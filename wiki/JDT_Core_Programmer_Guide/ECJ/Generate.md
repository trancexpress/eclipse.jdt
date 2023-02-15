## Participants

  - This phase is driven by `generatedCode()` methods in individual AST
    classes.
  - The result is accumulated into an instance of
    `org.eclipse.jdt.internal.compiler.ClassFile` per type.
  - The work horse for all the details of byte code is class
    `CodeStream`, with its subclasses `StackMapFrameCodeStream` and
    `TypeAnnotationCodeStream`, which are used es required by the target
    Java version.

## Modes

As mentioned above subtypes of CodeStream implement different
capabilities:

  - stack maps are generated for 1.6 and above
  - type annotations are generated for 1.8 and above

### Wide mode

Additionally, byte code can be generated in one of two addressing modes:
**regular** and **wide**. In regular mode, relative code offsets must be
smaller than 0x8000. In regular mode, opcodes like `goto` are used as
opposed to `goto_w` in wide mode. Code generation always starts in
regular mode, but as soon as a jump requires an offset larger than
0x7FFF code generation for the current method is aborted (by throwing
`new AbortMethod(CodeStream.RESTART_IN_WIDE_MODE, _)`, which causes a
**restart** in `AbstractMethodDeclaration.generateCode()` or similar.
The current mode is stored as `CodeStream.wideMode`.

A full restart is done, because instructions like goto vs goto_w have
different size in bytes. So changing a goto instruction to become
goto_w will invalidated all byte code offsets after the instruction.
OTOH, jump offsets are only known once the target label has been placed
into the stream. As a result code generation needs to now the length of
a jump before the position of the jump target is known. The easiest way
to find out is thus trial and error :)

### Local variable optimization

Restarting code generation for a given method may also happen for
another reason as detailed in :

**Local variables** are assigned to byte code positions during
`BlockScope.computeLocalVariablePositions(..)`, where for the sake of
optimization, **unused locals** may not get a position assigned.

For **"compound"** uses (postIncrement or compoundAssignment), however,
real usedness is found out only during generateCode() itself. The idea
here is that expressions like `i++` while technically reading the
variable, don't necessarily indicate that the value is actually used by
the program. When the expression is used as a statement, the read value
is discarded. When it appears in an expression position, the value is
used. For technical reasons, information about use as statement vs
expression is available only during generateCode (via parameter
`valueRequired`).

So if code gen started, thinking a local does not require a position,
and later detects this assumption was wrong, a restart is triggered with
`CodeStream.RESTART_CODE_GEN_FOR_UNUSED_LOCALS_MODE`.

[Category:JDT](Category:JDT "wikilink")