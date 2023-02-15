**Java™ 8 support for Eclipse is ready, see [JDT/Eclipse Java 8 Support
For Kepler](JDT/Eclipse_Java_8_Support_For_Kepler "wikilink")**. This
page summarizes the work that was done.

# Eclipse Support for Java 8 - Release Schedule

  - Feb 21st 2014: Early Access Release IV: **Release candidate 1**
      - What is new in this release ?
          - JDT/Core and JDT/APT are already feature complete - This is
            a bug fix release with numerous defects resolved

<!-- end list -->

  - March 7th 2014 : **Release candidate 2**
      - Bug fixes

<!-- end list -->

  - March 18th 2014: **General availability release** (**Available** -
    See below for install instructions)

# Installation instructions

Starting with I20140318-0830 all our Luna (4.4) builds contain the
Eclipse support for Java™ 8. For Kepler SR2 (4.3.2) a feature patch is
available. For details go to our [downloads
page](http://download.eclipse.org/eclipse/downloads/).

# Java 8 features ready to be tested (save for some very minor defects)

  - JSR 308 - Type Annotations
  - Default & static methods in interfaces
  - Support for lambda expressions and method/constructor references is
    substantially in place except in the areas of type inference, lambda
    serialization and varargs.
  - Support for Java 8 type inference specification.
  - Quick assist support for migrating anonymous classes to lambda
    expressions and vice versa.
  - Meta data enhancement specification:
      - JEP120: Repeating annotations
      - JEP118: Runtime access to parameter names
  - JSR269 Enhancements for Pluggable Annotation Processor API and
    javax.lang.model APIs
  - Formatter, code completion, code navigation, search & indexing,
    reconciler, incremental builder support for all of Java 8
  - Basic IDE enablement for all of Java 8
  - AST/APIs for all of Java 8
  - Serializable lambda support
  - JSR308 type annotations based null analysis (substantially complete
    - some open issues exist)
  - UI: Basic infrastructure like the Java-related views, Java Compiler
    compliance settings, Organize Imports, Mark Occurrences, Open
    Declaration (F3), Edit \> Expand Selection To \> ..., Content Assist
    (Ctrl+Space), Formatting, and Source Actions should work correctly
    in most situations when you start using them with Java 8 constructs.



# How to report defects ?

File defect reports
[here](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT).
Open work items can be perused
[here](https://bugs.eclipse.org/bugs/buglist.cgi?list_id=7107554&classification=Eclipse&query_format=advanced&bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&component=APT&component=Core&product=JDT&target_milestone=BETA%20J8).

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



# What to do to set up the IDE

  - You need to install a JDK 8 build as an installed JRE in order to
    compile org.eclipse.jdt.annotation and run the tests using the
    JavaSE-1.8 Execution Environment.
      - Use the exact name "JavaSE-1.8" for the JRE

<!-- end list -->

  - Checkout the BETA_JAVA8 branch of the following git repositories:
      - JDT Core repository -
        <git://git.eclipse.org/gitroot/jdt/eclipse.jdt.core.git>
      - JDT UI repository -
        <git://git.eclipse.org/gitroot/jdt/eclipse.jdt.ui.git>
      - JDT Debug repository -
        <git://git.eclipse.org/gitroot/jdt/eclipse.jdt.debug.git>
      - Equinox OSGi repository:
        <git://git.eclipse.org/gitroot/equinox/rt.equinox.framework.git>
      - For tests:
          - Releng repository -
            <git://git.eclipse.org/gitroot/platform/eclipse.platform.releng.git>
          - Text repository -
            <git://git.eclipse.org/gitroot/platform/eclipse.platform.text.git>

<!-- end list -->

  - For more information on how to work with git repositories, look at
    <http://wiki.eclipse.org/Platform-releng/Git_Workflows> .

<!-- end list -->

  - Set R4.3.1 as Target Platform.
  - Set R4.3.1 as [API
    Baseline](Version_Numbering#API_Baseline_in_API_Tools "wikilink").

<!-- end list -->

  - If you are going to be running Ant builds (stand-alone or as project
    builders) using the 1.8 javac target, you should read the following
    wiki: [Ant / Java 8](Ant/Java8 "wikilink").



# Disclaimer

This is a work in progress. The contents of the BETA_JAVA8 branch will
be updated as the changes are made to the JSR Specification. Please use
the early access builds only in a test/evaluation mode and not in the
real development environment. If you need any help with this, please
contact the JDT/Core team through either the
[forum](http://www.eclipse.org/forums/index.php/f/13/) or
[Bugzilla](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT).

[Category:JDT](Category:JDT "wikilink")