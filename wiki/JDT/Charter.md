# The JDT Charter

## Introduction

Java Development Tools or JDT is one of the key building blocks of the
Eclipse IDE project. This document details the processes and the best
practices followed in JDT. The JDT project itself has three parts –
JDT-UI, JDT-DEBUG, and JDT-CORE. Though there could be some tailoring
for each of the components, this document details the guidelines
expected to be followed when contributing to JDT as a whole. If there
are component-specific parts in the charter, those are mentioned
explicitly.

## Activities aka What to Contribute?

JDT has a continuous stream of incoming enhancements and bugs. Very
importantly, new Java Language features released every six months (March
& September) add to these enhancements. Given that JDT has a lot to do
with the limited resources, the question of what to contribute has a
major significance. We have categorized the work we do to make it easier
to focus on what's important. New contributors will also do well if they
adhere to picking up items from this list with appropriate expectations
for review timelines. 

At the planning phase of each release (See [JDT
Planning](https://wiki.eclipse.org/JDT/Charter#JDT_Planning.C2.A0)),
bugs are targeted to specific milestones of the release. We have
prioritized the bugs based on the following guideline. There are a set
of routine tasks that takes a significant bandwidth and time;
nevertheless, they are a must-do with high priority.

### JDT priorities

#### Blocker Issues 

Blocker bugs would be high on the priority. 

#### Java Language Enhancements associated with new Java Releases

New Java Language features released twice a year, brings along a host of
new features. 

#### Supporting new JUnit releases

Some JUnit releases will have new features to implement.

#### Routine Tasks 

1.  Daily
    1.  Inbox triaging for the new bugs
    2.  Read updates of bugs in a domain
    3.  Review jdt-dev mailing list and reply
    4.  Watch out for activities affecting jdt in cross-project dev
    5.  Keep a watch on some of the designated mailing lists for changes
        affecting jdt.
    6.  Review jdt eclipse forum and reply to the queries
    7.  Address problems in build reports
        1.  Test failures
        2.  compile warnings
        3.  Comparator errors
        4.  Version problems.
        5.  Newly added/deprecated API
2.  At Every Milestone
    1.  N\&N entry to be added while closing the enhancement bug or
        latest by Monday EOD of the milestone week
    2.  N\&N entries should be added only after the feature is committed
    3.  Making sure the bug targets are adjusted, if not follow-up and
        close the item before milestone target
    4.  All resolved and targeted bugs to be verified
    5.  Testing for multiple platforms and JREs
    6.  Giving a Go/No Go in the milestone bug
    7.  Regenerate javadoc bundles
3.  Release Activities
    (https://wiki.eclipse.org/Eclipse/Release_checklist/Tasks)
    1.  ReadMe
    2.  Acknowledgement
    3.  What's New
    4.  Tips and Tricks
    5.  Migration Guide
    6.  SWT Javadoc Bash
    7.  Publish to Maven Central
4.  Releng Activities
      - Verify ecj and javac comparison results weekly

#### Technical Debt 

Technical debts from the recent releases would come in the next priority
category. 

#### Bugs 

1.  General bugs would come next. 
2.  We may put explicit 'helpwanted' tags may be put on certain bugs to
    serve two purposes - a) to fix the issue and b) to help aspiring
    committers (contributors) to start and contribute something that
    fits in the overall plan.
3.  Contributors interested in improving JDT are encouraged to pick up
    these bugs for a particular release with priority.  Add a comment to
    the bug with a time-line if you want it to be assigned to you. 

#### Voted Features

1.  If a feature is requested by multiple people using the "vote"
    feature, this can be included in the plan. 
2.  General expectation is that, as a contributor, if you require help /
    guidance, that needs to be planned during the planning phase.  Hence
    it is important to let others know before the planning phase, if you
    need help / guidance. 

#### General Programming Improvement (GPI) bugs 

1.  We will term a bug as a GPI bug if it's primary objective is to
    rewrite existing code with later library constructs, or improving 
    readability or anything to do with general program improvement
    without fixing a bug or adding a new feature. 
2.  Each release will have a root GPI bug with the details listing
    allowed number of GPI bugs, milestones, explicit guidelines,
    exceptions etc. which will be the parent for all GPI bugs.
3.  A GPI bug should clearly state the rationale behind the solution
    with pros and cons.
4.  Project lead has the discretion of allowing a maximum number of GPI
    bugs which will be done during the planning phase, defaulting to 5
    if not mentioned explicitly.
5.  GPI bugs are planned for M1 unless explicitly mentioned by the
    Project Lead in the planning phase.
6.  Mass Changes
    1.  Any GPI bug that has changes of more than a certain number of
        lines or that changes a certain number of files would be
        considered as mass GPIs. If not explicitly mentioned, we suggest
        it to default to 1000 lines or 50 files.
    2.  All the mass changes should be targeted only for M1 milestone or
        a milestone designated by Project lead. 
    3.  Needless to mention explicitly, whoever contributing the changes
        are responsible for the follow up fixes or regressions.
        Otherwise, the initial changes will be reverted. 
    4.  All mass changes need to be reviewed carefully before releasing
        to ensure that they do not cause any unwanted modification (e.g.
        semantic change, addition of new warning, etc.).
    5.  We, as contributors or committers need to resolve any merge
        conflicts with the BETA_JAVA\# branch caused by the mass
        changes.
    6.  Not more than 3 mass changes per release is advised.
    7.  JDT Leads have the discretion, to be used as necessary only, to
        alter the maximum allowed number of mass changes.
7.  Performance GPIs
    1.  Any bug that claims to improve the performance should have data
        to prove that such a change will bring performance improvement. 
    2.  Additionally,  a performance improvement should also give data
        that the place of optimization is indeed a bottle-neck affecting
        existing code.
    3.  Post the gerrit patch, performance data before and after the
        patch to be listed in the bug.
    4.  Performance GPIs should also be targeted for M1 or a milestone
        as designated by Project Lead.
    5.  Expected maximum performance GPIs to be 3 per release or a
        number designated by the Project lead.

## Lifecycle of a JDT Bug

Previous section explained in detail on what to contribute. Except for
some tasks falling under routine category, everything else is tracked by
a bug. It is important to set the Bugzilla fields like Assignee, Target
Milestone, Status, etc. correctly for better querying and monitoring.
This section details the workflow of a JDT bug. As a bug moves through
various stages,  the tasks and information required are listed below:

1.  New
    1.  A newly reported bug is in this state.
    2.  Reporter is expected to provide a reproducible test case
    3.  In cases where it is not possible to provide a test case,
        provide the steps to reproduce - bugs that has a reproducible
        test case will always have a higher probability to get fixed.
    4.  Provide the version number of Eclipse used and any other
        relevant info.
    5.  Person triaging the inbox can either make someone the QA or the
        Owner.
    6.  In JDT.Core the bug is left in the New State
    7.  In JDT.Debug and JDT.UI the bug is moved to Assigned state if it
        is acknowledged as a bug but assignee is default unless someone
        takes up the work
    8.  Separate bugs to be created in JDT Core, UI, and Debug for
        releasing any changes in the respective components. 
2.  Assigned
    1.  A committer who is assigned the task or a project lead moves the
        bug into Assigned state.
    2.  In JDT.Core, this implies that the bug is being actively worked
        on and hence if anyone other than the assignee is interested in
        taking up the bug should contact the assignee before spending
        cycles to avoid duplication of effort.
    3.  For fixes involving Code changes, the following items need to be
        adhered to:
        1.  By first principles, code changes having only test case
            changes are allowed
        2.  All Code fixes should have test cases that test the fix. In
            some exceptional cases, project lead has the discretion to
            waive this off, with the reasons.
        3.  Any visible UI change on an existing feature (display
            strings, mnemonics, options, etc.) needs approval from a JDT
            lead and should have a N\&N entry (where applicable).
        4.  Any change in project settings needs approval from a JDT
            lead and should be highlighted on mailing lists if required.
        5.  Any change in preferences default values needs approval from
            a JDT lead and should be highlighted on mailing lists if
            required.
        6.  The code should be uploaded to the gerrit with the message
            having bug number and title in the format Bug 123456: Title
            of the bug
        7.  If this is a work in progress patch, put WIP in the bug
            title and/or mark work-in-progress in gerrit.
        8.  All the tests should pass, else rework and upload
        9.  Reviewers to be assigned in gerrit; Additionally, reviewers
            can be explicitly requested in Buzilla flags.
        10. Once the review comments are given, the author should rework
            and upload into the gerrit.
        11. After getting a +1 from JDT build and a +2 from the
            reviewer, the code can be committed.
        12. Code cannot be committed if there is a live -1 . No
            exceptions here.
3.  Resolved
    1.  Once the fix is pushed to the branch, the bug moves to resolved
        State.
4.  Reopened (Not in the normal workflow)
    1.  If a bug in resolved state is found to be incomplete or causing
        regressions, the bug is re-opened.
    2.  In case a release is over, and if it is about addressing
        additional scenarios, then creating a follow-up bug is
        preferred.
    3.  A bug gets into the re-opened state only for such cases and this
        is not part of a normal workflow.
5.  Verified
    1.  Once the fix is verified for a particular release, the bug is
        moved to the verified state by us the committers
    2.  Mention the IDE build number with which the bug was verified in
        a comment
    3.  In case the reporter himself verified that the fix works,
        additional verification is optional.

## Roadmap for a Contributor

How do you start contributing and eventually become a committer? How
long will the committer status remain? What are the roles and
responsibilities of a contributor or committer from the JDT perspective?

1.  What can a Contributor expect?
    1.  Contributor is expected to get help for the bugs - hence the
        project lead has to assign a QA for each of these bugs the
        contributor is assigned to.
    2.  Contributor has the right to get feedback in a reasonable amount
        of time, - subject to the bandwidth of committers - the QA
        responsible for the same, escalation path to the lead; in case
        QA and lead are the same, other leads of JDT can be requested to
        review.
    3.  After a reasonable number of major bugs with decreasing number
        of review comments, the contributor can expect to be recommended
        for a committer position, given that the committer has gained
        domain experience in the component. The reasonable number is at
        the discretion of the project lead and may vary with components
        and the state of code at that point in time.
2.  Responsibilities
    1.  We as new contributors are expected to contact the committers or
        the leads and choose appropriate bugs.
    2.  We as contributors are expected to do the routine tasks as well
        probably as a secondary owner as decreed by lead.
    3.  Initially, the contributor is expected to focus on one
        particular area and build expertise in that domain/component.
    4.  Ideal contributor will have a decreasing number of review
        comments in patches submitted in the chosen area of expertise.
    5.  We as contributors are expected to choose the bugs with the
        'helpwanted' tag.
    6.  In case the contributor is planning to contribute a feature,
        that needs to be in the plan for that particular release.
    7.  GPI bugs may not signify domain expertise and hence cannot be
        counted in isolation for getting commit rights.
3.  Committer
    1.  Once a contributor is made a committer, they are expected to
        continue to get the patches reviewed with the QA for a few more
        contributions.
    2.  Only when the project lead gives the go-ahead for the new
        committer to continue without the review, the committer can
        fully get the review waived. 
    3.  Follow up bugs from a committer's/contributor's previously
        released patches should take higher priority over new features. 
    4.  Committers should adhere to the JDT charter and help new
        contributors to do the same.

## JDT Planning 

JDT leads will plan activities for every release based on the items and
guidelines from the previous sections. Everything will be via root bug
for a given release.

Create a root bug in JDT immediately after RC2 of the previous release
for the next one - i.e., immediately at RC2, create a root planning bug
for the next release.

1.  Committers can propose to include items in the plan by listing them
    on the root bug during the planning phase. JDT contributors can
    discuss and evaluate the proposals based on the above guidelines and
    the leads can include them in the release plan based on the outcomes
    of these discussions. 
2.  Each component can choose to have individual bugs, if they choose
    to, then the approved planned item bugs should be the children of
    the JDT root bug for easy tracking.
3.  All the information listed in Section What to Contribute should be
    filled up in the root bug
4.  Project lead may choose to provide more information about who's
    tracking when, for e.g., the assignment/owners for routine tasks.

## How to Change This Document

Just open a bug in JDT for updating the document.

[Category:JDT](Category:JDT "wikilink")