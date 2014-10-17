## More Power

We can't clean this up with `(<*>)` because each individual part of an
`Applicative` expression doesn't have access to the results from any other
part's evaluation. What does that mean? If we look at the `Applicative`
expression from before:

```haskell
User <$> getParam "name" params <*> getParam "email" params
```

Here, the two results from `getParam "name"` and `getParam "email"` (either of
which could be present or not) are passed together to `User`. If they're both
present we get a `Just User`, otherwise `Nothing`. Within the `getParam "email"`
expression, you can't make reference to the (potential) result of `getParam
"name"`.

We need that ability to solve our current conundrum because we need to check the
value of the "type" param to know what to do next. We need... *monads*.

## And Then?

Let's start with a minor refactor; let's pull out a `loginByType` function:

```haskell
loginUser :: Params -> Maybe User
loginUser params = case getParam "type" params of
    Just t -> loginByType params t
    Nothing -> Nothing

loginByType :: Params -> String -> Maybe User
loginByType params "username" = case getParam "username" params of
    Just u -> findUserByUserName u
    Nothing -> Nothing

loginByType params "email" = case getParam "email" params of
    Just e -> findUserByEmail e
    Nothing -> Nothing

loginByType _ _ = Nothing
```

Things seem to be following a pattern now: we have some value that might not be
present and some function that needs the (fully present) value, does some other
computation with it, but may itself fail.

Let's abstract this concern into a new function called `andThen`:


```haskell
andThen :: Maybe a -> (a -> Maybe b) -> Maybe b
andThen (Just x) f = f x
andThen _ _ = Nothing
```

We'll use the function infix via backticks for readability:

```haskell
loginUser :: Params -> Maybe User
loginUser params =
    getParam "type" params `andThen` loginByType params

loginByType :: Params -> String -> Maybe User
loginByType params "username" =
    getParam "username" params `andThen` findUserByUserName

loginByType params "email" =
    getParam "email" params `andThen` findUserByEmail

-- Still needed in case we get an invalid type
loginByType _ _ = Nothing
```

This cleans things up nicely. The "passing along the `Maybe`" concern is
completely abstracted away behind `andThen` and we're free to describe the
nature of *our* computation only. If only Haskell had such a function...
