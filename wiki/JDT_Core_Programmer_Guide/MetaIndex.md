## Introduction

The new MetaIndex was introduced to reduce the time it takes to build
the TypeHierarchy of java types.

## Problem

Today the when building the TypeHierarchy then
IndexBasedHierarchyBuilder perform a recursive search to find subtypes
which the number of iterations are equal to the number of sub types
found.

So in a workspace where you have 5000 indexes this could lead to a very
long response time if you sub type tree is also large. For example i
have a workspace which has a TypeHierarchy as follows

    Interface1
     |-- Interface2
          |-- Class2
     |-- Interface3
          |-- Class3
     |-- Interface4
          |-- Class4
     |-- Interface5
          |-- Class5
     |-- Interface6
          |-- Class6

All these classes and interfaces are in the same project. And this
workspace has nearly 1000 indexes, try to build this TypeHierarchy takes
around 15 sec. Looking closer at the profiling snapshots shown that we
are performing more than 11 pattern searches on all 1000 indexes. The
number of iterations depends on how much super type entries you find in
indexes since the pattern search doesn't take into account the
inheritance hierarchy.

For example if you search for type hierarchy of `a.b.c.Value`, the first
iteration will find all types which has the simple name `Value` which
will result in all subtypes which has some super type called Value. In
the same way the types found keeps growing in each iteration based on
the types you have in your workspace.

## Solution

The solution we look at is to reduce the number of indexes that are
search in this 11+ iterations. in this particular example we should be
only searching in the index document for this project. But how does the
IndexBasedHierarchyBuilder select the correct indexes.

To address this we looked at where we should filter the indexes and we
choose to do that at IndexSelector.getIndexLocations. Then we end up
thinking what should be our criteria for this index selection and how we
could find these indexes. Because we cannot search all indexes to find
its eligibility.

To support this we decided to introduce a new MetaIndex which index all
index document names against a qualification which can be used for
filter indexes.

When performing a type hierarchy build we will set the *qualifier* of
the type in each iteration into `SearchPattern` and use that value to
perform a search in MetaIndex to find index names that needs to be
included in current iteration at IndexSelector.getIndexLocations and
filter out the index locations.

To support backward compatibility, when performing this MetaIndex search
at IndexManager.findMatchingIndexNames() we check if there are any
indexes which are not part of the MetaIndex and include them by default.
This handles

  - When the MetaIndex is not created in the workspace
  - When there are indexes which are not included in the MetaIndex
  - When there are prebuilt indexes

Lets look at different qualifications we used and their impact on
indexing and accuracy of the search and also how this new meta index can
be used for other search types in future.

### Package name based solution

The first qualification type we looked at we decided to use java package
names as the qualification value. So we ended up updating the MetaIndex
as follows

| Category       | Key              | ContainerPath  |
| -------------- | ---------------- | -------------- |
| TYPE_INDEX_Q | java.lang.util   | 1234634.index  |
| TYPE_INDEX_Q | java.lang.util   | 12348756.index |
| TYPE_INDEX_Q | com.google.guava | 1234634.index  |
|                |                  |                |

We have modified all Indexer implementation to update package usages of
each class and compilation unit each indexer scan into the index it self
with a new key **TYPE_INDEX_Q**, this keep track of all packages used
as

  - Imports
  - Fully qualified type references
  - Current package name of the class or compilation unit

The capturing of package names are done in BinaryIndexes by looking at
fully qualified reference names and extracting the package name from
them. Basically the reference is read until the last '.' and use the
value as package name. This is not a problem in class files the nested
types are seperated by '$'. Following are the place where the package
usages are extracted from

  - Class Declarations (superclass, superinterfaces)
  - Enum Declarations (superinterfaces)
  - Interface Declarations (superinterfaces)
  - Annotation Declarations (superclass)

The capturing of package names are done in SourceIndexer little bit
differently. The steps are as follows

  - Extract package names from all none \* imports
  - Extract package names from
      - Class Declarations (superclass, superinterfaces)
      - Enum Declarations (superinterfaces)
      - Interface Declarations (superinterfaces)
      - Annotation Declarations (superclass)

if they are qualified names, otherwise we check if the class name was
available in the imports we processed, if not we collect them to be
resolved later. Once we have scanned the whole compilation unit, we will
try to do a diet resolving of the classes we didn't found neither as
fully qualified nor in imports.

We do this diet resolving by try to match these classes in current
project's package fragment roots. This is implemented at
SourceIndexerRequestor.performDietResolution(), this will handle default
package types, types from java.lang package as well. After performing
this if we still find type which are not resolved, then that mean we
have some types which might be part of inheriting hierarchy like

    ProjectA
    in package p1:
       class Couter {
            protected class Inner { ... }
       }
    in package p2:
       class Middle extends p1.Couter {}

    ProjectB
    in package p3:
       class C extends p2.Middle {
            class I extends Inner {}
       }

So to resolve this Inner class usage we need to resolve the type
bindings of the compilation unit, therefore we mark the source document
as it needs to be IndexingResolvedDocument. Before introducing this the
only criteria for IndexingResolvedDocument was when there are lambda
expressions.

When doing this we found that it has an overhead on the Indexing because
resolving type bindings takes the considerable amount of time. So when
looking closer at this we found that most of the type binding resolution
time is spend at
`IndexBasedJavaSearchEnvironment.create(List`<IJavaProject>`,
ICompilationUnit[])`, and also we so that this environment is create for
each source file which needs to be resolved even though the source files
might be from the same project. So to reduce creating this
INameEnvironment repeated we introduce a LRU cache, which is cleared at
the end of indexing and also project entries are clear as soon as the
project change is detected. At a glance this could be seen as a increase
in memory, but since the LRU cache use SoftReferences the heavy
INameEnvironment objects will be garbage collected when they are not
accessed. Also this reduce the overhead for the GC of cleaning up large
INameEnvironment objects which was created repeated when every you need
resolution of the compilation unit.

