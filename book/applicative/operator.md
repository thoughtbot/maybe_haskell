## Operators Are Not Syntax

If a function name is only symbols. You can always write it infix, no backticks
required. This is how an expression like `1 + 1` does what you would expect and
doesn't need to be written as `(+) 1 1` even though `(+)` is a function like any
other. We call functions like this *operators*.

Since the `apply` we made up reads best when written infix, it was defined as an
operator named `(<*>)` in the `Applicative` type class. Like `fmap` it is a type
class function, meaning it can be implemented by many types. There are actually
two types in that type class, the first is:

```haskell
pure :: a -> f a
```

This is relatively simple and represents taking some value and placing in a the
*minimal* or *default* context of `f`, whatever that means for the particular
type. For `Maybe`, it is implemented as wrapping the value in `Just`. We won't
be discussing this function any further since the most common way it is used can
be accomplished with `fmap`, something you already know.

The second function is our friend, `apply`:

```haskell
(<*>) :: f (a -> b) -> f a -> f b
```

Both of these come with laws, but I won't be going into them. You can read all
about them in the [module][Control.Applicative] where these functions are
defined. They're more complicated than the ones for `Functor` and understanding
them can come later. Generally speaking, you don't need to understand a type
class's laws to effectively use types in the class, only to define instances for
your own types (since then you'd have to abide by them).

[Control.Applicative]: http://hackage.haskell.org/package/base-4.7.0.1/docs/Control-Applicative.html

The same module that defines this type class also exports an operator synonym
for `fmap` named `(<$>)`. The reason for this should become clear when you see
the affect the two operators have on our example:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params = User
    <$> getParam "name" params
    <*> getParam "email" params
```

Since the functions are operators, we don't need any backticks. Since they also
have the correct fixity, parenthesis are not required. The result is an elegant
expression with minimal noise. Compare *that* to the stair-`case` we started
with!

## Hiding Details

The point of all this is to write code that looks as if there is no `Maybe`
involved (because that's convenient) but correctly accounts for `Maybe` at every
step along the way (because that's safe). If there were no `Maybe`s involved,
and we were constructing a normal `User` value, the code may look like this:

```haskell
userFromValues :: User
userFromValues = User aName anEmail
```

Compared with a `Maybe` version:

```haskell
userFromMaybeValues :: Maybe User
userFromMaybeValues = User <$> aMaybeName <*> aMaybeEmail
```

The resemblance is uncanny.
