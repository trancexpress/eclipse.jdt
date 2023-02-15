# Parse

In classical compiler construction, the input text is tokenized by a
**Scanner**. The resulting token stream is then fed into the **Parser**
which creates the abstract syntax tree (AST). ECJ essentially follows
this design, but in reality things are more complex, because the Java
syntax is no longer amenable to a strict implementation of this
approach.

## Scanner

Responsibilities of the scanner:

  - interpret input unicode
  - recognize comments
      - recognize task tags within comments
  - record line ends (for translation between linear offsets and line
    based positions)
  - **disambiguate** tokens which depend on context. Some examples, why
    scanning needs more context than it should according to text books
    (cf. [JLS
    §3.9](https://docs.oracle.com/javase/specs/jls/se14/html/jls-3.html#jls-3.9)):
      - token `->` is ambiguous with respect to the two single tokens
        `-` and `>`
      - *restricted keywords* are opportunistically recognized when this
        enables the parser to recognize a legal module declaration
        (inside the modular compilation unit "module-info.java").
      - *restricted identifiers* like `var` and `yield` must also be
        recognized by the scanner in certain contexts.

Two main concepts are used for token disambiguation:

  - A **VanguardParser**, is a stripped-down parser that runs the parse
    automaton at a location of ambiguity to check if a specific goal can
    be reached using the remaining input. In particular, the
    VanguardParser does not produce any AST, and it does not really
    consume any input.
  - The scanner keeps track of the current **ScanContext** in order to
    decide if a restricted keyword should be classified as a keyword or
    as an identifier.

To support the above, the scanner allows limited look ahead and the
option to "unget" an erroneously consumed token (in order to allow
re-classification).

Scanner itself is 100% handcoded, but **ScannerHelper** contains several
tables for unicode handling. Olivier Thomann is the expert for unicode
handling.

**Scanner** has a public variant, **PublicScanner** which must be
updated after each relevant change in Scanner. PublicScanner shields
clients from generated constants in interface **TerminalTokens** and
maps those to stable constants in **ITerminalSymbols**.

## Parser

### Constituents of the parser:

  - **java.g** is the grammar definition that is used by **jikespg** to
    create the following:
      - method Parser.consumeRule(int)
      - constants in interface **TerminalTokens**: every token detected
        by the scanner is represented using one of the constants in this
        interface. Constant names are composed of `TokenName` + the
        terminal name used in the grammar.
      - constants in interface **ParserBasicInformation** (used mostly
        by the parser automaton).
  - Most of the main Parser class, however, is hand-written.

### Generating the parser (see also ):

  - install jikespg on your local machine
      - cf
      - cf [github
        branch](https://github.com/jikespg/jikespg/tree/fixes-combined)
  - define a per-workspace **String Substitution** "JIKESPG" pointing to
    the location where jikespg is installed
  - run `scripts/build-parser.launch`

### Main parsing mechanism:

  - the parser state is captured in various stacks, each of which is
    implementated by a pair of fields:
      - a field of an array type to contain the stack data
      - an int field representing the pointer to the current top element
        (initially -1 for: empty stack)
  - method Parser.parse() contains the main loop of the parse automaton
  - method Parser.consumeToken(int) generically records some information
    purely based on a token being consumed
  - method Parser.consumeRule(int) (generated) dispatches into the
    individual consumeXYZ methods.
  - each consume method may peek or pop elements from some stacks and
    push elements on one or more other stacks

### Parsing modes

  - in **diet** mode, any blocks like method bodies are skipped, which
    can later be filled in using dedicated methods like
    Parser.parse(MethodDeclaration,CompilationUnitDeclaration).
  - fields Parser.methodRecoveryActivated and
    Parser.statementRecoveryActivated control whether or not attempts
    should be made to create AST despite syntax errors.

### The Grammar

Grammar **java.g** is the input to the parser generator "jikespg". For
lack of documentation concerning "jikespg" here is a set of documents
from "LPG" (which is a variant derived from jikespg):
<https://github.com/A-LPG/LPG2/tree/main/lpg-generator-templates-2.1.00/docs>
-- *use with caution, some details may not apply, most details are
unnecessary for ecj*

For normal maintenance work on ecj, however, the following hints might
suffice for editing java.g:

  - Declarations at the top of the file are rarely modified, including
    options, defines (macros), terminal definitions, aliases.
      - Lines starting with `Goal ::=` are parser rules with the special
        intention to setup the parser for selectively parsing a subset
        of the grammar.
    <!-- end list -->
      -
        E.g., the rule `Goal ::= '&&' FieldDeclaration` defines that
        assigning `TokenNameAND_AND` to `Parser.firstToken` will allow
        to parse an individual field declaration (see also
        `Parser.goForFieldDeclaration()` and friends).
        Goals are also used by the special `VanguardParser` when trying
        to disambiguate a token that could mean different things
  - The actual parser rules are written in a **BNF** dialect, below the
    directive `$Rules`
      - Two different symbols are used to map a non-terminal to a
        grammar expression: `::=` and `->`.
    <!-- end list -->
      -
        Only rules using the former symbol carry actions to be performed
        by the parser (*it is not clear if this is enforced or by
        convention only*)

what’s the difference between ::= and -\>, here is what we gathered by
experimentation:

Both are equivalent in the sense they are both valid grammars without
any difference in terms of meaning. However, when you use -\>, then you
are giving the parser generator the freedom to optimize this production
out if there is a chance. If you use ::=, then you are mandating that
this production needs to be there – don’t optimize this out even if
there is a chance.

So why is this relevant to us? If you have a consume\*() method which is
dependent on the reduction of this production, and if that consume\*()
is \*important\* \[it should be important or atleast it should be having
a future use, otherwise you would not put that there in the first
place\], then you should use ::= so that we have that production and
consequently that consume\*() method in the Parser\[.java\]. Otherwise,
you are free to use -\>. You can see examples of both in java.g.

  - For lack of operators like `|`, alternatives are written by a set of
    rules with the same LHS non-terminal
  - The special word `$empty` matches the empty input
      - Non-terminals ending in `opt` *by convention* have one
        alternative producing `$empty`
  - Keywords are written in single quotes.
  - Parser **actions** are written *below* a rule
      - By convention each action looks like this: `/. $putCase
        consumeXYZ(); $break ./`, which will create one case statement
        for method `Parser.consumeRule()`.
      - By convention, each `consumeXYZ()` method (which itself is
        written by hand) should contain as a code comment the grammar
        rules through which it is invoked
  - Minimal compliance level for the applicability of a rule is denoted
    like `/:$compliance 1.8:/` (also on a separate line below its rule)
  - For display in syntax error messages each rule should define a
    readable name like `/:$readableName QualifiedName:/`

The tricky part about editing the grammar is the
[**LALR(1)**](https://en.wikipedia.org/wiki/LALR_parser) requirement. If
this requirement is not fulfilled, jikespg will simply report this as an
error and abort. Information will be available file `java.l`, but that's
an overwhelming amount of it. It has been said that left-factoring can
be a means to avoid conflicts, but the Java grammar is intrinsically not
LALR(1), which gave rise to many tweaks and kludges like forced
lookahead (`VanguardParser` and `Scanner.lookBack`) unusual
Parser-Scanner communication etc. For ultimate confusion see
[JDT_Core_Programmer_Guide/ECJ/Parse/../../Completion\#Lambda_Specifics](JDT_Core_Programmer_Guide/Completion#Lambda_Specifics "wikilink")

### Parser Variants:

  - **DiagnoseParser**: after detecting a syntax error, this parser
    heuristically tries several "repairs", and selects the repair with
    the smallest "distance" to the actual input. During this process,
    the parser automaton is repeatedly run, without producing any
    output, other then accept or reject.
  - **VanguardParser**: as mentioned above, this parser is used for
    nested parse attempts in order to disambiguate certain tokens, that
    could be classified in different ways according to context.
  - **AssistParser** with subclasses CompletionParser and
    SelectionParser: these parsers focus on a specific text location and
    implement incomplete parsing optimized for the task at hand.
      - See [JDT Core Programmer
        Guide/Completion](JDT_Core_Programmer_Guide/Completion "wikilink")
        for complications in and around the CompletionParser
  - **CodeSnippetParser**: parses incomplete code for the sake of
    **evaluation**
  - **CommentRecorderParser**: used for created DOM AST
  - **SourceElementParser**: used for building structure of Java
    Elements (see Java Model)
  - **IndexingParser**: used for creating the JDT/Core index
  - **DocumentElementParser**: used for creating the *obsolete* JDOM
    (packages `org.eclipse.jdt.core.jdom`,
    `org.eclipse.jdt.internal.core.jdom`).
  - **MatchLocatorParser**: used during Java Search to provide input to
    the **MatchLocator** (which refines index matches into resolved java
    matches).

### Stacks in detail

**astStack** is the target: in absence of syntax errors everything will
eventually be aggregated into this stack and finally combined into a
single ASTNode. astStack will only take some higher level ASTNodes,
while specific stacks exist for "smaller" nodes.

**astLengthStack** is a companion to the above that provides a view as a
stack of **lists**. Each element on this stack is the number of elements
on the main astStack that should be seen as siblings at the same level.
Examples are members of the same class, statements of the same block.
If, e.g., the astStack contains 2 TypeDeclarations and 3
MethodDeclarions then an astLengthStack \[ 2 3 \] will indicate that the
three methods are siblings in the same list of methods, and likewise the
two types are siblings in the same list of types.

  - method `pushOnAstStack()` always starts a new list (by advancing
    astLengthPtr) of initial length 1.
  - method `concatNodeLists()` combines the top two lists into a single
    list, typically the top list is a singleton (the most recently
    produced node) that is being appended to the current list, following
    a grammar rule like *TypeDeclarations ::= TypeDeclarations
    TypeDeclaration*

The same kind of companion stack (\*LengthStack) exists for other stacks
as well.

**expressionStack** as the name suggests, manages expressions while they
are being assembled and before they are included into statements on the
main astStack.

**genericsStack** is where type parameters and complex type references
are assembled.

**identifierStack** is for identifiers that haven't even been integrated
in any ASTNode.

**intStack** is a multipurpose stack mostly used for source positions.
Other purposes include array dimensions, modifiers, and more. Due to the
mix of ints with different meaning, bugs related to an unbalanced
intStack are particularly difficult to analyze. So balancing pushs and
pops for this stack for all cases should be done with great care.

#### Automaton stack

While all the above stacks hold work-in-progress elements of the target
AST, parsing itself is controlled by the master stack **stack**. Each
value on this stack is a "state" capturing information what tokens have
been seen, and which tokens can legally follow. Sometimes these states
are called "act" or "action". The central method `tAction` takes the
previous act plus the current token to produce a new act (its
implementation directly leverages the tables generated by jikespg). acts
are grouped as shift, shift-reduce, reduce and ERROR states. During
**shift**, a new token will be accepted from the scanner. When the
parser has enough information for **reduce**, the current act is passed
to `consumeRule` where it will select which of the `consume*` methods is
invoked (consumeRule being generated by jikespg as well). This is when
contents of the other stacks will be modified.

To see the automaton in action consider setting `DEBUG_AUTOMATON` to
true (but don't commit this change\! :) ).

### Recovery

Upon any **syntax error**, syntax recovery happens using the following
concepts:

  - field `Parser.currentElement` holds a "recovered" element which
    wraps a possibly incomplete AST node.
      - each class below **RecoveredElement** models a node in the
        recovered tree, with generic functions for adding new child
        nodes etc.
      - `currentElement` acts like a cursor into the tree of recovered
        elements, with `parent` links to the rest of the tree.
  - field `Parser.lastCheckpoint` marks a position up-to which parsing
    is considered "done". When hitting a syntax error, parsing will
    restart at the last checkpoint.
  - method `Parser.resumeOnSyntaxError()` applies a number of heuristics
    to put the parser into a state where parsing can resume.
      - initially `buildInitialRecoveryState()` converts existing
        ASTNodes in `Parser.astStack` into `RecoveredElements`.
      - More elements will be added later using
        `RecoveredElement.add(..)`. Each execution of `add(..)` will
        return either
          - `this` to signal that more elements should be added to the
            same node, or
          - `parent` to signal that the current element is complete and
            subsequent details should be added to the enclosing element.
      - Expressions are explicitly omitted from the structure of
        `Recovered*`. To enable recovering details from a **lambda
        body**, the following tweaks are applied:
          - The lambda block is inserted as a direct child of the
            current element (omitting the lambda itself)
          - If the current element is a local variable declaration
            (which cannot accept a block as its child), the local
            variable is considered as complete and the lambda block is
            inserted into the parent, i.e., as a sibling of the local
            variable declaration.
  - much of the incremental updating of recovered elements relates to
    source positions:
      - `updateOnClosingBrace()`, e.g., may set source end positions of
        a method being recovered.
      - source end positions, on the other hand, are used in `add(..)`
        to detect whether the current element is "complete".
  - recovery may proceed in several iterations starting at a slowly
    moving lastCheckpoint. Each iteration will start with empty stacks,
    but ast nodes contained in the recovered elements tree may survive
    this reset.
  - in the end `updateParseTree()` will collect the information from the
    recovered elements tree, and update the actual parse tree
    accordingly, such that `Parser.compilationUnit` will contain the
    fully recovered AST.

## HOWTO: Grammar Changes

eg: Let us say you want to add a new statement, MyNewStatement. Grammar
changes start with java.g - It should be a context free grammar since we
use an LALR(1) parser You can add your MyNewStatement similar to that of
EnhancedForStatement to the RHS of Statement, ie Statement -\>
MyNewStatement and then followed by the new rules (obeying the context
free rule) with a few consumeMyNewStatement(), other consume\* methods.

The parser generator jikespg is used to generate the parser from this
grammar. You may want to take the latest bits of the jikespg from \[1\]
since it contains a few bug fixes and then make. Additionally, the
generation of parser has been made easier - ref \[2\]. If you want to
look at the original documentation of how the parser is generated see
\[3\].

Let us say, now you have the modified java.g and the jikespg parser
ready. First run jikespg.exe java.g to check whether your grammar is
LALR(1).

Assuming that it is, then you need to just run
org.eclipse.jdt.core/scripts/build-parser.launch. In order to provide
the path to jikespg you need to define a per-workspace String
Substitution "JIKESPG", see also the usage notes inside
[build-parser.xml](https://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core/scripts/build-parser.xml).

This would generate the files as described in \[3\], but what would
interest you most would the Parser.java - here you will see your
consumeMyNewStatement() and other consume\* stubs inserted - both the
method call as per the generated automata (which you should not touch)
and the declaration - you would need to add code to the declaration, to
reduce and possibly create a new AST Node.

The AST Node created would be the internal compiler AST Node and not the
DOM AST node - they have similar/identical definitions - so you need to
create something similar to
org.eclipse.jdt.internal.compiler.ast.ForeachStatement, which is the
enhanced for statement node of compiler AST - look at
org.eclipse.jdt.internal.compiler.parser.consumeEnhancedForStatement() -
internally, it creates a ForeachStatement - \[we are not talking of DOM
AST node - the conversion to the DOM AST Node EnhancedForStatement comes
later in the ASTConverter method and is not relevant here - mentioning
to avoid confusion\]

The most tricky part about constructing new AST nodes is to correctly
manage the various stacks of the parser. As a general rule, each consume
method should consume all ast nodes mentioned on its RHS and replace
them by one node representing the LHS. The first distinction to be made
is between `astStack` and `expressionStack`. Less obvious is the use of
`intStack` which is also modified inside `consumeToken`, e.g.

## Generating the parser separately with jikespg

(1) You can also run the jikespg separately and then bring the generated
files. Once you copy all the generated files into the grammar folder
\[where java.g lives\], set the env in Eclipse JIKESPG=JIKESPG_EXTERNAL
to signal to the script to skip running the jikespg. Read below on how
to use a gerrit job to run jikespg for your change

(2) To make things easier for generation of java.g, a job has been
created - refer(Details verbatim from)
<https://bugs.eclipse.org/bugs/show_bug.cgi?id=576415#c4>

\- Go to:
<https://ci.eclipse.org/jdt/job/eclipse.jdt.core-jikespg-gerrit/> -
Build with Parameters -\> here give the Egit refs (GERRIT_REFSPEC) -
Goto to the patch and use copy option and click the Egit ref - something
similar to refs/changes/31/189931/1

`- Once the job completes, you'll see the output under build artifacts of the job run.`

\- Clicking in build artifacts will give you an option to download all
files as zip or individual files.-

## References

\[1\] <https://github.com/jikespg/jikespg/tree/fixes-combined>

\[2\] <https://bugs.eclipse.org/bugs/show_bug.cgi?id=562044>

\[3\]
<https://www.eclipse.org/jdt/core/howto/generate%20parser/generateParser.html>

[Category:JDT](Category:JDT "wikilink")