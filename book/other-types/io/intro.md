## IO

So far, we've seen three types: `Maybe a`, `Either e a`, and `[a]`. These types
all represent a value with some other bit of information: a *context*. If `a` is
the `User` you're trying to find, the `Maybe` says if she was actually found. If
the `a` is the `JSON` you're attempting to parse, the `Either e` holds
information about the error when the parse fails. If `a` is a number, then `[]`
tells you it is actually many numbers at once, and how many.

For all these types, we've seen the behaviors that allow us to add them to the
`Functor`, `Applicative`, and `Monad` type classes. These behaviors obey certain
laws which allow us to reason about what will happen when we use functions like
`fmap` or `(>>=)`. In addition to this, we can also reach in and manually
resolve the context. We can define a `fromMaybe` function to reduce a `Maybe a`
to an `a` by providing a default value for the `Nothing` case. We can do a
similar thing for `Either e a` with the `either` function. Given a `[a]` we can
resolve it to an `a` by selecting one at a given index (taking care to handle
the empty list).

The `IO` type, so important to Haskell, is exactly like the three types you've
seen so far in that it represents a value in some context. With a value of type
`IO a`, the `a` is the thing you want and the `IO` means some input or output
will be performed in the real word as part of producing that `a`. The difference
is that the only way we can combine `IO` values is through their `Functor`,
`Applicative`, and `Monad` interfaces. In fact, it's really only through its
`Monad` interface since the `Applicative` and `Functor` instances are defined in
terms of it. We can't ourselves resolve an `IO a` to an `a`. This has many
ramifications in how programs must be constructed.

### Effects in a pure world

One question I get asked a lot is, "how is it that Haskell, a *pure* functional
programming language, can actually do anything? How does it create or read
files? How does it print to the terminal? How does it serve web requests?"

The short answer is, it doesn't. To show this, let's start with the following
Ruby program:

```ruby
def main
  print "give me a word: "

  x = gets

  puts x
end
```

When you run this program with the Ruby interpreter, does anything happen? It
depends on your definition of *happen*. Certainly, no I/O will happen, but
that's not *nothing*. Objects will be instantiated, and a method has been
defined. By defining this method, you've constructed a blue-print for some
actions to be performed, but then neglected to perform them.

Ruby expects (and allows) you to invoke effecting methods like `main` whenever
and wherever you want. If you want the above program to do something, you need
to call `main` at the bottom. This is a blessing and a curse. While the
flexibility is appreciated, it's a constant source of bugs and makes methods and
objects impossible to reason about without looking at their implementations. A
method may look "pure", but internally it might access a database, pull from an
external source of randomness, or fire nuclear missiles. Haskell doesn't work
like that.

Here's a translation of the Ruby program into Haskell:

```haskell
main :: IO ()
main = do
  putStr "give me a word: "

  x <- getLine

  putStrLn x
```

Much like the Ruby example, this code doesn't *do* anything. It defines a
function `main` that states what should happen when it's executed. It does not
execute anything itself. Unlike Ruby, Haskell does not expect or allow you to
call `main` yourself. The Haskell runtime will handle that and perform whatever
I/O is required for you. This is how I/O happens in a pure language: you define
the blue-print, a *pure* value that says *how* to perform any I/O, then you give
that to a separate runtime, which is in charge of actually performing it.