#### Results

Indexing times on large workspace

    TODO: Add indexing time

Search times

    == Master search ================================
    search iteration time(ms) pcts (50,80,100): [  2.   2. 124.]
    search iteration cumulative time(ms) pcts (50,80,100): [  6.   8. 794.]
    iteration size: 2649
    total time: 5152ms
    =================================================
    == Package search =====================
    total meta index search time: 30ms
    meta index search time(ms) pcts (50,80,100): [0. 0. 1.]
    search iteration index selection pcts (50,80,100): [ 17.  24. 474.]
    search iteration total indexes pcts (50,80,100): [578. 578. 578.]
    search iteration time(ms) pcts (50,80,100): [  1.    1.2 148. ]
    search iteration cumulative time(ms) pcts (50,80,100): [  0.   1. 835.]
    iteration size: 600
    total time: 966ms
    =================================================

But looking at closer we found that for certain scenarios we miss some
subtypes. So we looked for more solutions which will not require us to
change a lot of Indexers like we did in this solution.

### Simple name based solution

In this solution we use the type simple name as the index qualifier when
creating the MetaIndex. We only had to change the `AbstractIndexer` to
extract the required information into MetaIndex. The MetaIndex looks as
follows

| Category       | Key           | ContainerPath  |
| -------------- | ------------- | -------------- |
| TYPE_INDEX_Q | List          | 1234634.index  |
| TYPE_INDEX_Q | List          | 12348756.index |
| TYPE_INDEX_Q | ImmutableList | 1234634.index  |
|                |               |                |

#### Results

Indexing times on large workspace

    TODO: Add indexing time

Search times

    == Master search ================================
    search iteration time(ms) pcts (50,80,100): [  2.   2. 124.]
    search iteration cumulative time(ms) pcts (50,80,100): [  6.   8. 794.]
    iteration size: 2649
    total time: 5152ms
    =================================================
    == SimpleName search ============================
    total meta index search time: 91ms
    meta index search time(ms) pcts (50,80,100): [0. 0. 1.]
    search iteration index selection pcts (50,80,100): [ 17.  18. 462.]
    search iteration total indexes pcts (50,80,100): [578. 578. 578.]
    search iteration time(ms) pcts (50,80,100): [  0.   1. 126.]
    search iteration cumulative time(ms) pcts (50,80,100): [  0.   1. 491.]
    iteration size: 2649
    total time: 2064ms
    =================================================

One problem we found in this solution is when the type you search for
has a very common name, When you have a very common name such as Value
and if you have the same class name in your dependent libraries under
different packages names, you will have many iterations of
SuperTypeReference search collecting lot of types which are not from the
same inheritance hierarchy as you are searching for.

So we looked at how workspace compositions, like number of projects,
number library references etc. In majority of the time you have more
library references than number of projects you have in the workspace.
Which means you have more binary indexes and source indexes when you are
searching.

We use this information to come up with the more optimized solution.

### Simple + Qualified name based solution

When indexing binary jar files, we have the qualified names of types. So
we decided to use that information and update the MetaIndex with
qualified names for binary indexes, but for projects, we use the
SimpleName of types since trying to resolve will take up more indexing
time.

Also to further improve super type search, we use two categories in
MetaIndex.

  - metaIndexTQ : is used for normal type references such as method
    return types, type declarations etc.
  - metaIndexSTQ : is used for super type references such as super class
    or super interface.

Each of above categories as a SimpleName and QualifiedName
representation. So the actual categories used looks like below

  - metaIndexSTQ : SimpleName Type Reference Qualifier
  - metaIndexQTQ : QualifiedName Type Reference Qualifier
  - metaIndexSSTQ : SimpleName Super Type Reference Qualifier
  - metaIndexQSTQ : QualifiedName Super Type Reference Qualifier

So with this change this how the MetaIndex looks like.

| Category      | Key                                     | ContainerPath  |
| ------------- | --------------------------------------- | -------------- |
| metaIndexSSTQ | List                                    | 1234634.index  |
| metaIndexQSTQ | java.util.List                          | 12348756.index |
| metaIndexSTQ  | ImmutableList                           | 1234634.index  |
| metaIndexQTQ  | com.google.common.collect.ImmutableList | 1234634.index  |
|               |                                         |                |

#### Results

Indexing times on large workspace

    TODO: Add indexing time

Search times

    == Master search ================================
    search iteration time(ms) pcts (50,80,100): [  2.   2. 124.]
    search iteration cumulative time(ms) pcts (50,80,100): [  6.   8. 794.]
    iteration size: 2649
    total time: 5152ms
    =================================================
    == Simple+Qualified Name search ================================
    total meta index search time: 56ms
    meta index search time(ms) pcts (50,80,100): [0. 0. 1.]
    search iteration index selection pcts (50,80,100): [ 17.  18. 182.]
    search iteration total indexes pcts (50,80,100): [578. 578. 578.]
    search iteration time(ms) pcts (50,80,100): [  1.   1. 131.]
    search iteration cumulative time(ms) pcts (50,80,100): [  0.   1. 767.]
    iteration size: 760
    total time: 1025ms
    =================================================

[Category:JDT](Category:JDT "wikilink")