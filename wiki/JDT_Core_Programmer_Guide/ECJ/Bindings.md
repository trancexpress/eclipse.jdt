## Main Kinds of Bindings

See the type hierarchy of
`org.eclipse.jdt.internal.compiler.lookup.Binding`. This class also
declares constants which define possible answers from `Binding.kind()`.

  - One reason why kinds are or-able bits: a NameReference can use an
    int as bitset to encode if it can legally resolve to a variable, or
    a type, or both, where variable can be either a field or a local.

As a special type, `UnresolvedReferenceBinding` is a placeholder for a
not-yet resolved ReferenceBinding. Resolving an
UnresolvedReferenceBinding will update its holder, too. This type is
used in signatures of members of `BinaryTypeBinding` to avoid the need
to read *all class file dependencies*. By construction, types in class
files are always represented by their fully qualified name, so lookup of
these types does not require any context.

Another group of bindings is **synthesized** by the compiler from thin
air, see [\#Synthetic Bindings](#Synthetic_Bindings "wikilink").

## General rules for Bindings

### No `null`

In constrast to AST, where `null` is typically a legal field value,
bindings typically use constants of the `NO_*` family to denote "nothing
here". `null` would typically indicate "not initialized".

### Comparison

Since bindings are unique by construction, it would normally be OK to
compare bindings using `==` or `!=`. Specifically with the introduction
of TYPE_USE annotations, even the same type can be represented by
different bindings to account for attached annotations. For that reason,
specific comparison methods have been added, which should be used
throughout:

  - `TypeBinding.equalsEquals()`
  - `TypeBinding.notEquals()`

These methods ignore difference only in annotations. Those rare cases
where annotation difference should indeed be considered have to be
marked in source with `//$IDENTITY-COMPARISON$`, to suppress a warning
implemented specifically for the compiler's own sake.

Internally, each `TypeBinding` has an `id`. Low values correspond to
well-known types (see `TypeIds`), while higher values are allocated
dynamically. Ids are interesting as the family of all type bindings
derived from the same original share the same id. Ids also simplify
checks for well-known types.

## Non-Uniqueness

Contrary to the general rule, a number of reasons exist, why the same
source entity can be represented by several distinct bindings:

  - When the same **package** exists in several modules, we create one
    `PlainPackageBinding` per module, plus a `SplitPackageBinding` to
    combine the slices into one. The current strategy concerning
    SplitPackageBinding was developed via  and friends, which has a lot
    of explanations.
  - From a **generic type** (`SourceTypeBinding` or `BinaryTypeBinding`)
    a number of parameterizations can be created using
    `ParameterizedTypeBinding`.
      - For each contained `MethodBinding` a
        `ParameterizedMethodBinding` is created to carry the
        instantiation of type variables.
      - For each contained `FieldBinding` a `ParameterizedFieldBinding`
        is created to carry the instantiation of type variables.
  - From a **generic method** (`MethodBinding`) a number of
    parameterizations can be created using
    `ParameterizedGenericMethodBinding`. While
    ParameterizedMethodBinding captures type variables of the declaring
    class, ParameterizedGenericMethodBinding captures type variables
    declared by the method itself.
  - From each `TypeBinding` a number of "clones" (of the same type) can
    be created with different sets of TYPE_USE **annotations**. Indeed,
    method `TypeBinding.clone()` is used for this process, and
    `TypeBinding.prototype()` will answer the original from which an
    annotated type was cloned. Also see
    [\#Comparison](#Comparison "wikilink") above.
  - From each `WildcardBinding` a number of `CaptureBinding` can be
    created, but that's not of ECJ's invention, but specified in JLS.

When comparing bindings, it is sometimes necessary, to explicitly strip
a wrapper binding using one of these methods:

  - `TypeBinding.original()` strips annotations, and erases
    parameterized, raw and array types
  - `TypeBinding.erasure()` full erasure (no type arguments), but may or
    may not retain annotations
  - `TypeBinding.unannotated()` strips annotations only
  - `TypeBinding.actualType()` answers the erasure from
    ParameterizedTypeBinding and WildcardBinding, `null` otherwise
  - `ParameterizedTypeBinding.genericType()`: like `actualType()` but
    performes lazy resolving of UnresolvedReferenceBinding
  - `MethodBinding.genericMethod()`: strips instantiation of a method's
    own type variables
  - `MethodBinding.original()`: strips any parameterization
  - `MethodBinding.shallowOriginal()`: strips instantiation of a
    method's own type variables, but leaves instantiations of class
    parameters in place (*unclear if different from genericMethod()*).

## Parameterization and nesting

A plain method inside a ParameterizedType will be represented by a
ParameterizedMethodBinding. But what about a nested type inside a
generic outer type?

  - If the nested type is **static**, any type parameters from its outer
    class are irrelevant for the nested type, as it cannot access them
    without an outer instance.
  - If the nested type is **non-static**, reference to this type are
    represented by a ParameterizedTypeBinding even if the nested type
    itself declares no type parameters, because here the nested type can
    refer to type variables of the enclosing type & instance (this was
    wrong prior to ).

## Flag vectors

  - `tagBits` (TypeBinding,MethodBinding,VariableBinding): set of bits
    as declared in `TagBits`
      - Is\*: fine grained classification of a binding
      - Begin\*, End\*: pairs of flags that indicate when a given
        processing step is active / complete (used to avoid re-entrance
        / recursion).
      - AreFieldsComplete, AreFieldsSorted, AreMethodsComplete,
        AreMethodsSorted: describes that status of arrays `fields` and
        `methods`
      - Has\*: various diagnostics
      - Annotation\*: marks when a given annotation has been applied to
        the element.
  - `extendedTagBits` (TypeBinding): overflow from tagBits, constants
    are in `ExtendedTagBits`.
    **New bits should preferrably be allocated here, rather than the
    crowded TagBits**.
  - `typeBits` (ReferenceBinding): classification of types, constants in
    `TypeIds`:
      - classification of resources (below `Closeable`), see
        [Analysis-\>Black lists / white
        lists](JDT_Core_Programmer_Guide/ECJ/Analyse#Black_lists_.2F_white_lists "wikilink")
      - BitUninitialized
      - BitUninternedType: classify JDT's own types `TypeBinding`
        (compiler) and `ITypeBinding` (DOM) as not suitable for
        reference comparison (`==`, `!=`), unless documented as
        `//$IDENTITY-COMPARISON`.
      - Bit\*Null\*Annotation: detect annotation types, configured for
        use by ECJ's annotation based null analysis.
      - BitMap, BitCollection, BitList: mark types which have methods
        with well-known problems (see
        [UnlikelyArgumentCheck](JDT_Core_Programmer_Guide/ECJ/AST#Algorithm_implementations "wikilink")).

## Synthetic Bindings

While the normal process goes AST -\> Bindings -\> byte code, some
elements are synthesized by the compiler skipping the initial AST stage.
These elements are implemented as `SyntheticFieldBinding`,
`SyntheticMethodBinding`, `SyntheticArgumentBinding`.

Some of these are managed in `SourceTypeBinding.synthetics`, an array of
maps, where the array index is one of `FIELD_EMUL`, `METHOD_EMUL`,
`CLASS_LITERAL_EMUL`.

Each `SyntheticMethodBinding` classifies itself in its field `purpose`.

The following constants represent the different purposes of synthetic
methods (best effort description, may not completely capture all usage
scenarios):

| Purpose               | bytecode name                       | Description                                                                                                                                      |
| --------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| FieldReadAccess       | access$*n*                          | Read access to a bytecode-inaccessible field                                                                                                     |
| FieldWriteAccess      | access$*n*                          | Write access to a bytecode-inaccessible field                                                                                                    |
| SuperFieldReadAccess  | access$*n*                          | Read access to a bytecode-inaccessible field                                                                                                     |
| SuperFieldWriteAccess | access$*n*                          | Write access to a bytecode-inaccessible field                                                                                                    |
| MethodAccess          | access$*n*                          | Invocation of a bytecode-inaccessible method                                                                                                     |
| ConstructorAccess     | <init>                              | Invocation of a bytecode-inaccessible constructor                                                                                                |
| SuperMethodAccess     | access$*n*                          | Invocation of an bytecode-inaccessible constructor in an anonymous instance creation                                                             |
| SuperMethodAccess     | *same as the original method*       | Invocation of a super method inherited from a non-public class into a public class                                                               |
| BridgeMethod          | *same as the original method*       | Method that overrides an inherited method and invokes an overriding method with a more specific signature (parameterization or covariant return) |
| EnumValues            | values                              | generated method `values()`                                                                                                                      |
| EnumValueOf           | values                              | generated method `valueOf()`                                                                                                                     |
| SwitchTable           | $SWITCH_TABLE$*enumName*           | generated lookup function for `switch` statements                                                                                                |
| TooManyEnumsConstants | " enum constant initialization$*n*" | generated method for enum initialization if byte code cannot fit into <clinit> method, see .                                                     |
| LambdaMethod          | lambda$*n*                          | Implementation of a lambda expression                                                                                                            |
| ArrayConstructor      | lambda$*n*                          | Lambda implementation of a method reference for array allocation                                                                                 |
| ArrayClone            | lambda$*n*                          | Lambda implementation of a method reference for array cloning                                                                                    |
| FactoryMethod         | lambda$*n*                          | Lambda implementation for a regular constructor reference                                                                                        |
| DeserializeLambda     | $deserializeLambda$*n*              | Generated method for deserializing a serializable lambda                                                                                         |

Here "bytecode-inaccessible" is typically an access from a nested class
to a private member of an enclosing class. Since in bytecode, nested
types appear like toplevel types, the privilege to access private
members of enclosings needs to be faked by synthetic delegation methods,
which are public, but should not be called from client code. Much of
this is **obsoleted** as of Java 11 due to
[JEP 181](https://openjdk.java.net/jeps/181) (Nest-Based Access
Control).

Many of these synthetic bindings are **created** in the
`manageSyntheticAccessIfNecessary()` family of methods in various AST
types.

Finally, `CodeStream` directly **generates the bytecode** from the
synthetic binding, using one of the `generateSyntheticBodyFor*` methods.

## Peculiarities of ModuleBinding

Firstly, four kinds of modules must be distinguished:

  - SourceModuleBinding (corresponds to some `module-info.java`
  - BinaryModuleBinding (corresponds to some `module-info.class`
  - AutomaticModule (corresponds to a jar file without
    `module-info.class`)
  - UnnamedModule (corresponds to code outside any "real" module,
    classes on the classpath as opposed to modulepath)

Even if `module-info` is found, this need not be the final word: through
command line options like `--add-reads` etc the declarations of a module
can be altered after the fact. To feed these tweaks into compilation, we
apply the following tricks:

  - `IUpdatableModule` super interface of `ModuleBinding` exposing
    mutators for a module.
  - Member classes of the above: `AddReads`, `AddExports` which encode
    changes to be performed once the module binding will be known.
    Method `accept()` will actually perform that change.
  - The **batch** compiler stores all updates seen on the command line
    in `FileSystem.moduleUpdates`, to be applied using
    `applyModuleUpdates()`
  - In the IDE, class `org.eclipse.jdt.internal.core.ModuleUpdater` is
    the hub for this information.
  - **Note** that module updates are persisted in the
    `org.eclipse.jdt.internal.core.builder.State` (from
    `ClasspathLocation#updates`) (we had a conflict, where a module
    update was implemented as a lambda (of type
    `Consumer`<IUpdatableModule>), which could not be persisted - this
    is still the case for the update setting the MODULE_MAIN_CLASS,
    but that update is not persisted at the moment.).
  - It is a bit tricky how and where exactly application of these
    updates is integrated into compilation.
  - At the bottom line, this architecture de-couples the compiler from
    the different ways how such updates are defined: command line or
    classpath attributes.

## Other classes in the lookup package

*the structure of this section should be improved.*

### Constants

  - TagBits - big bag of flags in bitset `tagBits`, see [\#Flag
    Vectors](#Flag_Vectors "wikilink")
  - ExtendedTagBits, see [\#Flag Vectors](#Flag_Vectors "wikilink")
  - ExtraCompilerModifiers - will appear in fields `modifier` but are
    never found in .class files
  - ProblemReasons - constants used in Problem\*Binding
  - TypeConstants - not only type, also well-known method names etc
  - TypeIds - integer constants relating to types

### Management of variants of a type binding

  - TypeSystem
      - AnnotatableTypeSystem

### Type Inference

Here be draggons, read [JLS
§18](https://docs.oracle.com/javase/specs/jls/se14/html/jls-18.html)
first.

  - InferenceContext18 (variant InferenceContext is used only below 1.8
    - not really maintained any more)
  - BoundSet
  - InferenceVariable
  - ReductionResult
      - TypeBound
      - ConstraintFormula
          - ConstraintExpressionFormula
          - ConstraintExceptionFormula
          - ConstraintTypeFormula
  - InferenceSubstitution

### Scopes

see
[JDT_Core_Programmer_Guide/ECJ/Bindings/../Lookups\#Scopes](JDT_Core_Programmer_Guide/ECJ/Lookups#Scopes "wikilink")

### Reading .class files

  - SignatureWrapper: incremental interpretation of a signature in
    binary format

### Visiting

  - TypeBindingVisitor

### Null Annotations

  - ImplicitNullAnnotationVerifier: check if method overriding is valid
    wrt null annotations
  - ParameterNonNullDefaultProvider: implements the effect of
    @NonNullByDefault on method parameters
  - ExternalAnnotationSuperimposer: See hierarchy of
    `org.eclipse.jdt.internal.compiler.env.ITypeAnnotationWalker`.

### Post-resolution computations

  - MethodVerifier
      - MethodVerifier15

[Category:JDT](Category:JDT "wikilink")