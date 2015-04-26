## Curried Form

Before moving on, I need to pause briefly and answer a question I dodged in the
Haskell Basics chapter. You may have wondered why Haskell type signatures don't
separate a function's argument types from its return type. The direct answer is
that all functions in Haskell are in *curried* form. This is an idea developed by and
named for the same [logician][] as Haskell itself.

[logician]: http://en.wikipedia.org/wiki/Haskell_Curry

A curried function is one that *conceptually* accepts multiple arguments by
actually accepting only one, but returning a function. The returned function
itself will also be curried and use the same process to accept more arguments.
This process continues for as many arguments as are needed. In short, all functions
in Haskell are of the form `(a -> b)`. A (conceptually) multi-argument function
like `add :: Int -> Int -> Int` is really `add :: Int -> (Int -> Int)`; this
matches `(a -> b)` by taking `a` as `Int` and `b` as `(Int -> Int)`.

The reason I didn't talk about this earlier is that we can mostly ignore it when
writing Haskell code. We define and apply functions as if they actually accept
multiple arguments and things work as we intuitively expect. Even partial
application (a topic I hand-waved a bit at the time) can be used effectively
without realizing this is a direct result of curried functions. It's when we dive
into concepts like `Applicative` (the focus of the next chapter) that we need to
understand a bit more about what's going on under the hood.

### The Case for Currying

In the implementation of purely functional programming languages, there is value
in having all functions taking exactly one argument and returning exactly one result.
Haskell is written this way, so users have two choices for defining
"multi-argument" functions.

We could rely solely on tuples:

```haskell
add :: (Int, Int) -> Int
add (x, y) = x + y
```

This results in the sort of type signatures you might expect, where the argument types are
shown separate from the return types. The problem with this form is that partial
application can be cumbersome. How do you add 5 to every element in a list?

```haskell
f :: [Int]
f = map add5 [1,2,3]

  where
    add5 :: Int -> Int
    add5 y = add (5, y)
```

Alternatively, we could write all functions in curried form:

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

While both forms are valid Haskell (in fact, the `curry` and `uncurry` functions
in the Prelude convert functions between the two forms), the curried version was chosen
as the default and so Haskell's syntax allows some things that make it more
convenient.

For example, we can name function arguments in whatever way we like; we don't
have to always assign a single lambda expression as the function body. In fact,
these are all equivalent:

```haskell
add = \x -> \y -> x + y
add x = \y -> x + y
add x y = x + y
```

In type signatures, `(->)` is right-associative. This means that instead of
writing:

```haskell
addThree :: Int -> (Int -> (Int -> Int))
addThree x y z = x + y + z
```

We can write the less-noisy:

```haskell
addThree :: Int -> Int -> Int -> Int
addThree x y z = x + y + z
```

And it has the same meaning.

Similarly, function application is left-associative. This means that instead of
writing:

```haskell
six :: Int
six = ((addThree 1) 2) 3
```

We can write the less-noisy:

```haskell
six :: Int
six = addThree 1 2 3
```

And it has the same meaning as well.

These conveniences are why we don't actively picture functions as curried when
writing Haskell code. We can define `addThree` naturally, as if it took three
arguments, and let the rules of the language handle currying. We can also apply
`addThree` naturally, as if it took three arguments and again the rules of the
language will handle the currying.

### Partial Application

Some languages don't use curried functions but do support *partial application*:
supplying only some of a function's arguments to get back another function that
accepts the arguments that were left out. We can do this in Haskell too, but it's not
"partial" at all, since all functions truly accept only a single argument.

When we wrote the following expression:

```haskell
maybeName = fmap userUpperName (findUser someId)
```

What really happened is that `fmap` was first applied to the function `userUpperName`
to return a new function of type `Maybe User -> Maybe String`.

```haskell
fmap :: (a -> b) -> Maybe a -> Maybe b

userUpperName :: (User -> String)

fmap userUpperName :: Maybe User -> Maybe String
```

This function is then immediately applied to `(findUser someId)` to ultimately
get that `Maybe String`. This example shows that Haskell's curried functions
blur the line between partial and total application. The result is a natural and
consistent syntax for doing either.
