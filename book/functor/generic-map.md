## Map, Not Just for Lists

Decoupling the concept of `map` such that it's useful for anything besides lists
requires looking at it through this lens of single-argument, single-result
functions. Viewed this way, the generic behavior of `map` is right there in the
name: map a key to a value.

Take another look at the type signature of `map`, this time with explicit
parentheses around the return type:

```haskell
map :: (a -> b) -> ([a] -> [b])
```

The "key" is a function from `a` to `b`. The "value" is a function from `[a]` to
`[b]`. It's common but incorrect to say "map a function over each element of a
list." Because the `map` function most commonly found in programming languages
applies a function to each element in a list, we've taken the word "map" to mean
"do something to each element in a list." This is unfortunate as taking a
function that operates in some domain (`a`s and `b`s) and mapping it to a
function that operates in a different, related domain (`[a]`s and `[b]`s) is far
more accurate and eases the mental leap to the generalized version, `fmap`:

```haskell
fmap :: (a -> b) -> (f a -> f b)
```

Here, we're still "mapping", but this time from a function whose domain is `a`s
And `b`s to one whose domain is `f a`s and `f b`s. We make no mention of "each
element of a list" because that's an implementation detail not at all related to
`fmap`. By instantiating `f` as any number of types, we can take all our normal
functions and map them to functions that operate on whole new sets of values.
