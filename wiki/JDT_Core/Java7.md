This page will summarize all the work that is done to add Java(tm) 7
support (jsr334: project coin) into Eclipse. It might also be extended
to other jsrs if needed. We still need to find out what changes are
supposed to be done to support the jsr 292 in the compiler. This is not
clear yet.

# Current Status at the first glance



| Switch on Strings  |           Binary Literals            | Underscores in Literals |     Multi-catch     | More Precise Rethrow  |
| :----------------: | :----------------------------------: | :---------------------: | :-----------------: | :-------------------: |
| Try-with-Resources | Simplified Varargs Method Invocation |         Diamond         | Polymorphic Methods |      Unicode 6.0      |
|  Update Indexing   |        Update Code Formatter         |     Update DOM/AST      |    Update Search    | Update Error Recovery |

**Java 7 features**



<table>
<thead>
<tr class="header">
<th><p>  </p></th>
<th><p>Completely implemented</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><br />
</p></td>
<td><p>Can be improved</p></td>
</tr>
</tbody>
</table>

# What has been released so far:

  - Add new nodes inside the compiler ast to cover:

<!-- end list -->

1.  Multi catch formal parameter ("catch( Ex1 | Ex2 | Ex3 e) {...").
    The new class is named:
    *org.eclipse.jdt.internal.compiler.ast.UnionTypeReference*

<!-- end list -->

  - The grammar has been updated to cover the new syntax for
    multi-catch, try-with-resources and diamond case.
  - A new DOM/AST node have been added for the union type. The new class
    is: *org.eclipse.jdt.core.dom.UnionType*
  - The class *org.eclipse.jdt.core.dom.TryStatement* has been updated
    to support the new Try-with-Resources statement.
  - Two new constants have been added for the DOM nodes. There are the
    constants for the node type:
       org.eclipse.jdt.core.dom.ASTNode\#UNION_TYPE
  - Experimental handling of
    java.dyn.MethodHandle.invokeExact(..)/invoke(..) (invokeGeneric(..)
    is now deprecated)
  - A new constant has been added for the JLS (JLS4) level in the
    org.eclipse.jdt.core.dom.AST class.
    This constant is needed for the new DOM/AST nodes to be created.
  - Content assist support for switch in strings, try with resources,
    multi-catch exceptions.



# Work complete

  - Support for switch on strings
  - Support for binary literals and underscores in number literals
  - Support for Unicode 6.0 in the scanner using tables like it was done
    for Unicode 5
  - Support for SafeVarargs annotation
  - Support for try with resources
  - Support for the more precise rethrow analysis (this concerns only
    the handling of the throw statement)
  - Support for the improved type inference for generic instance
    creation (Diamond).

If you find bugs in these areas, please report them against
[JDT/Core](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT)
using the tags \[1.7\]\[compiler\]. Thank you.

# What needs to be done:

  - Add statement recovery for the new syntax

# Current status

  - The code is no longer in the BETA state. Everything has been merged
    in HEAD stream (Juno) and in 3_7_maintenance stream (Indigo SR1)
  - JDT UI changes are tracked in [JDT
    UI/Java7](JDT_UI/Java7 "wikilink")
  - JDT Debug changes are tracked in
    [Debug/Java7](Debug/Java7 "wikilink")
  - API Tools changes are tracked in
    [PDE/API_Tools/Java7](PDE/API_Tools/Java7 "wikilink")

# What to do to set up the IDE

Download
[Eclipse 3.7SR2](http://www.eclipse.org/downloads/packages/eclipse-classic-372/indigosr2) for
out of the box support. Or, follow the steps below to get the JDT source
code in an earlier build and launch a runtime workbench which will
contain Java 7 features.

  - Import the following project sets using File \> Import, and then
    Team \> Team Project Set:
      - [BETA_JAVA7.psf](Media:BETA_JAVA7.psf "wikilink") the code
        bundles
      - [BETA_JAVA7_JDT_UI_Tests.psf](Media:BETA_JAVA7_JDT_UI_Tests.psf "wikilink")
        the UI tests
      - [BETA_JAVA7_JDT_Core_Tests.psf](Media:BETA_JAVA7_JDT_Core_Tests.psf "wikilink")
        the Core tests
      - [BETA_JAVA7_API_Tools_Debug_Tests.psf](Media:BETA_JAVA7_API_Tools_Debug_Tests.psf "wikilink")
        the API Tools and Debug tests

The tests project sets contain bundles that have not been branched.
Using the `:pserver:anonymous` connection for those prevents you from
accidentally committing something there.

  - You need to install a JDK7 build as an installed JRE in order to run
    the tests using the JavaSE-1.7 Execution Environment.

<!-- end list -->

  - For more information on how to configure Eclipse to update to the
    latest version of the Java 7 support see the [Java 7 BETA support
    wiki page](JDT/Eclipse_Java_7_Support_\(BETA\) "wikilink").

If you need any help with this, please contact the JDT/Core team through
either the [forums](Www.eclipse.org/forums/index.php "wikilink") or
[Bugzilla](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT).

[Category:JDT](Category:JDT "wikilink")