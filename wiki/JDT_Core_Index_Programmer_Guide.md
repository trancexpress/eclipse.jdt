JDT core contains two indices.

The legacy index mainly contains mappings of symbol names onto jar
files. The new index contains a complete model of the code, in
sufficient detail that the java model can be created without accessing
the original class files.

The new index is intended to completely replace the legacy index, but at
the time of this writing (4.7) the new index can only index .class and
.jar files, not .java files. Also, many APIs are still implemented
directly based on the legacy index, so both indices currently exist in
parallel.

## General Info

Stored in a single file,
workspace/.metadata/org.eclipse.jdt.core/index.db

Design doc:
<https://docs.google.com/document/d/1w3-ufZyISbqH8jxYv689Exjm0haAGufdcSvEAgl2HQ4/edit>

Bugs are tagged with the prefix \[newindex\] in the subject.

## Where is the code

org.eclipse.jdt.internal.core.nd.db

Contains the implementation of Database, the virtual memory system.
(Non-java-specific)

org.eclipse.jdt.internal.core.nd.field

Contains the implementation of the Field classes used to create the
database structures. (Non-java-specific)

org.eclipse.jdt.internal.core.nd

Contains the implementation of Nd, the high-level interface for the
database. (Non-java-specific)

org.eclipse.jdt.internal.core.nd.java

Contains the schema for the java language. The main entry point is
JavaIndex. Java-specific.

org.eclipse.jdt.internal.core.nd.indexer

Contains the implementation of the indexer. Java-specific.

## How do I benchmark the indexer?

Create a large workspace.

Enable the tracing option org.eclipse.jdt.core/debug/index/timing=true

Delete the index.db file. Then restart Eclipse.

Let the indexer run.

Eclipse will print out statistics about indexing time to the console
that look like this:

`   Indexing done at 2017-04-27 15:57:35.726`
`     Located 187 indexables in 340ms`
`     Tested 187 fingerprints in 4885ms, average time = 26.123ms`
`     Indexed 205598 classes (from 187 files containing 1018.934MiB) in 101104ms, average time per class = 0.492ms`
`     Chunks: total = 212942, in memory = 32768, dirty = 0, not in cache = 0`
`     Cache misses = 348755 (0.041%)`
`     Reads = 1362.324MiB, writes = 1399.242MiB`
`     Read speed = 1322.645MiB/s`
`     Write speed = 963.547MiB/s`
`     Time spent performing flushes = 15251ms (14.321%)`
`     Total indexing time = 106495ms`

When benchmarking, it is best to intentionally fragment and re-index the
workspace 2-3 times in a row, to get an idea of how the performance
changes after multiple reindexing passes. See the section on fragmenting
the index for details.

## How do I find the cause of index corruption?

If you're seeing IndexExceptions being thrown in the log, this is a sign
the index is getting corrupted.

Quite often, if corruption is caused by a bug in the index itself you'll
be able to reproduce it consistently by repeatedly fragmenting and
reindexing a large workspace with an initially-deleted index.

Enable the tracing option
org.eclipse.jdt.core/debug/index/logsizemegs=1024

The numeric argument is a size in megabytes. This logs all writes to the
database in a big circular buffer.

Then reproduce your corruption. When the IndexException is finally
thrown, it will have a traceback attached that looks like this:

