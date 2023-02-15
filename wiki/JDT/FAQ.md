Frequently Asked Questions for Java Development Tools (JDT)

## JDT Users

### Other FAQ collections

You may find your answer in one the following FAQ collection

  - [The Official Eclipse
    FAQs](The_Official_Eclipse_FAQs#Java_Development_in_Eclipse "wikilink")

<!-- end list -->

  - [IRC FAQ](IRC_FAQ#Java_Development_Tools_.28JDT.29 "wikilink")

<!-- end list -->

  - [Java Development User
    Guide](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/reference/ref-jdt-faq.htm)

<!-- end list -->

  - [JDT Debug FAQ](Debug/FAQ "wikilink")

<!-- end list -->

  - [JDT APT FAQ](http://www.eclipse.org/jdt/apt/faq.html)

### How to disable the navigation bar or the mini package explorer located above the Java editor?

The navigation bar is called the **Breadcrumb**. To disable the
breadcrumb, click **Toggle Java Editor Breadcrumb** in the main toolbar.
Since 3.7, you can also choose **Hide Breadcrumb** from the context menu
of a breadcrumb item. More details on the Java editor breadcrumb can be
found in [Eclipse
help](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/reference/ref-java-editor-breadcrumb.htm).

### Can I use JDT outside Eclipse to compile Java code?

Yes, the batch compiler can be invoked outside Eclipse. These options
exist:

  - Invocation from a plain **command line**. More details can be found
    in the [Java development user
    guide](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_batch_compiler.htm).

<!-- end list -->

  -
    Options listed there may also be available via the other approaches.

<!-- end list -->

  - In **Ant** builds using the `javac` compiler adapter, see [Using the
    ant javac
    adapter](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-ant_javac_adapter.htm)
  - In **Maven** builds using `tycho-compiler-jdt`, see
    [Tycho/FAQ\#Can_I_use_the_Tycho_compiler_support_in_non-OSGi_projects,_too?](Tycho/FAQ#Can_I_use_the_Tycho_compiler_support_in_non-OSGi_projects,_too? "wikilink")

### How can I disable auto-indentation in the Java editor?

You can disable smart insert mode (Edit \> Smart Insert Mode),
alternatively you can also configure the Smart Insert Mode (Windows \>
Preferences \> Java \> Editor \> Typing).

### Can I enable code completion to be activated as I type like how it works in Visual Studio?

Go to Preferences \> Java \> Editor \> Content Assist and paste
"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz." (note the dot
after z) into the "Auto activation triggers for Java:" field.

### Why is Content Assist displaying an empty proposal window?

Check your default proposal generators by navigating to:

`Window > Preferences > Java > Editor > Content Assist > Advanced`

Ensure the top-most table (defining the default content assist list) has
your desired proposal generators. You'll likely want "Java Proposals"

### Code completion inserts meaningless names for method parameters - arg0, arg1. What is the problem?

The argument names are fetched from the source or javadoc and if Eclipse
cannot find them then it can only suggest arg0, arg1, etc. To solve the
problem you can

  - Use a JDK, the sources are bundled with a JDK but not with a JRE
    (see also [IRC
    FAQ](IRC_FAQ#The_javadoc_for_the_standard_Java_classes_does_not_show_up_as_context_help._What_is_the_problem.3F_Should_I_download_the_javadocs.3F "wikilink")).

<!-- end list -->

  - Another solution is to have the Javadoc installed locally. Eclipse
    can also fetch it from the web but this can timeout for slow
    connections in which case you get arg0, arg1, etc.

<!-- end list -->

  - You can also configure Eclipse to insert the best guessed names, see
    Windows \> Preferences \> Java \> Editor \> Content Assist, select
    'Insert best guessed names'.

### Why are some files not getting copied into the output folder?

The Java builder compiles the Java files in the source folders and
copies the rest of un-filtered resources to the output folder. You can
modify the list of filtered resources at:

` Window > Preferences > Java > Compiler > Building > Output folder`

Note that these can also be configured per project.

### In IntelliJ I can map any folder to any package name, can I do the same in Eclipse?

You can, sort of -
<http://www.eclipse.org/forums/index.php/mv/msg/277485/782414/>

### I do not want to use the Eclipse builder, is there some way for me to tell Eclipse to use my program when building my project?

  - open the project's properties
  - go to 'Builders' page
  - click 'New...'
  - choose your program

Also, don't disable auto-build but rather disable the other builder(s)
in the project properties.

## JDT Extenders

### Other FAQ collection

You may find your answer in the [Java Development Tool
API](The_Official_Eclipse_FAQs#Java_Development_Tool_API "wikilink")
section of The Official Eclipse FAQs.

### Can I use JDT outside Eclipse to manipulate Java code?

JDT Core has no dependency on UI side, however it requires a
runtime-workbench. Hence you can use it in an Eclipse headless
application or include all the dependent jar files in the class path of
your application. (You will have to use ASTParser.setSource() and
ASTParser.setEnvironment() to be able to parse non Eclipse Java
projects.)

### How to go from one of IBinding, IJavaElement, ASTNode to another?

#### From an IBinding to its declaring ASTNode

org.eclipse.jdt.core.dom.CompilationUnit.findDeclaringNode(IBinding)

#### From an IBinding to an IJavaElement

org.eclipse.jdt.core.dom.IBinding.getJavaElement()

#### From an ASTNode to an IBinding

Look for a 'resolveBinding()' (or similarly named method) method in a
subtype of ASTNode. Note that not all subtypes of ASTNode have a
corresponding binding, e.g. MethodDeclaration, Expression and
VariableDeclaration have one but IfStatement and ForStatement do not.

#### From an IJavaElement to an IBinding

If you only need the binding key and not the binding object itself, look
for a 'getKey()' method in a subtype of IJavaElement. This method
returns the binding key, which can be useful in many situations e.g. see
next point. Note that not all subtypes of IJavaElement have a
corresponding binding, e.g. IType and IMethod have one but
IPackageFragment and IImportContainer do not.

If you really need the binding objects you can use
'org.eclipse.jdt.core.dom.ASTParser.createBindings(IJavaElement\[\],
IProgressMonitor)'. Note that this operation is slightly expensive,
compared to just getting the binding key, as the bindings have to be
created.

#### From an IJavaElement to its declaring ASTNode

org.eclipse.jdt.core.dom.CompilationUnit.findDeclaringNode(String) - The
string parameter is the binding key, see previous point.

## If your question is not answered above

Consider browsing through the frequently asked questions on
Stackoverflow

  - [FAQs on
    Stackoverflow](http://stackoverflow.com/questions/tagged/jdt?sort=faq)
    - Questions with tag *jdt*

<!-- end list -->

  - [FAQs on
    Stackoverflow](http://stackoverflow.com/questions/tagged/eclipse-jdt?sort=faq)
    - Questions with tag *eclipse-jdt*

Ask a question on the [Official JDT
forum](http://www.eclipse.org/forums/index.php/f/13/), however search
for topics that might be related before asking\!

[Category:JDT](Category:JDT "wikilink")
[Category:FAQ](Category:FAQ "wikilink")