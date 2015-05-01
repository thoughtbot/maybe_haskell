### Typed puzzles

Starting with the type of `main`, we immediately see something worth explaining:

```haskell
main :: IO ()
```

The type of `main` is pronounced *IO void*. `()` itself is a type defined with a
single constructor. It can also be thought of as an empty tuple:

```haskell
data () = ()
```

It's used to stand in when a computation affects the *context*, but produces no
useful *result*. It's not specific to `IO` (or monads for that matter). For
example, if you were chaining a series of `Maybe` values together using `(>>=)`
and under some condition you wanted to manually trigger an overall `Nothing`
result, you could insert a `Nothing` of type `Maybe ()` into the expression.

This is exactly how the [guard][] function works. When specialized to `Maybe`,
its definition is:

```haskell
guard :: Bool -> Maybe ()
guard True = Just ()
guard False = Nothing
```

It is used like this:

```haskell
findAdmin :: UserId -> Maybe User
findAdmin uid = do
    user <- findUser uid

    guard (isAdmin user)

    return user
```

[guard]: http://hackage.haskell.org/package/base-4.7.0.2/docs/Control-Monad.html#v:guard

If you're having trouble seeing why this expression works, start by de-sugaring
from *do-notation* to the equivalent expression using `(>>=)`, then use the
`Maybe`-specific definitions of `(>>=)`, `return`, and `guard` to reduce the
expression when an admin is found, a non-admin is found, or no user is found.

Next, let's look at the individual pieces we'll be combining into `main`:

```haskell
putStr :: String -> IO ()

putStrLn :: String -> IO ()
```

`putStr` also doesn't have any useful result so it uses `()`. It takes the given
`String` and returns an action that *represents* printing that string, without a
trailing newline, to the terminal. `putStrLn` is exactly the same, but includes
a trailing newline.

```haskell
getLine :: IO String
```

`getLine` doesn't take any arguments and has type `IO String` which means an
action that represents reading a line of input from the terminal. It requires
`IO` and presents the read line as its result.

Next, let's review the type of `(>>=)`:

```haskell
(>>=) :: m a -> (a -> m b) -> m b
```

In our case, `m` will always be `IO`, but `a` and `b` will be different each
time we use `(>>=)`. The first combination we need is `putStr` and `getLine`.
`putStr "..."` fits as `m a`, because its type is `IO ()`, but `getLine` does
not have the type `() -> IO b` which is required for things to line up. There's
another operator, built on top of `(>>=)`, designed to fix this problem:

```haskell
(>>) :: m a -> m b -> m b
ma >> mb = ma >>= \_ -> mb
```

It turns its second argument into the right type for `(>>=)` by wrapping it in a
lambda that accepts and ignores the `a` returned by the first action. With this,
we can write our first combination:

```haskell
main = putStr "..." >> getLine
```

What is the type of this expression? If `(>>)` is `m a -> m b -> m b` and we've
got `m a` as `IO ()` and `m b` as `IO String`. This combined expression must be
`IO String`. It represents an action that, *when executed*, would print the
given string to the terminal, then read in a line.

Our next requirement is to put this action together with `putStrLn`. Our current
expression has type `IO String` and `putStrLn` has type `String -> IO ()`. This
lines up perfectly with `(>>=)` by taking `m` as `IO`, `a` as `String`, and `b`
as `()`:

```haskell
main = putStr "..." >> getLine >>= putStrLn
```

This code is equivalent to the do-notation version I showed before. If you're
not sure, try to manually convert between the two forms. The steps required were
shown in the do-notation sub-section of the Monad chapter.

Hopefully, this exercise has convinced you that while I/O in Haskell may appear
confusing at first, things are quite a bit simpler:

- Any function with an `IO` type *represents* an action to be performed
- Actions are not executed, only combined into larger actions using `(>>=)`
- The only way to get the runtime to execute an action is to assign it the
  special name `main`

From these rules and the general requirement of type-safety, it emerges that any
value of type `IO a` can only be called directly or indirectly from `main`.
