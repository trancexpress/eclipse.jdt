This page explains how you can create a feature patch. E.g. to add that
fix for the most annoying bug ever to the otherwise perfect JDT UI
plug-in.

I'll just show how I've produced the output for , a feature patch on top
of 4.4.1 that patches the org.eclipse.jdt.ui plug-in in the
org.eclipse.jdt feature. You can adapt the versions/bundles to your
concrete use case. Note that patching multiple features is a bit more
complicated, since you need to create multiple patch features that
depend on each other and mirror the dependencies between the original
features.

## Setup

  - start up with the Eclipse version you want to create a patch for
    (4.4.1 / M20140925-0400)
  - [clone](EGit/User_Guide#Cloning_a_Repository "wikilink")
    <git://git.eclipse.org/gitroot/jdt/eclipse.jdt.ui.git>
  - import the projects you need to touch (org.eclipse.jdt.ui)
  - switch to the right branch (R4_4_maintenance) and make sure the
    patch is applied to your workspace

## Shortcut if you just want to patch your existing install

(Skip this section if you want to create a feature patch that others can
use as well\!)

  - select the project and choose **Export \> Deployable plug-ins and
    fragments**
  - select the plug-in to export (org.eclipse.jdt.ui)
  - select **Destination \> Install into host**
  - click **Finish**
  - confirm messages, restart, and enjoy

## Create patch feature

  - create **New \> Other... \> Feature Patch**
  - give the project a unique name
    (org.eclipse.jdt.ui.bug434941.sortmembers)
  - click **Browse** and select the feature that contains the plug-in
    (org.eclipse.jdt)
  - click **Finish**

<!-- end list -->

  - In the feature.xml editor, go to the **Plug-ins** tab and add the
    plug-in to patch (org.eclipse.jdt.ui)
  - fill out the rest of the forms.

<table>
<thead>
<tr class="header">
<th><p>feature.xml for this example</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><div class="sourceCode" id="cb1"><pre class="sourceCode xml"><code class="sourceCode xml"><span id="cb1-1"><a href="#cb1-1" aria-hidden="true"></a><span class="kw">&lt;?xml</span> version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;<span class="kw">?&gt;</span></span>
<span id="cb1-2"><a href="#cb1-2" aria-hidden="true"></a><span class="kw">&lt;feature</span></span>
<span id="cb1-3"><a href="#cb1-3" aria-hidden="true"></a><span class="ot">      id=</span><span class="st">&quot;org.eclipse.jdt.ui.bug434941.sortmembers&quot;</span></span>
<span id="cb1-4"><a href="#cb1-4" aria-hidden="true"></a><span class="ot">      label=</span><span class="st">&quot;Sort Members patch for bug 434941&quot;</span></span>
<span id="cb1-5"><a href="#cb1-5" aria-hidden="true"></a><span class="ot">      version=</span><span class="st">&quot;1.0.0&quot;</span></span>
<span id="cb1-6"><a href="#cb1-6" aria-hidden="true"></a><span class="ot">      provider-name=</span><span class="st">&quot;Eclipse.org&quot;</span><span class="kw">&gt;</span></span>
<span id="cb1-7"><a href="#cb1-7" aria-hidden="true"></a></span>
<span id="cb1-8"><a href="#cb1-8" aria-hidden="true"></a>   <span class="kw">&lt;description&gt;</span></span>
<span id="cb1-9"><a href="#cb1-9" aria-hidden="true"></a>      Sort Members patch for bug 434941</span>
<span id="cb1-10"><a href="#cb1-10" aria-hidden="true"></a>   <span class="kw">&lt;/description&gt;</span></span>
<span id="cb1-11"><a href="#cb1-11" aria-hidden="true"></a></span>
<span id="cb1-12"><a href="#cb1-12" aria-hidden="true"></a>   <span class="kw">&lt;copyright&gt;</span></span>
<span id="cb1-13"><a href="#cb1-13" aria-hidden="true"></a>      Copyright (c) 2014 IBM Corporation and others</span>
<span id="cb1-14"><a href="#cb1-14" aria-hidden="true"></a>   <span class="kw">&lt;/copyright&gt;</span></span>
<span id="cb1-15"><a href="#cb1-15" aria-hidden="true"></a></span>
<span id="cb1-16"><a href="#cb1-16" aria-hidden="true"></a>   <span class="kw">&lt;license</span><span class="ot"> url=</span><span class="st">&quot;http://www.eclipse.org/legal/epl-v10.html&quot;</span><span class="kw">&gt;</span></span>
<span id="cb1-17"><a href="#cb1-17" aria-hidden="true"></a>      Eclipse Public License v1.0</span>
<span id="cb1-18"><a href="#cb1-18" aria-hidden="true"></a></span>
<span id="cb1-19"><a href="#cb1-19" aria-hidden="true"></a>http://www.eclipse.org/legal/epl-v10.html</span>
<span id="cb1-20"><a href="#cb1-20" aria-hidden="true"></a>   <span class="kw">&lt;/license&gt;</span></span>
<span id="cb1-21"><a href="#cb1-21" aria-hidden="true"></a></span>
<span id="cb1-22"><a href="#cb1-22" aria-hidden="true"></a>   <span class="kw">&lt;requires&gt;</span></span>
<span id="cb1-23"><a href="#cb1-23" aria-hidden="true"></a>      <span class="kw">&lt;import</span><span class="ot"> feature=</span><span class="st">&quot;org.eclipse.jdt&quot;</span><span class="ot"> version=</span><span class="st">&quot;3.10.0.v20140910-2310&quot;</span><span class="ot"> patch=</span><span class="st">&quot;true&quot;</span><span class="kw">/&gt;</span></span>
<span id="cb1-24"><a href="#cb1-24" aria-hidden="true"></a>   <span class="kw">&lt;/requires&gt;</span></span>
<span id="cb1-25"><a href="#cb1-25" aria-hidden="true"></a></span>
<span id="cb1-26"><a href="#cb1-26" aria-hidden="true"></a>   <span class="kw">&lt;plugin</span></span>
<span id="cb1-27"><a href="#cb1-27" aria-hidden="true"></a><span class="ot">         id=</span><span class="st">&quot;org.eclipse.jdt.ui&quot;</span></span>
<span id="cb1-28"><a href="#cb1-28" aria-hidden="true"></a><span class="ot">         download-size=</span><span class="st">&quot;0&quot;</span></span>
<span id="cb1-29"><a href="#cb1-29" aria-hidden="true"></a><span class="ot">         install-size=</span><span class="st">&quot;0&quot;</span></span>
<span id="cb1-30"><a href="#cb1-30" aria-hidden="true"></a><span class="ot">         version=</span><span class="st">&quot;0.0.0&quot;</span></span>
<span id="cb1-31"><a href="#cb1-31" aria-hidden="true"></a><span class="ot">         unpack=</span><span class="st">&quot;false&quot;</span><span class="kw">/&gt;</span></span>
<span id="cb1-32"><a href="#cb1-32" aria-hidden="true"></a></span>
<span id="cb1-33"><a href="#cb1-33" aria-hidden="true"></a><span class="kw">&lt;/feature&gt;</span></span></code></pre></div></td>
</tr>
</tbody>
</table>

  - due to p2 , you also have to create a New \> Category Definition
      - save the category.xml file in your patch feature project
      - add a dummy category and add your new patch feature

<table>
<thead>
<tr class="header">
<th><p>resulting category.xml</p></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><div class="sourceCode" id="cb1"><pre class="sourceCode xml"><code class="sourceCode xml"><span id="cb1-1"><a href="#cb1-1" aria-hidden="true"></a><span class="kw">&lt;source</span><span class="ot"> lang=</span><span class="st">&quot;xml&quot;</span><span class="kw">&gt;</span></span>
<span id="cb1-2"><a href="#cb1-2" aria-hidden="true"></a><span class="kw">&lt;?xml</span> version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;<span class="kw">?&gt;</span></span>
<span id="cb1-3"><a href="#cb1-3" aria-hidden="true"></a><span class="kw">&lt;site&gt;</span></span>
<span id="cb1-4"><a href="#cb1-4" aria-hidden="true"></a>   <span class="kw">&lt;feature</span><span class="ot"> url=</span><span class="st">&quot;features/org.eclipse.jdt.ui.bug434941.sortmembers_1.0.0.jar&quot;</span><span class="ot"> id=</span><span class="st">&quot;org.eclipse.jdt.ui.bug434941.sortmembers&quot;</span><span class="ot"> version=</span><span class="st">&quot;1.0.0&quot;</span><span class="ot"> patch=</span><span class="st">&quot;true&quot;</span><span class="kw">&gt;</span></span>
<span id="cb1-5"><a href="#cb1-5" aria-hidden="true"></a>      <span class="kw">&lt;category</span><span class="ot"> name=</span><span class="st">&quot;org.eclipse.jdt.ui.sortmembers.bug434941&quot;</span><span class="kw">/&gt;</span></span>
<span id="cb1-6"><a href="#cb1-6" aria-hidden="true"></a>   <span class="kw">&lt;/feature&gt;</span></span>
<span id="cb1-7"><a href="#cb1-7" aria-hidden="true"></a>   <span class="kw">&lt;category-def</span><span class="ot"> name=</span><span class="st">&quot;org.eclipse.jdt.ui.sortmembers.bug434941&quot;</span><span class="ot"> label=</span><span class="st">&quot;Dummy category for p2 bug 262009&quot;</span><span class="kw">&gt;</span></span>
<span id="cb1-8"><a href="#cb1-8" aria-hidden="true"></a>      <span class="kw">&lt;description&gt;</span></span>
<span id="cb1-9"><a href="#cb1-9" aria-hidden="true"></a>         Dummy; please vote for https://bugs.eclipse.org/bugs/show_bug.cgi?id=262009 .</span>
<span id="cb1-10"><a href="#cb1-10" aria-hidden="true"></a>      <span class="kw">&lt;/description&gt;</span></span>
<span id="cb1-11"><a href="#cb1-11" aria-hidden="true"></a>   <span class="kw">&lt;/category-def&gt;</span></span>
<span id="cb1-12"><a href="#cb1-12" aria-hidden="true"></a><span class="kw">&lt;/site&gt;</span></span></code></pre></div></td>
</tr>
</tbody>
</table>

## Build and export feature patch

  - open the feature.xml \> Overview, and click **Export Wizard**, or
    open the "Deployable features" wizard under File \> Export...
  - make sure your feature is checked and specify an (empty) output
    directory
  - switch to **Options** tab
  - click the **Browse** button after **Categorize repository** and
    select your category.xml
  - click **Finish** and go get a coffee

## Publish

  - when the export is done, publish the output directory on a website
    or put it into a ZIP for manual installs

[Category:JDT](Category:JDT "wikilink")