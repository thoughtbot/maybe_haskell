### Statements and the curse of do-notation

The above Haskell function used *do-notation*. I did this to highlight that the
reason do-notation exists is for Haskell code to look like that equivalent,
imperative Ruby on which it was based. This fact has the unfortunate consequence
of tricking new Haskell programmers into thinking that `putStr` (for example) is
an imperative statement that actually puts the string to the screen when
evaluated.

In the Ruby code, each statement is implicitly combined with the next as the
interpreter sees them. There is some initial global state, statements modify
that global state, and the interpreter handles ensuring that subsequent
statements see an updated global state from all those that came before. If Ruby
used a semi-colon instead of whitespace to delimit statements, we could almost
think of `(;)` as an operator for combining statements and keeping track of the
global state between them.

In Haskell, there are no statements, only expressions. Every expression has a
type and compound expressions must be combined in a type-safe way. In the case
of `IO` expressions, they are combined with `(>>=)`. The semantic result is very
similar to Ruby's statements. It's because of this that you may hear `(>>=)`
referred to as a *programmable semicolon*. In truth, it's so much more than
that. It's a first-class function that can be passed around, built on top of,
and overloaded from type to type.

To see how this works, let's build an equivalent definition for `main`, only
this time no do-notation, only `(>>=)`.
