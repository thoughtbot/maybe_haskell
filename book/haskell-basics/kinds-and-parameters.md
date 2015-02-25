## Kinds and Parameters

Imagine we wanted to generalize this `Person` type. What if people were able to
hold arbitrary things? What if what that thing is (its type) doesn't really
matter, the only meaningful thing we can say about it is if it's there or not.
What we had before was a *person with an age* or a *person without an age*, what
we want here is a *person with a thing* or a *person without a thing*.

One way to do this is to *parameterize* the type:

```haskell
data Person a = PersonWithThing String a | PersonWithoutThing String
--          |                          |
--          |                          ` we can use it as an argument here
--          |
--          ` By adding a "type variable" here
```

The type we've defined here is `Person a`. We can construct values of type
`Person a` by giving a `String` and an `a` to `PersonWithThing`, or by giving
only a `String` to `PersonWithoutThing`. Notice that even if we build our person
using `PersonWithoutThing`, the constructed value still has type `Person a`.

The `a` is called a *type variable*. Any lowercase value will do, but it's
common to use `a` because it's short, and a value of type `a` can be thought of
as a value of *any* type. Rather than hard-coding that a person has an `Int`
representing their age (or not), we can say a person is holding some thing of
type `a` (or not).

We can still construct people with and without ages, but now we have to specify
in the type that the `a` is an `Int` in this case:

```haskell
patWithAge :: Person Int
patWithAge = PersonWithThing "pat" 29

patWithoutAge :: Person Int
patWithoutAge = PersonWithoutThing "pat"
```

Notice how even in the case where I have no age, we still specify the type of
that thing which I do not have. In this case, we specified an `Int` for
`patWithoutAge`, but values can have (or not have) any type of thing:

```haskell
patWithEmail :: Person String
patWithEmail = PersonWithThing "pat" "pat@thoughtbot.com"

patWithoutEmail:: Person String
patWithoutEmail = PersonWithoutThing "pat"
```

We don't have to give a concrete `a` when it doesn't matter. `patWithoutAge` and
`patWithoutEmail` are the same value with different types. We could define a
single value with the generic type `Person a`:

```haskell
patWithoutThing :: Person a
patWithoutThing = PersonWithoutThing "pat"
```

Because `a` is more general than `Int` or `String`, a value such as this can
stand in anywhere a `Person Int` or `Person String` is needed:

```haskell
patWithoutAge :: Person Int
patWithoutAge = patWithoutThing

patWithoutEmail :: Person String
patWithoutEmail = patWithoutThing
```

Similarly, functions that operate on people can choose if they care about what
the person's holding or not. For example, getting someone's name shouldn't be
affected by them holding something or not, so we can leave it unspecified:

```haskell
getName :: Person a -> String
getName (PersonWithThing name _) = name
getName (PersonWithoutThing name) = name

getName patWithAge
-- => "pat"

getName patWithoutEmail
-- => "pat"
```

But a function which does care, must both specify the type *and* account for the
case of non-presence:

```haskell
doubleAge :: Person Int -> Int
doubleAge (PersonWithThing _ age) = 2 * age
doubleAge (PersonWithoutThing _) = 1

doubleAge patWithAge
-- => 58

doubleAge patWithoutAge
-- => 1

doubleAge patWithoutThing
-- => 1

doubleAge patWithoutEmail
-- => Type error! Person String != Person Int
```

In this example, `doubleAge` had to account for people that had no age. The
solution it chose was a poor one: return the doubled age or `1`. A better choice
is to not return an `Int`; instead, return some type capable of holding both the
doubled age and the fact that we might not have had an age to double in the
first place. What we need is `Maybe`.
