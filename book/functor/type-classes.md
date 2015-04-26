## About Type Classes

Haskell has a concept called *type classes*. These are not at all related to the
classes used in Object-oriented programming. Instead, Haskell uses type classes
for functions that may be implemented in different ways for different data
types. These are more like the *interfaces* and *protocols* you may find in
other languages. For example, we can add or negate various kinds of numbers:
integers, floating points, rational numbers, etc. To accommodate this, Haskell
has a [`Num`][] type class that includes functions like `(+)` and `negate`.
Each concrete type (`Int`, `Float`, etc) then defines its own version of the
required functions.

[`Num`]: http://hackage.haskell.org/package/base-4.7.0.1/docs/Prelude.html#t:Num

Type classes are defined with the `class` keyword and a `where` clause listing
the types of any *member functions*:

```haskell
class Num a where
    (+) :: a -> a -> a

    negate :: a -> a
```

Being an *instance* of a type class requires that you implement any member functions
with the correct type signatures. To make `Int` an instance of `Num`, someone
defined the `(+)` and `negate` functions for it. This is done with the
`instance` keyword and a `where` clause that implements the functions from the
class declaration:

```haskell
instance Num Int where
    x + y = addInt x y

    negate x = negateInt x
```

Usually, but not always, *laws* are associated with these functions that
your implementations must satisfy. Type class laws are important for ensuring that type classes are useful. They allow us as developers to reason about what will happen
when we use type class functions without having to understand all of the
concrete types for which they are defined. For example, if you negate a number
twice, you should get back to the same number. This can be stated formally as:

```haskell
negate (negate x) == x -- for any x
```

Knowing that this law holds gives us a precise understanding of what will happen
when we use `negate`. Because of the laws, we get this understanding without
having to know how `negate` is implemented for various types. This is a simple example,
but we'll see more interesting laws with the `Functor` type class.
