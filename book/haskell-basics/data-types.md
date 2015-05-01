## Our Own Data Types

We're not limited to basic types like `Int` or `String`. As you might expect,
Haskell allows you to define custom data types:

```haskell
data Person = MakePerson String Int
--                       |      |
--                       |      ` The persons's age
--                       |
--                       ` The person's name
```

To the left of the `=` is the *type* constructor and to the right can be one or
more *data* constructors. The type constructor is the name of the type, which is
used in type signatures. The data constructors are functions that produce
values of the given type. For example, `MakePerson` is a function that takes a
`String` and an `Int`, and returns a `Person`. Note that I will often use the
general term "constructor" to refer to a *data* constructor if the meaning is
clear from the context.

When working with only one data constructor, it's quite common to give it the same
name as the type constructor. This is because it's syntactically impossible to
use one in place of the other, so the compiler makes no restriction. Naming is
hard. So when you have a good name, you might as well use it in both contexts.

```haskell
data Person = Person String Int
--   |        |
--   |        ` Data constructor
--   |
--   ` Type constructor
```

Once we have declared the data type, we can now use it to write functions that
construct values of this type:

```haskell
pat :: Person
pat = Person "Pat" 29
```

## Pattern Matching

To get the individual parts back out again, we use [pattern
matching][pattern-matching]:

```haskell
getName :: Person -> String
getName (Person name _) = name

getAge :: Person -> Int
getAge (Person _ age) = age
```

In the definitions above, each function is looking for values constructed with
`Person`. If it gets an argument that matches (which is guaranteed since that's
the only way to get a `Person` in our system so far), Haskell will use that
function body with each part of the constructed value bound to the variables
given. The `_` pattern (called a *wildcard*) is used for any parts we don't care
about. Again, this is using `=` for equivalence (as always). We're saying that
`getName`, when given `(Person name _)`, *is equivalent to* `name`. It works
similarly for `getAge`.

Haskell offers [other][records] [ways][lenses] to do this sort of thing, but we won't
get into those here.

[pattern-matching]: https://www.haskell.org/tutorial/patterns.html
[records]: http://en.wikibooks.org/wiki/Haskell/More_on_datatypes#Named_Fields_.28Record_Syntax.29
[lenses]: http://www.haskellforall.com/2012/01/haskell-for-mainstream-programmers_28.html

## Sum Types

As mentioned earlier, types can have more than one data constructor. These are
called *sum types* because the total number of values you can build of a sum
type is the sum of the number of values you can build with each of its
constructors. The syntax is to separate each constructor by a `|` symbol:

```haskell
data Person = PersonWithAge String Int | PersonWithoutAge String

pat :: Person
pat = PersonWithAge "Pat" 29

jim :: Person
jim = PersonWithoutAge "Jim"
```

Notice that `pat` and `jim` are both values of type `Person`, but they've been
constructed differently. We can use pattern matching to inspect how a value was
constructed and accordingly choose what to do. Syntactically, this is
accomplished by providing multiple definitions of the same function, each
matching a different pattern. Each definition will be tried in the order
defined, and the first function to match will be used.

This works well for pulling the name out of a value of our new `Person` type:

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
compile, but warn us about a *non-exhaustive* pattern match. What we've
created at that point is a *partial function*. If such a program ever attempts
to match `getAge` with a `Person` that has no age, we'll see one of the few
runtime errors possible in Haskell.

A person's name is always there, but their age may or may not be. Defining two
constructors makes both cases explicit and forces anyone attempting to access a
person's age to deal with its potential non-presence.
