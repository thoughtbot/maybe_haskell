## Maybe

Haskell's `Maybe` type should make all kinds of sense now:

```haskell
data Maybe a = Nothing | Just a
```

It's a bit like `PersonWith | PersonWithout`, except we're not dragging along a
name this time. This type is only concerned with representing a value (of any
type) which is either *present* or *not*.

We can use this to take functions which would otherwise be *partial* and make
them *total*:

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

This function has two definitions matching two different patterns: if given the
empty list, we immediately return `Nothing`. Otherwise, the (non-empty) list is
de-constructed into its `first` element and the `rest` of the list by matching
on the `(:)` (pronounced *cons*) constructor. Then we test if applying the
`predicate` function to `first` returns `True`. If it does, we return `Just` it.
Otherwise, we recurse and try to find the element in the `rest` of the list.

Returning a `Maybe` value forces all callers of `find` to deal with the
potential `Nothing` case:

```haskell
-- 
-- Warning: this is a type error, not working code!
-- 
findUser :: UserId -> User
findUser uid = find (matchesId uid) allUsers
```

This is a type error since the expression actually returns a `Maybe User`.
Instead, we have to take that `Maybe User` and inspect it to see if something's
there or not. We can do this via `case` which also supports pattern matching:

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
into the language. If you have to have this sort of matching expression peppered
throughout the code base, how is that better than the analogous "`nil` checks"?