`   org.eclipse.jdt.internal.core.nd.db.IndexException: Null data block found in metablock`
`   Related addresses:`
`   backpointer number 4 [address 116281380, size 4]: `
`       wrote [address 116281362, size 4070] at time 574379275`
`           Writing field 0, a FieldString in struct NdConstantString`
`       malloc'd [address 116281354, size 4094] at time 574379271`
`           Writing field 0, a FieldString in struct NdConstantString`
`       wrote [address 116281352, size 4080] at time 574379270`
`           Writing field 0, a FieldString in struct NdConstantString`
`       malloc'd [address 116281354, size 4094] at time 574194812`
`           Writing field 3, a FieldManyToOne in struct NdVariable`
`       wrote [address 116281352, size 4080] at time 574194811`
`           Writing field 3, a FieldManyToOne in struct NdVariable`
`       malloc'd [address 116281354, size 4094] at time 573874828`
`           Writing field 0, a FieldManyToOne in struct NdMethodParameter`
`       wrote [address 116281352, size 4080] at time 573874827`
`           Writing field 0, a FieldManyToOne in struct NdMethodParameter`
`       malloc'd [address 116281354, size 4094] at time 572846736`
`           Writing field 0, a FieldManyToOne in struct NdMethodParameter`
`       wrote [address 116281352, size 4080] at time 572846735`
`           Writing field 0, a FieldManyToOne in struct NdMethodParameter`
`       malloc'd [address 116281354, size 4094] at time 572264886`
`           Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`
`   `
`   field 0, a FieldPointer in struct RawGrowableArray [address 5844788, size 4]: `
`       wrote [address 5844788, size 4] at time 572264893`
`           Writing field 0, a FieldPointer in struct RawGrowableArray`
`           Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`
`   `
`   field 0, a FieldInt in struct GrowableBlockHeader [address 116281354, size 4]: `
`       wrote [address 116281354, size 4] at time 574379274`
`           Writing field 0, a FieldString in struct NdConstantString`
`       malloc'd [address 116281354, size 4094] at time 574379271`
`           Writing field 0, a FieldString in struct NdConstantString`
`       wrote [address 116281352, size 4080] at time 574379270`
`           Writing field 0, a FieldString in struct NdConstantString`
`       wrote [address 116281354, size 4] at time 574362897`
`           Writing field 0, a FieldInt in struct GrowableBlockHeader`
`           Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`
`       wrote [address 116281354, size 4] at time 574360856`
`           Writing field 0, a FieldInt in struct GrowableBlockHeader`
`           Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`
`       wrote [address 116281354, size 4] at time 574353349`
`           Writing field 0, a FieldInt in struct GrowableBlockHeader`
`           Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`
`       malloc'd [address 116281354, size 4094] at time 574194812`
`           Writing field 3, a FieldManyToOne in struct NdVariable`
`       malloc'd [address 116281354, size 4094] at time 573874828`
`           Writing field 0, a FieldManyToOne in struct NdMethodParameter`
`       malloc'd [address 116281354, size 4094] at time 572846736`
`           Writing field 0, a FieldManyToOne in struct NdMethodParameter`
`       malloc'd [address 116281354, size 4094] at time 572264886`
`           Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`

This shows all the memory addresses involved in the corruption and the
history of all write, malloc, and free calls that covered each address.
The lines that look like this are a poor-man's stack trace that describe
the point in the call where the particular write occurred:

`       Writing field 0, a FieldInt in struct GrowableBlockHeader`
`       Writing field 1, a FieldManyToOne in struct NdComplexTypeSignature`

## How do I detect corruption earlier?

The following tracing option causes the indexer to perform periodic
self-tests:

org.eclipse.jdt.core/debug/index/freespacetest=true

This slows down indexing quite a bit. If you have a reproducible
test-case that produces corruption at a certain time, you can manually
edit the Database.periodicValidateFreeSpace method to control when the
tests are performed.

You can call getLog().getWriteCount() to get a time integer that matches
the "at time" messages in the IndexException.

## How do I test the index in a fragmented state?

When the indexer runs on an initially-empty database, everything in the
index ends up being allocated in consecutive memory locations. This
produce artificially-optimistic performance numbers. You can force the
database to re-index everything in a fairly realistic way if you set all
the file fingerprints to something invalid and then retrigger the
indexer.

This can be done by modifying the implementation of Indexer.rebuildIndex
like this:

