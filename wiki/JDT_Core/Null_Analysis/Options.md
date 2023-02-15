<css> .source-java { border:\#2F6FAB 1px dashed;
background-color:\#F9F9F9; padding:4px; } .source-otj { border:\#2F6FAB
1px dashed; background-color:\#F9F9F9; padding:4px; } </css>

The analysis of possible null pointer exceptions performed by the
Eclipse Java Compiler is powerful, but the full power may result in a
large number of errors and warnings. Not all projects can afford
addressing all these issues (at once).

This page discusses the problem from two different directions:

  - On the road towards a fully checked, NPE-free program, what are the
    **risks** needing to be addressed before the solution can be
    considered complete, i.e., fully safe?
  - Should a more **lenient** analysis be applied to reduce the number
    of warnings perceived as uninteresting?
      - What are the risks that more relaxed rules would introduce?
      - What are the patterns of coding that (some) users would like to
        see accepted without a warning?
      - What assumptions need to be maintained to render these patterns
        safe?

## Risks of NPE despite annotation based null analysis

If annotation based null analysis applies strict pessimistic rules, a
full guarantee of absence of NPE is only threatened by two risks:

  - specification of nullness is either incomplete or incompletely
    checked
  - checked null specifications are bypassed at runtime

Aside from these an approach that strictly views null annotations as an
extension of the type system is fully safe.

### Incomplete Specification

If you start with an existing project and only turn on annotation based
null analysis, you will not notice any difference. This is because the
analysis implicitly supports *three* kinds of reference types:

  - @NonNull types
  - @Nullable types and
  - types with unspecified nullness (legacy types).

So before you actually add annotations to your methods and fields, all
types will be considered as legacy types and so the unsafe semantics of
Java apply where both assigning null *and* dereferencing are legal.

#### Adding individual annotations

This path is obvious: add just those annotations where you are certain
about the design intent and after each annotation added address the
problems reported by the compiler.

#### Making nonnull the default

Instead of going in tiny little steps you may want to do jumps of some
size (cf. ):

  - annotate a method as @NonNullByDefault to affect all its parameters
    and its return
  - annotate a type as @NonNullByDefault to affect all its methods and
    fieds
  - annotate a package as @NonNullByDefault (using package-info.java) to
    affect all its types
  - define a global policy that all packages should be
    @NonNullByDefault
    *Unfortunately, a global default cannot directly be established but
    only via the indirection of package level defaults (see  for
    background).*

A particular problem arises when a type affected by @NonNullByDefault is
a subtype of a legacy type with legacy signatures. It is illegal to
override a method with legacy parameters by a method with @NonNull
parameters. Specifying @Nullable for all parameters in such an
overriding method may be too pessimistic, forcing the method body to do
more null checks than actually useful.

For this situation a parameter has been added to the default annotation.
Declaring a type that inherits from a legacy type with
@NonNullByDefault({}) for v2 Java 8 type null annotations respectively
NonNullByDefault(false) for v1 cancels the applicable default.

#### Third-party code

Every project depends on third party code, which it doesn't control, so
adding null annotations is not directly possible.

To address this issue, the Eclipse Java Compiler should support nullity
profiles, aka external annotations: separate files that capture the
factual null contracts of all API methods and fields contained in a
given library.

Support for this feature is planned for version 3.9, see .

#### Avoiding the risk of incompleteness

In order to ensure that no unchecked legacy types are used in an
application, these things must be ensured:

  - @NonNullByDefault is the globally enforced default
  - @NonNullByDefault(false) (v1) / @NonNullByDefault({}) (v2, type) is
    never used
  - all libraries come with API-complete nullity profiles

Each of these steps brings a project closer to the safety guarantees of
*complete* analysis.

**TODO**: Issue a configurable warning when @NonNullByDefault({} /
false) is used.

**TODO**: Add a note to  for checking complete coverage of referenced
libraries by available nullity profiles.

-----

### Bypassing compile time checks

Any guarantees given by the analysis can still be bypassed at runtime if
one of the following is applied:

  - runtime reflection beyond pure introspection, i.e, whenever
    reflective field access or method invocation is involved.
  - bytecode manipulation, i.e., whenever the bytes being executed by
    the VM are not identical to the bytes produced at the time when null
    analysis was performed.

These risks are typically considered outside the scope of linguistic
means, developers know about these issues at a general level and don't
expect a static analysis to consider these.

## Lenient analysis

Some warnings/errors flagged by a strict analysis may seem counter
intuitive. Some may actually be irrelevant in specific contexts.

Practical investigations showed that many of those "unwanted"
warnings/errors relate to field access. When focusing on these, the most
obvious approach at distinguishing interesting from uninteresting
warnings is to apply some kind of flow analysis for fields (which the
pessimistic approach strictly avoids).

### Risks of flow analysis for fields

When flow analysis is applied to fields the following reasons may cause
unexpected NPE:

  - side effects
  - implicit control flows
  - aliasing
  - concurrency

Null analysis for the following kinds of fields is essentially
unaffected by these risks:

  - final fields
  - @NonNull fields

For both kinds of fields only object initialization may produce
unexpected results (yes, even Java's definite assignment rule can be
circumvented). Once initialized their null status is either known or
amenable to flow analysis.

*TODO: more precise distinction for final fields: is static relevant?
does initialization occur directly as part of the field declaration?*

When considering flow analysis for fields another annotation becomes
highly relevant, let's call it `@EventuallyNonNull` (aka @LazyNonNull
etc.). This annotation has been proposed by different researchers to
establish monotonicity for a field, which means that a field can never
change from non-null to null, which makes fields with this annotation
amenable to flow analysis. The semantics is defined like this

  - initially the field may be `null`
  - any value assigned to the field must be provably non-null.

