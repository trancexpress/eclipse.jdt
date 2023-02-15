JDT/Core is the Java infrastructure of the Java IDE. It includes:

  - An incremental Java compiler. Implemented as an Eclipse builder, it
    is based on technology evolved from VisualAge for Java compiler. In
    particular, it allows to run and debug code which still contains
    unresolved errors.
  - A Java Model that provides API for navigating the Java element tree.
    The Java element tree defines a Java centric view of a project. It
    surfaces elements like package fragments, compilation units, binary
    classes, types, methods, fields.
  - A Java Document Model providing API for manipulating a structured
    Java source document.
  - Code assist and code select support.
  - An indexed based search infrastructure that is used for searching,
    code assist, type hierarchy computation, and refactoring. The Java
    search engine can accurately find precise matches either in sources
    or binaries.
  - Evaluation support either in a scrapbook page or a debugger context.
  - Source code formatter.

The JDT/Core infrastructure has no built-in JDK version dependencies, it
also does not depend on any particular Java UI and can be run headless.

## Links

  - [JDT Wiki Page](JDT "wikilink")
  - [JDT Core Committer FAQ](JDT_Core_Committer_FAQ "wikilink")
  - [JDT Core Programmer Guide](JDT_Core_Programmer_Guide "wikilink")
      - [A Hitchhiker's Guide to
        ECJ](JDT_Core_Programmer_Guide/ECJ "wikilink")
  - [JDT/FAQ](JDT/FAQ "wikilink")
  - [JDT Core on
    github](https://github.com/eclipse-jdt/eclipse.jdt.core)

## Subpages

[Category:Eclipse_Project](Category:Eclipse_Project "wikilink")
[Category:JDT](Category:JDT "wikilink")