`   public void rebuildIndex(IProgressMonitor monitor) throws CoreException {`
`       final int iterations = 5;`
`       SubMonitor loopMonitor = SubMonitor.convert(monitor, iterations);`
`       for (int rebuildCounter = 0; rebuildCounter < iterations; rebuildCounter++) { `
`           SubMonitor iterationMonitor = loopMonitor.split(1).setWorkRemaining(100);`
`           JavaIndex index = JavaIndex.getIndex(this.nd);`
`           this.nd.acquireWriteLock(iterationMonitor.split(1));`
`           try {`
`               List`<NdResourceFile>` files = index.getAllResourceFiles();`
`               for (NdResourceFile next : files) {`
`                   FileFingerprint modifiedFingerprint = new FileFingerprint(0xbaadd00d, 0xbabef00d, 0xabbabaad);`
`                   next.setFingerprint(modifiedFingerprint);`
`               }`
`           } finally {`
`               this.nd.releaseWriteLock();`
`           }`
`           rescan(iterationMonitor.split(98));`
`       }`
`   }`

Then run the "rebuild java index" command in the UI. Doing so will
rebuild the index 5 times in a row using the invalid fingerprint trick.
Of course, you shouldn't commit this since it breaks the usual usage of
the rebuild java index command.

This is also a good way to test the index for corruption in a more
real-world use-case.

## What if the content in the index doesn't match the input files?

Enable this tracing option:

org.eclipse.jdt.core/debug/index/selftest=true

This will cause the indexer to read back every class immediately after
inserting it into the index and compare it with the original .class
file. If anything is different, it will throw an exception. Put a
breakpoint on the line that throws the exception, then use Drop To Frame
to re-run the addClassToIndex method. That will let you step through
what the indexer was doing while indexing the problematic class.

## How do I reduce the size of the index?

Enable this tracing option:

org.eclipse.jdt.core/debug/index/space=false

After each reindexing pass, this will cause the indexer to display a
histogram of where the space in the index is being allocated. You can
use this to identify hotspots and opportunities for savings. In the
code, the histogram bin for any given call is controlled by the second
argument to malloc or free. You can control how the statistics are
collected by rearranging the bins.

Typical output looks like this:

`   Allocated size: 793.441MiB`
`   malloc'ed: 812.253MiB`
`   free'd: 21.51MiB`
`   wasted: 2.698MiB`
`   Free blocks`
`   NdType 563819 allocations, 291.566MiB`
`   Short Strings 5824608 allocations, 271.815MiB`
`   NdMethod 1646995 allocations, 75.274MiB`
`   Growable Arrays 1511602 allocations, 55.234MiB`
`   NdComplexTypeSignature 684498 allocations, 52.239MiB`
`   NdTypeArgument 648059 allocations, 15.03MiB`
`   NdTypeId 131926 allocations, 10.068MiB`
`   NdConstantInt 137160 allocations, 4.19MiB`
`   B-Trees 30196 allocations, 3.686MiB`
`   NdTypeInterface 84485 allocations, 1.958MiB`
`   NdResourceFile 787 allocations, 1.754MiB`
`   NdConstantString 55505 allocations, 1.695MiB`
`   NdMethodAnnotationData 52955 allocations, 1.635MiB`
`   Linked Lists 58047 allocations, 1.55MiB`
`   NdConstantLong 29612 allocations, 0.905MiB`
`   NdBinding 24013 allocations, 0.872MiB`
`   Long Strings 174 allocations, 0.48MiB`
`   NdVariable 8520 allocations, 0.266MiB`
`   NdConstantBoolean 4095 allocations, 0.125MiB`
`   NdConstantClass 2382 allocations, 0.073MiB`
`   NdConstantShort 2318 allocations, 0.071MiB`
`   NdConstantByte 2105 allocations, 0.064MiB`
`   NdConstantArray 1246 allocations, 0.048MiB`
`   NdConstantEnum 1207 allocations, 0.046MiB`
`   NdConstantAnnotation 964 allocations, 0.044MiB`
`   NdConstantChar 1079 allocations, 0.033MiB`
`   NdConstantDouble 483 allocations, 0.015MiB`
`   NdConstantFloat 150 allocations, 0.005MiB`
`   Miscellaneous 15 allocations, 248B`
`   NdWorkspaceLocation 4 allocations, 64B`

