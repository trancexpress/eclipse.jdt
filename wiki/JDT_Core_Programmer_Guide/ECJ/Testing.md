For testing the compiler it is **not necessary** to ever run a **maven**
build.

Running `org.eclipse.jdt.core.tests.RunCompilerTests` (or any of its
constituents) as a JUnit (3) Plug-in Test suffices. For faster
launching, open the "Main" page of the run dialog and choose "**Headless
Mode**" as the application.

For **selectively** running only a **single test method**, or a group of
test methods, see that each test class has a static block with a
commented assignment to `TEST_NAMES` and similar. Uncomment and insert
the name of the desired test and run the class. Note, that the given
name will be used for prefix-matching, so tests `testMethod1`, ...
`testMethod13` can all be run by specifying `"testMethod"`. Please try
not to commit this change of `TEST_NAMES`.

## Parameters of the compiler test suite

### compliance

Specify at which compliance levels a test should be run by adding a VM
argument like this (using a comma separated list of values):

`-Dcompliance=1.8,14`

Some compiler tests not only compile but also execute the compiled class
files. For those ensure that the test is executed on a **JRE** that is
the same or newer as the newest compliance level specified.

Not specifying a compliance level causes the tests to be executed for
*each compliance level* compatible with the current running JRE, which
can be very time consuming.

### run.javac

This special test mode is used to compare the behaviors of ecj and
javac.

Typical VM arguments for this mode look like this:

`-Drun.javac=enabled -Djdk.root=/home/java/jdk1.8.0_212
-Dcompliance=1.8`

Dedicated Jenkins jobs exist to test in this mode for different
compliance levels:

  - [eclipse.jdt.core-run.javac-1.8](https://ci.eclipse.org/jdt/job/eclipse.jdt.core-run.javac-1.8)
  - [eclipse.jdt.core-run.javac-10](https://ci.eclipse.org/jdt/job/eclipse.jdt.core-run.javac-10)
  - [eclipse.jdt.core-run.javac-12](https://ci.eclipse.org/jdt/job/eclipse.jdt.core-run.javac-12)

These jobs are triggered once per week (on weekends).

Each of these jobs demonstrates a number of differences (as failures) as
summarized most recently in .

The general umbrella bug for differences between ecj and javac is .

Known differences are documented using constants of type
\<code.org.eclipse.jdt.core.tests.compiler.regression.AbstractRegressionTest.JavacTestOptions.Excuse</code>
during test invocation. For conveniently passing the correct "Excuse" in
each affected test, it is recommended to use class "Runner" as shown
[below](#different_styles_of_test_invocation "wikilink"), and its field
`javacTestOptions`.

### jdt.flow.test.extra

Flow analysis has separate implementations for the first 64 variables
and subsequent ("extra") variables. Since most tests only exercise the
first implementation, a mode has been added to explicitly test the
second implementation (by faking 64 additional variables).

`-Djdt.flow.test.extra=true`

For better test coverage, this mode is enabled for one of the
time-scheduled jobs:
[eclipse.jdt.core-run.javac-10](https://ci.eclipse.org/jdt/job/eclipse.jdt.core-run.javac-10)

Since that job is known to fail, watching for regressions of this job is
particularly relevant.

### jdt.performance.asserts

A few JDT tests include some sort of performance assertions, ideally not
tied to specific wall time thresholds, but testing the characteristics
at which complexity grows (linearly vs. O(2^n) etc).

Since these tests frequently fail on the fluctuating jenkins
infrastructure of gerrit jobs, those assertions are disabled for gerrit:

`-Djdt.performance.asserts=disabled`

Other test runs still keep them enabled (the default).

When new performance-related tests are added to the suite, they should
guard their performance assertions under the same flag, in code
accessible as
`org.eclipse.jdt.core.tests.util.AbstractCompilerTest.PERFORMANCE_ASSERTS`.

### jdt.test.output_directory

This property will tell the test suite to create test files in a
non-standard location (otherwise files will be created below the default
temp dir).

## Authoring / editing tests

### different styles of test invocation

Inside each test method of the compiler test suite, an invocation like
`runConformTest`, `runNegativeTest` passes the source file and
additional parameters to the actual testing infra structure.

Traditionally, a myriad of overloads of these methods has been created
to accommodate the needs of different tests, which makes picking the
appropriate method tedious and still uninteresting parameters have to be
provided in many cases.

More recently, class
`org.eclipse.jdt.core.tests.compiler.regression.AbstractRegressionTest.Runner`
has been created to enable providing exactly the relevant test
parameters before invoking one of the `run*Test()` methods.

[Category:JDT](Category:JDT "wikilink")