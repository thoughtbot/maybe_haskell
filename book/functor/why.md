## Why Is This Useful?

OK, enough theory. Now that we know how it works, let's see how it's used. Say
we have a lookup function to get from a `UserId` to the `User` for that id.
Since the user may not exist, it returns a `Maybe User`:

```haskell
findUser :: UserId -> Maybe User
findUser = undefined
```

I've left the implementation of `findUser` as `undefined` because this doesn't
matter for our example. I'll do this frequently throughout the book. `undefined`
is a function with type `a`. That allows it to stand in for any expression. If
your program ever tries to evaluate it, it will raise an exception. Still, it
can be extremely useful while developing because we can build our program
incrementally, but have the compiler check our types as we go.

Next, imagine we want to display a user's name in all capitals:

```haskell
userUpperName :: User -> String
userUpperName u = map toUpper (userName u)
```

The logic of getting from a `User` to that capitalized `String` is not terribly
complex, but it could be. Imagine something like getting from a `User` to
that user's yearly spending on products valued over $1,000. In our case the
transformation is only one function, but realistically it could be a whole suite
of functions wired together. Ideally, none of these functions should have to
think about potential non-presence or contain any "nil-checks," as that's not
their purpose; they should all be written to work on values that are fully
present.

Given `userUpperName`, which works only on present values, we can use `fmap` to
apply it to a value that may not be present to get back the result we expect
with the same level of *present-ness*:

```haskell
maybeName :: Maybe String
maybeName = fmap userUpperName (findUser someId)
```

We can do this repeatedly with every function in our system that's required to
get from `findUser` to the eventual display of this name. Because of the second
functor law, we know that if we compose all of these functions together then
`fmap` the result, or if we `fmap` any individual functions and compose the
results, we'll always get the same answer. We're free to design our system architecture as
we see fit, but still pass along the `Maybe`s everywhere we need to.

If we were doing this in the context of a web application, this maybe-name might
end up being interpolated into some HTML. It's at this boundary that we'll have
to "deal with" the `Maybe` value. One option is to use the `fromMaybe` function
to specify a default value:

```haskell
template :: Maybe String -> String
template mname = "<span class=\"username\">" ++ name ++ "</span>"

  where
    name = fromMaybe "(no name given)" mname
```
