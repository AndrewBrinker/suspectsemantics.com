---
layout: post
title: "Monomorphization Bloat"
date: 2016-12-03 10:38:20 -0800
comments: true
categories: Rust
---

In this post I will discuss monomorphization: what it is, why it's useful, and
what problems can potentially arise from it, with a particular focus on the
problem of monomorphization bloat. I will also look at options for dealing with
monomorphization bloat in your codebase.

<!-- more -->

Monomorphization is a compilation strategy to allow polymorphism with static
dispatch. To explain how it works, we'll look at the following function:

```rust
fn do_the_thing<T>(x: T) {
    // ... does the thing    
}
```

As you can see, this function takes in some type `T`. This means we can call
the function with any type, like so:

```rust
fn main() {
    do_the_thing(42);
    do_the_thing(8385.35);
    do_the_thing("a string!");
}
```

At compile time, the compiler sees that `do_the_thing` is called with three
different types: `i32`, `f64`, and `&str`. So it creates three different
functions based on `do_the_thing`, replacing the general `T` type with
each of the three concrete types the function is actually called with. In
the end, you get something like this:

```rust
fn do_the_thing_1(x: i32) {
    // ... do the thing, with an i32!
}

fn do_the_thing_2(x: f64) {
    // ... do the thing, with an f64!
}

fn do_the_thing_3(x: &str) {
    // ... do the thing, with a &str!
}

fn main() {
    do_the_thing_1(42);
    do_the_thing_2(8385.35);
    do_the_thing_3("a string!");
}
```

This is monomorphization! The generated code is equivalent to you having
written three different functions, but you didn't have to actually write
three functions! This is really cool, as writing the general version is
a lot less work than writing a version for every concrete type you'll
use, and a lot more flexible too.

## Bloat

The downside is that, by generating these functions, you potentially add
bloat to the resulting binary. If you have a function with size `n`, and
it's called with `m` number of concrete types, it'll have a size of
`n×m` in the resulting binary! In some contexts, a large binary size can
be a problem.

Thankfully, there are several options to deal with this. In my last post,
I talked about a small conversion trick using `Into`. This is a fairly
common trick in the Rust world, and can if you're not careful cause
the monomorphization bloat problem we're discussing here. But imagine
you have something like the following:

```rust
pub fn big_function<T: Into<i32>>>(x: T) {
    // This is a giant function with hundreds of lines!
    // And it gets called with a lot of concrete types!
    // Oh no!
}
```

The way monomorphization works, the entire body of the function gets copied.
But in the case of conversion traits like `Into<T>`, `From<T>`, `AsRef<T>`,
`Borrow<T>`, and `ToString`, you can actually separate the part that needs to
be monomorphized from the rest of the function like so:

```rust
pub fn big_function<T: Into<i32>>(x: T) {
    let x: i32 = x.into();
    _big_function(x)
}

fn _big_function(x: i32) {
    // This is where all the rest of the original function body is now!
}
```

By splitting out the conversion code from the function, you keep the part that
the compiler will monomorphize, and thereby duplicate, quite small. This helps
to keep the size of the resulting binary down.

Alternatively, you can remove the conversion trait abstraction, and instead
require callers to do the conversion themselves before calling the function.
This may make the API a little more tedious to use, but it avoids the potential
problem of bloat by not requiring monomorphization at all.

Additionally, you can always try to shrink the sizes of your functions by
refactoring. A big function is probably a sign of something gone wrong anyway.

## Conclusion

I don't say any of this to encourage you not to use trait bounds or not to
use some of the conversion trait niceties in your API. Generics and static
dispatch are an important part of Rust, and it would be silly not to take
advantage of these features. Just keep in mind the potential for bloat, and
keep an eye on the size of your binary. If it grows too large and becomes a
problem, hopefully you will be better equipped to understand what is happening,
and to correct it.

