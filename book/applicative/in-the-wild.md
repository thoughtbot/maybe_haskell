## Applicative In the Wild

This pattern is used in a number of places in the Haskell ecosystem.

### JSON parsing

As one example, the [aeson][] package defines a number of functions for parsing
things out of JSON values. These functions return their results wrapped in a
`Parser` type. This is very much like `Maybe` except that it holds a bit more
information about *why* the computation failed, not only *that* the computation
failed. Not unlike our `getParam`, these sub-parsers pull basic types (`Int`,
`String`, etc.) out of JSON values. The `Applicative` instance for the `Parser`
type can then be used to combine them into something domain-specific, like a
`User`.

[aeson]: http://hackage.haskell.org/package/aeson

Again, imagine we had a rich `User` data type:

```haskell
data User = User
    String        -- Name
    String        -- Email
    Int           -- Age
    UTCTime       -- Date of birth
```

We can tell aeson how to create a `User` from JSON, by implementing the
`parseJSON` function. That takes a JSON object (represented by the `Value` type)
and returns a `Parser User`:

```haskell
parseJSON :: Value -> Parser User
parseJSON (Object o) = User
    <$> o .: "name"
    <*> o .: "email"
    <*> o .: "age"
    <*> o .: "birth_date"

-- If we're given some JSON value besides an Object (an Array, a Number, etc) we
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

### Option parsing

Another example is command-line options parsing via the [optparse-applicative][]
library. The process is very similar: the library exposes low-level parsers for
primitive types like `Flag` or `Argument`. Because this may fail, the values are
wrapped in another `Parser` type. (Though it behaves similarly, this is this
library's own `Parser` type, not the same one as above.) The `Applicative`
instance can again be used to combine these sub-parsers into a domain-specific
`Options` value:

[optparse-applicative]: https://github.com/pcapriotti/optparse-applicative

```haskell
-- Example program options:
-- 
-- - A bool to indicate if we should be verbose, and
-- - A list of FilePaths to operate on
-- 
data Options = Options Bool [FilePath]

parseOptions :: Parser Options
parseOptions = Options
    <$> switch (short 'v' <> long "verbose" <> help "be verbose")
    <*> many (argument (metavar "FILE" <> help "file to operate on"))
```

You can ignore some of the functions here, which were included to keep the
example accurate. What's important is that `switch (...)` is of type `Parser
Bool` and `many (argument ...)` is of type `Parser [FilePath]`. We use `(<$>)`
and `(<*>)` to put these two sub-parsers together with `Options` and end up with
an overall `Parser Options`. If we add more options to our program, all we need
to do is add more fields to `Options` and continue applying sub-parsers with
`(<*>)`.
