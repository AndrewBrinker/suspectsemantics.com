---
layout: post
title: "Unsafe Core, Safe Interface"
date: 2016-11-28 19:20:44 -0800
comments: true
categories: Rust
---

There is a common pattern in writing Rust libraries that seems to be asked about
fairly often: the unsafe core + safe interface. The essence is that you write
the core of your library using unsafe Rust (usually for performance, or because
you're wrapping FFI code) and then write safe Rust to provide a nice, safe
interface for the unsafe core. In this post I am going to give a quick
explanation of how this is done, and what the considerations are. If you're
looking for a greater degree of detail on this, check out
[The Advanced Rust Programming Language][tarpl] (also called The Rustonomicon).

<!-- more -->

---

Unsafe Rust exists to allow the Rust programmer to subvert Rust's safety
mechanisms, with the expectation that when the unsafe code ends, Rust's safety
guarantees are maintained. Unsafe Rust comes in three flavors: unsafe functions,
unsafe blocks, and unsafe traits.

Unsafe functions are functions whose use allows the subversion of Rust's safety
guarantees. Unsafe blocks are blocks that call unsafe functions. Unsafe traits
are a little different. To explain how, let's talk a bit about what the relationship
between unsafe Rust and safe Rust, in particular about trust.

Safe Rust _has to_ trust unsafe Rust to be correct. If it isn't, there is no way
to defend against it without using even more unsafe Rust. So safe Rust just has to
trust that the author of the unsafe Rust has carefully ensured that Rust's safety
guarantees are maintained.

On the other hand, unsafe Rust can't trust safe Rust. For example, if a piece of unsafe
Rust relies on a type's `PartialOrd` implementation to ensure that Rust's safety
guarantees are maintained, that would be wrong. Because now, if the Rust programmer
changes their safe code to have a bad `PartialOrd` implementation, they can induce
unsafety. This violates the expectation of how safe Rust should work. This means that
unsafe Rust has to be written defensively, and can never trust that safe Rust is correct.

Unsafe traits are an answer to the question of how to handle generic interfaces in the
world of unsafe Rust. Implementing an unsafe trait indicates that you've made sure
that your implementation upholds whatever expectations exist for that trait to behave
well. Because the trait and any implementations of it are unsafe, other unsafe Rust
is free to trust that it is correct.

---

One thing that may be surprising about unsafe Rust is that it is generally infectious
toward anything in the same module. This is because Rust's visibility rules give
any code in the same module as a data type definition (an enum, struct, or what have
you) access to the internal fields of that type. Obviously, for a data type with
unsafe shenanigans happening, this could allow for subversion of Rust's safety
guarantees. So, when you're writing unsafe, you keep the fiddly fields that the unsafe
code touches hidden away as private fields, and provide only a nice safe API for external
users of the module.

This is the key design idea when making types that use safe internally, and its this
pattern that's used throughout Rust's standard collections.

But there are some times where you want to expose both an unsafe interface and a safe
interface. Here, the pattern is a little different. For example, it is a very common
thing to want to provide a Rust wrapper around some C API. For this, you use the Rust
FFI (Foreign Function Interface), which requires that calls to C code be wrapped in
unsafe (as the Rust compiler can't guarantee that the C code hasn't engaged in any
sort of shenanigans). So you start by writing direct Rust analogues to the C code, with
lots of `unsafe` used throughout.

Then, because the resulting code is usually not in the style of idiomatic Rust, and would
likely be a pain to use, you write a nice API on top of this collection of unsafe wrappers.
In the end, these two layers are often published as separate crates, with the unsafe
wrapper crate name ending in `-sys` (a little Rust community practice to indicate a C FFI
crate). In this way, if users want just a nice Rust API to the C library, they use the
cleaned up idiomatic crate. If they want to be down in the nitty-gritty, or do something
not provided for in the higher-level API, they can easily move down to the unsafe wrapper
crate.

---

This has been a little bit about writing unsafe libraries in Rust; why do it, what to
know and think about when doing it, and how it is often done. Hopefully this has clarified
some questions for you. If you want to know more, read [The Advanced Rust Programming Language][tarpl],
as it covers these issues far more completely than I have here.

[tarpl]: https://doc.rust-lang.org/nomicon/ "The Advanced Rust Programming Language"

