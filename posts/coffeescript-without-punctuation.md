_Once upon a time a CoffeeScript ninja was born._

_One-liners were his ultimate tool and space was the only punctuation he
needed._ Well, other punctuation marks was used too, but he accepted them as
a necessary evil and struggled each time he used a comma.

He fought a hard battle with the reality.

```coffee
options = followRedirects: (res) -> isTrusted res.headers 'X-Auth'
```

Compiles to the following JavaScript.

```js
options = {
  followRedirects: function(res) {
    return isTrusted(res.headers('X-Auth'));
  }
};
```

Ninja liked his CoffeeScript one-liner and hated JavaScript verboseness. That
one-liner was obvious to him, obvious as every complex thing is obvious to the
ninja.

---

That CoffeeScript code looked legit on review, but an buzzing thought annoyed
reviewer. _What if there is a tiny comma missing? Right after `res.headers`:_

```coffee
options = followRedirects: (res) -> isTrusted res.headers, 'X-Auth'
```

```js
options = {
  followRedirects: function(res) {
    return isTrusted(res.headers, 'X-Auth');
  }
};
```

Err, now it is completely different. What was original intention? No one knows.

Now we need to review way more code to understand the interfaces of `res.headers` and `isTrusted`.

Both cases could be legit, but trigger different behavior. Only one thing is true; error would manifest itself in a weird way when no one would expect it.

---

Consider some deviation of options:

```coffee
options = method: 'post', followRedirects: (res) -> isTrusted res
```

Now someone is tasked to add `jar: true` option to it:

```coffee
options = method: 'post', followRedirects: (res) -> isTrusted res, jar: true
```

That was easy, spot that properties are comma-separated, and add another comma and new property to the end of line.

The result is legit CoffeeScript, but the program does not work as expected. Compited JavaScript would show the fatal fallacy.

```js
options = {
  method: 'post',
  followRedirects: function(res) {
    return isTrusted(res, {
      jar: true
    });
  }
};
```

---

The most skilled ninja would add some more punctuation when adding a new
property to his one-liner.

```coffee
options = method: 'post', followRedirects: ((res) -> isTrusted res), jar: true
```

This would result in the expected behavior, but the point of _use less punctuation_ would be missed.

Code would be fine, till somebody sane would struggle to change it once again asking _Should I use explicit invocation brackets, surrounding brackets, or space is enough in my case?_

---

Good news is that it is not CoffeeScript fallacy. Tool should be used properly. Use its good parts.

The way out of this hell is to format code according to structures it creates.

---

Object literal should list all properties at the same indentation level:

```coffee
options =
  method: 'post'
  followRedirects: (res) -> isTrusted res
  jar: true
```

Now it is obvious how to change that code.

---

Functions should have their implementations on separate indentation level:

```coffee
options =
  method: 'post'
  followRedirects: (res) ->
    isTrusted res, 'X-Auth'
  jar: true
```

---

The rationale for this rule of thumb is that visual structure is what creates first impression. We are quick acting on it. The brain analyzes the image before it analyzes the syntax.

Code structure should be used to convey semantics for a human.
