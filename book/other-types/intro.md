The three abstractions you've seen all require a certain kind of value.
Specifically, a value with some other bit of information, often referred to as
its *context*. In the case of a type like `Maybe a`, the `a` represents the
value itself and `Maybe` represents the fact that it may or may not be present.
This potential non-presence is that other bit of information, its context.

Haskell's type system is unique in that it lets us speak specifically about this
other bit of information without involving the value itself. In fact, when
defining instances for `Functor`, `Applicative` and `Monad`, we were defining an
instance for `Maybe`, not for `Maybe a`. When we define these instances we're
not defining how `Maybe a`, a value in some context, behaves under certain
computations, we're actually defining how `Maybe`, the context itself, behaves
under certain computations.

This kind of separation of concerns is difficult to understand when you're only
accustomed to languages that don't allow for it. I believe it's why topics like
monads seem so opaque to those unfamiliar with a type system like this. To
strengthen the point that what we're really talking about are behaviors and
contexts, not any one specific *thing*, this chapter will explore types that
represent other kinds of contexts and show how they behave under all the same
computations we saw for `Maybe`.
