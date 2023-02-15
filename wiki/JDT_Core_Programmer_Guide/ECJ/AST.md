AST and [bindings](JDT_Core_Programmer_Guide/ECJ/Bindings "wikilink")
are the two main data structures of the compiler.

AST nodes directly capture all semantically relevant aspects of the
source code. In many regards the type hierarchy below
`org.eclipse.jdt.internal.compiler.ast.ASTNode` corresponds to the
standard approach in compiler construction.

## Traversing the tree

  - Three main compiler phases are implemented with explicit traversals
    below these methods of class CompilationUnitDeclaration:
      - resolve()
      - analyseCode()
      - generateCode()
  - For various smaller tasks the visitor pattern is used, see
      - class `org.eclipse.jdt.internal.compiler.ASTVisitor`
      - method ASTNode.traverse(ASTVisitor, BlockScope), plus variants
        for other scope types

## Special encodings

Throughout all AST classes, **source positions** are recorded as linear
text indices.

  - Fields sourceStart and sourceEnd typically hold the start and end of
    that region that should be highlighted if an error(/warning/info)
    has been detected concerning a given AST node.
  - Complex nodes have more locations, like bodyStart, bodyEnd etc.
  - In some cases, a pair of start & end position is encoded in a single
    long variable. This is used specifically where a list of positions
    is needed, like QualifiedTypeReference.sourcePositions (encoding the
    position of each text segment).
      - To create the long value, use `(((long) start) << 32) + end`
      - To extract the start position use `(int) (position>>>32)`
      - To extract the end position use `(int) position`

The field **ASTNode.bits** is used as a bitset. Unfortunately the use of
bits in this vector is quite crowded, some bits are used with different
semantics in different kinds of nodes. See class `ASTNode` for constants
and their terse documentation.

  - Some flags are set right during parsing, like `IsDiamond`
  - Other flags store information gathered throughout compilation.
  - Bits in `ParenthesizedMASK` encode the number of enclosing pairs of
    parentheses, so that no AST node ParenthesizedExpression is needed.
  - Flags `HasAllMethodBodies`, `HasBeenResolved` and `HasBeenGenerated`
    remember whether a given compilation step has already been performed
    for this node.

Similar bitsets are `TypeBinding.tagBits` and
`ReferenceBinding.typeBits`, see
[Bindings](JDT_Core_Programmer_Guide/ECJ/Bindings "wikilink").

## Additional types in package ast

### Additional classification

The AST structure very sparingly uses **interfaces** for additional
classification:

  - IJavadocTypeReference: subsumes JavaSingleTypeReference and
    JavaQualifiedTypeReference
  - InvocationSite: holds some context for resolving a reference to a
    member
  - Invocation: subsumes AllocationExpression, ExplicitConstructorCall
    and MessageSend (used during type inference)
  - IPolyExpression: expressions that can be poly expressions according
    to [JLS
    §15.2](https://docs.oracle.com/javase/specs/jls/se14/html/jls-15.html#jls-15.2).

### Synthetic ast node

The special node class `FakedTrackingVariable` does **not** correspond
to any source element, but is used to map resource leak analysis to the
existing infrastructure of null analysis. See
[Analyse](JDT_Core_Programmer_Guide/ECJ/Analyse "wikilink").

### Algorithm implementations

Class **NullAnnotationMatching** provides static methods for
'type-checking' null annotations. Some of these methods use instances of
NullAnnotationMatching to communicate the exact result of the analysis.

Class **UnlikelyArgumentCheck** provides static methods for advanced
'type-checking' of arguments of well-known methods like `contains` and
`remove` of interface `Collection`<T>, see  and [4.7M6
N\&N](https://www.eclipse.org/eclipse/news/4.7/M6/#unlikely-argument-types).
Both check methods return an instance of UnlikelyArgumentCheck to
communicate the exact result of the analysis.

[Category:JDT](Category:JDT "wikilink")