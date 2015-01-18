## Operators

One potential source of confusion when seeing Haskell for the first time is the
use of *operators*. This section intends to explain these particular kinds of
functions so their use is not so surprising. Hopefully, you'll see that they are
not special built-in syntax, they're functions like anything else. All there is
to operators is that they can be used in a few special (and useful) ways.

This section is an attempt to avoid later confusion. We're going to make use of
operators throughout the rest of the book and stopping to explain them at their
first or every site of use would detract from the actual topic being covered. If
you find this section to be dense or unimportant at this time, feel free to skim
it now and refer back to it as needed when you come across operators you find
confusing later.

[The Haskell Report][report] states that any functions made up entirely of
punctuation (where "punctuation" is defined very exactly in the linked report)
can be referred to as an operator. Operators can be defined and used in all the
same ways as any other function, but have the following three additional
behaviors:

[report]: https://www.haskell.org/onlinereport/haskell2010/haskellch2.html#x7-160002.2

1. Operators are placed between their arguments

  This is called *infix notation*. It's very intuitive and looks like this:

  ```hs
  2 + 2
  -- => 4
  ```

  Functions are usually called using *prefix notation*. It's not common, but if
  you do want to call an operator using prefix notation, you have to surround it
  in parenthesis:

  ```hs
  (+) 2 2
  -- => 4
  ```

  You also have to surround operators with parenthesis if passing them as an
  argument to another function, as in this `sum` example:

  ```hs
  sum = foldl (+) 0
  ```

  Without the parenthesis, Haskell would think you were trying to add `foldl`
  and `0` which is most certainly a type error.

2. Operators can be assigned a *fixity*

  An operator's *fixity* is its [associativity][] and [precedence][] relative to
  other operators. By default, function application binds most tightly and
  associates to the left. This means the following expression:

  [associativity]: http://en.wikipedia.org/wiki/Associative_property
  [precedence]: http://en.wikipedia.org/wiki/Order_of_operations

  ```hs
  2 + 2 * 6
  ```

  would normally be parsed as

  ```hs
  (((2 +) 2) * 6)
  -- => 24
  ```

  and that would not give the correct result by the normal rules of mathematics
  which state that multiplication should occur before addition. We could
  disambiguate this with explicit parenthesis:

  ```hs
  2 + (2 * 6)
  -- => 14
  ```

  This is tedious, noisy, and it's better to say once that `(*)` binds more
  tightly than `(+)`. We'll see examples of declaring exactly that a little
  later on.

3. Operators can be used in a *section*

  If we surround an operator and one of its arguments in parenthesis, we get a
  function that will accept the missing argument. We can also choose freely
  which argument to leave out. This can be seen in the following two
  expressions:

  ```hs
  map (/ 10) [100, 200, 300]
  -- => [10, 20, 30]

  map (10 /) [10, 5, 1]
  -- => [1, 2, 10]
  ```

  There is one gotcha to be aware of: the `(-)` operator. This operator can't do
  a right section since Haskell syntax interprets `(-2)` as the number *negative
  two*. The function `subtract` is available to work around this limitation. If
  you ever need to use `(- n)`, you can use `(subtract n)` instead.

As a final note, all of the things I've described here can also be used for any
normally-named Haskell function by surrounding it in backticks:

```hs
-- Normal usage of an elem function
elem needle haystack

-- Reads a little better infix
needle `elem` hastack

-- Or as a section, leaving out the needle
intersects xs ys = any (`elem` xs) ys
```

Now that we understand what operators are, let's explore two such operators
whose importance in writing Haskell can't be overstated: `($)` and `(.)`.

### Function Application

`($)` is an operator which represents applying a function to an argument. Its
complete and deceptively simple definition is the following:

```hs
infixr 0 $

($) :: (a -> b) -> a -> b

f $ x = f x
```

First, the `infixr` statement says that this operator associates to the right
and has the lowest precedence possible.

