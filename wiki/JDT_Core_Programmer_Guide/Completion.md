## General approach

While completion is a monster full of special case heuristics, let's
start by explaining the general idea:

  - The main driver is class `CompletionEngine` which acts as a variant
    of the compiler using these components:
      - a regular `LookupEnvironment` for anything related to resolving
      - a special parser: `CompletionParser`
  - In many situations the completion parser will create one specialized
    ASTNode as soon as it hits the cursor location: one from the family
    of `CompletionOn*` classes.
  - Ideally, when resolving
    (`this.lookupEnvironment.completeTypeBindings(compilationUnit,
    true);`) hits the completion node, it will throw
    `CompletionNodeFound` containing what information was available
    during resolving.
  - Based on data in that exception and depending on the kind of
    completion node a number of searches are performed, see the long
    list of `CompletionEngine.find*` methods
  - When suitable things are found, one `InternalCompletionProposal` is
    created for each, which again collect a lot of useful information:
      - `#completion` the string to be proposed
      - source positions
      - `#relevance` ranks the proposals so the most relevant can be
        listed on top
      - much more information for use by JDT/UI or other adopting tools,
        to support display, filtering, and applying each proposal in a
        semantically useful way.

Going deeper the big monster can be sub-divided into to sub-monsters

  - `CompletionParser` (together with `CompletionScanner`) ''(6 KLOC + 1
    KLOC):

<!-- end list -->

  -
    This component is responsible for trying to make sense of the
    current source code at and around the cursor location in a *purely
    syntactical* sense. Here one of the main challenges is to work
    around the fact that a regular parser would commonly just reject the
    input due to syntax errors -- we are by definition in the middle of
    editing, so syntax errors are the norm.

<!-- end list -->

  - `CompletionEngine` *(\>13 KLOC in one file)*.

<!-- end list -->

  -
    This guy is responsible for all semantic aspects, like figuring out
    all types involved, finding available members, and performing
    special matching strategies like camel case, subword etc. While we
    have a `MissingTypesGuesser` right in the engine, more guessing
    business happens in JDT/UI (e.g., `ParameterGuessingProposal`).

### Where information is maintained

