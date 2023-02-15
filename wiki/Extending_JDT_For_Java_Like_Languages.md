The Java Development tools provide a high quality development experience
when working with pure Java code. However, Java-like languages
(languages such as Groovy and Scala that compile to Java byte code and
run on the JVM) are not supported directly in JDT. The rising popularity
of these languages means that there is a push to provide the same
quality of tool support for them as is available for Java. Furthermore,
there is an expectation that tooling can support mixed langugage
code-bases.

Currently, there are a patchwork of solutions for JDT's lack of
extensibility, but there is a large amount of duplication between
solutions and some solutions are incompatible.

On this page, we will sketch out the general requirements for tooling
for these kinds of languages. There is a large amount of overlap of
required functionality between all these languages, but there is also
some that is specific to individual languages. There is no expectation
that all of the suggestions on this page will be adopted into JDT, but
there is a hope that some of these suggestions can be distilled down
into a viable patch that can considered for inclusion in future versions
of JDT.

## Stakeholder languages

There are currently several Java-like languages that use various methods
for providing Eclipse-based tools that extend JDT:

1.  AspectJ [AJDT](AJDT "wikilink")
2.  Groovy [Groovy-Eclipse](http://groovy.codehaus.org/Eclipse+Plugin)
3.  Scala [ScalaIDE](http://www.scala-lang.org/node/94)
4.  Object Teams [Object Teams Development
    Tooling](Object_Teams_Development_Tooling "wikilink")
5.  JavaFX [JavaFX Eclipse
    plugin](http://javafx.com/docs/gettingstarted/eclipse-plugin/)
6.  [JmlEclipse](http://users.encs.concordia.ca/~dsrg/main/projects/jmleclipse/)
7.  Gosu [1](http://gosu-lang.org)

These tools either use AOP to weave into JDT in order to open up
extensibility, or they ship with a feature patch on JDT core.

Over time, there is an expectation that this list will be growing as
there is a push for stronger tool support for more JVM languages.

## General areas of extensions

TODO: What are the general pieces that require extensibility?

## Existing Solutions

TODO: Describe existing solutions and catalogue extensibility points,
eg- aspects and patches

## Open issues against JDT by stakeholder

JDT enhancement ticket

1.  JDT support for java-like languages
    <https://bugs.eclipse.org/bugs/show_bug.cgi?id=36939>

Groovy-Eclipse

1.  Change to JUnit creation wizard:
    <https://bugs.eclipse.org/bugs/show_bug.cgi?id=312204>

[category:JDT](category:JDT "wikilink")