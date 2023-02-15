# A Hitchhiker's Guide to ECJ

## What IS the Compiler / ECJ?

Strange enough this question does not have a single true answer.

### Project organization before 4.27

The following locations contribute to the compiler:

  - org.eclipse.jdt.core/compiler
  - org.eclipse.jdt.core/batch
  - org.eclipse.jdt.compiler.tool
  - org.eclipse.jdt.compiler.apt

Since the compiler does not directly correspond to any project / plug-in
the following measures are relevant:

  - Classes in source folders `compiler` and `batch` are not allowed to
    access classes in other source folders of org.eclipse.jdt.core. To
    avoid any violations, a secondary project has been created:
    **org.eclipse.jdt.core.ecj.validation**. This project should be
    imported into the workspace before working on the compiler. It
    contains only links to the two mentioned source folders and will
    signal errors, if any class outside this scope is used. The project
    is not intended for editing.
  - During production builds class files from different projects need to
    be merged into the single **ecj.jar** (this jar file is created as
    org.eclipse.jdt.core-\*-SNAPSHOT-batch-compiler.jar and renamed to
    ecj.jar afterwards). Search for "batch-compiler" in pom files of the
    projects mentioned above, to see how the compiler is assembled.
  - Additionally, an ant script exists,
    org.eclipse.jdt.core/scripts/export-ecj.xml, that should allow
    manually creating ecj.jar from within Eclipse. This script is also
    executed when building org.eclipse.jdt.core using PDE/Build,
    *probably* happening also when interactively exporting
    org.eclipse.jdt.core as a deployable plug-in using the export
    wizard.

### Project organization since 4.27 M1

The compiler resides in plug-in `org.eclipse.jdt.core.compiler.batch`.
This plug-in has no dependencies at runtime and a (hidden) **optional**
compile time dependency to `ant.jar` (see `jars.extra.classpath` entry
in `build.properties`) and can thus be used in standalone mode. When
deployed as a jar file, this plug-in serves as the "batch compiler" and
is identical to the file **ecj.jar**.

For use in the IDE (or other OSGi-applications), clients may still refer
to the compiler via `org.eclipse.jdt.core` which re-exports all relevant
packages from `org.eclipse.jdt.core.compiler.batch`.

### The ant adapter

For using ecj with ant, **jdtCompilerAdapter.jar** is created from
selected classes inside `org.eclipse.jdt.core.compiler.batch`:

  - org.eclipse.jdt.internal.antadapter.\*
  - org.eclipse.jdt.core.JDTCompilerAdapter

The same class files are also added to ecj.jar, but JDTCompilerAdapter
is **not** exported in `MANIFEST.MF` and thus not visible in OSGI to
other bundles.

Documentation about peculiarities of this jar file are described in some
more detail in
<https://github.com/eclipse-jdt/eclipse.jdt.core/blob/master/org.eclipse.jdt.core.compiler.batch/src/org/eclipse/jdt/core/README.md>

#### Creating jdtCompilerAdapter.jar

