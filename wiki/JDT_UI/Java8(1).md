This page tracks the work in progress to add Java™ 8 support (mainly
jsr308 "Type Annotations" and jsr335 "Lambda Expressions") into Eclipse
JDT UI. See [JDT Core/Java8](JDT_Core/Java8 "wikilink") for the work in
the JDT Core plug-ins.

**Note: This is an internal project page that may be horribly out of
date.**

# Work areas

  - set ASTProvider.SHARED_AST_LEVEL to JLS8 ()
      - fix problems that need immediate handling
      - fix other references to JLS4
  - Annotations on types
      - Annotations that moved to another node in the AST (under
        discussion in )
      - MethodDeclaration\#thrownExceptions() -\>
        thrownExceptionsTypes() ()
      - Annotations on Extra Dimensions ()
      - Receiver parameter on instance methods and inner class
        constructors ()
      - Refactoring dialogs that allow to modify types
      - deferred: Render annotations in Javadoc hover/view. Do not
        render in Outline, etc. for now. ()
  - Default Methods
  - Lambda Expressions ()
      - check code that walks the parent chain to find the enclosing
        Block or BodyDeclaration
  - Method References

# IMPORTANT NOTE

  - The following lines must be added in all headers of modified files
    for Java™ 8 implementation:<code>

` * This is an implementation of an early-draft specification developed under the Java`
` * Community Process (JCP) and is made available for testing and evaluation purposes`
` * only. The code is not compatible with any specification of the JCP.`
` *`
</code>

  - Use the following @since tag on all newly added members: "3.9
    BETA_JAVA8"

# How to set up the IDE

