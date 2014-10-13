When we declare a function in Haskell, we first write a type signature:

```haskell
five :: Int
```

Then a definition:

```haskell
five = 5
```

The type signature is optional, as Haskell can almost always tell the type of an
expression by inspecting the types of its constituent parts. That said, good
Haskellers will include a type signature on all top-level definitions as it
provides executable documentation and may, in some cases, prevent errors which
occur when the compiler assigns a more generic type than you might otherwise
want. For example, if we omitted the type signature in the above, the compiler
would assign the type `five :: Num a => a` which means that the type of `five`
is `a`ny type that's `Num`eric in nature -- i.e. it can be added, negated and so
forth.

## Defining Our Own Types

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
--   |        ` Type constructor
--   |
--   ` Type name
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

In the above definitions, each function is looking for the `Person` constructor.
If it gets an argument that matches (which in this case is guaranteed), Haskell
will bind each part of the constructed value to the variables given then execute
body of the function. The `_` pattern (called a wildcard) is used for any parts
we don't care about.

There are [other][records] [ways][lenses] to do this sort of thing, but we won't
get into that here.

[pattern-matching]: https://www.haskell.org/tutorial/patterns.html
[records]: http://en.wikibooks.org/wiki/Haskell/More_on_datatypes#Named_Fields_.28Record_Syntax.29
[lenses]: http://www.haskellforall.com/2012/01/haskell-for-mainstream-programmers_28.html

## Sum Types

As alluded to earlier, types can have more than one constructor, each separated
by a `|` symbol. This is called a *Sum type*:

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
few runtime errors possible in Haskell -- that's why you always fix all
compilation warnings, kids.

![one armed man](http://images.thoughtbot.com/maybe/one-armed-man.jpg)

A person's name is always there, but their age may or may not be. Defining two
constructors makes both cases explicit and forces anyone attempting to access a
person's age to deal with its potential non-presence.

## Kinds and Parameters

Imagine we wanted to generalize this `Person` type. What if people were able to
hold arbitrary things? What that thing is (its type) doesn't really matter, the
only meaningful thing we can say about it is if it's there or not. What we had
before was a *person with an age* or a *person without an age*, what we want
here is a *person with a thing* or a *person without a thing*.

One way to do this is to *parameterize* the type:

```haskell
data Person a = PersonWith String a | PersonWithout String
--          |                     |
--          |                     ` we can use it as an argument here
--          |
--          ` By adding a "type variable" here
```

Any lowercase value will do, but it's common to use `a` because it's short, and
a value of type `a` can be thought of as a value of `a`ny type. Rather than
hard-coding that a person has an age (or not), we can say a person is holding
some thing of type `a` (or not).

Now we can construct people with or without something:

```haskell
patWithAge :: Person Int
patWithAge = PersonWith "pat" 29

patWithoutAge :: Person Int
patWithoutAge = PersonWithout "pat"
```

Notice how even in the case where I have no age, I can still specify the type of
that thing which I do not have.

Functions that operate on people can choose if they care about what the person's
holding or not. For example, getting someone's name shouldn't be affected by
them holding something or not, so we can leave it unspecified, again using `a`
to mean `a`ny type:

```haskell
getName :: Person a -> String
getName (PersonWith name _) = name
getName (PersonWithout name) = name
```

But a function which does care, must both specify the type *and* account for the
case of non-presence:

```haskell
doubleAge :: Person Int -> Int
doubleAge (PersonWith _ age) = 2 * age
doubleAge (PersonWithout _) = 1 -- perhaps provide a sane default?
```

Congrats. You now completely understand parameterized types.

## Maybe

Haskell's `Maybe` type should make all kinds of sense now:

```haskell
data Maybe a = Just a | Nothing
```

It's a bit like `PersonWith | PersonWithout`, except we're not dragging along a
name this time. This type is only concerned with representing a value (of any
type) which is either *present* or *not*.

We can use this in functions which would otherwise be *partial*, meaning they
can't produce a value for all possible inputs:

```haskell
-- | Find the first element from the list for which the predicate function
--   returns True. Return Nothing if there is no such element.
find :: (a -> Bool) -> [a] -> Maybe a
find _ [] = Nothing
find predicate (first:rest) =
    if predicate first
        then Just first
        else find predicate rest
```

This forces all callers of `find` to deal with the potential `Nothing` case.

```haskell
--
-- Warning: this is a type error, not working code!
--
findUser :: UserId -> User
findUser uid = find (matchesId uid) allUsers
```

This is a type error since the expression actually returns a `Maybe User`.
Instead, we have to take that `Maybe User` and inspect it to see if something's
there or not. We can do this via `case` which supports pattern matching not
unlike you've seen before:

```haskell
findUser :: UserId -> User
findUser uid =
    case find (matchesId uid) allUsers of
        Just u  -> u
        Nothing -> -- what to do? error?
```

Depending on your domain and the likelihood of Maybe values, you might find this
sort of "stair-casing" propagating throughout your system. This can lead to the
thought that `Maybe` isn't really all that valuable over some *null* value built
into the language. I assure you that's not the case.
