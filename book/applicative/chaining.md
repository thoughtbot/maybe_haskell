## Chaining

One of the nice things about this pattern is that it scales up to functions that, conceptually at least, can accept any number of arguments. Imagine that our `User` type had a third
field representing their age:

```haskell
data User = User String String Int
```

Since our `getParam` function can only look up parameters with `String` values,
we'll also need a `getIntParam` function to pull the user's age out of our
`Params`:

```haskell
getIntParam :: String -> Params -> Maybe Int
getIntParam = undefined
```

With these defined, let's trace through the types of our applicative expression
again. This time, we have to remember that our new `User` function is of type
`String -> (String -> (Int -> User))`:

```haskell
User :: String -> (String -> (Int -> User))

User <$> getParam "name" params :: Maybe (String -> (Int -> User))

User <$> getParam "name" params <*> getParam "email" params :: Maybe (Int -> User)
```

This time, we arrive at a `Maybe (Int -> User)`. Knowing that `getIntParam "age"
params` is of type `Maybe Int`, we're in the exact same position as last time
when we first discovered a need for `(<*>)`. Being in the same position, we can
do the same thing again:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params = User
    <$> getParam "name" params
    <*> getParam "email" params
    <*> getIntParam "age" params
```

As our pure function (`User`) gains more arguments, we can continue to apply it
to values in context by repeatedly using `(<*>)`. The process by which this
happens may be complicated, but the result is well worth it: an expression that
is concise, readable, and above all safe.
