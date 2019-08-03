---
layout: post
title: "Setting Expectations for Rust's Difficulty"
date: 2017-01-26 16:08:03 -0800
comments: true
categories: rust
---

In this post I discuss the experiences of people coming to Rust from other languages. What their expectations generally are, how Rust meets or doesn't meet those expectations, and how Rust might do better both at meeting those expectations, and at helping new users to calibrate expectations appropriately.

<!-- more -->

There have been several [blog](https://ayende.com/blog/176801/the-struggle-with-rust) [posts](https://blog.ntpsec.org/2017/01/18/rust-vs-go.html) of late with potential Rust users trying out the language, and deciding it isn’t for them. This is absolutely fine. Rust cannot be everything to everyone. The design decisions Rust makes make it more suitable for certain contexts than others. Making a decision not to use Rust based on a mismatch of language purpose and area of development interest is an entirely reasonable thing to do.

But in some of these posts, the authors comment on the amount of time necessary to get from a point of knowing no Rust at all to being able to program whatever it is they want to program (often characterized by the author as simple). I believe that the problem is not that Rust is unreasonably hard (although I do think there are things to be improved), but that Rust does not set expectations appropriately.

I do think there are certain ergonomic pain points in Rust today. In fact, the Rust teams are well aware of these points, and there is a concerted effort over the next year to address these. These include the introduction of [non-lexical lifetimes](http://smallcultfollowing.com/babysteps/blog/2016/04/27/non-lexical-lifetimes-introduction/), to make the borrow checker smarter, and reduce the likelihood of a situation where the code is actually correct, but the borrow checker is too dumb to tell. There are also discussions happening around auto-dereferencing in match expressions, making matching on a reference simpler, or on improving the `&str`/`String` confusion, to make string usage easier for new Rust programmers.

However, I also think there is a certain degree of fundamental complexity in the mix of performance and safety that Rust provides, and that the onus is then on Rust to better communicate to potential users the degree of difficulty they should expect.

In doing this, it is important to note that programmers come to Rust from a variety of languages. Broadly, these can be grouped into three categories, and programmers coming from each of the three categories experience different issues, appreciate different features, and have different expectations about Rust.

The three groups are:

- C & C++ programmers. I would say “systems” programmers, but the phrase is too overloaded to be particularly meaningful in comparative programming language discussions. C & C++ programmers tend to appreciate the additional safety that Rust can provide, but at the same time seem to have an expectation that Rust will be easier for them than it actually is. While it is true that Rust offers the same degree of performance that these two languages offer, and that it has (largely to avoid spending the weirdness budget frivolously) syntax similar to C, it is in reality a language of a substantial degree of difference. I think C and C++ programmers, in general, do not expect this, and are thus frustrated when Rust turns out to be harder than expected.
- Ruby, Python, Java, and other garbage collected imperative / OO languages. Programmers coming from these languages tend to like that Rust is faster, and appreciate the functional-like features it brings (functional or functional-like programming becoming hotter in these circles). For some, Rust feels like the first low-level programming language that they can understand. It opens up domains that may have otherwise been inaccessible, and feels generally empowering.
- Haskell, OCaml, and other functional languages. These programmers appreciate Rust for the performance and power of it, while feeling comfortable with the design ideas that Rust has blatantly stolen from them. Rust’s original compiler was written in OCaml, and the influence of the language remains in some places. Rust’s trait system is essentially a copy of Haskell’s typeclass system (with [some caveats](https://www.rust-lang.org/en-US/faq.html#how-do-rust-traits-compare-to-haskell-typeclasses)).

Essentially, the second two groups of programmers expect Rust to be hard, because it is low-level and systems-y and generally a little strange. And Rust is hard, but often not as hard as they seem to expect once they get going with it.

On the other hand, C and C++ programmers tend to expect Rust to be easy, and it isn’t. While it has the trappings of something familiar, the experience of Rust for C and C++ programmers is death by a thousand surprises. Rust traits and C++ templates bind differently. Rust defaults to moves, while C++ defaults to copies. The borrow checker is strange and alien and (they often seem to think) usually wrong (in my experience, the borrow checker is very rarely wrong. If you think it is, check your work first).

This mismatch between expectation and reality creates a lot of frustration for C and C++ programmers, who often end up choosing to ditch Rust altogether, and who sometimes write blog posts about their experience.

Now, all of this can sound like I am putting the blame squarely on the shoulders of these programmers. I am not. Rust doesn’t do a good enough job of on-boarding new programmers. This is mostly because on-boarding is really difficult, and the current edition of the Rust book was written very quickly in the run-up to 1.0. There have been improvements since then, but it is still a more limited document than is really desired.

Good news though! There is a new edition of the Rust book in the works, being written by Steve Klabnik and Carol Nichols. You can [read it yourself on GitHub](https://github.com/rust-lang/book), it is excellent, and it will be published by No Starch Press when it’s finished (yay!).

In addition, this year is the year of productivity in Rust. Last year the Rust teams put together a survey of Rust and potential Rust programmers, and the [results of that survey](https://blog.rust-lang.org/2016/06/30/State-of-Rust-Survey-2016.html) have directly informed [the priorities for the next year](https://github.com/rust-lang/rfcs/blob/master/text/1774-roadmap-2017.md). The focus now is on executing on those priorities and addressing the major pain points people identified. These are the same pain points people coming to Rust from other languages often bring up in explaining why they’ve chosen not to use Rust. We’re listening, and things are improving.

In the meantime, how can we improve the on-boarding experience, particularly for C and C++ programmers who seem so often frustrated by Rust? The first path may be by speaking more directly to them. I know there is a book in the works right now, [to be published by O’Reilly](http://shop.oreilly.com/product/0636920040385.do), that introduces Rust to C++ programmers. Additionally, having more C and C++ programmers writing about their experience with Rust would likely be of value, both as a form of social proof, and to more directly address the particular issues people coming from those languages have.

But none of these solutions addresses the core question of setting the expectation of difficulty. I am not sure what the answer is here. Perhaps directing new Rust users to [The Rust Programming Language](https://doc.rust-lang.org/book/) book more forcefully. Perhaps encouraging more programmers to reach out on [IRC](https://www.rust-lang.org/en-US/community.html#irc-channels) or [other](https://www.rust-lang.org/en-US/community.html#discussion-forums) [channels](https://users.rust-lang.org/), where individuals can more directly answer their questions and assuage their concerns. I’m curious to hear what other ideas people may have on this front.

As a final note, I will say that Rust is still a young language, and that a lot of the people contributing to it are volunteering to do so in their free time. It takes a long while to get high-quality stable libraries, build up institutional community knowledge and agreement about what libraries or patterns to use, and get the language into a state where you can use it as productively as you would use a long-standing language like Python or Java.

The Rust teams and contributors work really hard to actively address the issues people experience. Those (myself included) who’ve taken part in an issue, or pull request, or RFC can attest to how excellent the experience is. This is a great group, and they are doing their best to address the concerns potential users have with the language.

And hey, maybe in the end it’ll turn out that Rust just really isn’t for them. That’s alright. We can still be friends. <3