Next, the type signature is given. Since the operator is appearing alone, we
surround it with parenthesis. It takes a function from `a` to `b` and value of
type  `a`. It returns a value of type `b`, presumably applying the first
argument to the second to produce it. It's worth noting that this is a very
reasonable presumption because that's the only definition possible. This is the
first of many benefits of Haskell's strict type system that you'll see: there is
only one valid definition for a function of this type.

Finally, we get the expected definition. Operators can appear between their
arguments even during definition which can often improve readability, as it does
here. We can see that `f $ x` *is* (remember `=` means equivalence, wherever you
see the expression on the left it *is* the expression on the right) the
application `f x`. It can be useful to pronounce expressions using this operator
with the word *of*: f of x is the application `f x`.

Technically speaking, the application `f x` should itself be pronounced "f of
x", so this definition could be read as "f of x is f of x". In fact, that's
exactly right as `($)` is an [identity][] function. If you're not convinced,
try exploring the following Haskell code:

[identity]: http://en.wikipedia.org/wiki/Identity_function

```hs
id :: a -> a
id x = x

-- Adding explicit parenthesis shows how ($)'s type is the same but specialized
-- to a specific "shape" of a
($) :: (a -> b) -> (a -> b)

-- This means we could use id as ($)
id (+2) 2
-- => 4
```

At first, it can be hard to see why such a circular definition has any use at
all. Having this function available comes with (at least) two very real
benefits:

1. We can speak precisely about function application itself

  Function application is one of the most important things you can do in
  Haskell. For this reason, the authors of the language ensured the doing so was
  free of any unnecessary syntax: no parenthesis, no commas, put the function
  name next to its argument, `f x`, that's it. This is great, but sometimes it's
  useful to name this phenomenon.

  Take the following Haskell code:

  ```hs
  -- A list of functions (using sections)
  fs = [(+1), (+2), (+3)]

  -- A list of values
  xs = [1, 2, 3]

  -- Zip the lists together by calling ($) with an element of each list as we go
  zipWith ($) fs xs
  -- => [2, 4, 6]
  ```

  Pretty neat right? Here's another example using `($)` in a section:

  ```hs
  -- A list of predicate functions (also using sections)
  ps = [even, (< 10), (> 0)]

  -- Check if a number matches all of those conditions
  check x = all ($ x) ps

  check 2  -- => True
  check 5  -- => False
  check 12 -- => False
  ```

2. We can get rid of parenthesis

  Take the following expression:

  ```hs
  take 2 drop 5 filter even [1..]
  ```

  This expression is a type error, because Haskell's default precedence
  misaligns with our intention:

  ```hs
  (((take 2) drop 5) filter even) [1..]
  ```

  Our intention was for the applications to "go the other way". We can state
  that explicitly through parenthesis and get this to compile:

  ```hs
  take 2 (drop 5 (filter even [1..]))
  -- => [12, 14]
  ```

  That looks awfully Lispy. Remember the fixity declaration given to `($)`? It
  associates to the right, and has the lowest possible precedence. That's
  exactly what we need in this case (and in fact many cases):

  ```hs
  take 2 $ drop 5 $ filter even [1..]
  -- => [12, 14]
  ```

  *take 2 of drop 5 of filter even 1..*

  Much better.

### Function Composition

`(.)` is a function which denotes *composing* two functions together. This means
to create a new function representing applying one function *after* another. As
a quick example, if `appendX` takes a string and appends an "X" on the end, and
`appendY` does the same, but with a "Y", then `appendY . appendX` can be read as
*append y after append x* and represents a function that takes a string and
appends "XY" on the end.

Its complete, and again deceptively simple, definition is as follows:

```hs
infixr 9 .

(.) :: (b -> c) -> (a -> b) -> a -> c

(.) f g = \x -> f (g x)
```

