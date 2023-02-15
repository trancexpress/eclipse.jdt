This page is a simple starting point for where to begin when wanting to
contribute to the JDT project using Oomph. The goal of this wiki page is
to help users to set up coding workspace for JDT components using Oomph.

## Setting up Oomph

Oomph can be extracted to a local folder after downloading it from
[Eclipse Installer](Eclipse_Installer "wikilink") (by Oomph).
You can learn more about the Oomph project at
<https://projects.eclipse.org/proposals/oomph>
FAQ is available at [Eclipse_Oomph_FAQ](Eclipse_Oomph_FAQ "wikilink")

## Using Oomph

### Selecting Advanced Mode

Code setup using Oomph Installer for Eclipse is available in Advanced
mode.
![<File:OomphAdvancedMode.PNG>](OomphAdvancedMode.PNG
"File:OomphAdvancedMode.PNG")

### Commonly used Preferences

Eclipse has many workspace-specific preferences. Oomph helps us set the
preferences across all workspaces.
Oomph can manage following preferences and by enabling the last
preference (Enable Oomph Preference Recorder), any preference set later
can be managed by Oomph:
\* Refresh Resources Automatically?
\* Show Line Numbers in Editors?
\* Check Spelling in Text Editors?
\* Execute Jobs in Background?
\* Encode Text Files with UTF-8?
\* Enable Oomph Preference Recorder?

### Eclipse Product and Version selection

Select Eclipse Product as <B>"Eclipse IDE for Java Developers"</B>
Product version can be selected as required.

![<File:ProductSelection.PNG>](ProductSelection.PNG
"File:ProductSelection.PNG")

### Project Selection

Projects to be selected for components JDT Core, JDT UI and JDT debug:

![<File:CoreProject.PNG>](CoreProject.PNG
"File:CoreProject.PNG")![<File:UIProject.PNG>](UIProject.PNG
"File:UIProject.PNG")![<File:DebugProject.PNG>](DebugProject.PNG
"File:DebugProject.PNG")

### Variable Selection

Rules and details for Install, Workspace and JRE can be selected based
on user choice. For each project (Core, UI and Debug) Git Feature
repository connection method needs to be selected. For Core Project
there are 2 repositories( Core and Binaries). User can use Git or Gerrit
based on there convenience. User without Commit rights should use HTTPS
and with commit rights should use SSH. If they have Gerrit account, they
can use HTTPS(read-write, gerrit). HTTP (read only, anonymous, direct)
will be the simplest to use.

### Wait for the setup to complete

After Eclipse launches, the workspace is still being set up for JDT
development. See [the platform SDK provisioning
page](Eclipse_Platform_SDK_Provisioning#Provision_the_Workspace "wikilink")
for details.

## Using Oomph but checkout and prepare older branch (2020-06)

### Eclipse IDE for Eclipse commiters

Choose the "Product Version" you want. Take care of the java version.
For 2020-06 we need java 8.

![<File:ScreenshotOOMP1.png>](ScreenshotOOMP1.png
"File:ScreenshotOOMP1.png")

### Projects

Choose the projects - all JDT projects and platform news for the "News &
Noteworthy" pages could be useful.

![<File:ScreenshotOOMP2_checkprojects.png>](ScreenshotOOMP2_checkprojects.png
"File:ScreenshotOOMP2_checkprojects.png")

### Variables

It automatically increases a postfix number in case you have already a
oomph based project with the same name.

![<File:ScreenshotOOMP3_entervalues.png>](ScreenshotOOMP3_entervalues.png
"File:ScreenshotOOMP3_entervalues.png")

### Review

Review tasks

![<File:ScreenshotOOMP4_reviewtasks.png>](ScreenshotOOMP4_reviewtasks.png
"File:ScreenshotOOMP4_reviewtasks.png")

### Task execution

Now you have to wait for the git fetch operation.

![<File:ScreenshotOOMP5_waitforsetup.png>](ScreenshotOOMP5_waitforsetup.png
"File:ScreenshotOOMP5_waitforsetup.png")

![<File:ScreenshotOOMP6_finish.png>](ScreenshotOOMP6_finish.png
"File:ScreenshotOOMP6_finish.png")

### First build

After everything is there it builds all projects.

![<File:ScreenshotOOMP7_build.png>](ScreenshotOOMP7_build.png
"File:ScreenshotOOMP7_build.png")

### Switch git branch

To switch to the 4.16 (2020-06) branch you have to open the view for the
git repositories.

![<File:ScreenshotOOMP8_opengit.png>](ScreenshotOOMP8_opengit.png
"File:ScreenshotOOMP8_opengit.png")

Double click on the "Remote Tracking" branch for
"origin/R4_16_maintenance".

![<File:ScreenshotOOMP9_remoteTracking.png>](ScreenshotOOMP9_remoteTracking.png
"File:ScreenshotOOMP9_remoteTracking.png")

Confirm the dialog to checkout. Repeat that for all git repositories you
have.

![<File:ScreenshotOOMP10_createBranch.png>](ScreenshotOOMP10_createBranch.png
"File:ScreenshotOOMP10_createBranch.png")

### Switch baseline

Now go to Navigate-\>Open Setup-\>Workspace.

![<File:ScreenshotOOMP11_navigatesetupworkspace.png>](ScreenshotOOMP11_navigatesetupworkspace.png
"File:ScreenshotOOMP11_navigatesetupworkspace.png")

Copy and Paste the following snippet to the "workspace" node.

``` xml numberLines
<?xml version="1.0" encoding="UTF-8"?>
<xmi:XMI xmi:version="2.0"
    xmlns:xmi="http://www.omg.org/XMI"
    xmlns:setup="http://www.eclipse.org/oomph/setup/1.0">
  <setup:VariableTask
      name="eclipse.target.platform"
      value="2020-06"
      storageURI="scope://Workspace"/>
  <setup:VariableTask
      name="eclipse.api.baseline.target.platform"
      value="2020-03"
      storageURI="scope://Workspace"/>
</xmi:XMI>
```

![<File:ScreenshotOOMP12_pasteonworkspace.png>](ScreenshotOOMP12_pasteonworkspace.png
"File:ScreenshotOOMP12_pasteonworkspace.png")

### Task execution to fix baseline

Save. Then Help-\>Perform Setup Tasks

![<File:ScreenshotOOMP13_runsetuptasks.png>](ScreenshotOOMP13_runsetuptasks.png
"File:ScreenshotOOMP13_runsetuptasks.png")

Hit "Finish" button.

![<File:ScreenshotOOMP14_finish.png>](ScreenshotOOMP14_finish.png
"File:ScreenshotOOMP14_finish.png")

Now you have a checkout of 4.16 (2020-06) where you can work on.

## Further reading

  - [Oomph](Oomph "wikilink")
  - [Eclipse Oomph FAQ](Eclipse_Oomph_FAQ "wikilink")
  - [Eclipse Platform SDK
    Provisioning](Eclipse_Platform_SDK_Provisioning "wikilink") -- *if
    you want to provision more than just JDT*

[Category:JDT](Category:JDT "wikilink")