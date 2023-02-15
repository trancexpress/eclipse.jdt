This page is a starting point for where to begin when wanting to
contribute to the project. Contribution means not only contributing code
but also reporting bugs, triaging bugs, answering questions on
newsgroups, documentation, and even spreading the word by blogging or
giving a talk on your local Eclipse DemoCamp. The goal of this wiki page
is to educate and to be as up front as possible with expectations so
that the process can be as efficient as possible.

## Reporting Bugs

Report the bug for the change you want to see in the world.

If you find a bug or have a new feature request, log in to [Eclipse
bugzilla](https://bugs.eclipse.org/bugs/) system and open a bug on the
Eclipse \> JDT \> UI component via this link: [Open JDT UI
bug](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=JDT;component=UI).
See the description of how a [great bug
report](http://borisoneclipse.blogspot.com/2005/10/great-bug-report.html)
looks like. If you find a bug that you think is a duplicate, is not a
bug, etc. feel free to comment saying so.

If wanting to track bug changes in JDT UI there are a few ways:

  - Via e-mail. If you want to receive e-mail when a bug is logged you
    can watch the **jdt-ui-inbox@eclipse.org** user. You will receive
    e-mail anytime a new bug is logged to this user or an update is made
    while assigned to this user. To set this up see Preferences -\>
    Email Preferences -\> User Watching. This will e-mail you for all
    incoming JDT UI bugs.
  - Via Atom. You can convert any query in bugzilla to a feed that will
    update when a matched bug changes. To convert a search to a feed
    perform a search and select "Feed" at the bottom of the search
    results.

## Contributing Code

Whether you're wanting to fix a typo in javadoc or to implement the next
whiz bang feature for JDT UI you'll need to know a few things before you
contribute code. The JDT UI code is contained in a Git repo. Use git or
[EGit](EGit "wikilink") to clone this repo. If you are using the Eclipse
Git team provider, see [Platform-releng/Git Workflows\#Clone a
repo](Platform-releng/Git_Workflows#Clone_a_repo "wikilink") on
suggestions on how to clone a repo and set up a branch.

### Getting the code into your workspace

The JDT UI code is contained in the [Git
repo](http://git.eclipse.org/c/jdt/eclipse.jdt.ui.git/) at

<https://github.com/eclipse-jdt/eclipse.jdt.ui.git>

  - You may not need to import all projects in the repo to your
    workspace. The important JDT UI plug-ins are listed
    [here](http://www.eclipse.org/jdt/ui/dev.php#plugins).
  - To satisfy dependencies on test projects you need to clone the
    following git repositories

<git://git.eclipse.org/gitroot/platform/eclipse.platform.text.git>

for org.eclipse.core.filebuffers.tests, org.eclipse.jface.text.tests,
org.eclipse.text.tests

<git://git.eclipse.org/gitroot/platform/eclipse.platform.releng.git>

for org.eclipse.test.performance

  - It is also recommended to clone the JDT/Core repo and have the
    'org.eclipse.jdt.core' project in your workspace

<git://git.eclipse.org/gitroot/jdt/eclipse.jdt.core.git>

Our current branches:

  - master - development towards the next Eclipse IDE release


The documentation of the Eclipse platform projects including JDT UI can
be found in

<git://git.eclipse.org/gitroot/platform/eclipse.platform.common.git>

### Eclipse installation requirements

  - For development in the master branch, the Eclipse IDE you are using
    should be the latest I-build for the currently developed release.
    This latest build can be found on
    <http://download.eclipse.org/eclipse/downloads/> By default, the
    Eclipse SDK will use the Running Platform as the **target
    platform**.

### Configuring the workspace

Getting the code is the first and the biggest step, but there are a few
more steps

  - JDT UI uses project-specific settings for compiler warnings/errors,
    code formatting etc, which you will automatically get when you clone
    the Git repository. So you do not have to worry about configuring
    these.

<!-- end list -->

  - However we have the following **Save Actions** enabled in our
    workspaces:
      - Format edited lines
      - Organize imports
      - Remove trailing white spaces on non-empty lines

<!-- end list -->

  - Enable **API tooling**, and specify an [appropriate API
    baseline](Version_Numbering#API_Baseline_in_API_Tools "wikilink").
    For example, for Eclipse 4.13 development the baseline should be
    4.12.

<!-- end list -->

  - For each project in your workspace make sure that the corresponding
    JDK version is installed (usually 1.4 to 1.8).
      - On Mac, only 1.6 to 1.8 are available. Let the EEs for Java-SE
        1.4 to 1.6 point to the installed JDK 1.6. To get rid of the
        errors in your workspace, set 'Project Properties \> Java
        Compiler \> Building \> No strictly compatible JRE for execution
        environment available' to 'Warning' for each project and hide
        the modified settings files from Git by using 'context menu \>
        Team \> Advanced \> Assume Unchanged'.

### Contributing a change

#### Provide a contribution using Gerrit

1.  First, make sure you have agreed and signed the [Eclipse
    Contribution CLA](http://www.eclipse.org/legal/clafaq.php).
2.  Then, read **carefully** this documents: [Gerrit](Gerrit "wikilink")
    to set up Gerrit. We recommend to use EGit. There make sure that the
    options *Add Signed-off by* and *Compute Change-Id for Gerrit
    Code-Review* are selected in the Commit dialog.

In case you're not using EGit:

1.  Make your change locally, and *git commit --signoff* them in your
    local repo. Commit message must contain Bug Number.
2.  when you're ready, *git push* your change to Gerrit using the
    following command: ` git push
     `<ssh://username@git.eclipse.org:29418/jdt/eclipse.jdt.ui.git>`
    HEAD:refs/for/master `
3.  After the push, the log tells you about an URL which tracks the
    contribution

In any case

1.  Share this URL on the bug you're working on.

In case you want to provide a better patch for the same change, don't
forget to repeat the **Change-Id** in the footer of your commit message
so Gerrit will be able to link it to existing contribution.

#### or use Bugzilla

We accept patches on [Bugzilla](https://bugs.eclipse.org/bugs). While
attaching a patch on Bugzilla, you need to add an explicit [Certificate
of Origin](https://www.eclipse.org/legal/CoO.php) sign-off comment on
the bug. See [Contributing a patch via
Bugzilla](http://wiki.eclipse.org/Development_Resources/Contributing_via_Git#via_Bugzilla).

### Unit Testing

Testing is imperative to the health of the project. We have a
significant amount of tests. The quantity of tests will keep growing as
more functionality is added to JDT UI. If you are contributing a fix or
writing an enhancement, it is a requirement that tests are written. If
you don't write them a committer will have to and that could slow down
the contribution process.

There are a couple of things that you should know about our testing
process:

  - Most tests are included in **org.eclipse.jdt.ui.tests** and
    **org.eclipse.jdt.ui.tests.refactoring**, but you will need the
    other test plug-ins as well to satisfy dependencies.
  - The main test suite for org.eclipse.jdt.ui.tests is
    **org.eclipse.jdt.ui.tests.AutomatedSuite**, and the main test suite
    for org.eclipse.jdt.ui.tests.refactoring is
    **org.eclipse.jdt.ui.tests.refactoring.all.AllAllRefactoringTests**.
    Those two test suites must be green before submitting or committing
    a change.
  - If you create a new TestCase make sure to add it to the correct test
    suite.
  - If you want to make a good impression, write tests. This goes for
    any project, of course.
  - Testing code against different Java versions should follow a common
    file name pattern:

| Class file name     | Java Version |
| ------------------- | ------------ |
| CleanUpTest1d7.java | Java 7       |
| CleanUpTest1d8.java | Java 8       |
| CleanUpTest9.java   | Java 9       |
| CleanUpTest10.java  | Java 10      |
| ..                  | ..           |

### Coding Conventions

  - Follow the Eclipse Platform's
    [Development_Conventions_and_Guidelines](Development_Conventions_and_Guidelines "wikilink"),
    and the [additional rules used in JDT
    UI](http://www.eclipse.org/jdt/ui/dev.php#documents).
  - Don't duplicate code and don't reinvent the wheel. Consult the
    Javadoc of
    [JDTUIHelperClasses](http://git.eclipse.org/c/jdt/eclipse.jdt.ui.git/plain/org.eclipse.jdt.ui/core%20extension/org/eclipse/jdt/internal/corext/util/JDTUIHelperClasses.java)
    and reuse existing code.
  - The copyright header goes before the package declaration, starting
    in the very first line.
      - For new files, list yourself "and others" instead of "IBM and
        others" in the first line.
      - For each changed file:
          - Update the copyright year (if not the current year)
          - Enter your credentials by filling out the following template
            and adding it to the header comment:

`Your Name <email@example.com> - Bug Title - `<https://bugs.eclipse.org/BUG_NUMBER>

  - It is considered good practice to write code that does not have
    warnings. If possible, fix warnings existing whenever you see them,
    they can crop up due to compiler changes occasionally.
  - Non-externalized strings are considered errors, do not ship
    non-externalized strings.
  - Write/update Javadoc for all API you introduce/change. See [Evolving
    Java-based
    APIs](http://wiki.eclipse.org/index.php/Evolving_Java-based_APIs) by
    Jim des Rivières to understand what it means to maintain an API.
  - It might also be useful to read the detailed document on [things to
    remember when contributing code to JDT
    UI](http://www.eclipse.org/jdt/ui/contributions.txt)

### Before You Check In

  - Commit comments are required for all code commits, bugs should be
    logged to track work and the bug number and a description is then
    used in the commit comment to describe the change. For example when
    fixing a bug, use: "Fixed bug xxx:
    <title of bug>
    ". The "bug xxxx" part is really important as this is what is used
    to relate code changes to bugs.
  - Before committing changes, catch up to all changes made by others,
    and then run the tests.
  - Don't commit your changes if this will cause compile errors or API
    errors for others, or when there are test failures.
  - Check for spotbugs errors. Jenkins has got a separate build to check
    for spotbugs errors in jdt.ui since 4.18 at
    <https://ci.eclipse.org/jdt/job/eclipse.jdt.ui-SpotBugs/>.

### The Build

We commit a change only when the tests are green, hence build failures
should not occur normally. Even so things can go wrong, hence sign up
for the [platform-releng-dev mailing
list](https://dev.eclipse.org/mailman/listinfo/platform-releng-dev).
You'll receive e-mails when builds complete and when build and test
failures occur. It's always good to pay extra special attention on the
mornings after you make a commit or someone makes a commit on your
behalf. The normal reaction to "breaking the build" is to log a bug,
notify the platform-releng-dev list about it so that others can gauge
the quality of the build, and then fix the bug.

## Newsgroup/Forum

We try to be prompt and responsive on the
[newsgroup](http://www.eclipse.org/forums/index.php/f/13/) but there's
always room for improvement. If you know the answer to a query feel free
to respond. It is also helpful to answer questions on stackoverflow
which are tagged 'jdt' or 'eclipse-jdt'.

## Wiki

The wiki is open and can be edited by all. If you find a typo, a broken
link, or anything that you view as a small issue, feel free to fix it.
If you plan to contribute a significant amount of information or create
a new article, we request that you [log a
bug](JDT_UI/How_to_Contribute#Reporting_Bugs "wikilink") so that we're
aware of what you're contributing. This is so that we can ensure
consistency structurally and in the message conveyed.

## Website

In addition to the wiki we have
[website](http://www.eclipse.org/jdt/ui/). Unlike the wiki the website
is not open. However, if you see something wrong please feel free to
provide a patch. The website contents are in a git repository

<git://git.eclipse.org/gitroot/www.eclipse.org/jdt.git>

[Category:How to Contribute](Category:How_to_Contribute "wikilink")