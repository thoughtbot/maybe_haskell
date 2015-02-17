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
six = (5 :: Int) + 1
```

Type annotations and signatures are usually optional, as Haskell can almost
always tell the type of an expression by inspecting the types of its constituent
parts or seeing how it is eventually used. For example, Haskell knows that `six`
is an `Int` because it saw that `5` is an `Int`. Since you can only use `(+)`
with arguments of the same type, it *enforced* that `1` is also an `Int`.
Knowing that `(+)` returns the same type as its arguments, the final result of
the addition must itself be an `Int`.

Good Haskellers will include a type signature on all top-level definitions
anyway. It provides executable documentation and may, in some cases, prevent
errors which occur when the compiler assigns a more generic type than you might
otherwise want. For example, if we omitted the type signatures in our first
example, the compiler would assign the type `five :: Num a => a` which means
that the type of `five` is any type that's `Num`eric in nature -- i.e. it can
be added, negated and so forth.

*Inferred types*, as these are called, are usually fine but can open you up to
unclear error messages. If you were to take the result of our generalized `five`
and use it in two places, treating it as an `Int` in one and a `Float` in
another, the error message may not immediately identify the problem depending on
which expression the compiler sees first. On the other hand, if you explicitly
state the type as whichever one you want it to be, then using it incorrectly
will produce an error message leading you directly to the line and character of
your error.
