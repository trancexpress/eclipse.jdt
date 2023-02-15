<css> /\* box for bash listings \*/ .source-bash { border:\#2F6FAB 1px
dashed; background-color:\#F9F9F9; padding:4px; } </css>

## Strategies for investigating compiler bugs

### Comparing compiler versions

For initial triage it is useful to test how different compiler versions
respond to a given source file. This can be done quickly using a shell
and a few shell scripts. The following assumes `bash`.

**`ecj-HEAD.sh`: Run the compiler from class files of an existing
JDT/Core workspace / git clone**:

``` bash
#!/bin/sh

# Adjust path to the root of a git clone of JDT/Core:
JDT_GIT=/home/git/eclipse.jdt.core
# Adjust path to a suitable Java installation:
JAVA_HOME=${JAVA_HOME:-/home/java/jdk1.8.0}

ECJ_BINS=${JDT_GIT}/org.eclipse.jdt.core/bin:${JDT_GIT}/org.eclipse.jdt.compiler.apt/bin
PATH=${JAVA_HOME}/bin:${PATH}

java -classpath $ECJ_BINS org.eclipse.jdt.internal.compiler.batch.Main $*
```

**`ecj.sh` Run the compiler from one out of a set of stored ecj jar
files**:

Assume you have a directory (`$BASE` in the script below) holding
different versions of ecj, with file names like `ecj-4.16.jar`, then the
following script can be used to pick and use any of the stored versions:

``` bash
#!/bin/sh

BASE=${HOME}/jdt/ecj

ECJ=${BASE}/ecj-${1}.jar
if [ -f ${ECJ} ]
then
  echo "Compiling with $ECJ"
else
  echo "Not found: ${ECJ}"
  echo "Available:"
  cd ${BASE}
  ls ecj-*.jar
  exit 2
fi
EXTRA="-proc:none"
case ${1} in
    4.[4-7]*)
        JAVA_HOME=/home/java/jdk1.8.0
        PATH=${JAVA_HOME}/bin:${PATH}
        ;;
    4.[0-3].*)
    JAVA_HOME=/home/java/jdk1.6.0
        PATH=${JAVA_HOME}/bin:${PATH}
    ;;
    3.[0-9]*)
    JAVA_HOME=/home/java/jdk1.5.0
        PATH=${JAVA_HOME}/bin:${PATH}
    EXTRA=""
    ;;
esac

shift
echo java -jar ${ECJ} ${EXTRA} $*
java -jar ${ECJ} ${EXTRA} $*
```

Example invocation: `$ ecj.sh 4.16 -1.8 Hello.java`

The extra argument `-proc:none` is useful because some historic ecj
versions lack the annotation processing classes and thus crash when
trying to perform annotation processing.

Also note, that different version ranges of ecj require different Java
versions to run. Adjust paths in the script to your local installations.

#### Speedy bisecting

If the compiler behavior changed significantly at a yet unknown point in
time, the following appears to be the fasted approach to find the exact
commit causing the change:

1.  Use the parametric script `ecj.sh` to find the release or even
    milestone that first introduced the change.
2.  Identify git tags of last version with old behavior and first
    version with new behavior
3.  In a shell inside a JDT/Core git clone perform the `git bisect`
    subcommands
4.  In an Eclipse workspace that has org.eclipse.jdt.core from that git
    imported, refresh the git repo and wait for auto build to complete
5.  In another shell where you have your test source invoke
    `ecj-HEAD.sh`
6.  Check the result and go back to 3. until done

It is useful to have a git clone and eclipse workspace dedicated only to
bisecting, so that you never mess up your regular workspace. This is
reflected by a copy of `ecj-HEAD.sh` where `${JDT_GIT}` points to that
instance of org.eclipse.jdt.core.

### Comparing compiler output

In most cases, it is sufficient to check which compiler version
**accepts** / **rejects** the given test program, possibly checking also
for warnings.

In other cases it is also necessary to compare **executions** of the
compiled test program.

In rare case it may even be necessary to also check the compiler
**output files** (e.g., to check class file attributes like annotations
etc). When different compiler versions generate different byte code, the
following sequence would identify that difference:

``` bash
$ ecj.sh 4.10 Hello.java
$ javap -p -v -c Hello.class > Hello-4.10.list
$ ecj.sh 4.11 Hello.java
$ javap -p -v -c Hello.class > Hello-4.11.list
$ diff Hello-4.10.list Hello-4.11.list
```

Obviously, a graphical diff tool (like `kompare`) comes in handy for
this task.

### Typical breakpoints

**Error reporting**

  - Inside
    `ProblemHandler.handle(int,String[],int,String[],int,int,int,ReferenceContext,CompilationResult)`
      - Breakpoints after `case ProblemSeverities.Error` and `case
        ProblemSeverities.Warning` will be triggered by every detected
        problem of the given severity (although some may be filtered out
        during `CompilationUnitDeclaration.finalizeProblems`).
  - Breakpoints in constructors of `Problem*Binding`: this bindings are
    created when resolving found an error, sometimes the creation of the
    problem binding is closer to the cause then the above handle()
    method.
  - To find the locations responsible for a particular known error
    message proceed as follows:
      - Inside
        `org/eclipse/jdt/internal/compiler/problem/messages.properties`
        locate the error message and identify the error number at the
        start of that text line
      - Find the same number in `org.eclipse.jdt.core.compiler.IProblem`
        and search for references of the corresponding constant
      - You will have found one or more methods in
        org.eclipse.jdt.internal.compiler.problem.ProblemReporter, call
        hierarchy of its callers are what you are looking for
          - Shortcut: call hierarchy on the constant in IProblem (make
            sure the **Field access** option from the view menu is *not*
            set to **Write access**).

**Type lookup from the environment**

  - Methods `LookupEnvironment.askForType` (both overloads\!): every
    additional type pulled into compilation must be found by one of
    these methods.

**Parsing**

  - A breakpoint on the first line of method `Parser.consumeRule(int)`
    approximates single stepping through the parse process.
      - Note, that Parser & Scanner have useful toString() methods, so
        the progress of parsing can be watched in the details of the
        Variables view.

**TypeInference**

  - *I'll write a section dedicated only to debugging type inference*

**Checking compiler output during tests**

  - `AbstractRegressionTest.runTest()` (the core method, longest
    parameter list): a breakpoint immediately after
    `batchCompiler.compile()` let's you inspect the generated class
    files, to be found in your tmp directory at
    `comptest/run`<timestamp>`/regression`.

**Completion**

  - An exception breakpoint on
    `org.eclipse.jdt.internal.codeassist.complete.CompletionNodeFound`
    will trigger when the completion node is resolved.
      - Use this breakpoint if the AST looks OK, but problems exist
        downstream

**Other exceptions used in the compiler**

Exceptions where throw and catch are near to each other are not listed
here.

  - `AbortCompilation` and subtypes: thrown when severe problems suggest
    that nothing good will come afterwards
  - `CopyFailureException`: to enable multiple resolution attempts,
    lambda expressions are copied (AST). In case of syntax errors, this
    may not be possible.
  - `ClassFormatException`: when a .class file cannot be interpreted.
  - `InferenceFailureException`: initially used to detect gaps in the
    implementation of type inference, mostly obsolete by now.

[Category:JDT](Category:JDT "wikilink")