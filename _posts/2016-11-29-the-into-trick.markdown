---
layout: post
title: "The \"Into<Option>\" Trick"
date: 2016-11-29 20:30:41 -0800
comments: true
categories: rust
---

In this post I will talk about an interesting and useful addition to the
Rust standard library in version 1.12.0. One which allows for an interesting
API improvement to existing libraries, without harming backwards compatibility
if libraries decide to use it.

<!-- more -->

A fun little feature was quietly added to Rust in version 1.12, in the form
of the following trait implementation:

```rust
impl<T> From<T> for Option<T> {
    fn from(val: T) -> Option<T> {
        Some(val)
    }
}
```

While this may seem innocuous, this little trait implementation allows for a
really pleasing new API trick. To explain, let's look at a function that takes
in an optional parameter. (Note: this function is taken from [the pull request][pr]
that introduced this new feature.)

```rust
fn set_read_timeout(&mut self, timeout: Option<u32>) {
    // ...
}

fn main() {
    // x is defined somewhere in here...

    x.set_read_timeout(Some(30));
    x.set_read_timeout(Some(10));
    x.set_read_timeout(None);
}
```

This function looks fine, except it's a little annoying to have to wrap the numbers
in `Some`. Wouldn't it be nice if you didn't have to do that? Enter the new trait
implementation. You see, in the Rust standard library there exist two conversion
traits that are the dual of each other: `From` and `Into`. If you implement `From`,
you get the implementation of `Into` for free, thanks to the following blanket
implementation in the standard library:

```rust
impl<T, U> Into<U> for T where U: From<T> {
    fn into(self) -> U {
        U::from(self)
    }
}
```

That may seem a little crazy, but it just says that if a type `U` implements `From<T>`,
then `T` will automatically implement `Into<U>`. This means that for those two types,
both of the following are valid:

```rust
let original = OriginalType::new();
let new_thing_1: NewType = original.into();
let new_thing_2 = NewType::from(t);
```

Back to the original idea: using this new trait implementation, you _can_ in fact
write a new version of `set_read_timeout` that is used like so:

```rust
fn main() {
    x.set_read_timeout(30);
    x.set_read_timeout(10);
    x.set_read_timeout(None);
}
```

The new version of the function looks like this:

```rust
fn set_read_timeout<T: Into<Option<u32>>>(&mut self, timeout: T) {
    let timeout = timeout.into();
    // ...
}
```

In the new function, `T` is some type that implements `Into<Option<u32>>`. Because of
the new blanket trait implementation for `From`, and the already in-place blanket trait
implementation for `Into` (to automatically match any `From` implementation), the set
of types that meets `T`'s constraints includes `i32` itself! In addition, `From` (and
therefore `Into`) is implemented for any type to convert into itself, and so passing
`None` or `Some` still works the same as before. This means the new function header can
be used as a drop-in replacement for the old version without any problems!

This is just a small trick that you can use to make your Rust APIs nicer. It's may not
make sense to use everywhere, but it's something nice to reach for when it's worthwhile.


[pr]: https://github.com/rust-lang/rust/pull/34828 "The pull request that introduced this new trait implementation."

