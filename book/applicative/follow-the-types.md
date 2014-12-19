In the last section we saw how to use `fmap` to take a system full of functions
which operate on fully present values, free of any `nil`-checks, and employ them
to safely manipulate values which may in fact not be present. This immediately
makes many uses of `Maybe` more convenient, while still being explicit and safe
in the face of failure and partial functions.

There's another notable case where `Maybe` can cause inconvenience, one that
can't be solved by `fmap` alone. Imagine we're writing some code using a web
framework. It provides a function `getParam` which takes the name of a query
parameter (passed as part of the URL in a GET HTTP request), and returns the
value for that parameter as parsed out of the current URL. Since the parameter
you name could be missing or invalid, this function returns `Maybe`:

```haskell
-- Don't worry about how these are represented
data Params = Params

-- Or how this function works internally. All we care about is its type.
getParam :: String -> Params -> Maybe String
getParam = undefined
```

Let's say we have a `User` data type in our system. `User`s have a name and
email address, both `String`s.

```haskell
data User = User String String
```

How do we build a `User` from query params representing their name and email?

The simplest way is the following:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params =
    case getParam "name" params of
        Just name -> case getParam "email" params of
            Just email -> Just (User name email)
            Nothing -> Nothing
        Nothing -> Nothing
```

`Maybe` is not making our lives easier here. Yes, type safety is a huge implicit
win, but this still looks a lot like tedious, defensive coding you'd find in any
language:

```ruby
def user_from_params(params)
  if name = get_param "name" params
    if email = get_param "email" params
      return User.new(name, email)
    end
  end
end
```

## Follow the Types

`fmap` alone is not powerful enough to address this directly, but it's a start.
What happens when we apply `fmap` to `User`? It's not immediately clear because
`User` has the type `String -> String -> User` which doesn't line up with `(a ->
b)`. To reason about what happens, we have to remember that every function in
Haskell really only takes one argument: `User` takes a `String` and returns a
function of type `String -> User`.

```haskell
fmap :: (a -> b) -> f a -> f b

--      a      ->  b
User :: String -> (String -> User)

--           f     a      -> f      b
fmap User :: Maybe String -> Maybe (String -> User)
```

So now we have a function that takes a `Maybe String`. We happen to have one of
those. What happens when we apply `fmap User` to `getParam "name" params`?

```haskell
getParam "name" params :: Maybe String

fmap User :: Maybe String -> Maybe (String -> User)

fmap User (getParam "name") :: Maybe (String -> User)
```

Interesting. This is a common thing to do when starting out with Haskell or even
if you're experienced with Haskell but are learning a new abstraction or
library: follow the types, see what fits together. Through this process, we've
reduced things down to a smaller problem.

We have this:

```haskell
x :: Maybe (String -> User)
x = fmap User (getParam "name" params)
```

And we have this:

```haskell
y :: Maybe String
y = getParam "email" params
```

And we want this:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params = x ?+? y
```

We only have to figure out what that `?+?` should be. What it looks like we need
is some way to apply a `Maybe` function to a `Maybe` value to get a `Maybe`
result.