This is the definition you'll find if you look it up in the [Haskell
Prelude][prelude]. It can be a little hard to parse if you're not used to
[anonymous functions][lambda] and it doesn't take advantage of the fact that
operators can be placed between their arguments when being defined too. For
these reasons, I'd like to use an alternate but equivalent definition:

[prelude]: http://hackage.haskell.org/package/base-4.7.0.2/docs/Prelude.html#v:.
[lambda]: https://wiki.haskell.org/Anonymous_function

```hs
(f . g) x = f (g x)
```

Going back to the fixity declaration, we can see that `(.)` also associates to
the right, but that it's given the highest precedence possible. The reason
should make sense once we see how it works.

`(.)`'s type reads best if we imagine it taking two arguments and returning a
function:

```hs
(.) :: (b -> c) -> (a -> b) -> (a -> c)
```

Because of how `(->)` itself associates, the two signatures are equivalent. This
is related to an idea called *currying* which I'll talk more about later.

We can read this type as follows: given two functions, one from `b` to `c` and
the other from `a` to `b`, we get back a new function this time from `a` to `c`.
This is why such a high precedence is desirable. When you glue two functions
together with `(.)`, the result should be treated (syntactically speaking) as a
single function would.

Looking at the definition, we can see how we get from `a` to `c`: `(f . g)`,
when given an `x`, means to call `g` on `x`, then call `f` on the result of
that. If `g` is `(a -> b)` and we give its result to `f`, which is `(b -> c)`,
ultimately we have a function going all the way from `a` to `c`.

Using functions to build other functions is another frequent and important thing
to do in Haskell, so it should also be devoid of unnecessary syntax. Take the
following function for trimming whitespace from a string:

```sh
import Data.Char (isSpace)

trim s = reverse $ dropWhile isSpace $ reverse $ dropWhile isSpace s

trim "  foo bar "
-- => "foo bar"
```

Reading right to left, we first strip the trailing whitespace, then we strip the
leading whitespace by reversing the string, stripping trailing whitespace again,
then reversing it back. We immediately see that the form `reverse $ dropWhile
isSpace x` is repeated twice. We can reduce this repetition with a local
definition:

```hs
trim s = f $ f s

  where
    f x = reverse $ dropWhile isSpace x
```

We've used the tiny name `f` specifically because its meaning should be clear
from context, calling it anything else is unnecessary noise. Ruby developers are
probably twitching in their chairs at this point, but trust me -- you get used
to it. Even so, there is still quite a bit of noise in this definition.

What we really mean is that `f` is `reverse` *after* `dropWhile isSpace`.
There's no need to name an argument here; we're defining one function (`f`) as
the composition of two others (`reverse` and `dropWhile isSpace`). Beginner
Haskellers may get confused about which of `($)` or `(.)` we should use in this
case. Because we would read the function as reverse-after-drop and not
reverse-of-drop, we know that `(.)` is what we want:

```hs
trim s = f $ f s

  where
    f = reverse . dropWhile isSpace
```

Better, but still noisy. We can pull the same trick with `trim` itself:

```hs
trim = f . f

  where
    f = reverse . dropWhile isSpace
```

I would read this as "trim is f after f where f is reverse after drop-spaces".
Even though it's "full of punctuation" and uses terse variable names, I think
this Haskell code comes extremely close to expressing my intent

Functions like `trim` and `f` are known as *point-free*. That can be a source of
confusion because there's visually more "points". The reason is that the `(.)`s
are not the points we're talking about. `s` and `x` were "fixed points" of the
functions `trim` and `f` respectively (in a mathematical sense), so removing
them makes the functions point-free.

I go through all of this for two reasons:

1. Operators can be a huge source of confusion and fear in new Haskell
   developers. Hopefully this section has pre-emptively reduced that a bit.
2. In later sections, we'll be learning some new operators. I wanted to outline
   the rules ahead of time so I don't need to stop and explain it at points when
   you've already got a ton of new stuff on your plate.

Well that's it for functions, next stop: data!
