## Apply

The `(<*>)` operator is pronounced *apply*. Specialized to `Maybe`, its job is
to apply a `Maybe` function to a `Maybe` value to produce a `Maybe` result.

In our example, we have `fmapUser` of type `Maybe (String -> User)` and
`aMaybeEmail` of type `Maybe String`. We're trying to use `(<*>)` to put those
together and get a `Maybe User`. We can write that down as a type signature:

```haskell
(<*>) :: Maybe (String -> User) -> Maybe String -> Maybe User
```

With such a specific type, this function won't be very useful, so let's
generalize it away from `String`s and `User`s:

```haskell
(<*>) :: Maybe (a -> b) -> Maybe a -> Maybe b
```

This function is part of the `Applicative` type class, meaning it will be
defined for many types. Therefore, its actual type signature is:

```haskell
(<*>) :: f (a -> b) -> f a -> f b
```

Where `f` is any type that has an `Applicative` instance (such as `Maybe`).

It's important to mention this because it is the type signature you're going to
see in any documentation about `Applicative`. Now that I've done so, I'm going
to go back to type signatures using `Maybe` since that's the specific instance
we're discussing here.

The semantics of our `(<*>)` function are as follows:

- If both the `Maybe` function and the `Maybe` value are present, apply the
  function to the value and return the result wrapped in `Just`
- Otherwise, return `Nothing`

We can translate that directly into code via pattern matching:

```haskell
(<*>) :: Maybe (a -> b) -> Maybe a -> Maybe b
Just f <*> Just x = Just (f x)
_      <*> _      = Nothing
```

With this definition, and a few line breaks for readability, we arrive at our
desired goal:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params = User
    <$> getParam "name" params
    <*> getParam "email" params
```

The result is an elegant expression with minimal noise. Compare *that* to the
stair-`case` we started with!

Not only is this expression elegant, it's also safe. Because of the semantics of
`fmap` and `(<*>)`, if any of the `getParam` calls return `Nothing`, our whole
`userFromParams` expression results in `Nothing`. Only if they all return `Just`
values, do we get `Just` our user.

As always, Haskell's being referentially transparent means we can prove this by
substituting the definitions of `fmap` and `(<*>)` and tracing how the
expression expands given some example `Maybe` values.

If the first value is present, but the second is not:

```haskell
User <$> Just "Pat" <*> Nothing
-- => fmap User (Just "Pat") <*> Nothing        (<$> == fmap)
-- => Just (User "Pat")      <*> Nothing        (fmap definition, first pattern)
-- => Nothing                                   (<*> definition, second pattern)
```

If the second value is present but the first is not:

```haskell
User <$> Nothing <*> Just "pat@thoughtbot.com"
-- => fmap User Nothing <*> Just "pat@thoughtbot.com"
-- => Nothing           <*> Just "pat@thoughtbot.com"   (fmap, second pattern)
-- => Nothing
```

Finally, if both values are present:

```haskell
User <$> Just "Pat" <*> Just "pat@thoughtbot.com"
-- => fmap User (Just "Pat") <*> Just "pat@thoughtbot.com"
-- => Just (User "Pat")      <*> Just "pat@thoughtbot.com"
-- => Just (User "Pat" "pat@thoughtbot.com")            (<*>, first pattern)
```
