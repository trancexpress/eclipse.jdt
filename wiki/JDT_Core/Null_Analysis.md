<css> div.b {

`   padding:1px;`
`   background-color:#404040;`

} div.hover {

`   background-color:#ffffde;`

} div.hover a.image {

`   float: left;`
`   margin-right: 5px;`

} div.fix {

`   background-color:#ffffff;`

} div.fix a.image {

`   float: left;`
`   margin-right: 5px;`

} </css> This page describes continuing work on improving the static
null analysis of the JDT compiler.

The initial master bug for this work was , this part has been released
for Eclipse Juno (JDT 3.8).

Later, "type annotations" (JSR 308, part of Java 8), have been adopted
for null analysis via  — released with JDT's support for Java 8 and then
Luna (JDT 3.10).

Support for [external
annotations](JDT_Core/Null_Analysis/External_Annotations "wikilink") has
been added via , released with Eclipse Mars (JDT 3.11).

**User documentation** can meanwhile be found in the online help:

  - [Using null
    annotations](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_annotations.htm)
    (Java ≤ 1.7)
  - [Using null type
    annotations](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_type_annotations.htm)
    (Java ≥ 1.8)
  - [Using external null
    annotations](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_external_null_annotations.htm)

When **migrating** from SE5-style null annotations to null type
annotations in Java 8, a few [compatibility
considerations](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_type_annotations.htm%23compatibility)
are relevant.

## Introduction

The static analysis of the JDT compiler detects many potential
programming problems related to the null-ness of variables:
dereferencing a null value (-\> NPE), redundant null checks etc.

However, the analysis in JDT ≤ 3.7 is restricted to flow analysis
**within one method**. No assumptions can be made about

  - arguments flowing into a method
  - return values from method calls and
  - field reads.

In order to include these elements in the analysis one could either

  - use whole program analysis (very expensive - not feasible for a
    (incremental) compiler)
  - explicit contracts via an extended type system or annotations

The second option is well explored in research and some existing tools
(like the Checker Framework, JML, FindBugs) already introduce specific
annotations to this end.

One could argue that advanced analysis should be left to specialized
tools but having something like this in the JDT compiler should show two
benefits:

  - feedback is more immediate and it is available for all JDT users
    without installing more software
  - analysis might be more precise than some existing tools provide,
    because the actual flow analysis in the JDT compiler is already
    pretty strong (unproven claim).

<table>

<tr>

<td>

![Image:Video.png](Video.png "Image:Video.png")

</td>

<td>