## How do I debug race conditions?

Race conditions are often caused by what's in the index at any given
time and what's in the JDT model cache at any given time. For this
reason, one good approach is to add print statements all over the unit
test, log whenever anything is added or removed from the model cache,
and log whenever anything is added or removed from the index. Then you
can determine whether or not a particular entity was present in the
index or the cache at any given moment.

Enable this tracing option to list each class that is added/removed from
the index:

org.eclipse.jdt.core/debug/index/insertions=true

...and use this tracing option to list each class that is added/removed
from the model cache:

org.eclipse.jdt.core/debug/javamodel/insertions=true

It is sometimes useful to log the time at which the indexer was
scheduled, which can be done with this trace option:

org.eclipse.jdt.core/debug/index/scheduling=true

## What tracing options should I use when debugging?

The following tracing options produce useful high-level information
about what the index is doing without too much spam:

`   org.eclipse.jdt.core/debug/index/indexer=true`
`   org.eclipse.jdt.core/debug/index/space=true`
`   org.eclipse.jdt.core/debug/index/timing=true`

## How do I debug deadlocks?

If the deadlock appears to be caused by the read/write lock on the
Database, you can generate some useful diagnostics using this tracing
option:

org.eclipse.jdt.core/debug/index/locks=true

If the deadlock appears to be caused by waiting on the indexer job, use
the tracing options that track scheduling and running of the indexer.

## Where can I find examples of using the index?

BinaryTypeFactory.readFromIndex demonstrates how to create a model
object whose implementation is backed by the index. Also look at the
implementation of IndexBinaryType for examples of doing various types of
reads.

IndexBasedHierarchyBuilder.newSearchAllPossibleSubTypes demonstrates how
to perform a breadth-first search on the type hierarchy rooted at a
given class.

## Error Handling

If the index is corrupted, it should throw an IndexException as soon as
the corruption is detected. The correct response to an IndexException is
to delete the entire index and rebuild it. IndexException should \*NOT\*
be used for other kinds of errors since throwing it will result in the
index being deleted. It also initiates self-diagnostics that are only
appropriate for debugging index corruption.

Code that throws IndexException can attach address ranges to the
exception. Such code should attach the most specific addresses that were
read from when determining that the index was corrupt. For example, if
two ints were expected to be equal, the code should attach the addresses
from which each int was read. Don't attach the memory range for the
entire struct that contains the int - just include the memory range for
the int itself.

Programming errors should be detected using something like assertions,
IllegalStateException, etc.

I/O errors or corrupted input files should be detected by some kind of
checked exception (like CoreException).

## Known Problems

### Rework the model cache (Bug 497513)

The JDT model cache is not well optimized for the index. The index is
optimal when you do lots of small reads as lazily as possible. The JDT
model cache was designed to do a small number of big reads as eagerly as
possible (it assumes that opening a class file is slow and extracting
each field is fast). This means that it wastes a lot of the time saved
by the index in order to unnecessarily cache stuff that could be fetched
lazily from the index. In our experiments, we found that changes to the
model cache could more than double the speed of the type hierarchy view.
We expect similar speedups everywhere else in the UI as well.

### Fix fragmentation (Bug 512921)

When the indexer first runs, it puts all of its structures in
consecutive memory locations. That means that references between
structures within the same class are usually within the same database
page. After a number of reindexing passes, the number of
cross-references between pages increases and performance of the database
degrades. We could fix this by using arena allocators. That would ensure
that memory for any given class would always be within the same page.

Once fixed, the index should always stay as fast as it is on the first
indexing pass, no matter how many times it has to perform reindexing of
the same artifacts.

### Deadlock risk in Indexer.waitForIndex (Bug 509558)

