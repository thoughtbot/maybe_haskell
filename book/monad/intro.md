So far, we've seen that as `Maybe` makes our code safer, it also makes it less
convenient. By making potential non-presence explicit, we now need to correctly
account for it at every step. We addressed a number of scenarios by using `fmap`
to "upgrade" a system full of normal functions (free of any `nil`-checks) into
one that can take and pass along `Maybe` values. When confronted with a new
scenario that could not be handled by `fmap` alone, we discovered a new function
`(<*>)` which helped ease our pain again. This chapter is about addressing a
third scenario, one that `fmap` and even `(<*>)` cannot solve: dependent
computations.

Let's throw a monkey wrench into our `getParam` example from earlier. This time,
let's say we're accepting logins by either username or email. The user can say
which method they're using by passing a `type` param specifying "username" or
"email".

*Note*: this whole thing is wildly insecure, but bear with me.

Again, all of this is fraught with `Maybe`-ness and again, writing it with
straight-line `case` matches can get very tedious:

```haskell
loginUser :: Params -> Maybe User
loginUser params = case getParam "type" of
    Just t -> case t of
      "username" -> case getParam "username" of
          Just u -> findUserByUserName u
          Nothing -> Nothing
      "email" -> case getParam "email" of
          Just e -> findUserByEmail e
          Nothing -> Nothing
      _ -> Nothing
    Nothing -> Nothing
```

Yikes.