**jdtCompilerAdapter.jar** is packaged in project `org.eclipse.jdt.core`
(\!). The reason for doing this "remotely" (i.e., not in
..compiler.batch) is in the fact that `org.eclipse.jdt.core` needs a
**dedicated** library containing only ant related classes for
registration with extension point
`org.eclipse.ant.core.extraClasspathEntries` - and this extension point
has to be inside `org.eclipse.jdt.core` bundle, because
JDTCompilerAdapter class from batch compiler should be "not visible" to
OSGI to avoid conflicts between Ant and OSGI loaded classes at (OSGI)
runtime (see [the
readme](https://github.com/eclipse-jdt/eclipse.jdt.core/blob/master/org.eclipse.jdt.core.compiler.batch/src/org/eclipse/jdt/core/README.md)).

When interactively exporting `org.eclipse.jdt.core` from the IDE,
creation of jdtCompilerAdapter.jar is managed by PDE/Build in a
cooperation between `customBuildCallbacks.xml` and
`scripts/export-ecj.jar`.

During maven builds, jdtCompilerAdapter.jar is created by copying
required classes compiled in org.eclipse.jdt.core.compiler.batch module
to the org.eclipse.jdt.core and bundling them together as jar.

#### Using jdtCompilerAdapter.jar

In all PDE/Build scenarios (e.g., when interactively exporting arbitrary
plug-ins from the IDE) jdtCompilerAdapter.jar is used by ANT/PDE IDE
code as an entry point into the compiler (see
`org.eclipse.ant.internal.core.AntClassLoader`).

### Interfacing with other components

  - **Name Environments:** To interface with its environment, the
    compiler needs an instance of
    `org.eclipse.jdt.internal.compiler.env.INameEnvironment`. During
    batch compilation, class
    `org.eclipse.jdt.internal.compiler.batch.FileSystem` is used. But
    using different implementations of this interface other components
    like the builder can provide required classes into the compiler.
      - In terms of
        [JLS](https://docs.oracle.com/javase/specs/index.html) the name
        environment implements the **"host system"**, see in particular
        [JLS
        §7.2](https://docs.oracle.com/javase/specs/jls/se18/html/jls-7.html#jls-7.2).
  - **IBinaryType:** Different use cases use different implementations
    to represent existing .class files to which Java sources being
    compiled can refer.
  - **ITypeRequestor:** Whenever a new type is found from the name
    environment, it is first passed to methods of the type requestor.
    Normally, the `Compiler` itself acts as the type requestor, which
    will add a representation of the discovered type to the internal
    data structures of the compiler, but code assist, type hierarchy,
    search and indexing each have their own implementation of these
    hooks into the compiler.

### Variants of the compiler

The central class is `org.eclipse.jdt.internal.compiler.Compiler`, which
is used as-is in some use cases, but also a few subclasses exist, which
are variants of the compiler, with purposes different from generating
`.class` files. In other use cases, not the Compiler class, but
`org.eclipse.jdt.internal.compiler.parser.Parser` is subclassed to
achieve different functionality. The latter strategy is used notably for
code select and code complete functionality.

## Phases

As is standard in compiler technology, ecj operates on Java files in
several phases, which are roughly outlined as:

  - **[Scan and
    parse](JDT_Core_Programmer_Guide/ECJ/Parse "wikilink")**, i.e.,
    transform a character stream first into a stream of tokens, then
    into the abstract syntax tree
    ([AST](JDT_Core_Programmer_Guide/ECJ/AST "wikilink"))
  - **Build and connect type bindings**, i.e., overlay the syntactic
    tree structure ([AST](JDT_Core_Programmer_Guide/ECJ/AST "wikilink"))
    with a semantic graph of
    [bindings](JDT_Core_Programmer_Guide/ECJ/Bindings "wikilink").
  - **Verify methods**: analyse inheritance, overriding and overloading
    of methods
  - **Resolve**: interpret identifiers and link them to the
    [bindings](JDT_Core_Programmer_Guide/ECJ/Bindings "wikilink") which
    they represent using various
    [lookups](JDT_Core_Programmer_Guide/ECJ/Lookups "wikilink").
  - **[Analyse](JDT_Core_Programmer_Guide/ECJ/Analyse "wikilink")**:
    perform flow analysis in order to detect errors like variables read
    before assigned, final variables re-assigned, and also analysis of
    (potential) null pointers and resource leaks. This phase may also
    detect a few more errors that need the AST to be fully resolved.
  - **[Generate](JDT_Core_Programmer_Guide/ECJ/Generate "wikilink")**:
    Allocate positions to variables (as used in load and store
    operations of the byte code), then generate the byte code, in the
    steps shown below. Note that still during code generation some
    errors may be detected and reported.
      - Generate the general class file structure with relevant byte
        code attributes
      - Generate the `Code` attributes containing the actual byte code
        instructions for methods, constructors and initializers.

Looking at class `Compiler` the phases are written slightly differently:

  - `beginToCompiler`/`internalBeginToCompile`:
      - `parse` or `dietParse`
      - `buildTypeBindings`
      - `completeTypeBindings`, here bindings are linked / connected
        with each other, which requires all bindings to already exist.
  - *optionally*: `processAnnotations`
  - `processCompilationUnits` / `process` -- *at this point a separate
    compilation thread may be spawned: `ProcessTaskManager`*
      - `getMethodBodies`: the initial parse may have skipped method
        bodies, parse them now, perhaps only selectively
      - `faultInTypes`: ensure that all bindings are properly created
        and initialized
      - `verifyMethods`
      - `resolve`
      - `analyseCode`
      - `generateCode`
      - `finalizeProblems`: before errors and warnings are actually
        reported to the user, they are filtered by any
        `@SuppressWarnings` annotations found in the source.

## Processing order

  -
    *Sequentiel phases vs. demand-driven computations*

In addition to the sequential process outlined by the phases above, some
computations will be triggered on demand.

Existing tricks to fine-tune order of processing steps:

  - `ClassScope.deferredBoundChecks`:

<!-- end list -->

  -
    Inside `ParameterizedQualifiedTypeReference.internalResolveLeafType`
    we normally perform a `boundCheck()`. However, in some situations,
    notably during `Scope.connectTypeVariables()` we may not be ready
    yet to perform that check. If argument `checkBounds` is false, the
    check is deferred, adding an element to the list
    `deferredBoundChecks`, which will be processed via
    `ClassScope.checkParameterizedTypeBounds()`
    Similarly `MemberValuePair.resolveTypeExpecting(..)` may add
    runnables to the same list `deferredBoundChecks`. This accounts for
    the fact that resolving annotations may happen at particularly
    unexpected points in time.

<!-- end list -->

  - `LookupEnvironment.deferredEnumMethods`:

<!-- end list -->

  -
    During `scanMethodForNullAnnotation()` we want to mark the generated
    enum methods "valueOf" and "values" as returning a nonnull type. To
    do so we want to add the configured nonnull annotation, but null
    annotations may not yet be initialized, in particular because
    "@NonNullByDefault" depends on an enum, whose "valueOf" and "values"
    methods should be marked as returning nonnull. To cut this circular
    dependency, we check if null annotations have been initialized, and
    if not we add the enum method to `deferredEnumMethods` for
    processing from `usesNullTypeAnnotations()`.

<!-- end list -->

  - `LocalDeclaration.duplicateCheckObligation`:

<!-- end list -->

  -
    *This paragraph is outdated\!*
    To handle a specific scoping issue of instanceof pattern variables,
    the current implementation admits possibly-duplicate variables
    during `LocalDeclaration.resolve()`, because at that point we don't
    yet have the necessary flow information. Only later during
    `analyseCode` we process that deferred `duplicateCheckObligation`.
    ![Image:Warning.gif](Warning.gif "Image:Warning.gif")
    <font color="red">Note, that to be true to the spec, more situations
    need flow information right during resolving, so the entire design
    of separating phases resolve and analyseCode is at stake, see
    </font> ![Image:Warning.gif](Warning.gif "Image:Warning.gif").

# Subpages

## Compilation phases

  - [JDT_Core_Programmer_Guide/ECJ//Parse](JDT_Core_Programmer_Guide/ECJ/Parse "wikilink")
  - Resolve: see
    [JDT_Core_Programmer_Guide/ECJ//Bindings](JDT_Core_Programmer_Guide/ECJ/Bindings "wikilink")
    and
    [JDT_Core_Programmer_Guide/ECJ//Lookups](JDT_Core_Programmer_Guide/ECJ/Lookups "wikilink")
  - [JDT_Core_Programmer_Guide/ECJ//Analyse](JDT_Core_Programmer_Guide/ECJ/Analyse "wikilink")
  - [JDT_Core_Programmer_Guide/ECJ//Generate](JDT_Core_Programmer_Guide/ECJ/Generate "wikilink")

## Data structures

  - [JDT_Core_Programmer_Guide/ECJ//AST](JDT_Core_Programmer_Guide/ECJ/AST "wikilink")
  - [JDT_Core_Programmer_Guide/ECJ//Bindings](JDT_Core_Programmer_Guide/ECJ/Bindings "wikilink")
  - [JDT_Core_Programmer_Guide/ECJ//Lookups](JDT_Core_Programmer_Guide/ECJ/Lookups "wikilink")

## Java concepts

Some Java concepts pose specific challenges for the compiler

  - [JDT_Core_Programmer_Guide/ECJ//Lambda](JDT_Core_Programmer_Guide/ECJ/Lambda "wikilink")

## Strategies for working on the compiler

  - [JDT_Core_Programmer_Guide/ECJ//Investigating](JDT_Core_Programmer_Guide/ECJ/Investigating "wikilink")
  - [JDT_Core_Programmer_Guide/ECJ//Testing](JDT_Core_Programmer_Guide/ECJ/Testing "wikilink")

[category:JDT](category:JDT "wikilink")