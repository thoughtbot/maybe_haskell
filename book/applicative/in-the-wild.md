## Applicative In the Wild

This pattern is used in a number of places in the Haskell ecosystem.

As one example, the [aeson][] package defines a number of functions for parsing
things out of JSON values, these functions return their results wrapped in a
`Parser` type which is very much like `Maybe` except that it holds a bit more
information about *why* the computation failed, not only *that* the computation
failed. The `Applicative` instance for this type can be used to combine these
sub-parsers into something domain-specific.

[aeson]: http://hackage.haskell.org/package/aeson

Again, imagine we had a rich `User` data type:

```haskell
data User = User
    String        -- Name
    String        -- Email
    Int           -- Age
    UTCTime       -- Date of birth
```

We can tell aeson how to create one of these values from JSON, by implementing
the `parseJSON` function which takes a JSON object (represented by the `Value`
type) and returns a `Parser User`:

```haskell
parseJSON :: Value -> Parser User
parseJSON (Object o) = User
    <$> o .: "name"
    <*> o .: "email"
    <*> o .: "age"
    <*> o .: "birth_date"

-- If we're given some JSON value besides an object (an array, a string, etc) we
-- can signal failure by returning the special value mzero
parseJSON _ = mzero
```

Each individual `o .: "..."` expression is a function that attempts to pull the
value for the given key out of a JSON `Object`. Potential failure (missing key,
invalid type, etc) is captured by returning a value wrapped in the `Parser`
type. We can combine the individual `Parser` values together into one `Parser
User` using `(<$>)` and `(<*>)`.

If any key is missing, the whole thing fails. If they're all there, we get the
`User` we wanted. This concern is completely isolated within the implementation
of `(<$>)` and `(<*>)` for `Parser`.
