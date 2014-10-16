## Kinds and Parameters

Imagine we wanted to generalize this `Person` type. What if people were able to
hold arbitrary things? What if what that thing is (its type) doesn't really
matter, the only meaningful thing we can say about it is if it's there or not.
What we had before was a *person with an age* or a *person without an age*, what
we want here is a *person with a thing* or a *person without a thing*.

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
