## Do Notation

There's one more topic I'd like to mention related to monads: *do-notation*.

This bit of syntactic sugar is provided by Haskell for any of its `Monad`s. The
reason is to allow monadic code to read like imperative code when composing
monadic expressions. This is valuable because monadic expressions, especially
those of type `IO a`, are often best understood as a series of imperative steps:

```haskell
f = do
    x <- something
    y <- anotherThing
    z <- combineThings x y

    finalizeThing z
```

That said, this sugar is available for any `Monad` and so we can use it for
`Maybe` as well. We can use `Maybe` as an example for seeing how *do-notation*
works. Then, if and when you come across some `IO` expressions using
*do-notation*, you won't be as surprised or confused.

De-sugaring *do-notation* is a straight-forward process followed out during
Haskell compilation and can be understood best by doing it manually. Let's start
with our end result from the last example. We'll translate this code
step-by-step into the equivalent *do-notation* form, then follow the same
process backward, as the compiler would if we had written it that way in the
first place.

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid = findUser uid >>= userZip >>= shippingCost
```

First, to make things clearer, let's add some arbitrary line breaks:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    findUser uid >>=
    userZip >>=
    shippingCost
```

Next, let's name the arguments to each expression via anonymous functions,
rather than relying on partial application and their curried nature. We'll also
add another arbitrary line break to highlight the final expression in the chain.

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    findUser uid >>= \u ->
    userZip u >>= \z ->

    shippingCost z
```

Next, we'll take each lambda and translate it into a *binding*, which looks a
bit like variable assignment and uses `(<-)`. You can read `x <- y` as "x from
y":

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    u <- findUser uid
    z <- userZip u

    shippingCost z
```

Finally, we prefix the series of "statements" with `do`:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid = do
    u <- findUser uid
    z <- userZip u

    shippingCost z
```

Et Violà, you have the equivalent *do-notation* version of our function. When
the compiler sees code written like this, it follows (mostly) the same process
we did, but in reverse:

Remove the `do` keyword:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    u <- findUser uid
    z <- userZip u

    shippingCost z
```

Translate each binding into a version using `(>>=)` and lambdas:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    findUser uid >>= \u ->
    userZip u >>= \z ->

    shippingCost z
```

The compiler can stop here as all remaining steps are stylistic changes only
(removing whitespace and *eta-reducing*[^eta-reduce] the lambdas).

[^eta-reduce]: The process of simplifying `\x -> f x` to the equivalent form `f`.

## Will it Pipe?

Both notations have their place and which to use is often up to the individual
developer, but I do have a personal guideline I can offer.

As mentioned, *do-notation* is typically useful in the `IO` monad where the
computation is probably representing a series of dependent actions to take place
in the real world. If your process is a straight pipe-line, chaining expressions
together with `(>>=)` will usually read better:

```haskell
-- Read stdin, pass it to the given function, and print the result on stdout
interact :: (String -> String) -> IO ()
interact f = getContents >>= f >>= putStr

-- vs
interact f = do
    c <- getContents

    putStr (f c)
```

If instead you find yourself manipulating one result many times, *do-notation*
is probably the way to go:

```haskell
-- Build a user instance, then execute some actions with it before returning
createUser :: Params -> IO User
createUser params = do
    user <- buildUser params

    storeInDatabase user
    sendConfirmationEmail user

    return user

-- vs (something like)
createUser params = buildUser params >>= \user ->
    storeInDatabase user >> sendConfirmationEmail user >> return user
```

Don't worry if you don't follow all of the new information here (i.e. `IO ()` or
the `(>>)` and `return` functions). These examples were only to show the
differences between *do-notation* and relying only on `(>>=)` for composing
monadic expressions.
