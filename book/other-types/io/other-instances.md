### Other instances

Unlike previous chapters, here I jumped right into `Monad`. This was because
there's a natural flow from imperative code to monadic programming with
do-notation, to the underlying expressions combined with `(>>=)`. As I
mentioned, this is the only way to combine `IO` values. While `IO` does have
instances for `Functor` and `Applicative`, the functions in these classes
(`fmap` and `(<*>)`) are defined in terms of `return` and `(>>=)` from its
`Monad` instance. For this reason, I won't be showing their definitions. That
said, these instances are still useful. If your `IO` code doesn't require the
full power of monads, it's better to use a weaker constraint. More general
programs are better; weaker constraints on what kind of data your functions
can work with makes them more generally useful.

#### Functor

`fmap`, when specialized to `IO`, has the following type:

```haskell
fmap :: (a -> b) -> IO a -> IO b
```

It takes a function and an `IO` action and returns another `IO` action, which
represents applying that function to the *eventual* result returned by the
first.

It's common to see Haskell code like this:

```haskell
readInUpper :: FilePath -> IO String
readInUpper fp = do
    contents <- readFile fp

    return (map toUpper contents)
```

All this code does is form a new action that applies a function to the eventual
result of another. We can say this more concisely using `fmap`:

```haskell
readInUpper :: FilePath -> IO String
readInUpper fp = fmap (map toUpper) (readFile fp)
```

As another example, we can use `fmap` with the Prelude function `lookup` to
write a safer version of `getEnv` from the `System.Environment` module. `getEnv`
has the nasty quality of raising an exception if the environment variable you're
looking for isn't present. Hopefully this book has convinced you it's better to
return a `Maybe` in this case. The `lookupEnv` function was eventually added to
the module, but if you intend to support old versions, you'll need to define it
yourself:

```haskell
import System.Environment (getEnvironment)

-- lookup :: Eq a => a -> [(a, b)] -> Maybe b
-- 
-- getEnvironment :: IO [(String, String)]

lookupEnv :: String -> IO (Maybe String)
lookupEnv v = fmap (lookup v) getEnvironment
```

#### Applicative

Imagine a library function for finding differences between two strings:

```haskell
data Diff = Diff [Difference]
data Difference = Added | Removed | Changed

diff :: String -> String -> Diff
diff = undefined
```

How would we run this code on files from the file system? One way, using
`Monad`, would look like this:

```haskell
diffFiles :: FilePath -> FilePath -> IO Diff
diffFiles fp1 fp2 = do
    s1 <- readFile fp1
    s2 <- readFile fp2

    return (diff s1 s2)
```

Notice that the second `readFile` does not depend on the result of the first.
Both `readFile` actions produce values that are combined *at once* using the
pure function `diff`. We can make this lack of dependency explicit and bring the
expression closer to what it would look like without `IO` values by using
`Applicative`:

```haskell
diffFiles :: FilePath -> FilePath -> IO Diff
diffFiles fp1 fp2 = diff <$> readFile fp1 <*> readFile fp2
```

As an exercise, try breaking down the types of the intermediate expressions
here, like we did for `Maybe` in the Follow the Types sub-section of the
Applicative chapter.
