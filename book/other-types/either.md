## Either

Haskell has another type to help with computations that may fail:

```haskell
data Either a b = Left a | Right b
```

Traditionally, the `Right` constructor is used for a successful result (what a
function would have returned normally) and `Left` is used in the failure case. The
value of type `a` given to the `Left` constructor is meant to hold information
about the failure: why did it fail? This is only a convention, but it's a
strong one that we'll use throughout this chapter. To see one formalization of
this convention, take a look at [Control.Monad.Except][except]. It can appear
intimidating because it is so generalized, but [Example 1][example] should look
a lot like what I'm about to walk through here.

[except]: http://hackage.haskell.org/package/mtl-2.2.1/docs/Control-Monad-Except.html
[example]: http://hackage.haskell.org/package/mtl-2.2.1/docs/Control-Monad-Except.html#g:3

With `Maybe a`, the `a` was the value and `Maybe` was the context. Therefore, we
made instances of `Functor`, `Applicative`, and `Monad` for `Maybe` (not `Maybe
a`). With `Either` as we've written it above, `b` is the value and `Either a` is
the context, therefore Haskell has instances of `Functor`, `Applicative`, and
`Monad` for `Either a` (not `Either a b`).

This use of `Left a` to represent failure with error information of type `a` can
get confusing when we start looking at functions like `fmap`. Here's why: the
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

### ParserError

As an example, consider some kind of parser. If parsing fails, it would be nice
to include some information about what triggered the failure. To accomplish
this, we first define a type to represent this information. For our purposes,
it's the line and column where something unexpected appeared, but it could be
much richer than that including what was expected and what was seen instead:

```haskell
data ParserError = ParserError Int Int
```

From this, we can make a domain-specific type alias built on top of `Either`. We
can say a value that we parse may fail. If it does, 
error information will appear in a `Left`-constructed result. If it succeeds, we'll get the
`a` we originally wanted in a `Right`-constructed result.

```haskell
--              Either e           a    = Left e           | Right a
type Parsed a = Either ParserError a -- = Left ParserError | Right a
```

Finally, we can give an informative type to functions that may produce such results:

```haskell
parseJSON :: String -> Parsed JSON
parseJSON = undefined
```

This informs callers of `parseJSON` that it may fail and, if it does, the
invalid character and line can be found:

```haskell
jsonString = "..."

case parseJSON jsonString of
   Right json ->                 -- do something with json
   Left (ParserError ln col) ->  -- do something with the error information
```

### Functor

You may have noticed that we've reached the same conundrum as with `Maybe`: often,
the best thing to do if we encounter a `Left` result is to pass it along to our
own callers. Wouldn't it be nice if we could take some JSON-manipulating
function and apply it directly to something that we parse? Wouldn't it be nice if the
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
fmap :: (a -> b) -> Parsed   a -> Parsed   b
fmap f (Right v) = Right (f v)
fmap _ (Left e)  = Left e
```

If the value is there, we apply the given function to it. If it's not, we pass
along the error. Now we can do something like this:

```haskell
fmap (replace "admin" False) (parseJSON jsonString)
```

If the incoming string is valid, we get a successful `Parsed JSON` result with
the `"admin"` key replaced by `False`. Otherwise, we get an unsuccessful `Parsed
JSON` result with the original error message still available.

Knowing that `Control.Applicative` provides `(<$>)` as an infix synonym for
`fmap`, we could also use that to make this read a bit better:

```haskell
replace "admin" False <$> parseJSON jsonString
```

Speaking of `Applicative`...

### Applicative

It would also be nice if we could take two potentially failed results and pass
them as arguments to some function that takes normal values. If any result
fails, the overall result is also a failure. If all are successful, we get a
successful overall result. This sounds a lot like what we did with `Maybe`. The
only difference is that we're doing it for a different kind of context.

```haskell
-- Given two json objects, merge them into one
merge :: JSON -> JSON -> JSON
merge = undefined

jsonString1 = "..."

jsonString2 = "..."

