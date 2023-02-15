# Overview

The purpose of this document is give insight into JDT Core internals.

If you're looking for information on **JDT APIs** you should start by
visiting the [JDT Core](http://help.eclipse.org/indigo/index.jsp?nav=/3)
section in Eclipse Help. You can also check [How to Train the JDT
Dragon](https://github.com/aupsy/org.eclipsecon2012.misc.tutorial/blob/master/Slides/How%20To%20Train%20the%20JDT%20Dragon%20combined.ppt)
presentation by Ayushman Jain and Stephan Herrmann. Instructions for the
tutorial can be found
[here](https://github.com/aupsy/org.eclipsecon2012.misc.tutorial/tree/master/Handouts).

Many issues of **working with JDT/Core source code** are covered in the
[JDT_Core_Committer_FAQ](JDT_Core_Committer_FAQ "wikilink"),
specifically:

  - The [Coding](JDT_Core_Committer_FAQ#Coding "wikilink") section
    covers getting the code from git and getting it to compile in your
    workspace

If the answer to your question is neither there nor here, [ask the
question](http://wiki.eclipse.org/JDT/FAQ#If_your_question_is_not_answered_above).

# Concepts

## Java Model

Java model is a lightweight model for views.

## Search Engine

Indexes of declarations, references and type hierarchy relationships.

### Searching steps

1.  [\#Indexing](#Indexing "wikilink") phase
2.  Get the file names from indexes
3.  Validate the scope
4.  Parse the file and find out matching nodes
5.  Resolve types (see
    [CompilationUnitDeclaration\#resolve()](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core/compiler/org/eclipse/jdt/internal/compiler/ast/CompilationUnitDeclaration.java#n528))
    and narrow down matches reported from index
6.  Create the appropriate model element and call the requestor

### Indexing

~~For information about the new index, see [this
page](https://wiki.eclipse.org/JDT_Core_Index_Programmer_Guide)~~ As of
the "new index" has been removed.

Information about the ~~legacy~~ actual index:

  - Stored in files in workspace/.metadata/org.eclipse.jdt.core
  - Separate index file for each class path entry (shared across
    projects)
  - Dedicated daemon thread
  - Names of declaration and references of JavaElements used in the file
    are stored in the index
  - First stored in Memory and then goes into the Disk
      - In Memory “fileName-\>category-\>names”
      - In Disk “Category-\>names-\>fileName”
  - For jars, .class files are read (even if source is available)
  - References are read from the constant pool

See also [JDT Core Programmer
Guide/MetaIndex](JDT_Core_Programmer_Guide/MetaIndex "wikilink")

### Precise and non-precise matches

If there is a search result for the given pattern, but a problem (e.g.
errors in code, incomplete class path) occurred, the match is considered
as
[inaccurate](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core/search/org/eclipse/jdt/core/search/SearchMatch.java#n47).

Inaccurate matches used to be called
[potential](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core/search/org/eclipse/jdt/core/search/IJavaSearchResultCollector.java#n60)
matches and as such can still be seen in the
[tests](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core.tests.model/src/org/eclipse/jdt/core/tests/model/JavaSearchBugsTests2.java)
and tracing messages.

### Using the APIs, an example

Create a search pattern:

`SearchPattern pattern = SearchPattern.createPattern(`
` "foo(*) int", `
` IJavaSearchConstants.METHOD, `
` IJavaSearchConstants.DECLARATIONS, `
` SearchPattern.R_PATTERN_MATCH`
`);`

Create a search scope:

`IJavaSearchScope scope = SearchEngine.createWorkspaceScope();`

Collect results using SearchRequestor subclass:

`SearchRequestor requestor = new SearchRequestor() {`
` public void acceptSearchMatch(SearchMatch match) {`
`   System.out.println(match.getElement());`
` }`
`};`

Start search:

`new SearchEngine().search(`
` pattern, `
` new SearchParticipant[] { SearchEngine.getDefaultSearchParticipant()}, `
` scope, `
` requestor, `
` null /*progress monitor*/`
`);`

## AST

Precise, fully resolved compiler parse tree.

## Compiler

See -\>
[JDT_Core_Programmer_Guide//ECJ](JDT_Core_Programmer_Guide/ECJ "wikilink").

# Tests

## Unit tests

To run unit tests follow the instructions under
[JDT_Core_Committer_FAQ\#Unit_Testing](JDT_Core_Committer_FAQ#Unit_Testing "wikilink").

New Java Search tests should be added to
[JavaSearchBugsTests2](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core.tests.model/src/org/eclipse/jdt/core/tests/model/JavaSearchBugsTests2.java).

## Performance tests

To run performance tests you need to have the following projects in your
workspace:

  - org.eclipse.jdt.core.tests.performance
  - org.eclipse.jdt.core.tests.binaries
  - org.eclipse.test.performance
  - org.eclipse.test.performance.win32 (if you're on Windows)

More info about obtaining the JDT/Core source code can be found here:
[JDT_Core_Committer_FAQ\#Where_is_the_JDT.2FCore_code.3F](JDT_Core_Committer_FAQ#Where_is_the_JDT.2FCore_code.3F "wikilink").

Useful VM arguments:

  - \-Dmeasures=1 to reduce number of measurements to 1, default is 10
  - \-Dprint=true to print more test details on the console
  - \-Ddebug=true to print debug info on the console

# Subpages

[Category:JDT](Category:JDT "wikilink")