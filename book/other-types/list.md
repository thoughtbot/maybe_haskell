## List

At various points in the book, I relied on most programmers having an
understanding of arrays and lists of elements to ease the learning curve up to
`Maybe` and particularly `fmap`. In this chapter, I'll recap and expand on some
of the things I've said before and then show that `[a]` is more than a list of
elements over which we can map. It also has `Applicative` and `Monad` instances
that make it a natural fit for certain problems.

### Tic-Tac-Toe and the Minimax algorithm

For this chapter's example, I'm going to show portions of a program for playing
Tic-Tac-Toe. The full program is too large to include, but portions of it are
well-suited to using the `Applicative` and `Monad` instances for `[]`. The
program uses an algorithm known as [minimax][] to choose the best move to make
in a game of Tic-Tac-Toe.

[minimax]: http://en.wikipedia.org/wiki/Minimax

In short, the algorithm plays out all possible moves from the perspective of one
player and chooses the one that maximizes their score and minimizes their
opponent's, hence the name. Tic-Tac-Toe is a good game for exploring this
algorithm because the possible choices are small enough that we can take the
naive approach of enumerating all of them, then choosing the best.

To model our Tic-Tac-Toe game, we'll need some data types:

```haskell
-- A player is either Xs or Os
data Player = X | O

-- A square is either open, or taken by one of the players
data Square = Open | Taken Player

-- A row is top, middle, or bottom
data Row = T | M | B

-- A column is left, center, or right
data Column = L | C | R

-- A position is the combination of row and column
type Position = (Row, Column)

-- A space is the combination of position and square
type Space = (Position, Square)

-- Finally, the board is a list of spaces
type Board = [Space]
```

And some utility functions:

```haskell
-- Is the game over?
over :: Board -> Bool
over = undefined

-- The opponent for the given player
opponent :: Player -> Player
opponent = undefined

-- Play a space for the player in the given board
play :: Player -> Position -> Board -> Board
play = undefined
```

### Applicative

One of the things this program needs to do is generate a `Board` with all
`Square`s `Open`. We could do this directly:

```haskell
openBoard :: Board
openBoard =
    [ ((T, L), Open), ((T, C), Open), ((T, R), Open)
    , ((M, L), Open), ((M, C), Open), ((M, R), Open)
    , ((B, L), Open), ((B, C), Open), ((B, R), Open)
    ]
```

But that approach is tedious and error-prone. Another way to solve this problem
is to create an `Open` square for all combinations of `Row`s and `Column`s. We
can do exactly this with the `Applicative` instance for `[]`:

```haskell
openSpace :: Row -> Column -> Space
openSpace r c = ((r, c), Open)

openBoard :: Board
openBoard = openSpace <$> [T, M, B] <*> [L, C, R]
```

Let's walk through the body of `openBoard` to see why it gives the result we
need. First, `openSpace <$> [T, M, B]` maps the two-argument `openSpace` over
the list `[T, M, B]`. This creates a list of partially applied functions. Each
of these functions has been given a `Row` but still needs a `Column` to produce
a full `Space`. We can show this as a list of lambdas taking a `Column` and
building a `Space` with the `Row` it has already:

```haskell
(<$>) :: (a -> b) -> f a -> f b

--           a       b
openSpace :: Row -> (Column -> Space)

--                         f   b
openSpace <$> [T, M, B] :: [] (Column -> Space)

openSpace <$> [T, M, B]
-- => [ (\c -> ((T, c), Open))
-- => , (\c -> ((M, c), Open))
-- => , (\c -> ((B, c), Open))
-- => ]
```


Like the `Maybe` example from the `Applicative` chapter, we've created a
function in a context. Here we have the function `(Column -> Space)` in the `[]`
context: `[(Column -> Space)]`. Separating the type constructor from its
argument and writing  `[(Column -> Space)]` as `[] (Column -> Space)` shows how
it matches the `f b` in `(<$>)`s type signature. How do we apply a function in a
context to a value in a context? With `(<*>)`.

Using `(<*>)` with lists means applying every function to every value:

```haskell
openSpace <$> [T, M, B] <*> [L, C, R]
-- => [ (\c -> ((T, c), Open)) L        (first function applied to each value)
-- => , (\c -> ((T, c), Open)) C
-- => , (\c -> ((T, c), Open)) R
-- => , (\c -> ((M, c), Open)) L        (second function applied to each value)
-- => , (\c -> ((M, c), Open)) C
-- => , (\c -> ((M, c), Open)) R
-- => , (\c -> ((B, c), Open)) L        (third function applied to each value)
-- => , (\c -> ((B, c), Open)) C
-- => , (\c -> ((B, c), Open)) R
-- => ]
-- 
-- => [ ((T, L), Open)
-- => , ((T, C), Open)
-- => , ((T, R), Open)
-- => , ((M, L), Open)
-- => , ((M, C), Open)
-- => , ((M, R), Open)
-- => , ((B, L), Open)
-- => , ((B, C), Open)
-- => , ((B, R), Open)
-- => ]
```

### Monad and non-determinism

The heart of the minimax algorithm is playing out a hypothetical future where
each available move is made to see which one works out best. The `Monad`
instance for `[]` is perfect for this problem when we think of lists as
representing one non-deterministic value rather than a list of many
deterministic ones.

The list `[1, 2, 3]` represents a single number that is any one of `1`, `2`, or
`3` at once. The type of this value is `[Int]`. The `Int` tells us the type of
the value we're dealing with and the `[]` tells us that it's many values at
once.

