In the last section we saw how to use `fmap` to take a system full of functions
that operate on fully present values, free of any `nil`-checks, and employ them
to safely manipulate values that may in fact not be present. This immediately
makes many uses of `Maybe` more convenient, while still being explicit and safe
in the face of failure and partial functions.

There's another notable case where `Maybe` can cause inconvenience, one that
can't be solved by `fmap` alone. Imagine we're writing some code using a web
framework. It provides a function `getParam` that takes the name of a query
parameter (passed as part of the URL in a GET HTTP request) and returns the
value for that parameter as parsed out of the current URL. Since the parameter
you name could be missing or invalid, this function returns `Maybe`:

```haskell
getParam :: String -> Params -> Maybe String
getParam = undefined
```

Let's say we also have a `User` data type in our system. `User`s are constructed
from their name and email address, both `String`s.

```haskell
data User = User String String
```

How do we build a `User` from query params representing their name and email?

The most direct way is the following:

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
win, but this still looks a lot like the tedious, defensive coding you'd find in
any language:

```ruby
def user_from_params(params)
  if name = get_param "name" params
    if email = get_param "email" params
      return User.new(name, email)
    end
  end
end
```

## Hiding Details

So how do we do this better? What we want is code that looks as if there is no
`Maybe` involved (because that's convenient) but correctly accounts for `Maybe`
at every step along the way (because that's safe). If no `Maybe`s
were involved, and we were constructing a normal `User` value, the code might look like
this:

```haskell
userFromValues :: User
userFromValues = User aName anEmail
```

An ideal syntax would look very similar, perhaps something like this:

```haskell
userFromMaybeValues :: Maybe User
userFromMaybeValues = User <$> aMaybeName <*> aMaybeEmail
```

The `Applicative` type class and its `Maybe` instance allow us to write exactly
this code. Let's see how.
