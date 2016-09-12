Code is for humans to read.

```coffee
encodeUser = (cid, { email, _id: id }) -> jwt.sign { cid, id, email }, JWT_SECRET
```

`encode` is a verb, so there should be a function on right side... where is a
function body?

Well, CoffeeScript could be very implicit, but at least it allows to visually
express functional body, so semantics of operation are explicitly expressed by
the name and parameters of a function, and then there is an implementation.

```coffee
encodeUser = (cid, { email, _id: id }) ->
  jwt.sign { cid, id, email }, JWT_SECRET
```

Object destructuring is great at some places, but wait a minute... where is the
`user` in `encodeUser`?

```coffee
encodeUser = (cid, user) ->
  { email, _id: id } = user
  jwt.sign { cid, id, email }, JWT_SECRET
```

Now it's ugly, the implementation is ugly, but at least function signature is
better (note: it is ugly too, but lets change one thing at a time).

`{ _id: id } = user` means `id = user._id` but in what universe destructuring
became the tool for every problem?

```coffee
encodeUser = (cid, user) ->
  jwt.sign { cid, id: user._id, email: user.email }, JWT_SECRET
```

Heh, now I could clearly see that `id` is `user._id` and `email` is coming from
`user.email`.

---

Why function named `encodeUser` signs some other object while it should _encode user_?

Now it is time to fix the naming issue, but it is a separate story.
