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
class Functor Maybe where
    fmap = whenJust
```

In reality, there is no `whenJust` function; `fmap` is implemented directly:

```haskell
class Functor Maybe where
    fmap _ Nothing = Nothing
    fmap f (Just x) = Just (f x)
```

### List

The most familiar example for an implementation of `fmap` is the one for `[]`.
Like `Maybe` is the type constructor in `Maybe a`, `[]` is the type constructor
in `[a]`. You can pronounce `[]` as *list* and `[a]` as *list of a*.

The basic `map` function which exists in many languages, including Haskell, has
the following type:

```haskell
map :: (a -> b) -> [a] -> [b]
```

It takes a function from `a` to `b` and a *list of a*. It returns a *list of b*
by applying the given function to each element in the list. Knowing that `[]` is
a type constructor, and `[a]` represents applying that type constructor to some
`a`, we can write this signature in a different way; one that shows it is the
same as `fmap` when we replace the parameter `f` with `[]`:

```haskell
--     (a -> b) -> f  a -> f  b
map :: (a -> b) -> [] a -> [] b
```

This is the same process as replacing `f` with `Maybe` used to show that
`whenJust` and `fmap` were also equivalent.

`map` does exist in the Haskell Prelude (unlike `whenJust`), so the `Functor`
instance for `[]` is in fact defined in terms of it:

```haskell
instance Functor [] where
    fmap = map
```
