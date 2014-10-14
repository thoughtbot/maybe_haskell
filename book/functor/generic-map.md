## Map, Not Just for Lists

Decoupling the concept of `map` such that it's useful for anything besides lists
requires looking at it though this lens of single-argument, single-result
functions. Viewed this way, the generic behavior of `map` is right there in the
name: map a key to a value.

```haskell
map :: (a -> b) -> ([a] -> [b])
--     |           |
--     |           ` The value, a function that operates on lists of as and bs
--     |
--     ` The key, a function that operates on as and bs
```

We can look at `fmap` as the same, only not specialized for lists:

```haskell
fmap :: (a -> b) -> (f a -> f b)
--      |           |
--      |           ` The value, a function that operates on f as and f bs
--      |
--      ` The key, a function that operates on as and bs
```

We can instantiate `f` as one of any number of types and get a map which takes
the functional key and maps it to a different functional value, one that
operates in that type's space.

![fmap](images/fmap.png)

Functors come from Category theory, where they represent mapping *morphisms*
from one category to another. In Haskell, *morphisms* are functions and `fmap`
maps them from the "category" of `a`s and `b`s to the "category" of `f a`s and
`f b`s. Practically speaking, this means that if you have a lot of Maybe values
in your domain, you can use `fmap` to turn all your normal functions into more
useful forms.

For a more in-depth discussion on functors, I recommend Daniel Mlot's [What Does
fmap Preserve?][what-does].

[what-does]: http://duplode.github.io/posts/what-does-fmap-preserve.html
