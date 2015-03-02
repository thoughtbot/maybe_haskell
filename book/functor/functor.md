## Functor

Haskell defines the type class `Functor` with a single member function, `fmap`:

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

Type constructors, like `Maybe`, implement `fmap` by defining a function where
that `f` is replaced by themselves. We can see that `whenJust` has the correct
type:

```haskell
--          (a -> b) -> f     a -> f     b
whenJust :: (a -> b) -> Maybe a -> Maybe b
```

Therefore, we could implement a `Functor` instance for `Maybe` with the
following code:

```haskell
instance Functor Maybe where
    fmap = whenJust
```

In reality, there is no `whenJust` function; `fmap` is implemented directly:

```haskell
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap _ Nothing = Nothing
```

This definition is exactly like the one we saw earlier for `whenJust`. The only
difference is we're now implementing it as part of the `Functor` instance
declaration for `Maybe`. For the rest of the book, I'll be omitting the `class`
and `instance` syntax. Instead, I'll state in prose when a function is part of
some type class but show its type and definition as if it was a normal,
top-level function.
