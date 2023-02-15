## Data structures for flow analysis

### FlowInfo

The analysis status for each location in the execution graph of a method
body is captured by an instance of the light-weight class FlowInfo.

  - FlowInfo is abstract, mostly dedicated to defining a protocol for
    sub classes
      - UnconditionalFlowInfo is the main work horse, capturing real
        state.
      - ConditionalFlowInfo is a composite of two flow infos,
        `initsWhenTrue` and `initsWhenFalse`, which is able to capture
        all relevant information after evaluating an `if` condition,
        e.g..

Inside UnconditionalFlowInfo, the optimized encoding of this status uses
several `long` fields as bitvectors to capture information concerning 64
variables in scope. The bit position in this vector corresponds to
`VariableBinding.id` for fields and `VariableBinding.id + maxFieldCount`
for locals.

  - definiteInits: this bit indicates if the given variable is
    definitedly assigned at the current position
  - potentialInits: indicates if the given variable is potentially
    assigned (i.e., some flow exists leading to this position, where the
    variable is indeed assigned).
  - if both above bits are 0 then the given variable is definitely
    unassigned.

<!-- end list -->

  - nullBit1, ... nullBit4: for each variable, a bit slice through these
    four fields gives a four-bit value encoding the "null status". See
    comment in UnconditionalFlowInfo for the interpretation of the 15
    relevant values (value 1000 is unused). In this table, the following
    abbreviations are used:
      - n - null
      - nn - nonnull
      - u - unknown, this is used for values from an external source:
        method arguments, method call results. Note that this is not
        related to "potentially" as used in situations with local
        branching.
      - def - definitely
      - pot - potentially
      - prot - protected (prot nn means, the variable must be non-null,
        because it has been dereferenced before without through NPE -- I
        don't have an example for prot n).

Originally, the encoding has probably been chosen as to give individual
bits distinct meaning, but then some bit combinations are used in
unrelated ways. This implies that each combination of two null status
values is a complex bit operation. For normal compiler maintenance these
operations can safely be regarded as black box. Details follow below.

  - iNBit, iNNBit: additional bits added at a later point to improve
    analysis of loops (see , and these
    [N\&N](https://www.eclipse.org/eclipse/news/4.5/jdt.php#loop-flows).
    The (tricky) implementation idea is described in , with a slightly
    higher level explanation in .

![Image:Bug.png](Bug.png "Image:Bug.png") Bugs relating to flow analysis
with loops may be marked with **\[loop\]** as a prefix in the bug
summary.

#### No correlation analysis

Some bug reports complain that ecj doesn't recognize "obvious" flows of
null / nonnull. One cluster of these issues would require "correlation
analysis", i.e., an analysis that captures correlations between
different variables, like, "when flag f is true then reference x cannot
be null".

As of today, no-one has come forward with an implementation strategy
that would fit the typical requirements of ecj, in particular speed.
Hence, correlation analysis is considered out of scope for ecj,
unfortunately.

![Image:Bug.png](Bug.png "Image:Bug.png") Existing bugs have been marked
with prefix **\[correlation\]**.

#### More than 64 variables

Since the above talks of `long` bitvectors, it can encode the state only
for 64 variables. So how do we cope, if there are more than 64 variables
in scope?

Answer: field `extra`, a dynamically growing array-of-array-of-long.

  - First index encodes correspondence to the original fields:
      - 0 : definiteInits
      - 1 : potentialInits
      - X+1, X in \[1...4\]: nullBitX
      - IN(6): iNBit
      - INN(7): iNNBit
  - Second index corresponds to "variable id / 64" - this is the
    dynamically growing part, each additional slice adding space for 64
    additional variables.

Operations using this field `extra` are specifically **tested** using
the flag
[jdt.flow.test.extra](JDT_Core_Programmer_Guide/ECJ/Testing#jdt.flow.test.extra "wikilink").

### FlowContext

While ideally each AST node directly produces effects by computing a new
FlowInfo, checking of some effects must be deferred, until all flows in
a given context are known. The simplest example is a loop, where each
position inside the loop body should also consider complete flows
through the loop body which then return to the beginning of the loop
body (i.e., we are analysing something like 1.5 iterations of each
loop).

Specific flow contexts exist for loops, switches, try blocks, finally
blocks etc.

Flow contexts have two families of methods:

  - `record*` : during the *first pass* record the effect of a given AST
    node
  - `check*` and `complainOnDeferred*` : when all individual flows have
    been computed, recheck all recorded information if it indicates any
    problem.

### Simple encoded nullStatus

Outside FlowInfo, nullness is expressed using an `int` nullStatus, as
one of UNKNOWN, NULL, NON_NULL, POTENTIALLY_UNKNOWN,
POTENTIALLY_NULL, POTENTIALLY_NON_NULL (constants in FlowInfo).

  - `FlowInfo.nullStatus(LocalVariableBinding)` - Here a given flow info
    computes the nullStatus for the provided variable.
  - `Expression.nullStatus(FlowInfo,FlowContext)` - Give an incoming
    flow info and flow context, computes the null status of the given
    expression. Different kinds of expression provided different
    implementations.

## Performing flow analysis

*TBC*

## Using flow analysis for resource leak detection

When value of type `Closeable` is handled, a specific use of flow
analysis determines, if that resource is closed on all paths.

![Image:Bug.png](Bug.png "Image:Bug.png") Bugs relating to resource leak
detection should have **\[resource\]** as a prefix in the bug summary
for easier searching. Here are **[all currently open
bugs](https://bugs.eclipse.org/bugs/buglist.cgi?classification=Eclipse%20Project&component=Core&product=JDT&query_format=advanced&resolution=---&short_desc=%5Bresource%5D&short_desc_type=allwordssubstr)**
in this area.

### Implementation using FakedTrackingVariable

The core trick here is to synthesize a special LocalDeclaration of
subtype `FakedTrackingVariable` which pretends that we have an extra
local variable. Whenever we see activity concering the resource value,
we encode the observation as "null info" concerning that faked tracking
variable.

  - initially the tracking variable is marked as null
  - when we see a `close()` call, we mark the tracking variable as
    nonnull.

Simply speaking, at the end of each tracking variable's scope, the
tracking variable must be marked as definitely nonnull, or a leak
(potential or definite) exists. Exception: a try-with-resources block
automatically closes.

  -
    *Indeed, the motivation for this analysis was to alert users of
    cases where try-with-resources should be used (language feature and
    analysis were added in the Java 7 era).*

Since these checks are connected to the end of a block, this is
implemented as `Block.checkUnclosedCloseables()`.

Since leak analysis has a few requirements that differ from null
analysis, the following bit-wise information is stored in
`FakedTrackingVariable.globalClosingState`: CLOSE_SEEN,
SHARED_WITH_OUTSIDE, OWNED_BY_OUTSIDE, CLOSED_IN_NESTED_METHOD,
REPORTED_EXPLICIT_CLOSE, REPORTED_POTENTIAL_LEAK,
REPORTED_DEFINITIVE_LEAK, FOREACH_ELEMENT_VAR (see the code comments
for explanations).

To understand **nested resources** where closing the outermost resource
is assumed to close also all inner resources (like, e.g.,
BufferedInputStream), each FakedTrackingVariable can be linked to an
`innerTracker` and an `outerTracker`.

In compiler warning messages, we prefer to use the name of a real local
variable to signal which element lacks closing. For values that are
never assigned to a local variable, and still subject to flow analysis,
one of the constants UNASSIGNED_CLOSEABLE_NAME or
UNASSIGNED_CLOSEABLE_NAME_TEMPLATE is used ("\<unassigned Closeable
value ...\>")

### Black lists / white lists

Resource analysis is intended to avoid leaks of *gc-resistent*
resources, i.e., resources outside of what the Java garbage collector
can free.

Since subtypes of `Closeable` have different semantics wrt the need to
close, interface `TypeConstants` defines some black lists / white lists:

  - JAVA_IO_WRAPPER_CLOSEABLES, JAVA_UTIL_ZIP_WRAPPER_CLOSEABLES,
    OTHER_WRAPPER_CLOSEABLES: no gc-resistent resource of its own, but
    wraps another resource that *may* be gc resistent.
  - JAVA_IO_RESOURCE_FREE_CLOSEABLES: while generally Closeables in
    package `java.io` are considered as gc-resistent, classes on this
    list are *not*.
  - JAVA_UTIL_STREAM: generally all subclasses of Stream are
    considered as **not** gc-resistent.
  - RESOURCE_FREE_CLOSEABLE_J_U_STREAMS: additional classes in
    package `java.util.streams` which should be treated like `Stream`
    despite not being subtypes of it.
  - FLUENT_RESOURCE_CLASSES: resource classes (gc-resistent) meant for
    fluent programming: methods returning an instance of the same type
    are considered as returning `this` so that invoking `close()` on the
    final result of a call chain `result = rc.fluent().operation()` is
    sufficient to also close the original resource.
  - JAVA_NIO_FILE_FILES: where a method of this class returns a
    `Stream`, the result *must* be closed, despite the general rule for
    `Stream`

After evaluation of a type name, the effect of the above lists are
encoded into `ReferenceBinding.typeBits`. Here the following constants
from `TypeIds` are relevant:

  - BitAutoCloseable (inheritable)
  - BitCloseable (inheritable)
  - BitWrapperCloseable
  - BitResourceFreeCloseable

Of these bits, those which are marked "inheritable" are propagated to
all subtypes, unless canceled by a black list.

Additionally, `TypeConstants.closeMethods` lists a few library methods
known to class a closeable argument.

## Implementation of null status bit operations

*TBC*

[Category:JDT](Category:JDT "wikilink")