#### Side effects

The simplest problem with flow analysis for fields results from side
effects in methods:

``` java
class X {
   @Nullable Object f;
   void test() {
       if (this.f != null) {
           foo();
           this.f.bar();
       }
   }
   void foo() { /* arbitrary code here */ }
}
```

The execution of `foo()` could potentially assign null to `f` thus
invalidating the above null check.

#### Implicit control flows

The problem of side effects is aggravated by the fact, that in Java not
all control flows are visible in the source code. In particular, class
loading may happen at unexpected points in time and my trigger
additional behavior which can interfere with null analysis:

``` java
class X {
   @Nullable Object f;
   void test(Ojbect arg) {
       if (this.f != null && Z.CONSTANT != arg) {
           this.f.bar();
       }
   }
}
```

The mere fact, that evaluating `Z.CONSTANT` needs to access class Z,
which may trigger any static initializers of Z, implies that this access
to a constant may invalidate the flow analysis for field `this.f`.

#### Aliasing

When applying flow analysis for checking nullness of fields the
following snippet demonstrates how aliasing threatens the validity:

``` java
class X {
    @Nullable Y f;
    void test(X other) {
        if (this.f != null) {
            other.f = null;
            this.f.bar(); // potential NPE
        }
    }
    void breakIt() {
        test(this); // definitely triggers the NPE
    }
}
```

Without further annotations an intra-procedural analysis cannot see that
the assignment to `other.f` affects the value of `this.f`.

#### Concurrency

Also concurrency can invalidate the null related guarantees of flow
analysis for fields. In a concurrent application not even this snippet
is safe:

``` java
   if (this.f != null)
       this.f.bar();
```

Thus concurrency adds to the problems of side effects and aliasing,
because null information from one expression may already be invalid when
evaluating the very next expression.

### Drawing a line: what intermediate code to admit?

Given the above risks, we can admit certain styles of code if a certain
risk is considered irrelevant in a given context. The discussion will
use this basic question:

``` java
class X {
   @Nullable Object f;
   void test() {
       if (this.f != null) {
           // what code is allowed here?
           this.f.bar(); // want to be sure there's no NPE here
       }
   }
}
```

What code can we admit at the designated location? What assumptions must
a developer ensure to render that code valid?

