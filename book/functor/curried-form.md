## What's in a Map

The definition of `fmap` for `Maybe` is simple and its usage is intuitive, but
now that we have two points of reference (`Maybe` and `[]`) for this idea of
*mapping*, we can talk about some of the more interesting aspects of what it
really means.

For example, you may have wondered why Haskell type signatures don't separate
arguments from return values. The answer to that question should help clarify
why the abstract function `fmap`, which works with far more than only lists, is
aptly named.

## Curried Form

In the implementation of purely functional programming languages, there is value
in all functions taking exactly one argument and returning exactly one result.
Therefore, users of Haskell have two choices for defining "multi-argument"
functions.

We could rely solely on tuples:

```haskell
add :: (Int, Int) -> Int
add (x, y) = x + y
```

This results in type signatures you might expect, where the argument types are
shown separate from the return types. The problem with this form is that partial
application can be cumbersome. How do you add 5 to every element in a list?

```haskell
f :: [Int]
f = map add5 [1,2,3]

  where
    add5 :: Int -> Int
    add5 x = add (x, 5)
```

Alternatively, we could write all functions in "curried" form:

```haskell
-- 
--     / One argument type, an Int
--     |
--     |       / One return type, a function from Int to Int
--     |       |
add :: Int -> (Int -> Int)
add x = \y -> x + y
--  |   |
--  |   ` One body expression, a lambda from Int to Int
--  |
--  ` One argument variable, an Int
-- 
```

This makes partial application simpler. Since `add 5` is a valid expression and
is of the correct type to pass to `map`, we can use it directly:

```haskell
f :: [Int]
f = map (add 5) [1,2,3]
```

While both forms are valid Haskell without any additional changes to the
language (in fact, the `curry` and `uncurry` functions in the Prelude will
convert functions between the two forms), the latter form is preferred and some
decisions were made regarding Haskell's syntax to better support it.

For example, we can name function arguments in whatever way we like; we don't
have to always assign a single lambda expression as the function body. In fact,
these are all equivalent:

```haskell
add = \x -> \y -> x + y
add x = \y -> x + y
add x y = x + y
```

Haskell also defines `->` to be right-associative and function application to be
left associative. That means we don't need to add any parenthesis unless we want
to explicitly group in some other way. This means rather than writing:

```haskell
addThree :: Int -> (Int -> (Int -> Int))
addThree x y z = x + y + z

six :: Int
six = ((addThree 1) 2) 3
```

We can write:

```haskell
addThree :: Int -> Int -> Int -> Int
addThree x y z = x + y + z

six :: Int
six = addThree 1 2 3
```

And that's why Haskell type signatures have the form they do.

## Partial Application

As in the `map` example above, we can partially apply functions by supplying
only some of their arguments and getting back another function which accepts any
arguments we left out. Technically, this is not "partial" at all, since all
functions really only take a single argument. In fact, this mechanism happens
even in the cases you wouldn't conceptually refer to as "partial application":

When we wrote the following expression:

```haskell
maybeName = fmap userUpperName (findUser someId)
```

What really happened is `fmap` was first applied to the function `userUpperName`
to return a new function of type `Maybe User -> Maybe String`.

```haskell
fmap :: (a -> b) -> Maybe a -> Maybe b

userUpperName :: (User -> String)

fmap userUpperName :: Maybe User -> Maybe String
```

This function is then immediately applied to `(findUser someId)` to ultimately
get that `Maybe String`.