merge <$> parseJSON jsonString1 <*> parseJSON jsonString2
```

`merge <$> parseJSON jsonString1` gives us a `Parsed (JSON -> JSON)`. (If this
doesn't make sense, glance back at the examples in the Applicative chapter.)
What we have is a *function* in a `Parsed` context. `parseJSON jsonString2`
gives us a `Parsed JSON`, a *value* in a `Parsed` context. The job of `(<*>)` is
to apply the `Parsed` function to the `Parsed` value and produce a `Parsed`
result.

Defining `(<*>)` starts out all right: if both values are present we'll get the
result of applying the function wrapped up again in `Right`. If the second
value's not there, that error is preserved as a new `Left` value:

```haskell
--       f        (a -> b) -> f        a -> f        b
--       Either e (a -> b) -> Either e a -> Either e b
(<*>) :: Parsed   (a -> b) -> Parsed   a -> Parsed   b
Right f <*> Right x = Right (f x)
Right _ <*> Left e = Left e
```

Astute readers may notice that we could reduce this to one pattern by using
`fmap`. This is left as an exercise.

What about the case where the first argument is `Left`? At first this seems
trivial: there's no use inspecting the second value because we know something has
already failed, so let's pass that along, right? Well, what if the second value
was also an error? Which error should we keep? Either way we discard one of
them. Any potential loss of information should be met with pause.

It [turns out][gist], it doesn't matter, at least not as far as the
Applicative Laws are concerned. If choosing one over the other had violated any
of the laws, we would have had our answer. Beyond those, we don't know how this
instance will eventually be used by end-users and we can't say which is the
"right" choice standing here now.

[gist]: https://gist.github.com/pbrisbin/b9a0c142d6ccdb8580a5

Given that the choice is arbitrary, I present the actual definition from
`Control.Applicative`:

```haskell
Left e <*> _ = Left e
```

### Monad

When thinking through the `Monad` instance for our `Parsed` type, we don't have
the same issue of deciding which error to propagate. Remember that the extra
power offered by monads is that computations can depend on the results of prior
computations. When the context involved represents failure (which may not always
be the case!), any single failing computation must trigger the omission of all
subsequent computations (since they could be depending on some result that's not
there). This means we only need to propagate that first failure.

Let's say we're interacting with a JSON web service for getting blog post
content. The responses include the body of the post as a string of HTML:

```
{
  "title": "A sweet blog post",
  "body": "<p>The post content...</p>"
}
```

Parsing JSON like this includes parsing the value at the `"body"` key into a
structured `HTML` data type. For this, we can re-use our `Parsed` type:

```haskell
parseHTML :: Value -> Parsed HTML
parseHTML = undefined
```

We can directly parse a `String` of JSON into the `HTML` present at one of its
keys by binding the two parses together with `(>>=)`:

```haskell
-- Grab the value at the given key
at :: Key -> JSON -> Value
at = undefined

parseBody :: String -> Parsed HTML
parseBody jsonString = parseJSON jsonString >>= parseHTML . at "body"
```

First, `parseJSON jsonString` gives us a `Parsed JSON`. This is the `m a` in
`(>>=)`'s type signature. Then we use `(.)` to compose a function that gets the
value at the `"body"` key and passes it to `parseHTML`. The type of this
function is `(JSON -> Parsed HTML)`, which aligns with the `(a -> m b)` of
`(>>=)`'s second argument. Knowing that `(>>=)` will return `m b`, we can see
that that's the `Parsed HTML` we're after.

If both parses succeed, we get a `Right`-constructed value containing the `HTML`
we want. If either parse fails, we get a `Left`-constructed value containing the
`ParserError` from whichever failed.

Allowing such a readable expression (*parse JSON and then parse HTML at body*),
requires the following straightforward implementation for `(>>=)`:

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
variety of functions that may fail (with or without useful error information).

This is a great way to reduce a project's maintenance burden. If you start with
functions returning `Maybe` values but use generalized functions for (e.g.) any
`Monad m`, you can later upgrade to a fully fledged `Error` type based on
`Either` without having to change most of the code base.
