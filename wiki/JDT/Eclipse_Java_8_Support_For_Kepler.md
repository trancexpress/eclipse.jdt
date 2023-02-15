Starting with I20140318-0830 all Luna (4.4) builds on our [downloads
page](http://download.eclipse.org/eclipse/downloads/) contain the
Eclipse support for Java™ 8. For **Kepler SR2 (4.3.2)** a feature patch
needs to be installed. This page describes how to do this.

1.  Help \> Install New Software...
2.  enter the following URL into the 'Work with' field:
        http://download.eclipse.org/eclipse/updates/4.3-P-builds/
3.  press 'Enter'
4.  select category 'Eclipse Java 8 Support (for Kepler SR2)'
5.  for faster install, deselect 'Contact all updates sites during
    install to find required software'
6.  click 'Next'
7.  click 'Next'
8.  accept the license
9.  click 'Finish'
10. restart Eclipse when asked

In order to use the batch compiler only, you can follow the steps
described at the link [Using the batch
compiler](http://help.eclipse.org/topic/org.eclipse.jdt.doc.user/tasks/task-using_batch_compiler.htm),
but use the org.eclipse.jdt.core_3.9.50.v\*.jar from your updated
install.

[Category:JDT](Category:JDT "wikilink")