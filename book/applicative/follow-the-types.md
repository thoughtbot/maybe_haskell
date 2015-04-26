## Follow The Types

We can start by trying to do what we want with the only tool we have so far:
`fmap`.

What happens when we apply `fmap` to `User`? It's not immediately clear because
`User` has the type `String -> String -> User` which doesn't line up with `(a ->
b)`. Fortunately, it only *appears* not to line up. Remember, every function in
Haskell takes one argument and returns one result: `User`'s actual type is
`String -> (String -> User)`. In other words, it takes a `String` and returns a
function, `(String -> User)`. In this light, it indeed lines up with the type
`(a -> b)` by taking `a` as `String` and `b` as `(String -> User)`.

By substituting our types for `f`, `a`, and `b`, we can see what the type of
`fmap User` is:

```haskell
fmap :: (a -> b) -> f a -> f b

--      a      ->  b
User :: String -> (String -> User)

--           f     a      -> f      b
fmap User :: Maybe String -> Maybe (String -> User)
```

So now we have a function that takes a `Maybe String` and returns a 
`Maybe (String -> User)`. We also have a value of type `Maybe String` that we can give
to this function, `getParam "name" params`:

```haskell
getParam "name" params             :: Maybe String

fmap User                          :: Maybe String -> Maybe (String -> User)

fmap User (getParam "name" params) ::                 Maybe (String -> User)
```

The `Control.Applicative` module exports an operator synonym for `fmap` called
`(<$>)` (I pronounce this as *fmap* because that's what it's a synonym for). The
reason this synonym exists is to get us closer to our original goal of making
expressions look as if there are no `Maybe`s involved. Since operators are
placed between their arguments, we can use `(<$>)` to rewrite our
expression above to an equivalent one with less noise:

```haskell
User <$> getParam "name" params :: Maybe (String -> User)
```

This expression represents a "`Maybe` function". We're accustomed to *values* in a
context: a `Maybe Int`, `Maybe String`, etc; and we saw how these were
*functors*. In this case, we have a *function* in a context: a `Maybe (String ->
User)`. Since functions are things that *can be applied*, these are called
*applicative functors*.

By using `fmap`, we reduced our problem space and isolated the functionality
we're lacking, functionality we'll ultimately get from `Applicative`:

We have this:

```haskell
fmapUser :: Maybe (String -> User)
fmapUser = User <$> getParam "name" params
```

And we have this:

```haskell
aMaybeEmail :: Maybe String
aMaybeEmail = getParam "email" params
```

And we're trying to ultimately get to this:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params = fmapUser <*> aMaybeEmail
```

We only have to figure out what that `(<*>)` should do. At this point, we have
enough things defined that we know exactly what its type needs to be. In the
next section, we'll see how its type pushes us to the correct implementation.
