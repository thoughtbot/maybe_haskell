## The Functor Laws

Type class laws are a formal way of defining what it means for implementations
to be "well-behaved". If someone writes a library function and says it can work
with "any `Functor`", that code can rely both on that type having an `fmap`
implementation, and that it operates in accordance with these laws.

For example, let's look at the first Functor law:

```haskell
fmap id x == id x
-- 
-- for any value x, of type Maybe a
-- 
```

Where `id` is the *identity* function, one which returns whatever you give it:

```haskell
id :: a -> a
id x = x
```

Since pure functions always give the same result given the same input, it's
equally correct to say that the functions themselves must be equivalent, rather
than applying them to "any `x`" and saying the results must be equivalent. For
this reason, the laws are usually stated as:

```haskell
fmap id == id
```

This law says that if we call `fmap id`, the function we get back should be
equivalent to `id` itself. This is what "well behaved" means in this context. If
you think about `fmap` for `[]`, you would expect that applying `id` to every
element in the list (as `fmap id` does) gives you back the same exact list, and
that is exactly what you expect to get if you apply `id` directly to the list
itself. I encourage you to go through the same thought exercise for `Maybe` so
you can see that the law holds true for its implementation as well.

The second law has to do with order of operations, it states:

```haskell
fmap (f . g) == fmap f . fmap g
```

Where `(.)` is a function that takes two functions and *composes* them together:

```haskell
(.) :: (b -> c) -> (a -> b) -> a -> c
(f . g) x = f (g x)
```

What this law says is that if we compose two functions together, then `fmap` the
resulting function, we should get a function which behaves the same as when we
`fmap` each function individually, then compose the two results.

This one's a little trickier when you're not familiar with function composition.
Let's prove that this law holds for `Maybe` by walking through an example with
`actuallyFive` and `notReallyFive`. If you already feel comfortable with what
the law is stating and why it holds, feel free to skip this section.

First, lets define two concrete functions, `f` and `g`

```haskell
f :: Int -> Int
f = (+2)

g :: Int -> Int
g = (+3)
```

We can *compose* these two functions to get a new function, and call that `h`:

```haskell
h :: Int -> Int
h = f . g
```

Given the definition of `(.)`, this is equivalent to:

```haskell
h :: Int -> Int
h x = f (g x)
```

This new function takes a number and gives it to `(+3)`, then it takes the
result and gives it to `(+2)`. The result is a function that will add 5 to its
argument.

```haskell
h 5
-- => 10
```

We can give this function to `fmap` to get one that works with `Maybe` values:

```haskell
fmap h actuallyFive
-- => Just 10

fmap h notReallyFive
-- => Nothing
```

Similarly, we can give `f` and `g` to `fmap` to produce functions which can add
2 or 3 to a `Maybe Int` to produce another `Maybe Int`. The resulting functions
can also be composed with `(.)` to produce a new function, `fh`:

```haskell
fh :: Maybe Int -> Maybe Int
fh = fmap f . fmap g
```

Again, given the definition of `(.)`, this is equivalent to:

```haskell
fh :: Maybe Int -> Maybe Int
fh x = fmap f (fmap g x)
```

This function will call `fmap g` on its argument which will add 3 if the
number's there or return `Nothing` if it's not, then give that result to `fmap
f` which will add 2 if the number's there, or return `Nothing` if it's not. This
results in a function which will add 5 to a number if it's there, or return
`Nothing` if it's not:

```haskell
fh actuallyFive
-- => Just 10

fh notReallyFive
-- => Nothing
```

You should convince yourself that `fh` and `fmap h` behave in exactly the same
way. The second functor law states that this must be the case if your type is a
valid `Functor`. Because Haskell is referentially transparent, we can replace
functions with their implementations freely -- it may require some explicit
parenthesis here and there, but the code will always give the same answer. Doing
so brings us back directly to the statement of the second law:

```haskell
(fmap f . fmap g) actuallyFive
-- => Just 10

fmap (f . g) actuallyFive
-- => Just 10

(fmap f . fmap g) notReallyFive
-- => Nothing

fmap (f . g) notReallyFive
-- => Nothing

-- Therefore:
fmap (f . g) == fmap f . fmap g
```

Not only can we take normal functions (those which operate on fully present
values) and give them to `fmap` to get ones that can operate on `Maybe`, but
this law states we can do so in any order. We can compose our system of
functions together *then* give that to `fmap` or we can `fmap` individual
functions and compose *those* together -- either way, we're guaranteed the same
result. We can rely on this fact whenever we use `fmap` for any type that's in
the `Functor` type class.
