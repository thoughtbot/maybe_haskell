When we declare a function in Haskell, we first write a type signature:

```haskell
five :: Int
```

We can read this as `five` *of type* `Int`.

Next, we write a definition:

```haskell
five = 5
```

We can read this as `five` *is* `5`.

In Haskell, `=` is not variable assignment, it's defining equivalence. We're
saying here that the word `five` *is equivalent to* the literal `5`. Anywhere
you see one, you can replace it with the other and the program will always give
the same answer. This property is called *referential transparency* and it holds
true for any Haskell definition, no matter how complicated.

It's also possible to specify types with an *annotation* rather than a
signature. We can annotate any expression with `:: <type>` to explicitly tell
the compiler the type we want (or expect) that expression to have.

```haskell
almostThird = (3 :: Float) / 9
-- => 0.3333334

actualThird = (3 :: Rational) / 9
-- => 1 % 3
```

We can read these as `almostThird` is `3`, *of type* `Float`, divided by `9` and
`actualThird` is `3`, *of type* `Rational`, divided by `9`.

Type annotations and signatures are usually optional, as Haskell can almost
always tell the type of an expression by inspecting the types of its constituent
parts or seeing how it is eventually used. This process is called *type
inference*. For example, Haskell knows that `six` is an `Int` because it saw
that `5` is an `Int`. Since you can only use `(+)` with arguments of the same
type, it *enforced* that `1` is also an `Int`. Knowing that `(+)` returns the
same type as its arguments, the final result of the addition must itself be an
`Int`.

Good Haskellers will include a type signature on all top-level definitions
anyway. It provides executable documentation and may, in some cases, prevent
errors which occur when the compiler assigns a more generic type than you might
otherwise want.

### Arguments

Defining functions that take arguments looks like this:

```haskell
add :: Int -> Int -> Int
add x y = x + y
```

The type signature can be confusing because the argument types are not separated
from the return type. There is a good reason for this, but I won't go into it
yet. For now, feel free to mentally treat the thing after the last arrow as the
return type.

After the type signature, we give the function's name (`add`) and names for any
arguments it takes (`x` and `y`). On the other side of the `=`, we define an
expression using those names.

### Higher-order functions

Functions can take and return other functions. These are known as
[higher-order][] functions. In type signatures, any function arguments or return
values must be surrounded by parentheses:

[higher-order]: http://learnyouahaskell.com/higher-order-functions

```haskell
twice :: (Int -> Int) -> Int -> Int
twice f x = f (f x)

twice (add 2) 3
-- => 7
```

`twice` takes as its first argument a function, `(Int -> Int)`. As its second
argument, it takes an `Int`. The body of the function applies the first argument
(`f`) to the second (`x`) twice, returning another `Int`. The parentheses in the
definition of `twice` are grouping, not application. In Haskell, applying a
function to some argument is simple: stick them together with a space in
between. In this case, we need to group the inner `(f x)` so the outer `f` is
applied to it as single argument. Without these parentheses, Haskell would think
we were applying `f` to two arguments: another `f` and `x`.

You also saw an example of *partial application*. The expression `add 2` returns
a new function that itself takes the argument we left off. Let's break down that
last expression to see how it works:

```haskell
-- Add takes two Ints and returns an Int
add :: Int -> Int -> Int
add x y = x + y

-- Supplying only the first argument gives us a new function that will add 2 to
-- its argument. Its type is Int -> Int
add 2 :: Int -> Int

-- Which is exactly the type of twice's first argument
twice :: (Int -> Int) -> Int -> Int
twice f x = f (f x)

twice (add 2) 3
-- => add 2 (add 2 3)
-- => add 2 5
-- => 7
```

It's OK if this doesn't make complete sense now, I'll talk more about partial
application as we go.

### Operators

In the definition of `add`, I used something called an *operator*: `(+)`.
Operators like this are not special or built-in in any way; we can define and
use them like any other function. That said, there are three additional (and
convenient) behaviors operators have:

1. They are used *infix* by default, meaning they appear between their arguments
   (i.e. `2 + 2`, not `+ 2 2`). To use an operator *prefix*, it must be
   surrounded in parentheses (as in `(+) 2 2`).
2. When defining an operator, we can assign a custom [associativity][] and
   [precedence][] relative to other operators. This tells Haskell how to group
   expressions like `2 + 3 * 5 / 10`.
3. We can surround an operator and *either* of its arguments in parentheses to
   get a new function that accepts whichever argument we left off. Expressions
   like `(+ 2)` and `(10 /)` are examples. The former adds `2` to something and
   the latter divides `10` by something. Expressions like these are called
   *sections*.

[associativity]: http://en.wikipedia.org/wiki/Associative_property
[precedence]: http://en.wikipedia.org/wiki/Order_of_operations

In Haskell, any function with a name made up entirely of punctuation (where [The
Haskell Report][report] states very exactly what "punctuation" means) behaves
like an operator. We can also take any normally-named function and treat it like
an operator by surrounding it in backticks:

[report]: https://www.haskell.org/onlinereport/haskell2010/haskellch2.html#x7-160002.2

```haskell
-- Normal usage of an elem function for checking if a value is present in a list
elem 3 [1, 2, 3, 4, 5]
-- => True

-- Reads a little better infix
3 `elem` [1, 2, 3, 4, 5]
-- => True

-- Or as a section, leaving out the first argument
intersects xs ys = any (`elem` xs) ys
```

### Lambdas

The last thing we need to know about functions is that they can be *anonymous*.
Anonymous functions are called *lambdas* and are most frequently used as
arguments to higher-order functions. Often these functional arguments only exist
for a single use and giving them a name is not otherwise valuable.

The syntax is a back-slash, the arguments to the function, an arrow, then the
body of the function. A back-slash is used because it looks like the Greek
letter λ.

Here's an example:

```haskell
twice (\x -> x * x + 10) 5
-- => 1235
```

If you come across a code example using a lambda, you can always rewrite it to
use named functions. Here's the process for this example:

```haskell
-- Grab the lambda
\x -> x * x + 10

-- Name it
f = \x -> x * x + 10

-- Replace "\... ->" with normal arguments
f x = x * x + 10

-- Use the name instead
twice f 5
```
