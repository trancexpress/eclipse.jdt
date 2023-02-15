This page tracks the work in progress to add Java(TM) 7 support (mainly
jsr334: project coin) into Eclipse JDT UI. See [JDT
Core/Java7](JDT_Core/Java7 "wikilink") for the work in the JDT Core
plug-ins.

# Work areas

Dani will look at Content Assist. For other Eclipse features
(Refactorings, Quick Fixes/Assists, Hovers, Source Actions, Search,
etc.) see assignments below.

| Switch_on_Strings (Raksha) |            Binary_Literals (Raksha)             | Underscores_in_Literals (Raksha) |     Multi-catch (Deepak)      | More_Precise_Rethrow (Deepak) |
| :--------------------------: | :----------------------------------------------: | :--------------------------------: | :---------------------------: | :-----------------------------: |
| Try-with-Resources (Deepak)  | Simplified_Varargs_Method_Invocation (Markus) |          Diamond (Raksha)          | Polymorphic_Methods (Markus) |       Unicode_6.0 (Dani)       |

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
<td><p>Tested and works but can be improved (e.g. by adding Quick Assists or Quick Fixes)</p></td>
</tr>
<tr class="even">
<td><p><br />
</p></td>
<td><p>Not done yet</p></td>
</tr>
</tbody>
</table>

# Things to remember/caveats

  - The following lines should be added in all headers of modified files
    for the Java(TM) 7 implementation:

<code>` * This is an implementation of an early-draft specification developed under the Java`
` * Community Process (JCP) and is made available for testing and evaluation purposes`
` * only. The code is not compatible with any specification of the JCP.`
` *`</code>

  - Goal of the first pass is to make Eclipse work with the new language
    features. For more advanced support (new quick fixes / refactorings
    / templates / ...), please [file a
    bug](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT&component=UI&short_desc=%5B1.7%5D+)
    with the \[1.7\] tag.

<!-- end list -->

  - Bugs that went into the BETA_JAVA7 branch should be RESOLVED/FIXED
    with Target Milestone "---" and the \[1.7\] tag in front of the
    summary.

<!-- end list -->

  - For ASTNode changes:
      - Search for references to a changed AST Node and update code that
        relies on the concrete structure
      - Check references to superclasses of the changed/added node type.
        E.g. for UnionType, code that thinks it handles all known
        subtypes of Type needs to be adjusted.
      - Add new node types to switch or if-else-if statements
      - Think about cases where code could
        use StructuralPropertyDescriptors

<!-- end list -->

  - Think about cases where new bindings can show up.



# How to set up the IDE

See [JDT
Core/Java7\#What_to_do_to_set_up_the_IDE](JDT_Core/Java7#What_to_do_to_set_up_the_IDE "wikilink").

[Category:JDT](Category:JDT "wikilink")