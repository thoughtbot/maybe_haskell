## Do Notation

There's one more topic I'd like to mention related to monads: *do-notation*.

This bit of syntactic sugar is provided by Haskell for any of its `Monad`s. The
reason is to allow functional Haskell code to read like imperative code when
building compound expressions using `Monad`. This is valuable because monadic
expressions, especially those representing interactions with the outside world,
are often read best as a series of imperative steps:

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

First, let's add some arbitrary line breaks so the eventual formatting aligns
with what someone might write by hand:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    findUser uid >>=
    userZip >>=

    shippingCost
```

Next, let's name the arguments to each expression via anonymous functions,
rather than relying on partial application and their curried nature:

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

Et voilà, you have the equivalent *do-notation* version of our function. When
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

The compiler can stop here as all remaining steps are stylistic changes only. To
get back to our exact original expression, we only need to
*eta-reduce*[^eta-reduce] the lambdas:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    findUser uid >>=
    userZip >>=

    shippingCost
```

And remove our arbitrary line breaks:


```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid = findUser uid >>= userZip >>= shippingCost
```

[^eta-reduce]: The process of simplifying `\x -> f x` to the equivalent form `f`.