When debugging code completion, it is essential to first understand the
basic working of the Parser, and in particular its [various
stacks](JDT_Core_Programmer_Guide/ECJ/Parse#Stacks_in_detail "wikilink").
Also the story of `RecoveredElement currentElement` is of great
importance for completion parsing.

Additionally, the CompletionParser maintains its own data, which partly
overlaps with the regular parser state.

**assistNode** will hold the node on which assist was invoked. It should
be one of the `CompletionOn*` family of classes, which are instantiated
by various overrides `consume*` in CompletionParser.

**assistNodeParent** is used in lots of places of CompletionEngine to
infer additional context information. Typically this will be the
directly enclosing node, a parameterized type reference when completing
on a type argument, or an enclosing statement when completing on an
expression etc.

**enclosingNode** is another ancestor node of assistNode, but with
specific purpose:

  - for specific casted-receiver proposals, enclosingNode will hold the
    enclosing IfStatement(s) (or AND_AND_Statements after ), with
    instanceof expressions that narrow down the actual type of a
    receiver.
  - to determine an expected type inside a variable declaration or
    return statement, or in provides statements (it is also set for uses
    statements where it appears to be unused).

Additionally, the CompletionParser maintains yet another set of stacks
all filled in lock-step as controlled by **elementPtr** (all inherited
from `AssistParser`):

  - **elementKindStack** holds constants of the set `K_BLOCK_DELIMITER`
    ... `K_YIELD_KEYWORD` remembering what tokens have already been
    seen. This semantically overlaps with the automaton stack, but since
    the automaton stack uses generated constants that should not be
    interpreted by hand-written code, this addition stack is very useful
    for inspecting the syntactical context
  - **elementInfoStack** for each "stack frame" addressed by elementPtr,
    this stack holds additional information, specific to the element
    kind. See constants starting at `CompletionParser#IF`.
    Unfortunately, it has not been documented precisely, which constants
    are used for which element kind. Some flags also seem to be used for
    different kinds alike.
  - **elementObjectInfoStack** is the most tricky of these stacks, as it
    may hold on to actual ast nodes which may not be reachable
    otherwise, as these may survive a parser restart during recovery.
    See call hierarchy of `pushOnElementStack()` for who's putting what
    on this stack.

To summarize, the CompletionParser has four kinds of sources for ast
nodes:

|                                                                                        |                                                                                                       |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| the regular astStack, expressionStack ... family of stacks                             | *the top result of which is provided as the result of `parse()`*                                      |
| the individual nodes assistNode, assistNodeParent and enclosingNode                    | *directly accessible to CompletionEngine*                                                             |
| the currentElement 'cursor tree'                                                       | ''internal to the parser, i.e., it needs to be materialized into astStack before `parse()` terminates |
| nodes on the elementObjectInfoStack (with interpretation hints in the sibling stacks). | *same as above*                                                                                       |

## Source code range

### Steps towards today's solution

  - The **initial** strategy of `CompletionParser` was to restrict the
    source code range to \[method.bodyStart, cursorLocation\], i.e, when
    the scanner hits the `cursorLocation` it would answer
    `TokenNameEOF`.
      - This made the parser totally ignorant to any potential syntax
        errors after the cursor location
      - OTOH it required significant machinery in and around
        `attachOrphanCompletionNode()` to wrap up some parts that
        haven't yet made it into the main `astStack`.
  - With the introduction of **lambdas** this was no longer sufficient
    and so  implemented an option to extend the source when we discover
    we need more.
      - Later this proved problematic because at the time the source
        range was extended, the observed EOF had already muddled with
        some internal state / stacks.
  - In  first experiments to avoid setting EOF to cursorLocation caused
    regressions, so at that time the experiment was abandoned.
  - Since  the strategy has been reversed indeed: we always start with a
    range corresponding to the **entire method body**.
      - The new method `CompletionParser.fetchNextToken()` will only
        ever answer EOF at or after `cursorLocation` when it's "safe" to
        do so.
      - Consequently, more code locations need to check `cursorLocation`
        to detect when completion specific behavior should trigger.

Unfortunately, a few other locations still manipulate
`Scanner.eofPosition`:

  - `CompletionParser.consumeToken()` sets EOF for specific situations
    of fields / field initializers. -- *it would be great if this could
    be solved in some other way*.
  - The above is conditionally compensated inside
    `AssistParser.fallBackToSpringForward()`
  - A few more locations could be listed here, but those seem to be of
    less significance.

As a fine point in the implementation of
`CompletionParser.fetchNextToken()` conditionally delay EOF using the
following strategy:

  - If no lambda is in sight, answer EOF directly at cursorLocation or
    as the next following token.
  - Otherwise, keep parsing until the expression stack is empty, as to
    ensure that any complex expression involving a lambda will always be
    parsed in entirety.

## Coping with incomplete AST

Here we face two kinds of incompleteness:

1.  AST may be incomplete due to syntax errors, which can naturally
    occur since the user is in the middle of editing.
2.  AST may be incomplete because parsing is stopped at or near the
    cursor location.

Issue (1) is not particular to code completion and basically follows the
strategy outlined in
[JDT_Core_Programmer_Guide/ECJ/Parse\#Recovery](JDT_Core_Programmer_Guide/ECJ/Parse#Recovery "wikilink").

In this case *additional, completion-specific* **recovery** happens
below method `updateRecoveryState()` overridden in `CompletionParser`

  - relevant things may happen below `attachOrphanCompletionNode()`:
      - Directly in this method we try to splice the `assistNode` into
        `currentElement`
          - we possibly *synthesize* some complex ast node from parser
            stacks, of which the assist node will be a child.
      - \-\> `buildMoreCompletionEnclosingContext()` has special
        treatment for code nested in an if statement and specifically
        for **instanceof**. Also here ast nodes may be *synthesized*.
      - also `assistNodeParent` and `enclosingElement` may be assigned
        when we have that information.

In very specific situations, parsing will need specific help at the
input-side, to produce any useful nodes. In particular, parsing will
have difficulties with **incomplete lambda expressions**

  - see [\#Lambda_Specifics](#Lambda_Specifics "wikilink").

### Collecting nodes in absence of a syntax error

If **no syntax error** has been observed, we may cleanly parse all the
way into `CompletionParser.endParse()`, where we wrap up similar to
`attachOrphanCompletionNode()`, but with much simplified strategy, since
we expect all relevant AST nodes to exist on some stacks etc.

  - when parsing block statements of a method or constructor, we inspect
    `referenceContext` (in this case being an
    `AbstractMethodDeclaration`):
      - if `statements` is still `null` we assign the most suitable node
        from one of these locations: `astStack`, `assistNodeParent`,
        `assistNode`.
      - if statements exist, and also a `currentElement` we may splice
        the assist node (or its parent) into the current element and
        feed the result into the AST using `updateParseTree()`
  - additionally, a `CompletionNodeDetector` will be used to populate
    `assistNodeParent` and `enclosingNode`, if necessary. These elements
    will be directly used by `CompletionEngine`. This detector uses a
    few heuristics to find most-suitable parent/enclosing nodes.
      - `enclosingNode` is specifically selected for casted-receiver
        completions (see next).
      - `assistNodeParent` will specifically drive the computation of
        expected types (and some more tasks).

### Supporting casted-receiver completions

Here is a special kind of completion, for which the AST may be created
opportunistically, even inventing some structure may happen.

The core idea is that in statements like `if (o instanceof Foo) o.|`
completion should propose members of `Foo` even if `o` has a different
static type. We want this to apply in more situations, too: nested if
statements. `&&` expressions where the left operand is an `instanceof`
test.

NB: *I couldn't find similar treatment for `ConditionalExpression`,
which looks inconsistent / incomplete to me. Also
`CompletionEngine.computeExpectedTypes()` has a code comment indicating
room for improvement: "for future use"*.

This strategy consists of two parts:

  - `CompletionNodeDetector` will detect the outermost of a chain of
    `IfStatement` or `AND_AND_Expression` of which the condition / left
    operand is known to be true at the cursorLocation (i.e., the cursor
    is in the then-branch or right operand). The result will be found in
    `#enclosingNode`.
  - `CompletionEngine.findFieldsAndMethodsFromCastedReceiver(..)` needs
    to consider both kinds of nodes during traversal
    (`findGuardingInstanceOf()` and `findGuardedInner()`). It should be
    straight-forward to also integrate `ConditionalExpression` in the
    same manner.

## Lambda Specifics

Completion involving lambdas has essentially been implemented via  and .

### fallBackToSpringForward()

This method will essentially try two strategies:

1.  Can we close a pending lambda, either by the next real token, or by
    one of a fixed set of `RECOVERY_TOKENS`?
      - Since  "real next token" should only be relevant outside of
        methods (field / field initializer).
2.  If not, can we just jump to the next 'thing'?

For strategy (1) we first check if the `eofPosition` needs to be pushed
past the cursor location, allowing to scan at least up-to the end of the
current method body.

For strategy (2) we keep a stack (since ) of snapshots, represented by
instances of CompletionParser, which mainly hold copies of the various
stacks, plus various assist-specific fields.

  - The snapshots are created / updated by calling `commit()` when
    various elements are consumed which could contain a lambda
    expression (see `triggerRecoveryUponLambdaClosure()`).
  - In one arm, `fallBackToSpringForward()` will copy the state of the
    latest / matching snapshot into the current parser. After such
    copyState() the parser will be told, to `RESUME` (rather than
    `RESTART`) in order to keep the parsing state reached thus far.
  - It's crucial that each snapShot will be removed from the stack at
    the right point in time (from `triggerRecoveryUponLambdaClosure`,
    `consumeBlock`, or `consumeMethodDeclration`).

Within `fallBackToSpringForward()` we may signal
`shouldStackAssistNode()` which tells subsequent
`consumeEmptyStatement()` to replace the top of `astStack` with
`assistNode`.

While `shouldStackAssistNode()` is called on multiple arms, the method
can and should be refactored to call it only directly before
`this.currentToken = TokenNameSEMICOLON; return RESUME;` -- here the
semi allows the parser to proceed, but if it just creates and empty
statement that one will be replaced.

### actFromTokenOrSynthetic()

Via  another strategy for closing a pending lambda has been added:

Method `actFromTokenOrSynthetic()` may synthesize an arbitrary sequence
of `RECOVERY_TOKENS` as long as they can be accepted by the parser. Here
we are feeding the tokens right into the parser as if they were coming
from the scanner.

## Breakpoints and expectations

Here's a rough guide at how to narrow down a problem with completion.
It's given by way of some breakpoints and a description of what could be
checked at each of them.

(A) All of completion is orchestrated in method
`CompletionEngine.complete(ICompilationUnit, int, int, ITypeRoot)` so
this is a good place for first breakpoints.

  - **Parsing** is done in two phases (as usual), so you may check the
    results:
      - diet parsing right after `parser.dietParse(sourceUnit...)`
      - full parse (for method bodies) right after
        `parseBlockStatements(parsedUnit...)`

**Expectations:**

  - In the debugger's Variables view, the toString() of the resulting
    AST should show one of the <CompletionOn_:_> nodes. Is it the
    expected node, i.e., correctly classified, and showing the correct
    detail like prefix already typed by the user?
  - Does the completion node have suitable context? Specifically lambda
    expressions need a target type for resolving, so is the lambda
    embedded in a suitable assignment or invocation? Also invocations of
    generic methods need that target type during resolving.

-----

(B) If the AST structure looks good, the next stop could be the
`resolve_` method of the identified completion node (otherwise proceed
to (D)). Set a breakpoint in all candidate resolve or getBinding
methods.

**Expectations:**

  - Does the breakpoint fire at all?
  - Does resolving succeed to resolve relevant details that will
    determine reciever type and/or target type?
  - Is exception `CompletionNodeFound` thrown with all relevant
    information?

-----

(C) If `CompletionNodeFound` is correctly thrown, then debugging will
continue in `CompletionEngine`. Stepping through the private
`complete(ASTNode,ASTNode...)` will be useful.

**Expectations:**

  - does `computeExpectedTypes` call `addExpectedType()` with desired
    types?
  - if all looks well so far you may have to step via the long if-else
    cascade into the individual `completionOn_` method and try to follow
    its specific logic.
  - specifically check, if the engine fails due to insufficient
    information in the parser's field `assistNodeParent` and
    `enclosingElement` - in which case you should proceed at (D).

-----

(D) If the AST seen is insufficient (or assistNodeParent or
enclosingElement), the previous location to stop would be
`CompletionParser.endParse()`. Please see that it may be called twice
(during dietParse and parseBlockStatements), you want the second
execution.

**Expectations:**

  - if information was missing in the AST, you may inspect parser
    stacks, as well as `Parser.currentElement`, whether the desired
    nodes can be found in any of them. Specifically have a look at
    `AssistParser.elementObjectInfoStack` which may hold AST nodes, too.
  - if AST basically looks OK, but assistNodeParent or enclosingElement
    are not correctly set, then `CompletionNodeDetector` might be the
    culprit.

-----

(E) If at (D) you cannot find the expected nodes anywhere near or far,
check if recovery has done its work. A first breakpoint would be in
`attachOrphanCompletionNode()` *after* the early exit in the first line.

**Expectations:**

  - If `assistNode` holds a useful completion node, then this method
    should "understand" its context and splice the completion node into
    the most suitable location of existing AST snippets. You will have
    to step through the detailed logic of this method incl. any of the
    `buildMore_CompletionContext()` methods.

-----

(F) When in doubt whether a completion node was ever created, identify
the expected class `CompletionOn_` and set a breakpoint in its
contructor.

-----

(G) Completion proposals have a **relevance** for sorting the list in
the proposal popup. Constants used for raising the relevance according
to given criteria can be found in `RelevanceConstants`. When a proposal
has unexpected relevance, set a breakpoint in method
`CompletionEngine.computeBaseRelevance()`. After returning from that
method you will be able to step through a bunch of similar methods, each
responsible for one or more of these constants.

*NB:* it is recommended to specify test expectations using these
constants in order to express what reasons for relevance are expected -
*a recommendation not followed in all existing tests*.

-----

*to be continued*.

[Category:JDT](Category:JDT "wikilink")