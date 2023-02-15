## Meta FAQ

### What is the status of this FAQ?

### Who is this FAQ for?

This FAQ is for the JDT/Core committers, and anyone who is willing to
participate in the development of JDT/Core.

### What is this FAQ about?

This FAQ is about the rules that JDT/Core committers follow when
developing the JDT/Core component.

### How do I contribute to this FAQ?

Simply edit this page. You will need to [log
in](http://bugs.eclipse.org/bugs/index.cgi?GoAheadAndLogIn=1) using your
bugzilla username and password to gain access. If you don't have a
Bugzilla account, you can [create a new
one](http://bugs.eclipse.org/bugs/createaccount.cgi).

### What should I do after contributing to this FAQ?

You should send a message to jdt-dev@eclipse.org with the diff url (e.g.
<http://wiki.eclipse.org/index.php?title=JDT_Core_Committer_FAQ&diff=50788&oldid=50780>)
so that every JDT/Core committer is aware of the change.

## Setup IDE and workspace

Use either the **"Eclipse IDE for Eclipse Committers"** or **"Eclipse
IDE for RCP and RAP Developers"**. It can be downloaded from:
<https://www.eclipse.org/downloads/packages/>

### Cloning the JDT/Core sourcecode

All the JDT/Core code is in the Eclipse GIT repository. You need to
clone <git://git.eclipse.org/r/jdt/eclipse.jdt.core.git> and import the
following projects from that repository:

  - eclipse.jdt.core\\org.eclipse.jdt.annotation
  - eclipse.jdt.core\\org.eclipse.jdt.annotation_v1
  - eclipse.jdt.core\\org.eclipse.jdt.apt.core
  - eclipse.jdt.core\\org.eclipse.jdt.apt.pluggable.core
  - eclipse.jdt.core\\org.eclipse.jdt.apt.pluggable.tests
  - eclipse.jdt.core\\org.eclipse.jdt.apt.tests
  - eclipse.jdt.core\\org.eclipse.jdt.apt.ui
  - eclipse.jdt.core\\org.eclipse.jdt.compiler.apt
  - eclipse.jdt.core\\org.eclipse.jdt.compiler.apt.tests
  - eclipse.jdt.core\\org.eclipse.jdt.compiler.tool
  - eclipse.jdt.core\\org.eclipse.jdt.compiler.tool.tests
  - eclipse.jdt.core\\org.eclipse.jdt.core
  - eclipse.jdt.core\\org.eclipse.jdt.core.tests.builder
  - eclipse.jdt.core\\org.eclipse.jdt.core.tests.compiler
  - eclipse.jdt.core\\org.eclipse.jdt.core.tests.model
  - eclipse.jdt.core\\org.eclipse.jdt.core.tests.performance
  - eclipse.jdt.core\\org.eclipse.jdt.core.ecj.validation (Optional)

The project `org.eclipse.jdt.core.ecj.validation` only checks that the
classes in source folders `org.eclipse.jdt.core\compiler` and
`org.eclipse.jdt.core\batch` do not reference classes in other source
folders. This project should be imported into the workspace before
working on the compiler. It contains only links to the two mentioned
source folders and will signal errors, if any class outside this scope
is used. The project itself is not intended for editing.

You also need to clone
<git://git.eclipse.org/gitroot/jdt/eclipse.jdt.core.binaries.git> and
import the following project from that repository:

  - eclipse.jdt.core.binaries.git\\org.eclipse.jdt.core.tests.binaries

You also need to clone
<git://git.eclipse.org/gitroot/platform/eclipse.platform.releng.git> and
import the following projects from that repository:

  - eclipse.platform.releng\\org.eclipse.test.performance
  - eclipse.platform.releng\\org.eclipse.test.performance.win32 (only If
    you're using windows)

Please install and enable the POM Version Tool in all your workspaces:

  - [Pom Version
    Tool](https://wiki.eclipse.org/index.php/Version_Numbering#pom.xml_Versions)

### How do I create a Git repository connection?

Go through <http://wiki.eclipse.org/Platform-releng/Git_Workflows> to
see how to set up Egit and create repository connections. If you're not
a committer you should use http instead of ssh, with user id as
'anonymous' and password left blank.

### Which branch should I use?

Bugfixes and new features should be committed to **refs/for/master**
using gerrit. The following branches are used:

  - [master](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/?h=master):
    used for the development of the upcoming release.
  - R4_x_maintenance: used for the maintenance of a release. For
    example,
    [R4_20_maintenance](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/?h=R4_20_maintenance)
    is used for the maintenance of 4.20.

### Configuring the workspace

Getting the sourcecode is the first and the biggest step, but there are
a few more steps

  - JDT/Core uses project-specific settings for compiler
    warnings/errors, code formatting etc, which you will automatically
    get when you clone the Git repository. So you do not have to worry
    about configuring these.

<!-- end list -->

  - Get a 1.8 JDK and set it up in the "Installed JREs" section in the
    Eclipse preferences under "Java". You will need this for JDT/Core
    and tests. You cannot use language features or APIs of Java versions
    greater than 1.8 while writing your code.

<!-- end list -->

  - Install [execution environment
    descriptions](https://wiki.eclipse.org/Execution_Environments#Installing_Execution_Environment_Descriptions).

<!-- end list -->

  - For development in the master branch, the **target platform** should
    be the latest 4.6 I-build. By default, the Eclipse SDK will use the
    Running Platform as the target platform. However you may want to use
    a different Eclipse version for development, e.g. an older stable
    version of Eclipse. In such a case you should still download the
    latest 4.6 I-build and use that as the target platform.

<!-- end list -->

  - Enable **API tooling**, and specify an [appropriate API
    baseline](Version_Numbering#API_Baseline_in_API_Tools "wikilink").
    For example, for 4.5 development the baseline should be 4.4.2 or
    4.4.1 or 4.4 in that order depending on availability of the release.
    Read [Ayushman's
    blogpost](http://eclipseandjazz.blogspot.in/2011/10/of-invalid-references-to-system.html)
    here for more on this.

### How should I format my code?

We currently don't have a formatter profile available. But the rule is
that you should try to format your code as the surrounding code. If you
are not sure, use the Eclipse \[built-in\] formatter profile which is
very close to the JDT/Core team's formatting style. Always follow
the [Eclipse Platform's Standards, Conventions and
Guidelines](http://dev.eclipse.org/conventions.html),

### Comments in code

  - Don't add comments that just describe what the code does. If the
    code is not clear enough, refactor it (rename, extract method, ...).
  - Only add comments that explain non-obvious algorithms or data
    structures. Only add a reference to a bug if it contains a lot of
    discussions that lead to the current solution, or if the code is a
    workaround for an open bug. Show Annotations (git blame) is usually
    good enough to reveal the history.
  - References to **JLS** sections are actually very useful when trying
    to understand the relation of our implementation to the
    specification.

### Unit Testing

Testing is imperative to the health of the project. We have a
significant amount of tests. The quantity of tests will keep growing as
more functionality is added to JDT Core. If you are contributing a fix
or writing an enhancement, it is a requirement that tests are written.
If you don't write them a committer will have to and that could slow
down the contribution process.

There are a few things that you should know about our testing process:

  - Most tests are included in **org.eclipse.jdt.core.tests.\***
    projects.
  - If you create a new TestCase make sure to add it to the correct test
    suite.
  - To launch an existing test, open it in the editor, click on "Run
    Configurations..." in the coolbar, and create a new "JUnit Plug-in
    Test". Go to the "Main" tab and choose "Run an application" and
    select "\[No Application - Headless Mode\]". Also select the runtime
    JRE as the latest and greatest version of java that JDT supports.
    Currently, it is the JavaSE 12. Make sure you have a 12 JDK with
    you.
  - To launch all the JDT/Core tests at once,
    [org.eclipse.jdt.core.tests.RunJDTCoreTests](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/tree/org.eclipse.jdt.core.tests.model/src/org/eclipse/jdt/core/tests/RunJDTCoreTests.java)
    should be run.
  - If you want to make a good impression, write tests. This goes for
    any project, of course.

#### Running tests from the command line

Running tests from the IDE is certainly the easiest approach during
development. If you still need to run tests from the command line, you
might actually let jenkins do it, by submitting your change to gerrit.
If you still want to run tests locally from the command line, here's
how:

Since all Eclipse SDK projects are configured also for building with
Maven/Tycho, you can alternatively build and test JDT/Core on the
command line. This is actually part of what the full SDK build is doing,
see
[Platform-releng/Platform_Build](Platform-releng/Platform_Build "wikilink").

Before doing so, you have to tell Maven, where to find the required
JREs. This is done using a **`toolchains.xml`** file in your user's
`.m2` directory. See
[Platform-releng/toolchainsExample](Platform-releng/toolchainsExample "wikilink")
for an example. Unfortunately, just for all JDT/Core bundles you'll need
versions 1.4, 1.5, 1.6 and 1.8 installed locally. At least we don't use
any versions below 1.4, so you *don't* need to install the execution
environment descriptions mentioned elsewhere.

~~**Caveat**: inside the toolchain file, it says you're specifying a
path to a JDK, which appears to be imprecise: you need to point to a
JRE, so if you have a JDK installed in `/your/path/jdk1.8` the toolchain
must point to `/your/path/jdk1.8/jre`\!~~ *According to  this should no
longer be an issue in tycho 1.1.0 and newer.*

Also note, that apparently you need to define `JAVA_HOME` to point to
your JDK 1.8 install.

At this point you can run the full build and test of JDT/Core saying

`mvn clean verify -V -B -Dmaven.test.skip=false  -Dmaven.repo.local=/path/to/local/maven/repo \`
` -f pom.xml -P build-individual-bundles -P bree-libs`

In this command line only the two profiles `build-individual-bundles`
and `bree-libs` are Eclipse specific, the rest is standard Maven.

We have a series of optional profiles to select which JRE the tests
should run on (if different from the one running the build):
`test-on-javase-9`, `test-on-javase-10`, `test-on-javase-11`,
`test-on-javase-12`.

In theory, you should be able to build/test individual modules, but when
trying this I couldn't convince tycho to find the results of its own
previous module builds (using `install`).

Since running all tests takes a long time, you may want to reduce this
to an individual test class. You can do this by saying s.t. like this:

` mvn verify ... -Dtest=org.eclipse.jdt.core.tests.compiler.regression.NullAnnotationTest`

## What should I do before committing a fix?

  - Make sure the code is free of errors and also does not introduce any
    additional warnings. Make sure all tests are green. You can choose
    to run the tests manually or request Hudson to do that for you.
  - Each fix should have one or more corresponding regression test(s)
    (except fixes for race conditions, fixes for problems that cannot be
    reproduced and doc fixes).
  - When ready, prepare to commit to git (Do not 'push' yet). If you're
    a committer and you want to release your code without a review, you
    can continue reading these steps , otherwise jump to [\#Releasing to
    gerrit code review](#Releasing_to_gerrit_code_review "wikilink").
  - Choose **Team \> Commit**... in EGit to open the Commit dialog. In
    the commit comments, mention the complete bug summary, starting with
    the bug number. E.g.: "Fixed bug 12345: Eclipse compiler bug".
      - Alternatively, a commit can be incrementally crafted using the
        **Git Staging** view.
  - Make sure the author and committer fields contain correct
    credentials. If you're releasing a patch for another person, please
    add their name and email in the "author" field. Click 'Commit'.
  - Do a 'pull' on the jdt.core git repo so that you have the latest
    version of the code.
  - Right-click on the jdt.core repo in the "Git Repositories" view and
    click "Push...". Click next after ensuring that the destination git
    repo is the correct one
    (ssh://userid@git.eclipse.org:29418/jdt/eclipse.jdt.core.git). In
    the ref spec setting, choose the current branch you're working on as
    "source" and in the "destination", type refs/heads/master (or
    refs/heads/R4_x_maintenance or refs/for/userid/topicBranch
    according to where you want to push). Now 'push'.
      - In newer versions of EGit and *if* you are working on master you
        can simply say **Push to Upstream**.
  - Add a comment in the Bugzilla bug that includes a link to the commit
    (<http://git.eclipse.org/c/jdt/eclipse.jdt.core.git/commit/?id=><commitId>).

### Releasing to gerrit code review

1\. Choose **Team \> Commit**... in Egit to open the Commit dialog. In
the commit comments, mention the complete bug summary. Eg: "Fixed bug
12345: Eclipse compiler bug". Also select the "Compute change-id for
gerrit code review" button on the top right corner of the dialog as
shown below. The change-id will be 00... here but as soon as you press
the commit button, egit will populate this with a valid change-id. (The
change-id makes sure that the patch goes in as a fresh patch set to the
existing gerrit review request. For each subsequent patch, you should
use the same change-id.)

![Image:%247FE9CAF0B8F0829E.png](%247FE9CAF0B8F0829E.png
"Image:%247FE9CAF0B8F0829E.png")

2\. If you're not a committer, select the 'Add Signed-off-by' button to
add a 'Signed-off-by: Your Name \<email@example.com\>' line after the
change-id.
3\. Make sure the author and committer fields contain correct
credentials. If you're releasing a patch for another person, please add
their name and email in the "author" field. Click 'Commit'.
4\. Do a 'pull' on the jdt.core git repo so that you have the latest
version of the code.
5\. Right-click on the jdt.core repo in the "Git repositories"
perspective and click "Push to Gerrit...". Ensure that the destination
git repo is the correct one
(ssh://userid@git.eclipse.org:29418/jdt/eclipse.jdt.core.git). Set the
Gerrit branch to refs/for/master (or refs/for/R3_x_maintenance or
refs/for/userid/topicBranch according to where you want to push). Now
click 'Finish'. The result will show a new branch getting created, but
in fact the actual result is the patch is pushed to Gerrit\!
6\. Go to the URL <https://git.eclipse.org/r/#/>. Here you will see the
new changes you just pushed. Click on them and mark someone for
review.
*Things to note:*

  - If you're not a committer, you should ask for your patch to be
    reviewed by a committer. Additionally, you need to update copyrights
    on each file you touched as shown below:
      - The copyright header goes before the package declaration,
        starting in the very first line.
      - For new files, list yourself "and others" instead of "IBM and
        others" in the first line
      - If you're a committer releasing a contributor's patch, make sure
        that the contributor's name and email address is set in the
        "author" field.

### Additional steps to take if you're a committer releasing a contributor's patch

If you're a committer reviewing a contributor's patch in gerrit and want
to +2 the review, hold on and read the steps below

1.  The contributed patch will have both author and committer fields set
    to the credentials of the contributor. You need to change the
    'committer' field to reflect your credentials.
2.  To do so, please fetch the patch into Eclipse from gerrit (see the
    section "Fetching a patch from gerrit").
3.  Without changing anything, choose "Team\>Commit.." and then click
    the "Amend commit" button on top right corner of the commit dialog.
    Now change the 'committer' field to fill in your name and email id
    "registered with Gerrit" (imp. otherwise push will fail).
4.  Now make sure the change-id is part of the commit message and click
    commit.
5.  (Optional, but recommended to keep the repo history clean) Rebase on
    top of origin/master if the change in targetted for master. In case
    the rebase fails, ask the contributor to do it, no need to waste
    your time solving conflicts.
6.  Push to refs/for/<destination branch> as instructed in step 4 of
    "Releasing to gerrit code review" above.
7.  Go to the gerrit link, and mark the patch as +2 for review, and +1
    each for verified and ip clean.
8.  Gerrit auto-releases the patch to git. Close the bugzilla bug.

### Fetching a patch from gerrit

1\. To fetch a patch from gerrit into Eclipse, go to the Gerrit URL
where the review is requested. Click the patch set you want to fetch.
Copy the ref as shown below. This will be the source branch to fetch
from Gerrit.

![Image:Gerrit-patch-branch.png](Gerrit-patch-branch.png
"Image:Gerrit-patch-branch.png")

2.Now go to "Git Repositories" perspective in Eclipse and right-click on
jdt.core repo and choose "Fetch from Gerrit...". 3.Paste the path copied
in step 1 above in the "Change" as shown below.

![Image:Fetch-from-gerrit.png](Fetch-from-gerrit.png
"Image:Fetch-from-gerrit.png")

4\. Click Finish and you will have a new branch created with the
changes.

See
[EGit/User_Guide\#Fetching_a_change_from_a_Gerrit_Code_Review_Server](EGit/User_Guide#Fetching_a_change_from_a_Gerrit_Code_Review_Server "wikilink")
for more info.

### What if another committer makes changes in my code?

Each committer is responsible for checking incoming changes in his/her
own code. So if another committer makes changes in your code, you should
make sure that the change is correct when pulling from the GIT
repository.

### What should I do to backport a fix?

When a bug is fixed in master, it is sometimes interesting (or
necessary) to backport the fix to the maintenance branch. The fix needs
first to be modified in order to be applied on top of the maintenance
branch's code. It may be easy or tricky depending on how the patch's
code area has been touched in the branch.

The bug is either in RESOLVED/FIXED or VERIFIED/FIXED state, which means
respectively either in **State 2** or in **State 2.1** according to the
[JDT/Core bug life
cycle](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink").

So depending of that state, when the fix is ready, you have to:

1.  Set the target of the bug and all its duplicates to the
    corresponding maintenance target (e.g. 3.4.2)
2.  Release the fix in the maintenance branch (e.g. R3_4_maintenance
    branch)
3.  Add the corresponding comment (e.g. *Released for 3.4.2*)
4.  Set the Status Whiteboard to:
    1.  if the bug is in **State 2.1**: "To be verified for
        <maintenance target>" (e.g. *To be verified for 3.4.2*)
        *Note that in this case, the bug might need to be reopened if
        the maintenance stream was frozen owing to a service release end
        of cycle (see [JDT/Core bug life cycle (State
        2.2)](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink").*
    2.  if the bug is in **State 2**: "To be verified for
        <next milestone>" (e.g. *To be verified for 3.5M3*).
        *Note that in this case, the bug might not be reopened
        (typically if the fix is the same than in master branch or
        released at the same time...)*

## Releasing to Git

### How do I maintain the build notes?

Starting from 3.9, an automated e-mail is sent to the release
engineering group with the list of projects that have changed and bugs
that were fixed for the current build. It's been decided not to maintain
a separate build notes for the JDT/Core team. However, should there be a
need to make special announcement for a particular build, we can do so
by keeping a file namely org.eclipse.jdt.core\\buildnotes_jdt-core.html
and putting the content in it. When doing so, also make sure that the
file is either removed or updated for the subsequent build(s).

### What comment should I associate with a commit?

The comment of a commit should be of the form: fixed bug
<bug number with summary>, e.g.

  - fixed bug 12345: Eclipse compiler bug
  - fixed doc for bug 12345

### Can I release code during the milestone week?

Initial answer is: **NO**

The milestone stream has to stay clean until the milestone is declared.
Then, we would be able to fix a stop shipping bug and contribute again
without introducing any other changes since the [verification
process](#What_is_the_verification_process_for_a_milestone.3F "wikilink").

However, looking at milestone history, re-contributing seldom happened,
so **if this is really necessary and only for master branch**, code
releases are allowed if the verification process has well finished and
the stream reopened for development.

See also *[What should I do if I need to contribute after the
verification was
finished?](#What_should_I_do_if_I_need_to_contribute_after_the_verification_was_finished.3F "wikilink")*
question...

### Can I release code during the build input?

Again, the answer is: **NO**

Note: The Build Input process is no longer required/followed as it's all
automated.

The development stream also needs to stay clean during the build input
process. The build input can start between 1.30 AM EST to 5 AM EST on
the day of the build input and it ends when "*Released v_XXX for
today's build input*" message is sent on jdt-core-dev@eclipse.dev
mailing list.

It is generally advisable to not release anything on the build input day
till the build completes.However, if you have an important fix to
release and you're not 100% sure that it will be done before the build
input starts , then just ask to delay it until you're ready by sending
 a note to jdt-core-dev. As the process can be shorten a little bit, it
is possible to accept **<u>safe</u>** contribution until one hour before
the build input starts... But of course you should not abuse this
flexibility.

### How do I warn users after I changed the build state version or the index file version?

Changing the build state version
`org.eclipse.jdt.internal.core.builder.State#VERSION` will cause a full
build when the user restarts the workspace with the new version. As well
changing the index file version
`org.eclipse.jdt.internal.core.index.DiskIndex#SIGNATURE` will cause all
files in the workspace to be reindexed when the user restarts the
workspace with the new version. In both case, you should add an entry in
the build notes in the "What's new" section so that users are aware of
this change.

  - If the build state version is changed, you can add the following
    entry:

<!-- end list -->

    <li>Fix for <a href="http://bugs.eclipse.org/bugs/show_bug.cgi?id=?????">bug ?????</a> required the build state format to change.
        As a consequence, a full rebuild is expected when reusing existing workspaces.
    </li>

  - If the index version is changed, you can add the following entry:

<!-- end list -->

    <li>Fix for <a href="http://bugs.eclipse.org/bugs/show_bug.cgi?id=?????">bug ?????</a> required the index version to be incremented.
        Indexes will be automatically regenerated upon subsequent search queries (accounting for indexing notification in search progress
        dialogs).
    </li>

### What are the rules to release code in the maintenance branch?

You should not start on a fix in maintenance branch (either specific or
backport from master) before having got the +1 from the JDT/Core lead
added as a comment in the corresponding bug. Then, when the fix is
ready, it must be attached to the bug (as usual) and be reviewed by
another JDT developer. No code should be released in the maintenance
branch without these two conditions verified.

### What are the guidelines for release mass cleanup changes?

Time to time, people may want to release mass cleanup changes. It is
agreed upon that the mass changes are welcome in JDT Core at M1
milestones. A bug should be created first explaining the mass changes
and if there is a go ahead, the patch should be put to gerrit for the M1
time frame. IT should be noted that the BETA branch of Java support will
be merged to master every six months during the M1 time frame, and the
owner of the cleanup bug should resolve any conflicts after the BETA
branch is merged, and all the tests should pass. If there are many mass
cleanup bugs lined up, these may be taken up in a phased manner in
subsequent M1 milestones depending on the bandwidth and complexity on a
case to case basis.

## Bugzilla

### What is the JDT/Core bugs life cycle?

**Incoming bugs**

  - Incoming bugs are triaged by a full-time committer on a rotating
    basis. The goal is to have an empty inbox, no bug should stay in the
    inbox for longer than a few days.
  - As part of the triage, we need to make sure that the bug contains
    enough information, and that the "severity" is set to a value that
    makes sense. Enhancement request need to be marked as such, and the
    same is true for blocker/critical bugs. The bug also needs to be
    prefixed with the tag \[componentArea\].
  - Once the bug is triaged and established as a valid bug, it may not
    necessarily be assigned to a committer straightaway. If the bug is
    valid and we feel that we currently do not have enough resources to
    spend time on that bug, the following is done:

<!-- end list -->

1.  The bug is assigned to the jdt-core-triaged@eclipse.org inbox
2.  The QA contact is set to the committer responsible for fixing bugs
    in the particular area.

<!-- end list -->

  - componentArea tags you can use are roughly given below:

`     Prefix                                               Bug related to`
` ----------------------               ----------------------------------------------------------------------------`
` [1.5]                                  JDK 1.5 dvpt`
` [1.6]                                  JDK 1.6 dvpt`
` [1.7]                                  JDK 1.7 dvpt`
` [1.8]                                  JDK 1.8 dvpt`
` [9]                                    JDK 9 dvpt`
` [classpath]                            Classpaths`
` [compiler]                             Compiler (syntax, code gen, etc...)`
` [inference]                            Type Inference in Java 8+`
` [options]                              Compiler options`
` [prefs]                                JavaCore preferences`
` [javadoc]                              Javadoc comments`
` [search]                               Search (engine, participant, etc...)`
` [vm]                                   VM bug occurring in JDT/Core (no action)`
` [index]                                Indexing + search in indexes`
` [newindex]                             New Indexing as of `<https://bugs.eclipse.org/481796>
` [builder]                              Java Builder`
` [select]                               Code select`
` [assist][content assist]               Code assist`
` [format][formatter]                    Code formatter`
` [dom]                                  DOM/AST nodes`
` [model]                                Java model`
` [type hierarchy]                       Type Hierarchy`
` [null]                                 Flow analysis for null pointer problems`
` [external]                             External Null Annotations`
` [resource]                             Flow analysis regarding resource leaks`

  - Additional classification can be handled via the "Tags" field.
    Tokens currently in use include:
      - spec -- *Need clarification re JLS*
      - javac -- *Need clarification re javac*
      - rawtypes -- *Type checking problem involves raw types*
      - lombok -- *Bug is or may be caused by lombok*
      - scala -- *Bug occurs when interfacing to Scala*
      - so -- *For discussions on StackOverflow*

*Note: A bug in the jdt-core-triaged inbox means that it is open to
contributions from anyone who wants to take a shot at fixing it. If you
want to contribute to such a bug, please leave a comment showing your
interest in doing so, or if you're a committer, assign the bug to
yourself.*

When a person is **assigned as the QA contact** for a bug:

  - Double-check that there is enough information in the bug, and that
    the severity is accurate.
  - If it is a blocker or critical bug (e.g. loss of data, crash), it
    needs to be fixed as soon as possible. If you are not able to fix
    the bug yourself, assign it to someone else who is.
  - If it is a regression, it needs to be fixed during the current
    development cycle, or potentially in a maintenance stream as well.
    Schedule the bug accordingly, or assign it to someone else who has
    time to work on it.
  - For all other bugs, follow any activity closely and when you are
    ready to start working on it, assign it to yourself.

Here's the diagram for the main states of the JDT/Core bug life cycle:

![Image:JDTCore_bugs2.jpg](JDTCore_bugs2.jpg "Image:JDTCore_bugs2.jpg")

Please note the following points which may help to better understand the
picture shown above:

1.  not all bug states have been put in this diagram due to an obvious
    the lack of space. So, the represented states are _only_ those for
    which committers need to have a peculiar attention and has to apply
    some rules defined in this FAQ when changing the state of a bug
2.  some branch may not exist for certain bug. Typically, change the
    target of a bug to "Y" will not occur when the bug is not backported
    to another stream than "X". E.g. the left-most branch of this
    diagram will end at the first "VERIFIED" state for this kind of
    bug...
3.  \[ target = X \] means that the committer *[sets the bug target to
    X](#What_target_milestone_should_be_used.3F "wikilink")*
4.  \[ Released for X \] means that the committer released a fix for the
    bug and *[adds the comment 'Released for
    X'](#What_comment_should_be_used_when_fixing_a_bug.3F "wikilink")*
    to the bug
5.  verif X ok ---- means that *[the bug
    verification](#What_is_the_bugs_verification_phase.3F "wikilink")*
    was OK for the milestone/rollup X (failing verifications are not in
    the diagram to keep it readable...)

### What bugs are currently opened against JDT/Core?

[Bugs opened and not resolved
yet.](https://bugs.eclipse.org/bugs/buglist.cgi?query_format=advanced&short_desc_type=allwordssubstr&short_desc=&classification=Eclipse&product=JDT&component=Core&long_desc_type=allwordssubstr&long_desc=&bug_file_loc_type=allwordssubstr&bug_file_loc=&status_whiteboard_type=allwordssubstr&status_whiteboard=&keywords_type=allwords&keywords=&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&bug_status=RESOLVED&resolution=LATER&resolution=REMIND&resolution=---&emailtype1=substring&email1=&emailtype2=substring&email2=&bugidtype=include&bug_id=&votes=&chfieldfrom=&chfieldto=Now&chfieldvalue=&cmdtype=doit&order=Reuse+same+sort+as+last+time&field0-0-0=noop&type0-0-0=noop&value0-0-0=)

### What target milestone should be used?

1.  If you mark the bug FIXED (<b>[State
    1](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b> -\>
    <b>[State
    2](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b>):
      -
        First, assign it to you, (as we always do).
        Then, set the target with the next milestone (as we always do)
2.  If you mark the bug as WORKSFORME (<b>[State
    1](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b> -\>
    <b>[State
    3](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b>):
      -
        First, assign it to you *(this will help bugs dispatching during
        the verification process - see [What are the bugs I have to
        verify](#What_are_the_bugs_I_have_to_verify.3F "wikilink"))*.
        Then,
        a) if you tested the problem with a pure milestone build, use
        this milestone (e.g. if you've tested that a problem works with
        a pure 3.4M1 build, then use 3.4M1)
        b) if you tested the problem with an integration build, use the
        next milestone (e.g. if you've tested that a problem works with
        an integration build after 3.4M1, then use 3.4M2)
3.  If you mark the bug as WONTFIX, INVALID, NOT_ECLIPSE (<b>[State
    1](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b> -\>
    <b>[State
    3](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b>):
      -
        First, assign it to you.
        Then, set the target with the next milestone (in this case, the
        target is simply used to identify that the bug has been closed
        during this specific period...)
4.  If you mark the bug as DUPLICATE (<b>[State
    1](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b> -\>
    <b>[State
    4](#What_is_the_JDT.2FCore_bugs_life_cycle.3F "wikilink")</b>):
      -
        First, assign it to you.
        Then,
        a) if the original bug has not been resolved yet, don't set the
        milestone on the duplicate (it will be set when the original bug
        target is set)
        b) if the original bug has been resolved:
          -
            \- use the same milestone on the duplicate as on the
            original bug
            \- add the following formatted string to the Status
            Whiteboard: "To be verified for <next milestone or rollup>"
            (e.g. *To be verified for 3.5M3* or *To be verified for
            3.4.2*)
            <i>Note that the next milestone or rollup may be different
            than the target of the original bug.</i>

<i><u>Notes</u>
</i> <i> - If a bug is backported to a maintenance stream, then see
*[What should I do to backport a
fix?](#What_should_I_do_to_backport_a_fix.3F "wikilink")* question for
more details</i>
<i> - As soon as you modify the target of a bug, then you have to go
through all its duplicates if any and set these duplicates with the same
milestone as the one used for the original bug.</i>

### What comment should be used when fixing a bug?

When changing a bug to RESOLVED/FIXED a comment containing the following
string should be use: "released for \<target milestone | maintenance
release\>". E.g.

  - Fixed and released for 3.4M2
  - Released for 3.3.1

This string is used when querying the bugs to verify for a particular
milestone.

### The resolution LATER is deprecated. What should I use instead?

Bugs that we don't plan to fix used to be resolved as LATER. Since this
resolution is deprecated, we shall now use the P5 priority.

### The resolution REMIND is deprecated. What should I use instead?

Bugs that needed more information used to be resolved as REMIND. Since
this resolution is deprecated, we shall now use the needinfo keyword.

### What status/resolution should I use after marking a bug duplicate?

  - If the original bug is marked RESOLVED/FIXED, leave it to
    RESOLVED/DUPLICATE
  - If the original bug is marked VERIFIED/FIXED, add the following
    formatted string to the Status Whiteboard: To be verified for
    <next milestone>, e.g. To be verified for 3.4M6

### When can I set a bug to VERIFIED?

The bugs *Resolution* status is only set to VERIFIED during the
verification process the week of the milestone delivery (see *[How do I
verify my bugs?](#How_do_I_verify_my_bugs.3F "wikilink")*)

## Release engineering

### What steps should I follow to make a build contribution?

1.  Ensure that your local clone for **org.eclipse.jdt.core** repository
    is in sync with the remote.
2.  If you would like to announce something via the build notes, i.e.
    apart from the bugs that got fixed in this build, you can do so by
    updating the build notes as mentioned in the build notes section.
3.  For the master branch alone, there is a dedicated **integration**
    branch.
4.  Ensure local master is in sync with the remote repository and does
    not have any local commits or changes.
5.  Checkout integration branch and rebase with master (with integration
    branch checked out, do a **git rebase master** ).
6.  Run all the tests.
7.  Push the rebase integration branch to remote (**git push origin
    integration**)
8.  If required, repeat the above steps for the
    org.eclipse.jdt.core.binariers repository.
9.  For a maintenance stream, since we don't have integration branch,
    there is no build-input involved. Before the build is started,
    ensure sync up the stream concerned and ensure that all tests pass.
    It's possible, but it's been decided not, to have a dedicated
    integration branch for the maintenance streams as well by
    specifying/updating the corresponding entry in
    org.eclipse.releng/tagging/repositories.txt file.

### What is the milestone verification process?

The verification of a milestone is made of two main phases:

1.  bugs verification
2.  build test usage

#### What is the bugs verification phase?

##### What are the bugs I have to verify?

All bugs fixed in the milestone's stream must be verified the week of
the milestone delivery.
(*this is why setting the correct target for bugs is really important -
see [What target milestone should be
used?](#What_target_milestone_should_be_used.3F "wikilink")*)

To avoid a bug verifying by several committers at the same time, the
whole bugs list is split into several ones. An email is sent to each
verifier with the corresponding list of bugs to verify.

For example, the following bugzilla requests were used to follow the
verification progress for milestone 3.4 M2:

  -
    <u>all bugs to verify</u>: [all bugs
    request](https://bugs.eclipse.org/bugs/buglist.cgi?query_format=advanced&classification=Eclipse&product=JDT&component=Core&target_milestone=3.3.1&target_milestone=3.4+M2&order=map_assigned_to.login_name%2Cbugs.bug_id&field0-0-0=bug_status&type0-0-0=notequals&value0-0-0=VERIFIED)
    <u>bugs that Frederic must verify</u>: [verifier's bugs
    request](https://bugs.eclipse.org/bugs/buglist.cgi?query_format=advanced&classification=Eclipse&product=JDT&component=Core&target_milestone=3.3.1&target_milestone=3.4+M2&order=map_assigned_to.login_name%2Cbugs.bug_id&field0-0-0=bug_status&type0-0-0=notequals&value0-0-0=VERIFIED&field0-1-0=assigned_to&type0-1-0=regexp&value0-1-0=frederic)

##### When do I have to verify my bugs?

Usually this verification starts as soon as you get your bugzilla
request and a warm-up build is available on [Eclipse Project Downloads
page](http://download.eclipse.org/eclipse/downloads).

*Note: The corresponding milestone stream is frozen until this
verification is finished (see [When can I consider the verification
finished?](#When_can_I_consider_the_verification_finished.3F "wikilink"))*

##### How do I verify my bugs?

For each bug of your list:

1.  Verify that the resolution state is correct:
      - FIXED
          -
            the problem described in the bug should no longer occur
            using the milestone build
            all duplicate bugs (if any) have to have the same target as
            this bug
      - DUPLICATE
          -
            the problem described in the bug should no longer occur
            using the build <b>and</b> the bug is really a duplicate of
            the original
      - WORKSFORME
          -
            the problem described in the bug does not occur using the
            milestone build
      - INVALID, NOT_ECLIPSE
          -
            the reason why the bug is invalid has to be clearly
            explained and justified
      - WONTFIX
          -
            the reason why the bug was not fixed has to be clearly
            explained
2.  Modify the bug according to the result of your verification:
      - if the verification is OK:
          - add a formatted comment to the bug: <b>"Verified for
            \<*current verified milestone*\> using \<*build name*\>
            build"</b>
              -
                (''Note that for build name is not necessary when
                resolution state is INVALID, NOT_ECLIPSE or WONTFIX).
          - <u>only</u> if the target of the bug matches the
            verification process, set the bug *Status* to VERIFIED
          - reset the *Status Whiteboard* contents if it contains the
            formatted string *To be verified for ...*
      - if the verification is KO:
          - reopen the bug with a comment if the described scenario is
            not fixed
          - open a new bug if a part of the problem is solved but there
            are still remaining issues using different scenarios

##### What if I my fix is not in the build used for the verification?

If your bug fix is not in the build used for the verification (typically
if you commit the fix during or after the verification day), you must
coordinate with a verifier and tell him what build to expect the fix in.

##### When can I consider the verification finished?

When all your bugs are verified, wait for the email sent to all team
members and to the jdt-core-dev@eclipse.org list. This mail describes
the status of the verification and indicates which JDT/Core version is
eligible for the milestone. You are also informed whether the
corresponding stream is reopened for development or not.

Here's a sample of the email sent at then end of the 3.4M2 verification

  -
    *Version v_813 of org.eclipse.jdt.core project should be the
    JDT/Core contribution for 3.4 M2 milestone...*
    *So you can consider JDT/Core projects HEAD reopened for development
    :-)*

##### What should I do if I need to contribute after the verification was finished?

Do NOT panic ;-)

The process depends whether the stream was modified since the last
version or not:

  - when no change was done in the stream:
    1.  send an email to all committers saying that you need to
        contribute again
          -
            ***Note: this freezes the milestone stream until the build
            input's email is sent on jdt-core-dev@eclipse.org***
    2.  release your fix
    3.  do a build input with version v_NNN+1 (see *[What steps should
        I follow to make a build
        contribution?](#What_steps_should_I_follow_to_make_a_build_contribution.3F "wikilink")*)
  - but if some code was already released in the stream since the last
    version:
    1.  if not already done, create a branch which identifies the
        milestone, e.g. ***zz3.4M2***
    2.  release the fix in this branch
    3.  do a build input with version v_NNN+1 (see *[What steps should
        I follow to make a build
        contribution?](#What_steps_should_I_follow_to_make_a_build_contribution.3F "wikilink")*)
    4.  backport the fix into the milestone stream
    5.  backport the version v_NNN+1 into the buildnotes_jdt-core.html
        and also into the messages.properties files

#### What is the build test usage?

When the bugs verification is done and OK, all committers have to use
the **latest** candidate build until the milestone is officially
declared. They should test it either as normal development usage or, if
some more intensive testing is needed, as described in a specific test
plan (e.g. <http://www.eclipse.org/jdt/core/r3.3/test-3.3.1.php>)

### How do I deliver a patch for a maintenance stream?

Note that you should contact the JDT/Core lead first if you need to
deliver a patch for a maintenance stream. In case he/she is not
available, here are the steps to follow:

1.  Increment the plugin version id by finding all text references to
    the previous version id in the org.eclipse.jdt.core project and
    replacing them with the incremented version id.
2.  Add the following entry to the *What's new* section of the build
    notes:
      -

            <li>Plugin version ID got incremented to [plugin version id].</li>
3.  Add a reference to the JDT/Core update area for the current stream.
    E.g. for a patch for 3.0.x, add the following entry to the *What's
    new* section of the build notes:
      -

            <li>Patch available at <a href="http://www.eclipse.org/jdt/core/r3.0/main.html#updates">http://www.eclipse.org/jdt/core/r3.0/main.html#updates</a></li>

        Or for a patch for 3.3.x, add the following entry:

            <li>Patch available at <a href="http://www.eclipse.org/jdt/core/r3.3/index.php#UPDATES">http://www.eclipse.org/jdt/core/r3.3/index.php#UPDATES</a></li>
4.  Close the build notes using the JDT/Core tool (right click on the
    file, then JDT Core tools \> Close build notes).
5.  Add 'Posted on JDT/Core update area' after the date. E.g.
      -

            Eclipse SDK 3.0.3 - February 22, 2008 - Posted on JDT/Core update area
6.  Commit the build notes and the files modified by the plugin version
    id increment.
7.  Tag all JDT/Core projects (even those that haven't changed) with the
    current version (found in the build notes), eg. v_810
8.  Update the `/org.eclipse.releng/maps/jdtcore.map` file with the same
    tag and commit it to the maintenance branch so that the next
    official build will have the same level of fixes
9.  Run the `exportplugin.xml` script with the default target
10. Copy the resulting jar (for 3.2.x or above) or the resulting zip to
    `/jdt-core-www/patches`
11. Remove the previous jar/zip for the current stream
12. Edit the `UPDATES` area of the `index.php` file (or `main.html` for
    3.0.x or below) of the maintenance stream, and change:
    1.  the reference to the jar
    2.  the date
    3.  the size of the jar (as it appears in the Properties of Windows
        Explorer)
    4.  the plugin version id
    5.  the CVS version tag
    6.  the list of bugs that are addressed
    7.  the build notes revision
13. Commit the `index.php` file as well as the new jar addition and the
    old jar removal

## Misc

### How do I get the JDT/Core tools?

The JDT/Core tool can be obtained from this update site:
<http://www.eclipse.org/jdt/core/tools/jdtcoretools/update-site/> using
Eclipse update manager.

### An integration or maintenance build is red. Should I take it?

A red build doesn't mean that it is not usable. It just means that some
tests failed. The impact of failing tests is often negligeable. E.g. if
the releng tests are failing, it just means that there is a problem in
the Javadoc. So you should always take the latest integration or
maintenance build, unless someone posted on the platform-releng-dev
mailing list that it is not usable.

### What about contributions by non-committers?

The Eclipse project uses the Eclipse Foundation's [Automatic IP
Log](Development_Resources/Automatic_IP_Log "wikilink") system. There
are three categories of interesting content for the IP log:

(1) "Small" contributions (patches) coming from non-committers. For
this, we ask you to use the attachment flag "iplog+". Please remember to
add this flag whenever you commit a patch from a non-committer.

(2) "Large" contributions (code \> 1000 lines, new feature work) coming
from non-committers. For this, remember that it has to go through the
proper IP review process with the Eclipse Foundation through IPZilla.

(3) Any new, updated (i.e. new versions being used), modified (i.e. we
modify the code) or removed dependencies on third-party code, usually on
bundles that reside in Orbit. This also has to go through the Eclipse
Foundation IP review process.

"Non-committer", in this context, is defined as: anyone who is not a
committer on the Eclipse project (Platform, JDT, PDE) at the time they
make the contribution.

## Links

  - [JDT Core Wiki Page](JDT_Core "wikilink")
  - [JDT/Core Home Page](http://www.eclipse.org/jdt/core/index.php)
  - [JDT/Core Git
    repo](http://git.eclipse.org/c/jdt/eclipse.jdt.core.git)

[Category:FAQ](Category:FAQ "wikilink")
[Category:JDT](Category:JDT "wikilink")
[Category:How_to_Contribute](Category:How_to_Contribute "wikilink")