#### Admitting any code for @EventuallyNonNull fields

The interesting property of fields declared as `@EventuallyNonNull` is,
that even the most unsafe examples shown below become safe. The
monotonicity property guarantees that the information from the null
check remains valid for the entire remainder of that particular flow,
i.e. here: until the end of the then-block, *no matter what code comes
in between*.

#### Admitting method calls

If the code between check and dereference contains any method call, we
have the fragment used for demonstrating side effects:

``` java
class X {
   @Nullable Object f;
   void test() {
       if (this.f != null) {
           foo();
           this.f.bar();
       }
   }
}
```

It turns out that all three risks mentioned above are effective in this
situations.

This requires to confidently state the assumption that one's application
is free from side effects, aliasing and concurrency. In typical
object-oriented code that assumption, however, is unfounded. Thus
admitting any method call in this location lacks theoretical nor
practical foundation.

#### Admitting field assignments

``` java
class X {
   @Nullable Object f;
   void test(X other) {
       if (this.f != null) {
           other.f = null;
           this.f.bar();
       }
   }
}
```

When demonstrating the effects of aliasing we discussed that the above
fragment is not OK.

Still similar patterns could be accepted, if both of the following hold:

  - no other thread concurrently accesses the field f, *and*
  - we can statically determine that a field f being assigned is
    *different* from the field being dereferenced afterwards.

For the latter assumption we could apply one of these strategies:

  - full alias analysis (not realistic for the JDT)
  - static approximation: ignore that different objects hold different
    incarnations of the same field, treat any assignment to a field as
    invalidating the analysis results for all objects of that class.

Using static approximation the above fragment would be considered as
unsafe, because the assignment refers to the same field, although at
runtime different incarnations (of different receiver instances 'this'
and 'other') may be involved. This static approximation will thus flag a
few locations that might be OK given more knowledge about the instances
'this' and 'others', but the guarantees given by this approach can only
be broken by concurrency.

### Drawing a line: what kinds of field references to include?

So far we have focused on field references of the shape `this.f`. Can
the previous discussion be extended to other shapes of field references?

#### Accepting local-relative field references

When we want to apply the above to field references `local.f`, where
`local` is any local variable in scope, we have to ensure one of these:

  - `local` is declared `final`, *or*
  - analysis discards all information for `local.f` at any assignment to
    `local` itself.

So this is safe:

``` java
class X {
   @Nullable Object f;
   void test() {
       X local = new X();
       if (local.f != null) {
           // code without method calls nor assignments to local
           local.f.bar();
       }
   }
}
```

while this is unsafe:

``` java
class X {
   @Nullable Object f;
   void test(X other) {
       X local = new X();
       if (local.f != null) {
           local = other; // status of other.f is unknown
           local.f.bar();
       }
   }
}
```

This strategy is conceptually sound. It is a idiosyncrasy of the
analysis engine applied inside the JDT, that implementation of this is
far from trivial.

#### Accepting multi-segment field references

For field references that contain more than one dot (and also those
`f.f2` where `f` is already a field reference, relative to the implicit
'this') more must be considered as illustrated by the following example:

``` java
class Y {
   @Nullable Z f2;
}
class X {
   Y f;
   void test(X other, Y yippie) {
       X local = other;
       if (local.f.f2 != null) {
           // either of the next three assigments invalidates the check result:
           other.f = yippie;
           local.f = yippie;
           local = other;
           local.f.f2.bar();
       }
   }
}
```

At this point, the additional segment means we need to ensure that no
link in the chain from `local` to `f2` breaks. Again two possible
strategies are possible:

  - require all links in the chain to be final
  - discard analysis results for `local.f.f2` at every assignment that
    could affect one of these links.

Without full alias analysis the second strategy will amount to a static
approximation where, e.g., assignment to any `f` would trigger
discarding the analysis result.

#### Applying @EventuallyNonNull to different shapes of field references

While results for @Nullable fields are loosing more and more precision
when relaxing the rules in one of the ways discussed above, we saw that
@EventuallyNonNull fields can still support strong guarantees.

