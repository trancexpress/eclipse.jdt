## Global Conventions

Throughout the following lookup structures a convention exists that
methods `getPackage0()`, `getType0` and a few more like this will only
check, if a sought element already exists in one of the caches (call
them "**passive**"). Typically, a sibling method exists without the `0`
suffix, which in case of a cache miss will actively search for the
sought element ("**active**").

To record the fact that an element was not found in a particular
location of the tree, special instances `TheNotFoundModule`,
`TheNotFoundPackage`, `TheNotFoundType` are inserted into the tree to
avoid repeated, unsuccesful resolution attempts.

## LookupEnvironment

The central place where the following bindings are globally stored:

  - ModuleBinding (`knownModules`)
  - PackageBinding (`knownPackages`, recursively)
      - TypeBinding (`knownTypes`)

Note, that the structure below `knownPackages` forms a **tree**, where
each edge corresponds to a **simple name segment**. Based on this most
lookup is done **incrementally**, i.e., we resolve one simple name at a
time, relative to the result so far. The only exception to this rule is
field `ModuleBinding.declaredPackages` (see below).

Since Java 9, different instances of LookupEnvironment are used, to
implement the perspective of one module each. In this situation

  - the root lookup environment represents the *unnamed module*
  - module-specific lookup environments are found via the `environment`
    field of elements of `knownModules`
  - each lookup environment has a back link `root` to the root
    environment
  - fields of LookupEnvironment that are not module specific are
    documented as either ROOT_ONLY or SHARED (see javadoc of
    LookupEnvironment\#root).

### Other "singletons"

The root lookup environment also links a few unique instances:

  - `globalOptions` : all compiler options in effect
  - `nameEnvironment` : sometimes called the *oracle*, since it's an
    external/opaque entity that can answer sought types and packages.
  - `typeRequestor` : callback for newly discovered types
  - `typeSystem` : manages variants of known type bindings
    (parameterizations, arrays, annotations)
  - `verifier` :
  - `problemReporter` : a fall-back problem reporter, which is used for
    reporting errors that cannot be associated to a particular AST node.
  - `classFilePool` :

*TBC*

## ModuleBinding

Since Java 9 also module bindings act as lookups.

The current design has been developed in :

### Declared Packages

Each module binding stores a flat map of `declaredPackages`.

  - This map is indexed by **qualified names**
  - Values in this map are `PlainPackageBinding`

The map represents exactly those packages that are declared in this
module. It does not take into account possible name clashes with
packages declared in another module (`SplitPackageBinding`, see below).

  -
    **NB:** ''Much of the complexity of package lookup is necessary to
    *admit* and *distinguish* same-named packages in different modules.
    Interestingly, the JVM will not be able to handle such programs
    unless a custom `Layer` implementation is provided.''

During compilation, the map `declaredPackages` is *eagerly* filled with
all packages mentioned in **exports** and **opens** directives. This is
done as to avoid re-entrant package lookup as we observed before  due to
crazy traversal of the module graph etc. For automatic modules, which
have no such declarations, the structure is filled using a lazy,
one-time scan using `IModuleAwareNameEnvironment.listPackages()`.

For explicit modules additional (non-exported) packages may be added
lazily, but those have no impact on resolving of references outside this
module.

### Visible and Accessible Packages

Additionally, two methods of name `getVisiblePackage` exist, which will
search the current module and all other modules read by it. If more than
one visible module declares the requested package, the result will be
represented as a `SplitPackageBinding`. JLS distinguishes visibility and
accessibility. The former is defined just by the module graph, whereas
the latter, stronger form also takes **exports** declarations into
account. To query accessibility, the method `canAccess(PackageBinding)`
can be used.

## PackageBinding

As mentioned, package bindings are organized as a tree below
`LookupEnvironment.knownPackages`. Packages bindings can be used to look
up child packages (`getPackage()`) and types (`getType`). For qualified
names of unknown kind method `getTypeOrPackage()` does all the hard
work.

Since Java 9 all such lookup should pass the client module (where the
reference happens), so that lookup can obey to the rules of visibility
and accessibility of JPMS.

Java has a bit of a split mind as to whether or not "**empty packages**"
"exist". We have this tree of packages where leaf packages are
"children" of some "parent" package. Many of these "parent" packages are
empty. OTOH, some parts of JLS only consider packages that "exist" in
terms of being declared by a compilation unit. In that sense those empty
"parent" packages do not "exist". Note, that strictly speaking, a
package containing a non-Java resource, does not exist either.
PackageBinding sports a method `hasCompilationUnit` to check for the
stricter forms of existence. When passing `true` for the parameter
`checkCUs` the batch compiler will actually parse any found .java file,
to check if it indeed declares the expected package (which is of course
a bit expensive).

### SplitPackageBinding

In a way this particular strange guy acts as a lookup too: It represents
a package with incarnations in different modules and allows:

  - navigating to a child package (given a module perspective), which
    may or may not again be a SplitPackageBinding
  - extracting a specific package incarnation corresponding to to the
    slice of this package within a particular module

## ReferenceBinding

At the level below packages, `ReferenceBinding` and subclasses represent
all classes and interfaces.

Lookup of **fields** and **memberTypes** works just as expected, because
still here names are unique.

Only when diving into **methods**, a lot more work is needed to consider
type inference and overload resolution, so that's a separate story to be
told another time.

A special caveat on methods `unResolvedFields` and `unResolvedMethods`:
initially these have been implemented with the intention to access known
members *without triggering type resolution* (which can cause
detrimental re-entrance in some situations). Unfortunately, the picture
is blurred here: in particular the implementation in `ReferenceBinding`
simply delegates to the other `getMethods()` which *may* trigger type
resolution\!

## Scopes

In addition to the global lookup (which serves source and binary
elements), the following AST nodes have their dedicated `Scope` which
performs the initial, location-aware part of name lookup:

  - CompilationUnitDeclaration -\> CompilationUnitScope
  - ModuleDeclaration -\> ModuleScope
  - TypeDeclaration -\> ClassScope
  - AbstractMethodDeclaration -\> MethodScope
  - Block -\> BlockScope (also used by block-like nodes like
    ForeachStatement)

A TypeDeclaration may hold additional scopes `initializerScope` and
`staticInitializerScope` for resolving field initializers
(implementation uses MethodScope also here).

Each scope has a **parent** link to support inside-out searches during
resolution. *TBC*

[Category:JDT](Category:JDT "wikilink")