This page gives an overview of how to proceed when you want to generify
a Java project (use Java 5 generics). It describes the steps we took to
generify the org.eclipse.jdt.ui project. Most of these steps should also
be applicable for other projects.

# Basics

  - Make sure you understand what you're doing:
      - Read [Evolving Java-based
        APIs](Evolving_Java-based_APIs "wikilink"), especially [Evolving
        Java-based APIs
        2\#Turning_non-generic_types_and_methods_into_generic_ones](Evolving_Java-based_APIs_2#Turning_non-generic_types_and_methods_into_generic_ones "wikilink"):
          -
            <small>So if you plan to generify an API, remember that you
            only get one chance (release), to get it right. In
            particular, if you change a type in an API signature from
            the raw type "List" to "List\<?\>" or "List<Object>", you
            will be locked into that decision. The moral is that
            generifying an existing API is something that should be
            considered from the perspective of the API as a whole rather
            than piecemeal on a method-by-method or class-by-class
            basis.</small>
      - Read about the relevant [Java Programming Language
        Enhancements](http://download.oracle.com/javase/8/docs/technotes/guides/language/enhancements.html).
      - Read the [Java Language
        Specification](http://java.sun.com/docs/books/jls/), or at least
        the
        [Generics](https://docs.oracle.com/javase/tutorial/java/generics/index.html)
        tutorial.
      - Understand the concept of erasure, especially with respect to
        API compatibility (in short: For binary compatibility, only the
        erasure matters).
      - Consult other sources for guidelines. Examples from Angelika
        Langer's excellent [Generics
        FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html):
          - \[<http://www.angelikalanger.com/GenericsFAQ/FAQSections/ProgrammingIdioms.html#Which%20types%20should%20I%20design%20as%20generic%20types>?
            Which types should I design as generic types instead of
            defining them as regular non-generic types?\]
          - \[<http://www.angelikalanger.com/GenericsFAQ/FAQSections/ProgrammingIdioms.html#Should%20I%20use%20wildcards%20in%20the%20return%20type%20of%20a%20method>?
            Should I use wildcards in the return type of a method?\]

<!-- end list -->

  - Generifying APIs requires special care:
      - Communicate to API clients that you're going to generify APIs.
      - Perform the work in a branch and only merge back once you've
        validated the changes by updating several clients of your API.
      - If in doubt, don't add type parameters to types. If really
        necessary, you can still parameterize types later. But you
        cannot add/remove/modify type parameters after the initial
        release\! If you add type parameters, all your clients will have
        to add type arguments all over their code to resolve raw type
        problems.

# Setup

  - Run with enough memory (e.g. `-Xmx1024M`)
  - Make sure you're in sync with HEAD and that other committers are
    informed about your plans (you don't want to merge non-trivial
    changes, since you're going to touch a big percentage of your LOC).
  - In MANIFEST.MF, set `Bundle-RequiredExecutionEnvironment:
    JavaSE-1.7`
  - On the build path, change the "JRE System Library" to "Execution
    Environment: JavaSE-1.7"
      - this will also set the right compliance options on the "Project
        Properties \> Java Compiler" page
  - Make sure the [minor
    segment](Version_Numbering#When_to_change_the_minor_segment "wikilink")
    of the bundle version has been incremented.
  - Make a pass over "Project Properties \> Java Compiler \>
    Errors/Warnings" and turn interesting problems for 1.5 language
    constructs to Warning or Error:
      - Set all options under Annotations to Warning/Enabled, except for
        the last one (disable "Suppress optional errors...")
      - Our general rule (JDT UI team) is that we set options to Error
        iff:
          - there are no false positives (i.e. no "potential" problems)
          - we don't have hundreds of instances of the problem and
            fixing all of them would take too much time
          - it's not a local problem that often occurs during editing
            and where an error would be distracting (e.g. unused stuff)
          - it's not a problem that is regularly triggered by compatible
            API changes (e.g. raw type, unchecked operation)
      - Before we commit a file, we always try to get rid of all
        warnings (if possible with reasonable effort)
      - If you can't generify all dependencies, the [Ignore unavoidable
        generic type
        problems](http://help.eclipse.org/index.jsp?topic=/org.eclipse.jdt.doc.user/tips/jdt_tips.html&anchor=unavoidable-generic-type-problems)
        compiler option can help you get rid of uninteresting raw type
        warnings
          - To disable these warnings in PDE build add
            `javacWarnings..=-unavoidableGenericProblems` to your
            build.properties file
          - To disable these warnings in Tycho in Eclipse SDK builds,
            see the strategy documented for \<code.ignoredWarnings\> in
            the
            [eclipse-platform-parent/pom.xml](http://git.eclipse.org/c/platform/eclipse.platform.releng.aggregator.git/tree/eclipse-platform-parent/pom.xml#n118)
  - Adjust code formatter settings for generics and annotations if
    necessary

# Clean Up

  - Run Clean Ups:
      - Missing Code \> Add missing Annotations \> @Override
      - Missing Code \> Add missing Annotations \> @Deprecated
      - Unnecessary Code \> Remove unnecessary casts (you can also
        enable the others in Unnecessary Code)

The @Override annotation will make some comments unnecessary. Here are
two regular expressions to find boilerplate comments (replace matches
with nothing):

`^[ \t]*/\*\*[\s\*]*\{@inheritDoc\}[\s\*]*\*/\s*\R`
`^[ \t]*/\*[\s\*]*`\(non-Javadoc\)`[\s\*]*@see [^@/]+(?:@since \d+\.\d+)?\s*\*/\s*\R`

Or both in one little monster:

`^[ \t]*/\*(?:\*[\s\*]*\{@inheritDoc\}[\s\*]*|[\s\*]*`\(non-Javadoc\)`[\s\*]*@see [^@/]+(?:@since \d+\.\d+)?\s*)\*/\s*\R`

I also found a few matches with this pattern:

`^[ \t]*/\*[\s\*]*`\(non-Javadoc\)`[\s\*]*Method declared [oi]n \w+\.?[\s\*]*\*/\s*\R`

And this pattern finds pure @see comments that refer to a method with
the same name as the following declaration (replace matches with `\2`):

`^[ \t]*/\*\s[\s\*]*@see [^#@/]+#(\w+)`\([^)]*\)`[\s\*]*(?:@since \d+\.\d+)?\s*\*/\s*\R(\s*@Override[^(]*\1\()`

As always, make sure you look at the changes before committing them\!
Some comments started as (non-Javadoc), but accumulated interesting
explanations later on.

# Required Projects

  - Generify APIs of required projects. If you have non-generified
    dependencies, the "Infer Generic Type Arguments" refactoring will
    wrongly infer <Object> in too many cases (since that's the best
    guess when APIs use raw types).

> E.g to generify the org.eclipse.jdt.core.dom APIs, I used this regex
> search on the whole project:
>
>     \(element ?[Tt]ype: \{@link (\w+)\}\)[^/]*\R\t \*/\R\tpublic List
>
> ... and then I just replaced the match with:
>
>     $0<$1>
>
> This worked quite well because the Javadocs of these APIs are very
> clean, consistent, and they contain the element type information.
>
> Caveat: After this, client code that calls these APIs and then adds
> elements to the parameterized list must make sure the elements are of
> the right type. In some cases, this means that you have to add type
> parameters to other methods as well.
>
> E.g. in ASTRewrite, the right generic version of:
>
>     ASTNode createCopyTarget(ASTNode node)
>
> is:
>
>     <T extends ASTNode> T createCopyTarget(T node)
>
> If you can't release these changes, you'll run into trouble when
> compiling certain client code, e.g.:
>
> ```
>     SingleVariableDeclaration p1= xxx;
>     SingleVariableDeclaration newParam = (SingleVariableDeclaration) rewrite.createCopyTarget(p1);
> ```
>
> The original version of createCopyTarget requires a cast on the second
> line, but the generified version doesn't. I don't have a good solution
> for this problem.
>
> Here's another regex I used to generify property descriptors in the
> DOM AST:
>
>     Search: (public static final Child(?:List)?PropertyDescriptor)( [A-Z_]+\s*=\s*new Child(?:List)?PropertyDescriptor)(\(\w+\.class, "\w+", (\w+)\.class)
>     Replace: \1<\4>\2<\4>\3

  - If you can't generify all dependencies, the [Ignore unavoidable
    generic type
    problems](http://help.eclipse.org/indigo/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FwhatsNew%2Fjdt_whatsnew.html&anchor=unavoidable-generic-type-problems)
    compiler option can help you get rid of uninteresting raw type
    warnings.

# Generify your project

  - Manually add type arguments to your subtypes of generic types:
      - Search for implementations of Comparable, Comparator, Iterator,
        and of the Collection types (Collection, List, Set, Map)
  - Set the Problems view to "Group By \> Java Problem Type", open the
    "Type Safety and Raw Types" group, and look at the "Class is a raw
    type" problems
      - References to java.lang.Class are a bit special, and it usually
        pays out to generify them manually
        (`IAdaptable#getAdapter(Class`<T>`)`, I'm looking at you).
  - Generify your own container types (e.g. MultiMaps, adapters for UI
    list or tree widgets).
  - Run "Refactor \> Infer Generic Type Arguments" on the whole project
    (both options checked)
  - Organize Imports
  - Fix remaining compile errors manually.
  - Check that the changes make sense and don't affect APIs:
      - Be aware that the Infer Generic Type Arguments refactoring does
        not create wildcard type bounds, but in APIs, that would often
        be the right thing to do.
  - Note that some constructs cannot be automatically generified, e.g.
    custom map implementations, or code that uses the same slots for `E`
    and `Collection`<E>.
  - Search for commented type arguments that have sometimes been used
    (e.g. `Map/*<String, Integer>*/ fStringToInteger;`):
      - If equal to the actual arguments, remove the comments:

>
>
>     Search: \s*/\*\s*(<[^>]+>)\s*\*/\s*\1
>     Replace: \1
>     (Caveat: Search expression does not work for nested types like List<List<String>>)

:\* If different from the actual arguments, take a closer look:

>
>
>     Search:  \s*/\*\s*(<[^>]+>)\s*\*/\s*(?!\1)<[^>]+>
>     Search:  \w+\s*/\*\s*[^/]+)\s*\*/\s*<[^>]+>

:\* Query for Collection/\*String\*/<String> (note the missing \< \> in
the comment):

>
>
>     Search: \s*/\*\s*<?\s*([^*]+)\s*>?\s*\*/\s*<\1>
>     Replace: <\1>

  - Make a last manual pass over types with Object as type argument
    (could be correct, but is often not):

>
>
>     Search: <Object|Object>

  - Check all APIs to make sure their erasure didn't change and the type
    arguments have the necessary wildcards (roughly "\<? super X\>" for
    containers on which you need to call setters, "\<? extends X\>" for
    containers on which you need to call getters, and "<X>" for cases
    that should be completely restricted). Note that adding type
    parameters and arguments to APIs is like adding new API: You have
    exactly *one* chance to get it right. Changing generics later will
    be a breaking change most of the time.
      - Search in method parameter lists for parameterized types that
        don't use wildcards:
        >
        >
        >     \w+\s*\([^)]*?\w+\s*<\s*\w+[^()>;:=]+>[^();:=]+\)\s*(\{|;)
      - Method return types typically don't use wildcards, because that
        forces callers to deal with wildcard types.
  - Run "Clean Up \> Enhanced for loop" if you want.

# Closing remarks

  - Set the Problems view to "Group By \> Java Problem Type" and try to
    get the "Type Safety and Raw Types" group empty.
  - Collection\#toArray(T\[\]) is not typesafe, see
    [Bug 7023484](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=7023484).
    Be careful when writing new code, since the compiler will not detect
    bugs in this case.
      - In Java 8, Stream\#toArray(IntFunction\<A\[\]\>) is unsafe for
        the same reason.
  - Methods whose parameter list ends with an array type can be
    converted into a varargs method iff you expect them to be invoked
    with short list of statically known arguments. Be careful in these
    three cases:
      - Array type is `Object[]`, `Cloneable[]`, or `Serializable[]`:
        Invocations of such methods with a single actual argument
        (including `null`) can be confusing, because it's not obvious
        whether the argument should be considered as component type or
        as an array type. And you can lose type safety (see the famous
        example of `Arrays#asList(Object[])`).
      - Component type is a non-reifiable type such as `T` or
        `List`<String>: This can cause heap pollution and compile-time
        warnings for callers.
      - Passing `null` is a common use case: Callers get a warning about
        an inexact type match and need to cast to the array type to
        suppress the warning.

[Category:JDT](Category:JDT "wikilink")