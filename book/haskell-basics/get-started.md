## Don't Give Up

The above might leave you feeling underwhelmed. That code doesn't look all that
better than the equivalent Ruby:

```ruby
def find_user(uid)
  if user = all_users.detect? { |u| u.matches_id?(uid) }
    user
  else
    # what to do? error?
  end
end
```

First of all, the Haskell version is type safe: `findUser` must always return a
`User` since that's the type we've specified. I'd put money on most Ruby
developers returning `nil` from the `else` branch. The Haskell type system won't
allow that and that's a good thing. Otherwise, we have these values floating
throughout our system that we assume are there and in fact are not. I understand
that without spending time programming in Haskell, it's hard to see the benefits
of ruthless type safety employed at every turn. I assure you it's a coding
experience like no other, but I'm not here to convince you of that -- at least
not directly.

The bottom line is that an experienced Haskeller would not write this code this
way. `case` is a code smell when it comes to `Maybe`. Almost all code using
`Maybe` can be improved from a tedious `case` evaluation using one of the three
abstractions I'll be exploring in this book.

Let's get started.
