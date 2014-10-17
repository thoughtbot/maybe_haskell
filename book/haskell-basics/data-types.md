## Our Own Data Types

We're not limited to basic types like `Int` or `String`. As you might expect,
Haskell allows you to define custom data types:

```haskell
data Person = MakePerson String Int
```

Here we've stated that the people in our system have a name (a `String`) and an
age (an `Int`). To the left of the `=` is the name of the type and to the right
can be one or more *constructors*. In almost all cases, you can treat a
constructor like any other function; for example, `MakePerson` is a function
that takes a `String` and an `Int`, and returns a `Person`.

It's quite common to give the same name to the type and its constructor. This is
because it's syntactically impossible to use one in place of the other, so the
compiler makes no restriction. Naming is hard, so if you have a good one, you
might as well use it in both contexts.

```haskell
data Person = Person String Int
--   |        |
--   |        ` A constructor
--   |
--   ` The type's name
```

With this data type declared, we can now use it to write functions that
construct values of this type:

```haskell
pat :: Person
pat = Person "Pat" 29
```

## Pattern Matching

To get these values back out again, we would use something called [pattern
matching][pattern-matching].

```haskell
getName :: Person -> String
getName (Person name _) = name

getAge :: Person -> Int
getAge (Person _ age) = age
```

In the above definitions, each function is looking for values constructed with
`Person`. If it gets an argument that matches (which in this case is guaranteed
since that's the only way to get a `Person` in our system so far), Haskell will
use that function body with each part of the constructed value bound to the
variables given. The `_` pattern (called a *wildcard*) is used for any parts we
don't care about. Again, this is using `=` for equivalence (as always). We're
saying that `getName`, when given `(Person name _)`, *is equivalent to* `name`.
Similarly for `getAge`.

There are [other][records] [ways][lenses] to do this sort of thing, but we won't
get into that here.

[pattern-matching]: https://www.haskell.org/tutorial/patterns.html
[records]: http://en.wikibooks.org/wiki/Haskell/More_on_datatypes#Named_Fields_.28Record_Syntax.29
[lenses]: http://www.haskellforall.com/2012/01/haskell-for-mainstream-programmers_28.html

## Sum Types

As alluded to earlier, types can have more than one constructor, each separated
by a `|` symbol. This is called a *sum type*:

```haskell
data Person = PersonWithAge String Int | PersonWithoutAge String

pat :: Person
pat = PersonWithAge "Pat" 29

jim :: Person
jim = PersonWithoutAge "Jim"
```

Haskell allows for multiple definitions of the same function, so long as they
match different patterns. They will be tried in the order defined, and the first
function to match will be used. This works well for pulling the name out of a
value of our new `Person` type:

```haskell
getName :: Person -> String
getName (PersonWithAge name _) = name
getName (PersonWithoutAge name) = name
```

But we must be careful when trying to pull out a person's age:

```haskell
getAge :: Person -> Int
getAge (PersonWithAge _ age) = age
getAge (PersonWithoutAge _) = -- uh-oh
```

If we decide to be lazy and not define that second function body, Haskell will
compile, but warn us about the *non-exhaustive pattern*. If such a program ever
attempts to match `getAge` with a `Person` that has no age, we'll see one of the
few runtime errors possible in Haskell.

A person's name is always there, but their age may or may not be. Defining two
constructors makes both cases explicit and forces anyone attempting to access a
person's age to deal with its potential non-presence.
