### Null Contracts

Using [null annotations](JDT_Core/Null_Analysis "wikilink") in method
signatures can be explained as contracts between the method
implementation and its clients. Each part of a null contract implies an
obligation for one side and a guarantee for the other side.

#### Method Parameters

When a method parameter is specified as **nullable** this defines the
<font color="red">obligation</font> for the **method implementation** to
cope with null values without throwing NPE. **Clients** of such a method
enjoy the <font color="green">guarantee</font> that it is safe to call
the method with a null value for the given parameter.

When a method parameter is specified as **nonnull** all **clients** are
<font color="red">obliged</font> to ensure that null will never be
passed as the value for this parameter. Thus the **method
implementation** may rely on the <font color="green">guarantee</font>
that null will never occur as the value for this parameter.

#### Method Returns

The situation is reversed for method returns. All four cases are
summarized by the following table:

|                    | caller                                                           | method implementation                                  |
| ------------------ | ---------------------------------------------------------------- | ------------------------------------------------------ |
| nullabel parameter | <font color="green">may safely pass null without checking</font> | <font color="red">must check before dereference</font> |
| nonnull parameter  | <font color="red">must not pass null </font>                     | <font color="green">may use without checks</font>      |
| nullable return    | <font color="red">must check before dereference</font>           | <font color="green">can safely pass null</font>        |
| nonnull return     | <font color="green">may use without check</font>                 | <font color="red">must not return null</font>          |
|                    |                                                                  |                                                        |

#### Local Variables

Null contracts can also be defined for local variables although this
doesn't improve the analysis by the compiler, because local variables
can be fully analyzed without annotations, too. Here the main advantage
of null annotations is in documenting intentions.

The following is an example of a program where all sides adhere to their
respective part of the contract:

``` java
public class Supplier {
    // this method requires much but delivers little
    @Nullable String weakService (@NonNull String input, boolean selector) {
        if (selector)
            return input;
        else
            return null;
    }
    // this method requires nothing and delivers value
    @NonNull String strongService (@Nullable String input) {
        if (input == null)
           return "";
        else
           return input.toUpperCase();
    }
}
public class Client {
    void main(boolean selector) {
        Supplier supplier = new Supplier();
        @Nullable String value = supplier.weakService("OK", selector);
        @NonNull String result = supplier.strongService(value);
        System.out.println(result.toLowerCase());
    }
}
```

Notes:

  - Although we know that toUpperCase() will never return null, the
    compiler does not know as long as `java.lang.String` does not
    specify null contracts. Therefor the compiler has to raise a warning
    that nullness of the return value is unknown and thus contract
    adherence cannot be guaranteed statically.
  - Null annotations for the local variables are redundant here, as the
    nullness information can be fully derived from the variable's
    initialization (and no further assignments exist).

### Null Contract Inheritance

A method that overrides or implements a corresponding method of a super
type (class or interface) inherits the full obligations put forward by
the super method's null contract. For clarity of the code the overriding
method must repeat the inherited signature including any null
annotations.

  -
    *This behavior has been made configurable through  -
    \[compiler\]\[null\] inheritance of null annotations as an option*

However, the rules for safe polymorphic calls still allow the following
ways of redefining the inherited null annotations:

  - A nonnull method **parameter** may be **relaxed** to a nullable
    parameter. The additional checks have to be performed in the body of
    the overriding method. Callers of the super type must still pass
    nonnull, while callers which are aware of the sub type may pass
    null.
  - A nullable method **return** (or a return with no null annotation)
    may be **tightened** to a nonnull return. The additional checks must
    again be performed in the body of the overriding methods. Callers of
    the super type still have to check for null, whereas callers which
    are aware of the sub type may safely assume nonnull return values.

Any overrides that attempt to change a null contract in the opposite
directions will raise a compile time error.

This explicitly implies that callers only need to inspect the null
contract of the statically declared type of a call target to safely
assume that all runtime call targets will adhere (at least) to the
contract as specified in the statically declared type, even if the
runtime type of the call target is any sub type of the declared type.

[Category:JDT](Category:JDT "wikilink")