The indexer sometimes obtains the workspace lock. That means if you call
Indexer.waitForIndex from a thread that holds the workspace lock, it is
likely deadlock the indexer. Currently this is not a problem since we
don't (yet) call waitForIndex in such situations, but this needs to be
fixed before the index can be adopted in SearchEngine, which will need
to make such calls.

This could be fixed by changing Job.join to share locks from the joining
thread to the joined thread - or by rewriting the indexer to avoid
obtaining the workspace lock. Preferably both.

### Improve the event model for the index (No bug)

Currently JDT UI always uses the blocking variant of the JDT core calls
(it always waits for the indexer to finish before doing any kind of
search). With the new index, the nonblocking variants will produce much
better results since the index is still functional during indexing. This
will resolve a lot of UI freezes in JDT UI. However, in order for this
to work, JDT UI will need to do something like this (pseudocode):

`   void openSomeView() {`
`       updateUINow();`
`       jdtCore.addListenerToBeNotifiedWhenIndexerIsDone(() -> {updateUiNow();});`
`   }`
`   `
`   void updateUINow() {`
`       results = jdtCore.searchForStuffWithoutBlockingOnIndexer();`
`       ui.showResults(results);`
`   }`

When the UI first shows search results, they may be incomplete since the
indexer may still be running. For this reason, JDT UI should be
listening to events from JDT core that indicates that the indexer is
done running and refresh itself appropriately.

Currently there is no good event for this. When its done indexing, the
indexer currently fires model deltas for external jar files in the same
way it would if those jars had changed on disk, but this isn't ideal
since the UI can't see the difference between a real change on disk and
indexer completion. It also means that code which observes the .jar file
itself would see two change events for each change, which may be
inefficient for some observers.

This problem should really be sorted out. We could either add a new type
of event that indicates that the indexer has completed or we could delay
all model deltas until the content can be found in the index.

## Next Features to add

### Implement a source indexer (bug 496136)

Before we can delete the legacy indexer, the new indexer needs to start
indexing .java files as well as .class files. When the autobuilder is
on, it could do this by piggybacking on the compiler and indexing the
.class files generated from each .java file every time the compiler runs
on it. When the autobuilder is off, we could compile .class files to a
temporary location and index those results.

I'd recommend against trying to index the .java files directly -- in the
presence of lambdas, this is almost as expensive as compiling the .class
files would be, so it's not as efficient as it would have been in older
versions of Java.

### Add method call sites

In order to optimize the call hierarchy view, the index needs to contain
call site information. Currently this is not collected by the indexer or
stored in the index.

### Port SearchEngine to the new index

Searches are significantly faster in the new index, so porting
SearchEngine to the new APIs should provide noticeable speedups
everywhere in JDT.

### Rework JDT UI from using blocking calls to asynchronous calls + change events

With the new index, nonblocking calls will be both faster and return
better results. However, any call that blocks on the index may be
slightly slower while the indexer is running due to the slower indexer.

For this reason, every call in JDT UI that currently blocks on the
indexer to produce user output should be refactored to perform a
nonblocking call, display those results, and then update those results
when and if a change event arrives from the indexer.

Calls in JDT UI that modify code (refactoring, etc) or otherwise produce
irreversible operations should continue to block as they do now.

I believe this is the cause of the current slowness in the type
hierarchy when the indexer is running.

### Add incremental indexing (Bug 510212)

Currently, the indexer reacts to any change by rescanning the entire
workspace for changes and comparing the file fingerprints to the
database. This is more efficient than it sounds, but it could be made a
lot faster if we made use of change deltas and only rescanned parts of
the workspace that were known to have changed. Full rescans could still
be performed to catch and recover from errors, but they would be very
infrequent.

This would also work as a strategy for removing workspace locks from the
indexer. The component that reacts to the deltas could copy all data
that needs to be obtained while holding the workspace lock and copy it
somewhere for the indexer to access later. The indexer itself could then
run lock-free. (see bug 509558).

This would speed up the unit tests considerably and speed up many
operations that block on the index.

[Category:JDT](Category:JDT "wikilink")