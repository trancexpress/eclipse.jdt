<css> span.status {

`   color: green;`
`   text-decoration: underline;`

} </css>

## Attaching External Annotations to libraries

[Annotation-based null analysis](JDT_Core/Null_Analysis "wikilink") is
incomplete as long as libraries consumed by a project have no null
annotations in their API. To fill this gap, it is now possible to attach
"external annotations" to a library after the fact, i.e., without
touching and re-packaging the library itself.

### Configuration in the IDE

JDT now understands a new **classpath attribute** called
"annotationpath", which - similar to attaching sources and javadoc -
attaches a collection of external annotations to a particular classpath
entry pointing to a library. The value of the new attribute can have any
of these four <span id="shapes">shapes</span>:

  - workspace absolute: starts with '/' and is interpreted relative to
    the workspace
  - project relative: does *not* start with '/' and is interpreted
    relative to the enclosing project
  - variable relative: starts with the name of a defined classpath
    variable
  - file system absolute: if a path starting with '/' cannot be
    interpreted relative to the workspace, it is interpreted as an
    absolute path in the local file system.

<span class="status">Status:</span> all four shapes are support
![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")

Several <span id="packaging">**packagings**</span> of such a collection
of external annotations have been considered:

  - directory tree: this packaging is similar to how Java sources are
    stored in a directory tree, where each directory represents a
    package and each contained file represents one primary type (class,
    interface, enum)
    <span class="status">Status:</span> implemented
    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
  - zip:this packaging simply stores a directory tree in one compressed
    archive file, inside of which the same structure as above is
    preserved
    <span class="status">Status:</span> implemented
    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
  - big <file:if> annotations are sparse, instead of many tiny files it
    might be convenient to combine annotations for many types into one
    file. This could be one file per library or one file per package
    <span class="status">Status:</span> *not implemented*
  - combined:this packaging is a combination of "zip" and "directory
    tree". It is proposed in order to consume existing distributions of
    external annotations in zip format, while still being able to
    incrementally add more annotations as required by the consuming
    project. This packaging is inspired by Eiffel's [melting ice
    technology](https://docs.eiffel.com/book/eiffelstudio/melting-ice-technology).
    For external annotations to melt would mean that annotations for a
    particular type would be extracted from the zip archive and stored
    as an editable file in the corresponding directory tree. Changes are
    then made to this file, which will override the content of the zip
    archive.
    <span class="status">Status:</span> *not implemented*

Individual annotation files use the **file extension** ".eea" for
"Eclipse External Annotations". In zipped format either of ".zip" or
".jar" is accepted.

<span class="status">Details are tracked in these bugs</span>

  - JDT/Core

    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif") \[null\]
    Infrastructure for feeding external annotations into compilation

  - JDT/UI

    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif") External
    annotations need UI to be attached to library jars on the classpath

  - JDT/Debug

    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[launching\] Support attaching external annotations to a JRE
    container

<b>See also:</b> ![Image:new-small.gif](new-small.gif
"Image:new-small.gif") [External
annotations](https://www.eclipse.org/eclipse/news/4.5/M6#external-annotations)
in the [New\&Noteworthy
for 4.5M6](https://www.eclipse.org/eclipse/news/4.5/M6).

### Headless consumption

It is possible to consume external annotations when compiling outside
the IDE or under the control of a build system like ant, maven or
gradle. For this reason the batch compiler must be configured in
situations where no Eclipse project is available as the context for
compilation.

  - dedicated path:Using the command line option "-annotationpath
    *location*" the batch compiler can be configured to read external
    annotations from one or more specified locations. While this allows
    to work with several annotation packages, no connection exists
    between a library and its specific annotation package (see the
    packagings discussed above)
    <span class="status">Status:</span> implemented
    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
  - from classpath:Alternatively, the batch compiler can be configured
    to scan the classpath used for compilation in order to search
    external annotations for any type it is reading from a library. This
    option is less efficient than other approaches, but it provides the
    easiest means for integrating external annotations into existing
    concepts of dependency management - external annotation packages can
    be just additional artifacts to put on the classpath for
    compilation. This option is enabled by specifying "-annotationpath
    CLASSPATH"
    <span class="status">Status:</span> implemented
    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")

<span class="status">Details are tracked in this bug:</span>

  - JDT/Core

    ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[batch\]\[null\] improve command line option for
    external annotations

## File format

On the one hand, the file format *could* remain a private implementation
detail of the compiler, if the IDE just provides the necessary
operations for manipulating external annotations (like, e.g., quick
assists). This *would* make the binary .class file format a natural
choice for implementation, as it allows to store all required
information and can already be interpreted by the compiler.

On the other hand, annotation files should be amenable to storing,
comparing and merging using any version control system. This advocates
the use of a textual format.

Additionally, a publicly defined textual file format will allow other
tools to operate on annotation files (e.g., conversion from/to other
formats, automatic merging etc.), and seasoned users could even directly
edit annotation files (although no dedicated editors are planned).

The design is strongly influenced by the need for a low-footprint
implementation: files must be reasonably small (speaking against any xml
formats) and implementation of file reading must be small and efficient.

The specification of this file format is split into the aspects
**layout** and **encoding**.

### File layout

The basic layout is line-based, i.e., each line represents one element
and linebreaks within an element are not allowed. Each line can be
either of **empty**, **typeHeader**, **superType**, **memberName**,
**originalSignature**, or **annotatedSignature**.

<u>Explanation of elements:</u>

  - typeHeader: starts with the keyword **class** (subsuming also
    interfaces and enums) followed by the qualified name of the type. If
    the next line is a signature, it is interpreted as the type
    parameters of the type.
    superType: starts with the keyword **super** followed by the
    qualified name of a super type (superclass or superInterface).
    Signature on the next line will describe the type arguments of the
    super type.
    memberName: the simple name of a field or method
    originalSignature: directly follows a typeHeader, superType or
    memberName; starts with a single blank followed by the original
    signature of the preceding element using the encoding discussed
    below
    annotatedSignature: optionally follows the originally signature, and
    shares the same format, except that this will contain the actual
    information about annotations

<u>Overall grammar:</u>

  - *annotationFile:*
    ''typeHeader \[ originalTypeParameters \[ annotatedTypeParameters \]
    \] {empty}
      -
        *{ superType \[ originalTypeArguments \[ annotatedTypeArguments
        \] \] {empty} }*
        *{ member }*
  - *originalTypeParamters:*
    *originalSignature*
  - *annotatedTypeParameters:*
    *annotatedSignature*
  - *originalTypeArguments:*
    *originalSignature*
  - *annotatedTypeArguments:*
    *annotatedSignature*
  - *member:*
    *memberName \[ originalSignature \[ annotatedSignature \] \]
    {empty}*

To provide for maximum stability, members of a type should be
**alphabetically sorted**.

It may become relevant to include **meta data** (version of a library,
CRC of original jar, origin of an annotation etc.). While no format for
this has been defined as of yet, it might be prudent to specify that any
of the line formats may contain meta data, which are not to be
interpreted by the compiler. To allow for such future extensions, each
line may already contain arbitrary trailing content separated from what
has been specified above by any amount of white space (blanks and tabs).

<span class="status">Status:</span> implemented
![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")

### Textual encoding of signatures

The textual format is based on signatures as defined in
[JVMS 4.7.9.1](http://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.9.1).
This format is used unaltered for "original signatures", and for
"annotated signatures" the following changes are applied to the grammar
from JVMS (additions in bold face):

  - *ClassTypeSignature:*
    L ***\[Annot\]** \[PackageSpecifier\] SimpleClassTypeSignature
    {ClassTypeSignatureSuffix}* ;
  - *TypeVariableSignature:*
    T ***\[Annot\]** Identifier* ;
  - *ArrayTypeSignature:*
    \[ ***\[Annot\]** JavaTypeSignature*
  - *TypeArgument:*
    *\[WildcardIndicator **\[Annot\]** \] ReferenceTypeSignature*
    *\* **\[Annot\]***
  - *TypeParameter:*
    ***\[Annot\]** Identifier ClassBound {InterfaceBound}*
  - *Annot:*
    **0**
    **1**

This basically means, that after any of the tokens {L, T, \[, +, -, \*}
an optional "0" or "1" can occur, where "0" represents the nullable
annotation, and "1" represents the nonnull annotation. Thus this format
is independent of the concrete configured annotations to be used by the
JDT compiler, and thus will not create a conflict, when external
annotations are contributed from different sources.

Note that in this format, nullness of an array dimension *follows* the
corresponding "\[" token, whereas in Java source syntax it *precedes*
that token, i.e., "@NonNull String @NonNull\[\] @Nullable\[\]"
translated to "\[1\[0L1java/lang/String;". *Curiously, the class file
signature better represents the order of annotations from outer to
inner.*

<span class="status">Status:</span> implemented
![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif"), *except:
files are not yet validated against the specified format nor against the
host library*.

  -

    \[null\] textual encoding of external null annotations

In the future, **alternative formats** may be supported by import /
export functionality.

### Type Annotations (Java 8)?

The work has been started targeting type annotations from Java 8,
because that's the long term goal and provides the greater challenge.
The implementation demonstrates that a subset of these annotations can
be consumed in projects at 1.7-. Technically, annotations are written as
if they were type annotations on a top-level type of a field/method
parameter/method return, but internally these are then interpreted as
declaration annotations of the corresponding field/method
parameter/method.

Ergo: the approach is compatible with both styles of annotations, 1.7-
and TYPE_USE at 1.8+.

### Example

Declaring that "V Map.get(Object key)" may return null (independent of
constraints on V) would be written like this:

`class java/util/Map`
` `<KV>
`get`
` (Ljava/lang/Object;)TV;`
` (Ljava/lang/Object;)T0V;`

## Support for working with external annotations

Some support may be provided by JDT/UI, but the design should be open
for contributions in separate plug-ins.

### Quick fix

~~Existing quick fixes that are offered on compiler errors/warnings
regarding null problems should be extended so that they produce external
annotations instead of inserting annotations into Java source code.~~

Withdrawn. A quick fix of this kind would encourage people to quickly
insert annotations for libraries while only looking at their own client
code. Instead people should be directed to closely investigate the
library and insert annotations from there. See next.

### Point-and-click annotating

A new command "Annotate" is offered when looking at the (attached source
code of) a library (technically: when working with a ClassFileEditor).
This command allows users to precisely point to any type in a signature,
including type parameters etc. This way not only declaration annotations
but also type annotations can be attached to library methods.

<span class="status">Status:</span> implemented
![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")

  -

    \[null\] "Annotate" proposals for adding external null annotations
    to library classes

    based on new UI mechanics from

<b>See also:</b> ![Image:new-small.gif](new-small.gif
"Image:new-small.gif") [Annotate
command](https://www.eclipse.org/eclipse/news/4.5/M6#annotate-command)
in the [New\&Noteworthy
for 4.5M6](https://www.eclipse.org/eclipse/news/4.5/M6).

### Visualization

In the IDE external annotations should be visualized where appropriate.
Javadoc hovers and the Javadoc view show the effective annotated
signature (in Java syntax). Also visualization in the outline would be
interesting, but here a more compact representation would be needed,
perhaps using decorations like <font color="red">?</font> and
<font color="green">\!</font>.

<span class="status">Status:</span> implemented
![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")

  -

    \[1.8\] Render TYPE_USE annotations in Javadoc hover/view

### Additional operations

The following operations can be contributed by a separate plug-in:

  - **Creating an annotation template file from any class file.** The
    template would contain only original signatures, users could then
    copy any signature to create an annotated signature, i.e., manually
    creating external annotations would consist of:
      - create a template for the type
      - copy the signature of a member
      - insert 0's and 1's representing annotations
  - **Import/export from/to other formats.** Given that existing tools
    already define formats for external annotations, a new import wizard
    could support to import and convert a file from such format into the
    Eclipse format. Correspondingly for export.
  - **Inferring null annotations**. Tools exist that analyse the class
    or source files of a library, and infer null annotations. A UI
    operation should support invoking such a tool and converting the
    output into the Eclipse format.
  - Integrations like **m2e** could recognize project dependencies as
    containing external annotations and set up the project's classpath
    in Eclipse to point to the external annotation attachment.
  - ...

All this should be possible based just on the following API:

  - Definition of the file format
  - Specification of the extra classpath attribute

<span class="status">See also these bugs:</span>

  - JDT/Core:

    \[null\] allow more fine grained mapping from class path containers
    to annotation paths

      - PDE:

        \[patch\] Prepare for external null annotations

      - M2E:

        Prepare for external null annotations

## Collecting annotations

To become really effective the technical approach should be accompanied
with a **community effort** to create and collect external null
annotations for all relevant libraries.

### Manually maintain like source code

The simplest approach for this community effort would be to create a
forest of source repositories holding the external annotation files,
either one library per repo, or "big" repos comprising multiple
libraries. No new technology is needed for this.

From a technical p.o.v. even packaging external annotations as zip,
uploading to some central server and discovery of these artifacts using
the build technology of your choice are all straight-forward.

Issues to address here include: defining the **rules** and
**conventions** for the above, and the questions whether enough people
are willing to invest the effort for such manual maintenance of an
external annotation packge.

### Facilitate crowd-sourcing

To lower the barrier of contributing to a global collection of external
annotations, techniques from crowd-sourcing could be integrated into the
typical workflows, to automatically collect the relevant data directly
from members of the crowd. This approach consists of a phase of
**collection** and a phase of **discovery**.

**Collecting** external annotations could be directly hooked into some
of the *operations* mentioned
[above](#Support_for_working_with_external_annotations "wikilink").
Interesting events to observe for this purpose include:

  - changes of annotation files in the workspace can be observed using
    an IResourceChangeListener. Based on the definition of the file
    format, such a listener can recognize relevant changes and further
    process / upload as desired.
  - adding an external annotation to a particular signature.

<!-- end list -->

  -

    has been filed to investigate, what should be provided by JDT to
    allow other plugins to receive the relevant events / information.

From this it appears, that by resolving the mentioned  the design should
be sufficiently open for implementing a collector as a separate plug-in.

**Discovering** external annotations: External plug-ins are free to
analyse each project's classpath (perhaps triggered by a classpath
change event), and to consult any well-known server(s) whether external
annotations for referenced libraries (version) are available. If so,
after download the project can simply be updated by adding the necessary
extra classpath attribute to point to the external annotation
attachment.

No additional API in JDT seems to be necessary for this purpose.

<span class="status">Details are discussed in these bugs:</span>

  -

    Repository for external null annotations

    \[null\] extension point to enable crowd-sourcing of external
    annotations

      -
        *here "extension point" is said as a placeholder for any kind of
        API addition*

## LastNPE.org

<http://lastNPE.org> is an open source community with resources related
to developing NullPointerException safe code in Eclipse. It maintains a
growing repository of External Annotations ("EEAs") for various
libraries as well as the core JDK, and welcomes contributions.

## References

  - Formats:

:\*[Annotation File
Format](http://types.cs.washington.edu/annotation-file-utilities/annotation-file-format.html),
extension .jaif, by the group of Michael Ernst.

:\*IntelliJ have their own XML format for external annotations, *but I
couldn't find a specification of that format, anybody?*

  - Tools:
    [Nit: Nullability Inference Tool](http://nit.gforge.inria.fr/)
    [JastAddJ
    NonNullInference](https://bitbucket.org/jastadd/jastaddj-nonnullinference)
    [KAnnotator](https://github.com/jetbrains/kannotator)

[Category:JDT](Category:JDT "wikilink")