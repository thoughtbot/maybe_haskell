## Functor

To say that a type (like `Maybe`) is a `Functor`, we have to define `fmap`. The
`Functor` class definition states that it must have the following type:

```haskell
fmap :: (a -> b) -> f a -> f b
```

Where `f` is the type you're instantiating as a `Functor`, which in our case is
`Maybe`. We can see that `whenJust` has the correct type:

```haskell
--          (a -> b) -> f     a -> f     b
whenJust :: (a -> b) -> Maybe a -> Maybe b
```

Many types implement `fmap`. Another notable example is `[]` (List), it's
implementation is the `map` you're probably familiar with. If we remove some
syntactic sugar and write the type for *a list of `a`s* as `List a` rather than
`[a]`, you can confirm it too has the correct type:

```haskell
--
-- List a === [a]
--
--     (a -> b) -> f    a -> f    b
map :: (a -> b) -> List a -> List b
```
