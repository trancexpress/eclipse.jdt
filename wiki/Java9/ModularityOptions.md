Oxygen.1a brought initial support to configure new options as defined in
[JEP 261](http://openjdk.java.net/jeps/261) (*beware, we have been told
that the JEP contains outdated information*).

The new UI extending the Java Build Path configuration does not yet
cover all possible & desirable use cases. This page collects the open
issues and discusses how we can support everything in a maximally
consistent way.

When discussing integration in the Java Build Path configuration, will
this be picked up by launch configurations automatically, or is more
needed also in JDT/Debug?

## \--add-reads

### What's missing?

  -
  - `--add-reads my.mod=XXX`

      - special case `--add-reads my.mod=ALL_UNNAMED`

So far, --add-reads can only be applied to a module on which the current
project depends, but not on the current module itself.

### Use cases

Why is this relevant / what is the user trying to achieve by this
option?

  - Does it make a difference whether reading named or unnamed modules?
      - Yes, module-info cannot say "reads ALL_UNNAMED".
  - Do we still need to support named modules?
      - Yes, for test code. See .
          - \[SH\]: Let me check if I can construct a user story for
            this (correct me if I'm wrong):
              - A project defines a module M1
              - Additionally, it has a test source folder, that depends
                on module M2
              - M1 should not require M2, ergo "reads" is not a property
                of the project but of the test folder
              - The test folder cannot define its own module-info, so it
                cannot easily declare test-specific **requires** (files
                in this folders will be associated to the 'main' module
                of the project and can only use dependencies declared
                there).
              - Hence we need a way to express that M1 comes in two
                variants, with / without test source & test
                dependencies.
                  - \[NL\]: This is a possibility, or at least a way to
                    think about it.
              - Question: Is this issue limited to "reads" or would the
                test variant possibly need other directives, too?
                  - Export packages to test folders of downstream
                    projects? My guess: yes\!
                  - Declare service consumption? My guess: yes\!
                  - \[NL\]: It could also need --add-exports from the
                    module it reads to itself if the module being read
                    does not export the needed packages by itself. This
                    is already possible in the UI.
                      - \[SH\]: *(minor:)* If module-info already
                        requires M1 and tests need M1 to export p1, that
                        package would then be accessible also from main
                        code of the project. This *might* be
                        undesirable.
          - \[SH\]: If the above bullet is a true rendition of the issue
            at hand, could the same be expressed in module-info.java
            syntax, rather than as directives on the build path? Reason
            for asking: I see explicitly tweaking "Modularity Details"
            mainly as a tool during migration towards modules - in a
            perfect modular world none of this would be needed - whereas
            different main / test dependencies is a part of good
            software engineering, designed to stay.
              - \[NL\]: You can add the readability edges in module-info
                directly, but this means that you will need
                Eclipse-specific module-info files because it's the only
                IDE that treats test folders this way. As things stand,
                whether the case is a full modular world or a migration
                world, the problem of test folders still needs to be
                solved.
          - \[SS\] My vote will be to use module-info.java for all named
            modular dependencies.
          - \[SH\]: How close does "require static" come to expressing
            test dependencies?
              - \[NL\]: I \*think\* this is exactly the alternative
                mentioned above with "could the same be expressed in
                module-info.java syntax".
          - \[NL\]: I think that ideally Eclipse should do what other
            IDEs do (do they treat test sources as "real non-modules",
            which adds reads for everything?). Maybe have every source
            folder its own module - I think this is best. If not, allow
            to specify in the UI read edges (can be done directly in the
            classpath).
              - \[SH\]: Have "other IDEs" converged on a single solution
                that is both intuitive and powerful? Otherwise I'd
                prioritize consistency *within* our approach over
                copying other IDEs' concepts.
  - How does the request interact with the distinction main/test
    sources?
      - \[NL\]: main sources are can do everything in module-info so
        they are fine. Test sources are restricted by module-info, but
        cannot change it, so they need a way to express their special
        needs.

### Solution discussion

  - Specifically for test code we seem to have two conflicting mental
    models, which should be sorted out before discussing details of a
    solution:
      - Test code is not part of the project's module, it has all the
        privileges of an unnamed module.
      - Test code is part of the project's module, but two different
        views of that module exist: *main* and *main-and-test*
    <!-- end list -->
      -
        (*I believe this is the appropriate description for our current
        implementation*).
    <!-- end list -->
      - \[TB\] Currently, test code reads the unnamed module (as maven
        does). For the use case where test code should read additional
        named modules, maven-compiler-plugin 3.8.0 now seems to support
        a secondary module-info.java in the test source folder (but I
        haven't checked how it is compiled or executed.
        <https://issues.apache.org/jira/browse/MCOMPILER-341>). Maybe we
        should try to support that, too?
  - Where in the UI should this option be represented?
      - \[TB\] In that context: should we have a "test-only" flag for
        add-reads configurations or in general extra
        test-module-options?
      - Is it a property of a source folder?
          - How do we handle multiple source folders per project?
      - Is it a property of the project?
          - If so, which part of the Build Path dialogs best represents
            "the project"?
  - What should be the gestures / dialogs etc. to define this?

## \--patch-module

### What's missing?

  - , see also

  - `--patch-module other.mod=third.location`

So far, --patch-module can only declare that the *current* project
patches an existing module on which it declares a dependency.

In  it is argued that we should also support the case where one
dependency (a non-modular(?) jar, project ...) patches another module on
the build path.

### Use cases

Why is this relevant / what is the user trying to achieve by this
option?

  - What kind of locations can be declared as patching another module?
      - JEP 261 speaks of a "filesystem path name of a module
        definition", but then if a module-info is found there, a warning
        is raised and the file ignored.
      - \[TB\] This is the way to run with a test-specific
        module-info.java (actually main code needs to be configured to
        patch the test code). I think Remi Forax's pro build tool
        <https://github.com/forax/pro> does that.
  - Would we even need the option to declare some external code to patch
    the *current* module?

\[SS\] If we want to test with different implementations before
adopting, it might help but I feel it is good to have but not a blocker.

  - How does the request interact with the distinction main/test
    sources?
      - Automatically added --patch-module for test execution would need
        to be merged with user-configured --patch-module directives.

### Solution discussion

  - Where in the UI should this option be represented?
  - What should be the gestures / dialogs etc. to define this?

## \--upgrade-module-path

  - mentioned in

### Use case

Replace a module with a custom variant of the same name, where the
original module could either be

  - a system module, or
  - an application module on the module path

<!-- end list -->

  -
    (\[SH\]: *why would one put a module on the module path, if later
    you want to replace that module with another one??*).

### Solution discussion

  - User's intention could already be expressed like this:
      - exclude the given module from the "Contents" of the JRE node on
        the Java Build Path
          - \[NL\]: Isn't this how limit-module is expressed? If so,
            there will be a runtime issue as mentioned in the bug.
              - \[SH\]: As mentioned below I was thinking of using that
                UI while generating a specific upgrade-module-path for
                launching - if exclusion and addition match.
      - add the replacement to the module path
  - If that is too implicit, a warning could be given requiring the user
    to add a flag "yes I mean to replace a system module"
  - Alternatively, Upgrade Modulepath could be added as a third node
    below Classpath & Modulepath

Even if the user doesn't explicitly say "upgrade-module-path" this
option could be synthesized behind the scenes (for launching).

## \--add-opens

  - not a build time concept but  argues that the Java Build Path
    *might* still be an intuitive location

### Solution discussion

  - Build Path \> Modular Options
  - Not in Build Path.

## Defining the main class

  -
### Solution discussion

  - Build Path \> Source
  - Build Path \> Modular Options

[Category:JDT](Category:JDT "wikilink")