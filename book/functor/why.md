## Why Is This Useful?

OK, enough theory. Now that we know how it works, let's see how it's used. Say
we have a lookup function to get from a `UserId` to the `User` for that id.
Since the user may not exist, it returns a `Maybe User`:

```haskell
findUser :: UserId -> Maybe User
findUser = undefined
```

Now imagine we have a view somewhere that is displaying the user's name in all
capitals:

```haskell
userUpperName :: User -> String
userUpperName u = map toUpper (userName u)
```

The logic of getting from a `User` to that capitalized `String` is not terribly
complex, but it could be -- imagine something like getting from a `User` to
their yearly spending on products valued over $1,000. In our case the
transformation is only one function, but realistically it could be a whole suite
of functions wired together. Ideally, none of these functions should have to
think about potential non-presence or contain any "nil-checks" as that's not
their purpose; they should all be written to work on values that are fully
present.

Given `userUpperName`, which works only on present values, we can use `fmap` to
apply it to a value which may not be present to get back the result we expect
with the same level of *present-ness*:

```haskell
maybeName :: Maybe String
maybeName = fmap userUpperName (findUser someId)
```

We can do this repeatedly with every function in our system that's required to
get from `findUser` to the eventual display of this name. Because of the second
functor law, we know that if we compose all of these functions together then
`fmap` the result, or if we `fmap` any individual functions and compose the
results, we'll always get the same answer. We're free to architect our system as
we see fit, but still pass along the `Maybe`s everywhere we need to.

The view template, being the only place that has to "deal with" the Maybe value,
can do so in a number of ways. Here, I'm using the `fromMaybe` function to
specify a default value of the empty string:

```haskell
widget = "<span class=\"username\">" ++ fromMaybe "" maybeName ++ "</span>"
```