For `this.f` references to an @EventuallyNonNull field we can accept any
code between a check and a dereference.

For `local.f` references we only have to ensure that `local` is not
locally assigned a different value between check and dereference. If
that is given, we can again accept any code between a check and a
dereference.

For `local.f.f2` references even to @EventuallyNonNull fields the
desired guarantee can be violated, as demonstrated by this example:

``` java numberLines
class Y {
   @EventuallyNonNull Z f2;
}
class X {
   Y f;
   void test(X other, Y yippie) {
       if (other.f.f2 != null) {
           foo();
           other.f.f2.bar();
       }
   }
   void foo() {
       this.f = new Y();
   }
   void breakIt() {
       test(this, new Y());
   }
}
```

Due to the alias created between 'this' and 'other' (line 16), the
assignment inside the execution of foo changes the meaning of 'other.f',
thus the assumed knowledge about 'other.f.f2' is wrong, and no strategy
short of full alias analysis can restore the desired guarantee.

Additionally, concurrency could also affect `f` thus invalidating the
safety guarantee.

This means the promise given by applying @EventuallyNonNull cannot be
fulfilled when including multi-segment field references in flow
analysis. Positively speaking: for analyzing a reference to an
@EventuallyNonNull field, both of the following must hold:

  - the first segment must be under the control of the current method
    (either 'this' or a final local or a local with discarding
    information at every assignment to this local), *and*
  - if the field reference has intermediate links all these must be
    `final`

### Introducing syntactic special cases

Independent of the discussion above, the following special case can
easily be detected by the compiler:

``` java
class X {
   void test(X other) {
       if (other.f.f2 != null) {
           other.f.f2.bar();
       }
   }
}
```

This is to say, the code could be considered as safe if a dereference
directly follows a null check where both expressions are exactly
identical references at the syntax level. Considering this style as safe
only risks violation by concurrency.

## Summary

For non-final @Nullable fields the simplest strategy for avoiding any
nullness related risks is to pessimistically assume potential null at
*every* read. This means for strict checking no flow analysis should be
applied to @Nullable fields.

While this appears to be a very drastic restriction, the remedy is quite
easy: before dereferencing a @Nullable field it has to be assigned to a
local variable. Flow analysis is then safely applied to the local
variable with no risk of side effects, aliasing nor concurrency, since
local variables are not shared with any code locations that would be
outside the scope of the analysis. I.e., the flow analysis can see
everything it needs to consider regarding local variables.

Any flow analysis performs much better for local variables than for
fields. Shifting all serious computation to local variables makes for
much safer code. Whether or not a project can afford to fully enforce
this strategy depends on many factors.

Next to the use of local variables also @EventuallyNonNull makes the
code much more amenable to flow analysis, thus fully reliable analysis
can be achieved even without the abundant use of local variables. If
null checks directly against a field should be exploited using flow
analysis, applying @EventuallyNonNull provides the biggest gain.
However, the guarantees of this annotation are bounded by the kind of
field references used, they don't hold for the general case of
multi-segment references.

In situations where concurrency and aliases are considered as irrelevant
some limited forms of flow analysis can also be applied to @Nullable
fields. Here the kind of field reference only mildly affects the
precision of the analysis, which is already very limited anyway. Perhaps
a large portion in this domain is already covered by the syntactic
special case proposed above.

Aside from the technical discussion of what assumptions support which
guarantees, also the following questions deserve further investigation:

  - How can gradual migration to a fully safe coding style fully based
    on local variables be supported. I.e., what intermediate levels are
    useful, what configuration options are needed and what warning
    messages best convey the vagueness/certainty of each issue?
  - How can the above options be communicated to users? Is it OK to
    offer options like "perform flow analysis for fields", or should
    options be labeled as "consider aliasing", or "pessimistically
    consider concurrency for null analysis"?
  - How can initialization be made safer? How relevant are these issues
    in practice? Which annotation / set of annotations is the best buy?
    Is support for several styles required?

[Category:JDT](Category:JDT "wikilink")