See [JDT
Core/Java8\#What_to_do_to_set_up_the_IDE](JDT_Core/Java8#What_to_do_to_set_up_the_IDE "wikilink").

# Early access binaries

Please see [JDT/Eclipse Java 8 Support
(BETA)](JDT/Eclipse_Java_8_Support_\(BETA\) "wikilink") for details.

# UI decisions

## TYPE_USE annotations

ITypeBindings can now carry TYPE_USE annotations. API of
ITypeBindings\#equals needs to be updated and we eventually need to
replace usages of == with \#equals(Object).

\=\> Conclusion: ITypeBindings don't carry TYPE_USE annotations.

Should TYPE_USE annotations show up in the Java model (IJavaElements)?
The intention of the Java model is to provide a high-level structure for
rendering elements in an outline and to serve as enclosing elements in
search results. TYPE_USE annotations don't show up in type signatures
and the compiler doesn't consult them. They are second-class citizens.
The Java model cannot contain every detail of an element (e.g. it also
doesn't contain comments). Furthermore, it's not clear how TYPE_USE
annotations should be represented. They can't be part of a signature.

\=\> Conclusion: IJavaElements don't carry TYPE_USE annotations.

## Lambda expressions in the Java model

Currently, anonymous classes are represented in the Java model. Should
lambda expressions and method references also be part of the model? The
problem with that is that lambda expressions are perceived more
lightweight than anonymous classes, so lambda expressions will be used
much more frequently. Method references are another form of implicit
class declarations, since some forms already bind a receiver object.

Anonymous classes must be part of the Java model, because they can
define multiple members (fields/methods) that should be visible in an
Outline and that should support operations like "Open Super
Implementation". OTOH, lambda expressions and method references don't
have sub-members. However, a lambda expression can contain anonymous and
local classes, and the lambda expression itself would be interesting as
the enclosing element in a search result.

\=\> Conclusion: Lambda expressions are currently not represented in the
Java model but might be added later.

# Things to remember/caveats

  - Goal of the first pass is to make Eclipse work with the new language
    features. For more advanced support (new quick fixes / refactorings
    / templates / ...), please file an [enhancement
    request](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT&component=UI&short_desc=%5B1.8%5D+&bug_severity=enhancement)
    with the \[1.8\] tag.

<!-- end list -->

  - Bugs that went into the BETA_JAVA8 branch should be RESOLVED/FIXED
    with Target Milestone "BETA_JAVA8" and the \[1.8\] tag in front of
    the summary.

<!-- end list -->

  - For ASTNode changes:
      - Search for references to a changed AST Node and update code that
        relies on the concrete structure
          - in most cases, JDT UI only supports ASTs with the latest AST
            level (internal ASTProvider.SHARED_AST_LEVEL)
          - we only do AST\#apiLevel() checks where old ASTs can be
            passed in via JDT UI APIs:
              - CodeGeneration -\> StubUtility
              - ASTFlattener
              - AddUnimplementedConstructorsAction -\>
                StubUtility2\#createConstructorStub(..)
              - AddUnimplementedMethodsOperation -\> -\>
                StubUtility2\#createImplementationStub(..)
              - in general: search for references to type
                "org.eclipse.jdt.core.dom.\*" in org.eclipse.jdt.ui and
                subpackages
      - Remember there are multiple ways to access AST node properties:
          - via getter/setter or List-valued accessor
          - via the StructuralPropertyDescriptor constant
          - for properties declared in an abstract class: via
            get\*Property() instance method
      - Check references to superclasses of the changed/added node type.
        E.g. for IntersectionType, code that thinks it handles all known
        subtypes of Type needs to be adjusted.
      - Add new node types to switch or if-else-if statements
      - Think about cases where code could use
        StructuralPropertyDescriptors

<!-- end list -->

  - Think about cases where new bindings can show up.

# DOM AST changes in JLS8

*If everything is correct, then you should find the ASTNode changes as
references to ASTNode\#unsupportedIn2_3_4() and
\#supportedOnlyIn2_3_4().*

`~AST`
` ~newArrayType(Type): used to take a component type; now only accepts an element type that is not an array type`
` ~newArrayType(Type, int): (same as above)`

`+AnnotatableType (abstract superclass):`
`  +annotations: List`<Annotation>` (also in subtypes PrimitiveType, SimpleType, QualifiedType, NameQualifiedType, WildcardType)`

`~ArrayType`
` -componentType: Type`
` +setElementType(Type)`
` +dimensions: Dimension`

`+NameQualifiedType extends AnnotatableType`
` +qualifier: Name`
` +annotations: List`<Annotation>
` +name: SimpleName`

`+IntersectionType extends Type`
` +types: List`<Type>

`+Dimension extends ASTNode`
`  +annotations: List`<Annotation>

`+LambdaExpression extends Expression`
`  +parentheses: boolean`
`  +parameters: List`<SingleVariableDeclaration>` or List`<VariableDeclarationFragment>
`  +body: Block or Expression`
`  +resolveMethodBinding(): IMethodBinding`

`~MethodDeclaration:`
`  -setExtraDimensions(int)`
`  +extraDimensions: List`<Dimension>` (incl. annotations on extra dimensions)`
`  +receiverType: AnnotatableType`
`  +receiverQualifier: SimpleName`
`  -thrownExceptions: List`<Name>
`  +thrownExceptionTypes: List`<Type>

`+MethodReference extends Expression (abstract class)`
`  +typeArguments: List`<Type>
`  +resolveMethodBinding(): IMethodBinding`
`  `
`  (+Subypes:)`
`  `
`  +CreationReference`
`    +expression: Expression`
`  `
`  +ExpressionMethodReference`
`    +expression: Expression`
`    +name: SimpleName`
`  `
`  +SuperMethodReference`
`    +qualifier: Name`
`    +name: SimpleName`
`  `
`  +TypeMethodReference`
`    +type: Type`
`    +name: SimpleName`

`~SingleVariableDeclaration:`
`  +varargsAnnotations: List`<Annotation>

`~TypeParameter:`
`  +annotations: List`<Annotation>

`~VariableDeclaration:`
`  +extraDimensions List`<ExtraDimension>` (also in subtypes SingleVariableDeclaration, VariableDeclarationFragment)`

`~ITypeBinding:`
` +getTypeAnnotations(): IAnnotationBinding[]`
` +getFunctionalInterfaceMethod(): IMethodBinding`

For JLS4, the changes were:

`~TryStatement:`
`  +resources: List`<VariableDeclarationExpression>

`+UnionType`
`  +types: List`<Type>

Annotated array dimensions are not clearly defined by the current JLS8.
See [this
post](http://mail.openjdk.java.net/pipermail/type-annotations-spec-comments/2014-March/000070.html).
The assumption is that array dimensions are nested like this, from
innermost (element type) to outermost:

  - element type (with annotations just in front of the type)
  - extra dimensions, from left to right
  - type dimensions, from left to right

# TODOs

  - 'default' flag:
      - show in the UI?
      - update JdtFlags?

<!-- end list -->

  - The ITypeBinding of a TYPE_USE-annotated expression (e.g.
    ClassInstanceCreation or reference to a variable) doesn't contain
    the TYPE_USE annotations. References to "String s" and "@NonEmpty
    String ns" have identical type bindings. Should TYPE_USE
    annotations show up in reference bindings? Note that for some
    expressions, the annotated type is not defined by jsr 308, e.g. for
    the conditional expression here:

>
>
>     @Deprecated @NonEmpty String s = new Random().nextBoolean()
>         ? new @NonEmpty String("hi")
>         : new String();

  - A VariableDeclarationFragment can now also show up in a
    LambdaExpression
      - check usages of VariableDeclarationFragment.getParent()
      - check usages of VariableDeclarationFragment that assume there's
        an associated type (e.g. ASTNodes.getType(VariableDeclaration))

<!-- end list -->

  - Check helper methods in ASTNodes
      - e.g. getTopMostName(Name) is not correct any more to find an
        exception type

<!-- end list -->

  - Check all ASTVisitors that override one of the visit/endVisit
    methods for one of the changed AST node types

## TYPE_USE annotations

  - Code affected by lack of TYPE_USE annotations in ITypeBindings: The
    code in UI that currently loses TYPE_USE annotations will need to
    be adjusted once () is fixed.
      - Introduce Indirection refactoring:
        IntroduceIndirectionRefactoring\#createIntermediaryMethod()
      - Move Method refactoring:
        MoveInstanceMethodProcessor\#createMethodArguments(...)\#getArgumentNode(...)

<!-- end list -->

  - Code affected by lack of TYPE_USE annotations in IJavaElements: ()
      - Javadoc hover and view cannot display TYPE_USE annotations.
      - Method signature previews in refactoring wizards cannot display
        TYPE_USE annotations - Change Method Signature, Introduce
        Parameter Object, Introduce Parameter, Extract Method.

[Category:JDT](Category:JDT "wikilink")