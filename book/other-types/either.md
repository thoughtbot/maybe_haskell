The three abstractions you've seen all require a certain kind of value.
Specifically, a value with some other bit of information, often referred to as
its *context*. In the case of a type like `Maybe a`, the `a` represents the
value itself and `Maybe` represents the fact that it may or may not be present.
This potential non-presence is that other bit of information, its context.

Haskell's type system is unique in that it lets us speak specifically about this
other bit of information without involving the value itself. In fact, when
defining instances for `Functor`, `Applicative` and `Monad`, we were defining an
instance for `Maybe`, not for `Maybe a`. When we define these instances we're
not defining how `Maybe a`, a value in some context, behaves under certain
computations, we're actually defining how `Maybe`, the context itself, behaves
under certain operations.

This kind of separation of concerns is difficult to understand when you're only
accustomed to languages that don't allow for it. I believe it's why topics like
monads seem so opaque to those unfamiliar with a type system like this. To
strengthen the point that what we're really talking about are behaviors and
contexts, not any one specific *thing*, this chapter will explore types which
represent other kinds of contexts and show how they behave under all the same
computations we saw for `Maybe`.

## Either

Haskell has another type to help with computations that may fail:

```haskell
data Either a b = Left a | Right b
```

Traditionally, the `Right` constructor is used for a successful result (what a
function would've returned normally) and `Left` is used in the failure case. The
value of type `a` given to the `Left` constructor is meant to hold information
about the failure -- i.e. why did it fail? This is only convention, but it's a
strong one that we'll use throughout this chapter. To see one formalization of
this convention, take a look at [Control.Monad.Except][except]. It can appear
intimidating because it is so generalized, but [Example 1][example] should look
a lot like what I'm about to walk through here.

[except]: http://hackage.haskell.org/package/mtl-2.2.1/docs/Control-Monad-Except.html
[example]: http://hackage.haskell.org/package/mtl-2.2.1/docs/Control-Monad-Except.html#g:3

With `Maybe`, the `a` was the value and `Maybe` was the context. Therefore, we
made instances of `Functor`, `Applicative`, and `Monad` for `Maybe` (not `Maybe
a`). With `Either` as we've written it above, `b` is the value and `Either a` is
the context, therefore Haskell has instances of `Functor`, `Applicative`, and
`Monad` for `Either a` (not `Either a b`).

This use of `Left a` to represent failure with error information of type `a` can
get confusing when we start looking at functions like `fmap` since the
generalized type of `fmap` talks about `f a` and I said our instance would be
for `Either a` making that `Either a a`, but they aren't the same `a`!

For this reason, we can imagine an alternate definition of `Either` that uses
different variables. This is perfectly reasonable since the variables are chosen
arbitrarily anyway:

```haskell
data Either e a = Left e | Right a
```

When we get to `fmap` (and others), things are clearer:

```haskell
--      (a -> b)    f        a -> f        b
fmap :: (a -> b) -> Either e a -> Either e b
```

## ParserError

As an example, consider some kind of parser. If the parser fails, it would be
nice to include the line and character that triggered the failure. To accomplish
this, we first define a type to represent information about the failure. For our
purposes, we'll say it's the line and character where something unexpected
appeared, but it could be much richer than that including what was expected and
what was seen instead:

```haskell
data ParserError = ParserError Int Int
```

From this, we can make a domain-specific type alias built on top of `Either`. We
can say that a value which we parse may fail, and if it does, there will be
error information in a `Left`-constructed result. If it succeeds, we'll get the
`a` we originally wanted in a `Right`-constructed result.

```haskell
--              Either e           a    = Left e           | Right a
type Parsed a = Either ParserError a -- = Left ParserError | Right a
```

Finally, we can give functions that may produce such results an informative
type:

```haskell
parseJSON :: String -> Parsed JSON
parseJSON = undefined
```

This informs callers of `parseJSON` that it may fail and, if it does, the
invalid character and line can be found:

```haskell
jsonString = "..."

case parseJSON jsonString of
   Right json ->                -- do something with json
   Left (ParserError ln ch) ->  -- do something with the error information
```

### Functor

You may have noticed that we've reached the same conundrum as `Maybe`: often,
the best thing to do if we encounter a `Left` result is to pass it along to our
own callers. Wouldn't it be nice if we could take some JSON-manipulating
function and apply it directly to something we parse? Wouldn't it be nice if the
"pass along the errors" concern were handled separately?

```haskell
-- Replace the value at the given key with the new value
replace :: Key -> Value -> JSON -> JSON
replace = undefined

-- 
-- This is a type error!
-- 
--  replace "admin" False is (JSON -> JSON), but parseJSON returns (Parsed JSON)
-- 
replace "admin" False (parseJSON jsonString)
```

`Parsed a` is a value in some context, like `Maybe a`. This time, rather than
only present-or-non-present, the context is richer. It represents
present-or-non-present-with-error. Can you think of how this context should be
accounted for under an operation like `fmap`?

```haskell
--      (a -> b) -> f        a -> f        b
--      (a -> b) -> Either e a -> Either e b
fmap :: (a -> b) -> Parsed a   -> Parsed   b
fmap f (Right v) = Right (f v)
fmap _ (Left e)  = Left e
```

If the value is there, we apply the given function to it. If its not, we pass
along the error. Now we can do something like this:

```haskell
fmap (replace "admin" False) (parseJSON jsonString)
```

If the incoming string is valid, we get a successful `Parsed JSON` result with
the `"admin"` key replaced by `False`. Otherwise, we get an unsuccessful `Parsed
JSON` result with the original error message still available.

Knowing that `Control.Applicative` provides `(<$>)` is an infix synonym for
`fmap`, we could also use that to make this read a bit better:

```haskell
replace "admin" False <$> parseJSON jsonString
```

Speaking of `Applicative`...

### Applicative

It would also be nice if we could take two potentially failed results and pass
them as arguments to some function that takes normal values. If any result
fails, the overall result is also a failure. If all are successful, we get a
successful overall result. This sounds a lot like what we did with `Maybe`, the
only difference is we're doing it for a different kind of context.

```haskell
-- Given two json objects, merge them. Duplicate keys result in those in the
-- second object being kept.
merge :: JSON -> JSON
merge = undefined

jsonString1 = "..."

jsonString2 = "..."

merge <$> parseJSON jsonString1 <*> parseJSON jsonString2
```

Defining `(<*>)` starts out all right: if both values are present we'll get the
result of applying the function wrapped up again in `Right`. If the second
value's not there, that error is preserved as a new `Left` value:

```haskell
--       f        (a -> b) -> f        a -> f        b
--       Either e (a -> b) -> Either e a -> Either e b
(<*>) :: Parsed   (a -> b) -> Parsed   a -> Parsed   b
Right _ <*> Left e = Left e
```

Astute readers may notice that we could reduce this to one pattern by using
`fmap` -- this is left as an exercise.

What about the case where the first argument is `Left`? At first this seems
trivial: there's no use inspecting the second value, we know something has
already failed so let's pass that along, right? Well, what if the second value
was also an error? Which error should we keep? Either way we discard one of
them, and any potential loss of information should be met with pause.

It [turns out][gist], it doesn't matter -- at least not as far as the
Applicative Laws are concerned. If choosing one over the other had violated any
of the laws, we would've had our answer. Beyond those, we don't know how this
instance will eventually be used by end-users and we can't say which is the
"right" choice standing here now.

[gist]: https://gist.github.com/pbrisbin/b9a0c142d6ccdb8580a5

Given that the choice is arbitrary, I present the actual definition from
`Control.Applicative`:

```haskell
Left e <*> _ = Left e
```

## Monad

When thinking through the `Monad` instance for our `Parsed` type, we don't have
the same issue of deciding which error to propagate. Remember that the extra
power offered by monads is that computations can depend on the results of prior
computations. When the context involved represents failure (which may not always
be the case!), any single failing computation must trigger the omission of all
subsequent computations (since they could be depending on some result that's not
there). This means we only need to propagate that first failure.

Let's say that our domain has some JSON responses whose values contain HTML.
Parsing these values into an `HTML` data type may also fail. Since our `Parsed`
type is polymorphic in its result (i.e. it's `Parsed a` not `Parsed JSON`), we
can reuse it here:

```haskell
parseHTML :: Value -> Parsed HTML
parseHTML = undefined
```

We can directly parse a `String` of JSON into the HTML present at one of its
keys by binding the two parses together with `(>>=)`:

```haskell
-- Grab the value at the given key
at :: Key -> JSON -> Value
at = undefined

parseBody :: String -> Parsed HTML
parseBody jsonString = parseJSON jsonString >>= parseHTML . at "body"
```

First, `parseJSON jsonString` gives us a `Parsed JSON`. This is the `m a` in
`(>>=)`'s type signature. Then we use `(.)` to compose a function for getting
the value at the `"body"` key and passing it to `parseHTML`. The type of this
function is `(JSON -> Parsed HTML)` which aligns with the `(a -> m b)` of
`(>>=)`'s second argument. Knowing that `(>>=)` will return `m b`, we can see
that that's the `Parsed HTML` we're after.

If both parses succeed, we get a `Right`-constructed value containing the `HTML`
we want. If either parse, fails we get a `Left`-constructed value containing the
`ParserError` from whichever failed.

Allowing such a readable expression (*parse JSON and then parse HTML at body*),
requires the following straight-forward implementation for `(>>=)`:

```haskell
--       m        a -> (a -> m        b) -> m        b
--       Either e a -> (a -> Either e b) -> Either e b
(>>=) :: Parsed   a -> (a -> Parsed   b) -> Parsed   b
Right v >>= f = f v
Left e >>= _ = Left e
```

Armed with instances for `Functor`, `Applicative`, and `Monad` for both `Maybe`
and `Either e`, we can use the same set of functions (those with `Functor f`,
`Applicative f` or `Monad m` in their class constraints) and apply them to a
variety of functions which may fail (with or without useful error information).

This is a great way to reduce a project's maintenance burden: if you start with
functions returning `Maybe` values but use generalized functions for (e.g.) any
`Monad m`, you can later upgrade to a fully fledged `Error` type based on
`Either` without having to change most of the code base.
