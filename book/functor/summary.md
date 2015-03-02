## Recap

So far, we've seen an introduction to Haskell functions and its type system, a
new and powerful way to use that type system to describe something about your
domain (that some values may not be present), and a type class (`Functor`) that
allows for strict separation between value-handling functions and the need to
apply them to values that may not be present.

We then saw some real-word code that takes advantage of these ideas and
discussed type class laws as a means of abstraction and encapsulation: they give
us a precise understanding of how our code will behave without having to know
its internals. Finally, we took a brief detour into the world of currying, a
foundational concept responsible for many of the things we'll explore next.

In the next chapter, we'll talk about *applicative functors*. If we think of a
*functor* as a value in some context, supporting an `fmap` operation for
applying a function to that value while preserving its context, *applicative
functors* are functors where the value itself *can be applied*, i.e. it's a
function. These structures must then support another operation for applying that
function from within its context. That operation, combined with currying, will
grant us more power and convenience when working with `Maybe` values.
