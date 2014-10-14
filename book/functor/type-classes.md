## About Type Classes

Haskell uses type classes for functions which may be implemented in different
ways for different data types. For example, we can add or negate various kinds
of numbers: integers, floating points, rational numbers, etc. To accommodate
this, Haskell has a [`Num`][] type class with functions like `(+)` and `negate`
as part of it. Each concrete type (`Int`, `Float`, etc) then defines its own
version of the required functions.

[`Num`]: http://hackage.haskell.org/package/base-4.7.0.1/docs/Prelude.html#t:Num

Being a member of a type class requires you implement any *member functions*
with the correct type signatures. For example, to make `Int` a `Num` someone
defined a `negate` function with the type `Int -> Int`.

Usually, but not always, there are *laws* associated with these that your
implementations must satisfy. For example, if you negate a number twice, you
should get back to the same number. This can be stated formally as:

```haskell
negate (negate x) == x -- for any x
```

The first requirement is enforced by the type system. If your code compiles, you
got this part right. Unfortunately, the second requirement cannot be enforced by
the compiler. You'll need to verify that you got this right using tests. Most
laws can be stated as *properties* and thoroughly tested with a tool like
[QuickCheck][].

[quickcheck]: http://www.haskell.org/haskellwiki/Introduction_to_QuickCheck1