Under this interpretation, `Functor`'s `fmap` represents *changing*
probabilities: we have a number that can be any of `1`, `2`, or `3`. When we
`fmap (+1)`, we get back a number that can be any of `2`, `3`, or `4`. We've
changed the non-determinism without changing *how much* non-determinism there
is. That fact, that `fmap` can't increase or decrease the non-determinism, is
actually guaranteed through the Functor laws.

```haskell
fmap (+1) [1, 2, 3]
-- => [2, 3, 4]
```

`Applicative`'s `(<*>)` can be thought of as *combining* probabilities. Given a
function that can be any of `(+1)`, `(+2)`, or `(+3)` and a number that can be
any of `1`, `2`, or `3`, `(<*>)` will give us a new number that can be any of
the combined results of applying each possible function to each possible value.

```haskell
[(+1), (+2), (+3)] <*> [1, 2, 3]
-- => [2,3,4,3,4,5,4,5,6]
```

Finally, `Monad`'s `(>>=)` is used to *expand* probabilities. Looking at its
type again:

```haskell
(>>=) :: m a -> (a -> m b) -> m b
```

And specializing this to lists:

```haskell
--       m  a -> (a -> m  b) -> m  b
(>>=) :: [] a -> (a -> [] b) -> [] b
```

We can see that it takes an `a` that can be one of many values, and a function
from `a` to `[b]`, i.e. a `b` that can be one of many values. `(>>=)` applies
the function `(a -> [b])` to every `a` in the input list. The result must be
`[[b]]`. To return the required type `[b]`, the list is then flattened. Because
the types are so generic, this is the only implementation this function can
have. If we rule out obvious mistakes like ignoring arguments and returning an
empty list, reordering the list, or adding or dropping elements, the only way to
define this function is to map, then flatten.

```haskell
xs >>= f = concat (map f xs)
```

Given our same number, one that can be any of `1`, `2`, or `3`, and a function
that takes a (deterministic) number and produces a new set of possibilities,
`(>>=)` will expand the probability space:

```haskell
next :: Int -> [Int]
next n = [n - 1, n, n + 1]

[1, 2, 3] >>= next
-- => concat (map next [1, 2, 3])
-- => concat [[1 - 1, 1, 1 + 1], [2 - 1, 2, 2 + 1], [3 - 1, 3, 3 + 1]]
-- => [0,1,2,1,2,3,2,3,4]
```

We can continue expanding by repeatedly using `(>>=)`:

```haskell
[1, 2, 3] >>= next >>= next
-- => [-1,0,1,0,1,2,1,2,3,0,1,2,1,2,3,2,3,4,1,2,3,2,3,4,3,4,5]
```

If we picture the `next` function as a step in time, going from some current
state to multiple possible next states, we can think of `>>= next >>= next` as
looking two steps into the future, exploring the possible states reachable from
possible states.

### The Future

If the theory above didn't make complete sense, that's OK. Let's get back to our
Tic-Tac-Toe program and see how this works in the context of a real-word
example.

When it's our turn (us being the computer player), we want to play out the next
turn for every move we have available. For each of those next turns, we want to
do the same thing again. We want to repeat this process until the game is over.
At that point, we can see which choice led to the best result and use that one.

One thing we'll need, and our first opportunity to use `Monad`, is to find all
available moves for a given `Board`:

```haskell
available :: Board -> [Position]
available board = do
    (position, Open) <- board

    return position
```

In this expression, we're treating a `Board` as a list of `Space`s. In other
words, it's one `Space` that is all of the spaces on the board at once. We're
using `(>>=)`, through *do-notation*, to map, then flatten, each `Space` to its
`Position`. We're using *do-notation* to take advantage of the fact that if we
use a pattern in the left-hand side of `(<-)`, but the value doesn't match the
pattern, it's discarded. This expression is a concise map-filter that relies on
`(>>=)` to do the mapping and pattern matching to do the filtering.

### Return

The `return` function, seen at the end of `available`, is not like `return`
statements you'll find in other languages. Specifically, it does not abort the
computation, presenting its argument as the return value for the function call.
Nor is it always required at the end of a monadic expression. `return` is
another function from the `Monad` type class. Its job is to take some value of
type `a` and make it an `m a`. Conceptually, it should do this by putting the
value in some default or minimal context. For `Maybe` this means applying
`Just`. For `[]`, we put the value in a singleton list:

```haskell
--        a -> m  a
return :: a -> [] a
return x = [x]
```

In our example, `position` is of type `Position` (i.e. `a`) and we need the
expression to have type `[Position]` (i.e. `m a`), so `return position` does
that.

Another `Monad`-using function of the minimax algorithm is one that expands a
given board into the end-states reached when each player plays all potential
moves:

```haskell
future :: Player -> Board -> [Board]
future player board = do
    if over board
        then return board
        else do
            space <- available board

            future (opponent player) (play player space board)
```

First we check if the `Board` is `over`. If that's the case, the future is a
singleton list of only that `Board`--again, `return board` does that.
Otherwise, we explore all available spaces. For each of them, we explore into
the future again, this time for our opponent on a `Board` where we've played
that space. This process repeats until someone wins or we fill the board in a
draw. To fit this function into our Tic-Tac-Toe-playing program, we would score
each path as we explore it and play the path with the best score.

While a full program like this is very interesting, it quickly gets complicated
with things not important to our discussion. To see a complete definition of a
minimax-using Tic-Tac-Toe-playing program written in Ruby, check out Never Stop
Building's [Understanding Minimax][minimax-post].

[minimax-post]: http://neverstopbuilding.com/minimax
