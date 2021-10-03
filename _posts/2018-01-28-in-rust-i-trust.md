---
title:      In Rust I trust
date:       2018-01-28
summary:    My thoughts on Rust
categories: rust
---

![rusty gears](/images/2018-01-28-gears.jpg)

So it's time to write my first post in 2018. I've been busy learning the Rust language. In this post I'll try to explain why I decided to learn it, how I'm learning it and awesome things this language can be used for.

### My motivation to learn Rust

It first occurred to me to learn Rust in July of 2017. At that time we were trying to implement [the Ethereum protocol in Elixir](https://github.com/exthereum).

In Ethereum's [Merkle Patricia Tree](https://github.com/ethereum/wiki/wiki/Patricia-Tree) SHA3 hashes are used for node keys. The only implementation of SHA3 I found in Erlang/Elixir ecosystem was [erlang-keccakf1600](https://github.com/potatosalad/erlang-keccakf1600) NIF written in C. But I'm not a fan of either Erlang or C.
I had heard that there is a new player in system programming language market that can compete with C/C++. I decided to write Elixir NIF in the Rust language because this whole 'Ethereum in Elixir' thing was a side project, I didn't have any deadlines and I wanted it to be perfect from the ground up. Also it was a good opportunity to learn new things. My adventures with Rust began.

My plans to learn Rust were crushed when I was assigned a new project at my work in Bookmate: I had to write a microservice in Kotlin, the language that I didn't know at the time. Thankfully Kotlin is an easy to learn language and I had written a couple of microservices in Java before. Kotlin positions itself as a pragmatic and easy-to-use alternative to Java. But unfortunately I had to suspend my playing with Rust to direct all my attention to this project.
After finishing the Kotlin project my motivation to finish the Ethereum project has died so I didn't have reasons to learn Rust (by the way the other guys from the Exthereum github organization are still working on the project).

My second attempt to learn the language began in December of 2017. I got an idea for a simple game. And do you know what language did I choose to implement it? That's right, I chose Rust. The journey continues to this date.

### My plan to learn Rust

<blockquote>
  <p>
    ...I want to stress that Rust isn’t the kind of language you can learn in a couple days and just deal with the hard/technical/good-practice stuff later. You will be forced to learn strict safety immediately and it will probably feel uncomfortable at first. However in my own experience, this has led me towards feeling like compiling my code actually means something to me again.
  </p>
  <footer><cite title="Mitchell Nordine">Mitchell Nordine</cite></footer>
</blockquote>

I created a plan to learn Rust as effectively as possible:

1. Read O'Reilly's [Programming Rust book](http://shop.oreilly.com/product/0636920040385.do).
2. Finish all of 78 exercises from [Exercism](http://exercism.io/languages/rust/about). You can start doing exercises while reading the book.
3. Write a couple of simple libraries.

I hope after finishing step 3 I can say that I have a basic knowledge of Rust. At the time of writing this post I have finished 47 of 78 exercises (finished exercises are available in the github repository - [exercism_rust](https://github.com/ayrat-playground/exercism_rust)).

I experienced firsthand the steep learning curve of this language. Concepts of ownership, references and lifetime were the hardest to grasp.

### Usage

Rust is a new systems programming language. Like C and C++, Rust gives developers fine control over the use of memory, so it can be used for every task that C/C++ are used:

- Game development.
There is a great site called [Are we game yet?](http://arewegameyet.com/) that lists all game development resources in Rust ecosystem.
- Operating systems.
[Redox](https://www.redox-os.org/) is a Unix-like operating system written in Rust
https://os.phil-opp.com/ - This blog series creates a small operating system.
- Elixir Nifs.
[Rustler](https://github.com/hansihe/rustler) - Safe Rust bridge for creating Erlang NIF functions.
- Ruby extensions.
[Helix](https://github.com/tildeio/helix)
- Etc
