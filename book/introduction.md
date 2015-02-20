As a programmer, I spend a lot of time dealing with the fallout from one
specific problem: partial functions. A partial function is one that can't
provide a valid result for all possible inputs. If you write a function (or
method) to return the first element in an array that satisfies some condition,
what do you do if no such element exists? You've been given an input for which
you can't return a valid result. Aside from raising an exception, what can you
do?

The most popular way to to deal with this is to return a special value that
indicates failure. Ruby has `nil`, Java has `null`, and many C functions return
`-1` in failure cases. This is a huge hassle. You now have a system in which any
value at any time can either be the value you expect or `nil`, always.

If you try to find a `User`, and you get back a value, and you try to treat it
like a `User` when it's actually `nil`, you get a `NoMethodError`. What's worse,
that error may not happen anywhere near the source of the problem. The line of
code that created that `nil` may not even appear in the eventual backtrace. The
result is various "`nil` checks" peppered throughout the code. Is this the best
we can do?

The problem of partial functions is not going away. User input may be invalid,
files may not exist, networks may fail. We will always need a way to deal with
partial functions. What we don't need is `null`.

## An Alternate Solution

In languages with sufficiently-expressive type systems, we have another option:
we can state in the types that certain values may not be present. Functions
normally written in a partial way can instead return a type that captures any
potential non-presence. Not only does it make it explicit and "type checked"
that when a value may not be present, you have code to handle that case, but it
also means that if a value is *not* of this special "nullable" type, you can
feel safe in your assumption that the value's really there -- No `nil`-checks
required.

The focus of this book will be Haskell's implementation of this idea via the
`Maybe` data type. This type and all of the functions that deal with it are not
built-in, language-level constructs. All of it is implemented as libraries,
written in a very straightforward way. In fact, we'll write most of that code
ourselves over the course of this short book.

Haskell is not the only language to have such a construct. For example, Scala
has a similar `Option` type and Swift has `Optional` with various built-in
syntax elements to make its usage more convenient. Many of the ideas implemented
in these languages were lifted directly from Haskell. If you happen to use one
of them, it can be good to learn where the ideas originated.

## Required Experience

I'll assume no prior Haskell experience. I expect that those reading this book
will have programmed in other, more traditional languages, but I'll also ask
that you *actively combat* your prior programming experience.

For example, you're going to see code like this:

```haskell
countEvens = length . filter even
```

This is a function definition written in an entirely different style than you
may be used to. Even so, I'll bet you can guess what it does, and even get close
to how it does it: `filter even` probably takes a list and filters it for only
even elements. `length` probably takes a list and returns the number of elements
it has.

Given those fairly obvious facts, you might guess that putting two things
together with `(.)` must mean you do one and then give the result to the other.
That makes this expression a function which must take a list and return the
number of even elements it has. Without mentioning any actual argument, we can
directly assign this function the name `countEvens`. There's no need in Haskell
to say that count-evens *of some x* is to take the length after filtering for
the even values *of that x*. We can state directly that count-evens is taking
the length after filtering for evens.

This is a relatively contrived example, but it's indicative of the confusion
that can happen at any level: if your first reaction is "So much syntax! What is
this crazy dot thing!?", you're going to have a bad time. Instead, try to
internalize the parts that make sense while getting comfortable with *not*
knowing the parts that don't. As you learn more, the various bits will tie
together in ways you might not expect.

## Structure

We'll spend this entire book focused on a single *type constructor* called
`Maybe`. We'll start by quickly covering the basics of Haskell, but only so far
that we see the opportunity for such a type and can't help but invent it
ourselves. With that defined, we'll quickly see that it's cumbersome to use.
This is because Haskell has taken an inherently cumbersome concept and put it
right in front of us by naming it and requiring we deal with it at every step.

From there, we'll explore three *type classes* whose presence will make our
lives far less cumbersome. We'll see that `Maybe` has all of the properties
required to call it a *functor*, an *applicative functor*, and even a *monad*.
These terms may sound scary, but we'll go through them slowly, each concept
building on the last. These three *interfaces* are crucial to how I/O is handled
in a purely functional language such as Haskell. Understanding them will open
your eyes to a whole new world of abstractions and demystify notoriously opaque
topics.

Finally, with a firm grasp on how these concepts operate in the context of
`Maybe`, we'll discuss other types which share these qualities. This is to
reinforce the fact that these ideas are *abstractions*. They can be applied to
any type that meets certain criteria. Ideas like *functor* and *monad* are not
specifically tied to the concept of partial functions or nullable values. They
apply broadly to things like lists, trees, exceptions, and program evaluation.

## What This Book is Not

I don't intend to teach you Haskell. Rather, I want to show you *barely enough*
Haskell so that I can wade into more interesting topics. I want to show how this
`Maybe` data type can add safety to your code base while remaining convenient,
expressive, and powerful. My hope is to show that Haskell and its "academic"
ideas are not limited to PhD thesis papers. These ideas result directly in
cleaner, more maintainable code that solves practical problems.

I won't describe how to set up a Haskell programming environment, show you how
to write and run complete Haskell programs, or dive deeply into every language
construct we'll see. If you are interested in going further and actually
learning Haskell (and I hope you are!), then I recommend following Chris Allen's
great [learning path][learnhaskell].

[learnhaskell]: https://github.com/bitemyapp/learnhaskell

Lastly, a word of general advice before you get started:

The type system is not your enemy, it's your friend. It doesn't slow you down,
it keeps you honest. Keep an open mind. Haskell is simpler than you think.
Monads are not some mystical burrito, they're a simple abstraction which, when
applied to a variety of problems, can lead to elegant solutions. Don't get
bogged down in what you don't understand, dig deeper into what you do. And above
all, take your time.