See also these presentations:

  - recording of the [ECE 2011](http://eclipsecon.org/europe2011/)
    [session](http://eclipsecon.org/europe2011/sessions/bye-bye-npe):
    **[Bye, bye,
    NPE](http://www.fosslc.org/drupal/content/bye-bye-npe)**
  - [ECNA 2014](https://www.eclipsecon.org/na2014) session **[JDT
    Embraces Type
    Annotations](https://www.eclipsecon.org/na2014/session/jdt-embraces-type-annotations)**
  - [ECE 2014](https://www.eclipsecon.org/europe2014/) session [A Deep
    Dive into the Void - Advanced Null Type
    Annotations](https://www.eclipsecon.org/europe2014/session/deep-dive-void-advanced-null-type-annotations-35-minute-standard-talk)
  - [ECE 2016](https://www.eclipsecon.org/europe2016/) session [The End
    of the world as we know it - AKA your last NullPointerException $1B
    bugs\!](https://www.eclipsecon.org/europe2016/session/end-world-we-know-it-aka-your-last-nullpointerexception-1b-bugs)
    by Michael Vorburger
  - [ECE 2017](https://www.eclipsecon.org/europe2017/) session [Null
    type annotations in
    practice](https://www.eclipsecon.org/europe2017/session/null-type-annotations-practice)
    by Till Brychcy

</td>

</tr>

</table>

## Actual Strategy in the JDT

By default the JDT does not support inter-procedural null analysis,
however, starting with 3.8 the JDT can be configured to use annotations
for extended null checking.

Up-to-date documentation for the annotation-based null analysis and its
new configuration options can be found in the Eclipse help (Eclipse 3.8
and greater):

  - **Java development user guide**
      - **Reference \> Preferences \> Java \> Compiler \>
        Errors/Warnings**
          -
            scroll down to **Null analysis** -- [read
            online](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/reference/preferences/java/compiler/ref-preferences-errors-warnings.htm)
      - **Tasks \> Improving Java code quality**
    <!-- end list -->
      -
        **\> Using null annotations** [read
        online](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_annotations.htm?cp=1_3_9_0)
        **\> Using null type annotations** [read
        online](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_type_annotations.htm&cp=1_3_9_1)
        (*since Luna*)

### Specifying nullness

Null annotations in method signatures can be interpreted as [null
contracts](JDT_Core/Null_Analysis/Null_Contracts "wikilink"), however, a
more general approach considers null annotations as an extension of the
type system. Since the availability of JSR 308 all type references
should either include or exclude null, which allows for complete
checking of any possible dereferencing of null. In other words, a fully
annotated program which passes the type checker will never raise an NPE
at runtime.

To achieve this guarantee two annotations are used. The specific
annotations types can be selected as a preference, but the following
defaults are provided:

  - [org.eclipse.jdt.annotation.NonNull](http://help.eclipse.org/topic/org.eclipse.jdt.doc.isv/reference/api/org/eclipse/jdt/annotation/NonNull.html)
  - [org.eclipse.jdt.annotation.Nullable](http://help.eclipse.org/topic/org.eclipse.jdt.doc.isv/reference/api/org/eclipse/jdt/annotation/Nullable.html)

For any variable who's type is annotated with @NonNull (or the
configured equivalent) the following rules apply:

  - It is illegal to bind null or a value that can be null to the
    variable. (For fields and local variables this applies to
    initialization and assignments, for method argument binding a value
    means to pass an actual argument in a method call).
  - It is legal and safe to dereference such a variable for accessing a
    field or a method of the bound object.

For any variable who's type is annotated with @Nullable (or the
configured equivalent) the following rules apply:

  - It is legal to bind null or a value that can be null to the variable
    (see details above).
  - It is illegal to dereference such a variable for either field or
    method access.

The above rules imply that the value from a @NonNull variable can be
bound to a variable annotated with @Nullable, but the opposite direction
is generally illegal. Only after an explicit null check can a @Nullable
variable be treated as being @NonNull for the sake of binding to another
@NonNull variable or for dereferencing. For fields the situation is
actually more complex — please read ["The case of
fields"](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_annotations.htm?cp=1_3_9_0_4#fields).

For interaction with inheritance see [Null Contract
Inheritance](JDT_Core/Null_Analysis/Null_Contracts#Null_Contract_Inheritance "wikilink").

### Usage

In order to try the new analysis against any existing Java project the
following steps should help:

  - Open the compiler preferences for your project:

:\* Ensure compliance is 1.5 or higher

:\* Find the section **Null analysis** and select **Enable
annotation-based null analysis**

::![Image:annotation-based-null-analysis.png](annotation-based-null-analysis.png
"Image:annotation-based-null-analysis.png")

:\* You will be prompted to update the severity of some null-related
problems, this is recommended.

  - Apply any of the annotations `@NonNull`, `@Nullable` or
    `@NonNullByDefault` in your code.
      - The annotation will be unresolvable at first, but a quick fix is
        offered to update the project setup:
    <!-- end list -->
      -
        (see also: [Help: Setup of the build
        path](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_null_annotations.htm&cp=1_3_9_0_2&anchor=buildpath_setup))
          - **Copy library with default annotations to build path**
            (plain Java projects)
        For Plug-in projects it is recommended to add an optional,
        versioned dependency to the bundle `org.eclipse.jdt.annotation`
  - Define `@NonNull` as the default at the granularity of your choice
    (package/type):
      - **package**: add a file `package-info.java` with contents like
        this:
          -
            `@NonNullByDefault package org.my.pack.age;`
      - **type**: add `@NonNullByDefault` to the type declaration.
  - At this point you should see plenty of new errors and warnings

#### Cleaning up

When applying the new analysis to a big existing project, the sheer
number of new problems may look intimidating but that's where quick
fixes will come to the rescue. Currently the following problems offer a
quickfix:

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Null type mismatch: required
    '@NonNull Foo' but the provided value is null

    </div>

    </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Null type mismatch: required
    '@NonNull Foo' but the provided value is specified as @Nullable

    </div>

    </div>

    ''not in JDT 3.8.0 - see

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Null type mismatch: required
    '@NonNull Foo' but the provided value is inferred as @Nullable

    </div>

    </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_warning_obj.gif](Quickfix_warning_obj.gif
    "Image:Quickfix_warning_obj.gif") Null type safety: The expression
    of type Foo needs unchecked conversion to conform to '@NonNull Foo'

    </div>

    </div>

      -
        **Fixable for these locations: return statements**:
        Note that the mentioned @NonNull declaration may be implicit via
        an applicable default
        In cases 3) and 4) use only with care: the compiler has no clear
        indication if @Nullable was actually intended or not
        The fix is:
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Declare method return as
        @Nullable
        </div>
        </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Null comparison always yields false:
    The variable x is specified as @NonNull

    </div>

    </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Redundant null check: The variable x
    is specified as @NonNull

    </div>

    </div>

      -
        **Fixable for these locations: null check for a method
        parameter**
        The fix is:
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Declare method parameter as
        @Nullable
        </div>
        </div>
        Otherwise a null check may indeed be unnecessary and should be
        deleted.

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") The return type is incompatible with
    the @NonNull return from SuperFoo.foo()

    </div>

    </div>

      -
        **Location: declaration of an overriding method**
        Note again that the mentioned @NonNull declaration may be due to
        a default.
        Possible fixes are:
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Change return type of foo(..) to
        '@NonNull'
        </div>
        </div>
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Change return type of overridden
        foo(..) to '@Nullable'
        </div>
        </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Illegal redefinition of parameter a,
    inherited method from SuperFoo declares this parameter as @Nullable

    </div>

    </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Illegal redefinition of parameter a,
    inherited method from SuperFoo does not constrain this parameter.

    </div>

    </div>

      -
        **Location: Parameter declaration of an overriding method**
        The second form occurs when no null default applies at the scope
        of the super method.
        Possible fixes are:
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Change parameter type to
        '@Nullable'
        </div>
        </div>
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Change parameter type in
        overridden 'foo(..)' to '@NonNull'
        </div>
        </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Missing non-null annotation:
    inherited method from SuperClass declares this parameter as @NonNull

    </div>

    </div>

  -

    <div class="b">

    <div class="hover">

    ![Image:Quickfix_error_obj.gif](Quickfix_error_obj.gif
    "Image:Quickfix_error_obj.gif") Missing nullable annotation:
    inherited method from SuperClass declares this parameter as
    @Nullable

    </div>

    </div>

      -
        **Location: Parameter declaration of an overriding method**
        Quick fix is either of:
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Change parameter type to @NonNull
        </div>
        </div>
        <div class="b">
        <div class="fix">
        ![Image:Correction_change.gif](Correction_change.gif
        "Image:Correction_change.gif") Change parameter type to
        @Nullable
        </div>
        </div>

These quick fixes can be applied...

  - individually (Ctrl-1)
  - all occurrences per file (via the hover)
  - all occurrences (via context menu in the Problems view)

Note, that some quick fixes require to modify another compilation unit
(file) than the one where the problem was observed. For these quickfixes
the current implementation doesn't support fixing several equal issues
in bulk (for the technical background see ).

### Defaults at different levels

If no null annotations are used, the compiler uses the original Java
semantics, where the following is legal for all variables of reference
types:

  - assign `null`, *and*
  - dereference without check.

To generally avoid these weak semantics you may want to declare that by
default all types should be considered as nonnull.

This is done using the annotation 'NonNullByDefault'. The qualified type
name of this annotation can be configured using the preference
"'NonNullByDefault' annotation". The built-in value for these preference
is `org.eclipse.jdt.annotation.NonNullByDefault`.

  - This annotation takes an optional parameter that can be used to
    *cancel* a default that may possible apply at the current location.
    This is useful when, e.g., sub-classing a legacy class without null
    annotation, where the sub-class sits in a place that would otherwise
    apply non-null as the default, which would make all overrides
    incompatible with inherited methods.
      - When using version 1.x of `org.eclipse.jdt.annotation` specify
        **`false`** as the annotation argument.
      - When using version 2.x of `org.eclipse.jdt.annotation` specify
        **`{}`** as the annotation argument.

This annotation can be applied to any package, Java type or method, and
has the following effect:

  - Java ≤ 1.7: The annotation affects all method returns and parameters
    with undefined null status within their scope.
    Starting with Eclipse Kepler, also fields are affected (see ).

  - Java ≥ 1.8: Many more
    [locations](http://help.eclipse.org/topic/org.eclipse.jdt.doc.isv/reference/api/org/eclipse/jdt/annotation/DefaultLocation.html)
    are affected, but local variables are intentionally unaffected by
    any default.

### External Annotations

To close the gap of 3rd party libraries without formally defined null
contracts, starting with Eclipse Mars JDT supports the concept of
[external
annotations](JDT_Core/Null_Analysis/External_Annotations "wikilink").

## Status

### Done

At the current point the following bugs are resolved:

**Since 3.8 (Juno):**

  - .- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] Using annotations for null checking

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif") UI for
    new preferences regarding null annotations (plus a dup: ).

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] check compatibility of inherited null contracts

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] support flexible default mechanism for
    null-annotations

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[batch\] command line options for annotation based null
    analysis

**Since 4.3 (Kepler):**

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] consider null annotations for fields

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] syntactic null analysis for field references

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif") \[quick
    fix\] Add quickfixes for null annotations

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] inheritance of null annotations as an option

**Since 4.4 (Luna):**

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[1.8\]\[compiler\]\[null\] Apply null annotation on types for null
    analysis - *requires Java 8*

**Since 4.5 (Mars):**

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[compiler\]\[null\] Support external null annotations for libraries
    - see
    [JDT_Core/Null_Analysis//External_Annotations](JDT_Core/Null_Analysis/External_Annotations "wikilink")

**Since 4.8 (Photon):**

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif") \[9\]
    Consider @NonNullByDefault in module-info

  - \- ![Image:Ok_green.gif](Ok_green.gif "Image:Ok_green.gif")
    \[null\] Null-checks should honor @TypeQualifierDefault to reduce
    the scope of what's checked

### Future

The following bugzillas address future improvements of the above
strategy:

  - \- ![Image:Glass.gif](Glass.gif "Image:Glass.gif")
    \[compiler\]\[null\] Support a @LazyNonNull annotation for fields

[Category:JDT](Category:JDT "wikilink")