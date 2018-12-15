---
layout:     post
title:      Pride and Prejudice
date:       2018-12-15
summary:    This year's results
categories: common
---

![resolutions](https://i.imgur.com/wJKftyQ.jpg)

### Introduction

2018 was a very busy year for me. I learnt a lot of new things this year, was a part of one big project and wrote a couple of smaller libraries. In this post, I'll highlight my accomplishments of 2018 along with disappointments.

### This year's results

#### Learning Rust

The end of the previous year and the beginning of this year I was learning Rust:

- I read [O'Reilly's Programming Rust book](http://shop.oreilly.com/product/0636920040385.do) and official [Rust book](https://doc.rust-lang.org/book/).
- I did a lot of exercises from [exercism](https://github.com/ayrat-playground/exercism_rust).
- I wrote a simple cron command entry parser - [cronenberg](https://github.com/ayrat555/cronenberg)
- I started writing Part-of-speech (POS) tagger - [possibility](https://github.com/ayrat555/possibility)
- I spent a lot of time reading Rust code in the parity-ethereum. It's related to the next section of this post.

#### Mana - Ethereum client

[Mana](https://github.com/mana-ethereum/mana) is an open-source Ethereum client written in Elixir. I started contributing to the Mana project in the spring of 2017 but this year I started spending much more time on the project. We made a great progress:

- At the beginning of the year Mana wasn't passing any blockchain tests. Now, it passes all blockchain tests from all hardforks starting from Frontier and ending with the most recent hardfork - Constantinople.
- We fully synced `ropsten` test network.
- Mainnet sync is in-progress.
- Work on P2P layer is in-progress
- Work on JSON RPC is in-progress.
- In the process of writing an Ethereum client, I learnt how it works under the hood and a lot of things directly related to the blockchain technology. Some of these things are:
  - ZkSnarks that are used for Zero-knowledge proofs
  - Ethereum uses BN128 elliptic curve for ZkSnarks. So I implemented it in Elixir and published as a [separate library](https://github.com/mana-ethereum/bn). It's used in the Mana project.
  - Node discovery based on Kademlia algorithm

I'm happy with the progress we made this year. We almost finished our Ethereum client and I hope we'll release an alpha version soon.

#### Emacs/Emacs Lisp

I've been using Emacs for more than 2 years. But in the summer of this year I decided to dive into it and level up my Emacs skills:

- I read [Emacs Lisp Reference Manual](https://www.gnu.org/software/emacs/manual/elisp.html)
- I finished all Emacs Lisp exercises from [exercism](https://github.com/ayrat-playground/exercism_elisp)
- I rewrote my [Emacs configuration](https://github.com/ayrat555/dot-emacs) from 1_100_000 lines of code to only a couple of hundreds of lines of code using [`use-package`](https://github.com/ayrat555/dot-emacs).
- I learnt more emacs packages (org-mode, magit, swiper/counsel etc)

#### Blog posts

You're reading this post from my blog [Kraken of Thought](https://www.badykov.com/). I created it to share my thoughts and feelings about things that are interesting to me. This year I tried to be consistent and write posts every once in a while. I'm pretty proud of сontent of these post because they capture my feelings and concerns directly related to the work I was doing at a specific period of time. Here's a full list of posts:

- [In Rust I trust](https://www.badykov.com/rust/2018/01/28/in-rust-i-trust/) - My thoughts on Rust.
- [Cronenberg](https://www.badykov.com/rust/2018/02/27/cronenberg/) - Simple cron command entry parser in Rust.
- [GraphQL subscriptions using Absinthe](https://www.badykov.com/elixir/2018/03/25/graphql-subscriptions-using-absinthe/) - GraphQL in Elixir.
- [Ethereum Virtual Machine in Elixir](https://www.badykov.com/elixir/2018/04/29/evm-basics/) - EVM execution basics in Elixir.
- [Ethereum's Recursive Length Prefix encoding in Elixir](https://www.badykov.com/elixir/2018/05/06/rlp/) - Implementing RLP with recursive protocols.
- [Ethereum's node discovery in Elixir](https://www.badykov.com/elixir/2018/06/02/node-discovery/) - Mostly Kademlia in Elixir.
- [Message calls in Ethereum](https://www.badykov.com/ethereum/2018/06/17/message-calls-in-ethereum/) - Summary of existing message calls in Ethereum.
- [Why Emacs is a great text editor](https://www.badykov.com/emacs/2018/07/31/why-emacs-is-a-great-editor/) - My thoughts on Emacs.
- [Things to consider for Ethereum dev community](https://www.badykov.com/ethereum/2018/08/11/things-to-consider-for-eth-dev-com/) - Just some inconveniences I encountered.
- [Be productive with Org-mode](https://www.badykov.com/emacs/2018/08/26/be-productive-with-org-mode/) - A note manager and organizer.
- [zkSNARK in Ethereum](https://www.badykov.com/ethereum/2018/09/15/zkSNARK-in-Ethereum/) - Zero-knowledge proofs in Ethereum.
- [Story about one small library](https://www.badykov.com/ethereum/2018/10/22/story-about-one-small-library/) - Ethereumex evolution.
- [Storage in Ethereum](https://www.badykov.com/ethereum/2018/11/10/storage-in-ethereum/) - Storage levels in the Mana-Ethereum client.

### Disappointments

#### Gamedev frameworks in Rust

I have a couple of ideas for computer games. And I started learning Rust mainly for Game development. But all game engines in Rust turn out to have big downsides:

- they all are in an unfinished state.
- they lack good documentation.

So the current state of Rust game dev tools is not suitable for making complex games.

#### Inconveniences for developers in Ethereum

As described in the first section of this post I was involved in development of Ethereum client this year. I encountered a couple of inconveniences in Ethereum dev community. Below are excerpts for my post [Things to consider for Ethereum dev community](https://www.badykov.com/ethereum/2018/08/11/things-to-consider-for-eth-dev-com/):

- Ethereum has common tests that protocol level developers should use when implementing the protocol itself. The problem with these tests is that they’re not documented at all and it’s very hard to understand what exactly any particular test is testing.
- Most Ethereum clients are backward compatible with older versions of hard forks. A developer should keep all hard fork changes in the head, and in most of the clients, these changes look like dirty hacks with many conditional clauses.
- Some aspects of Ethereum are poorly documented. One of these is DEVp2p - a protocol used for peer-to-peer node communication

#### Unfinished projects

I feel sad because some projects that I started this year weren't finished.

- I started learning Emacs Lisp to write plugins for my favourite text editor. I have a couple of ideas for plugins but I didn't have time to implement them.
- I started working on web application this year. It isn't open source yet and I think it has some interesting ideas. I'm planning to use it myself first.

I hope to return to them when I have some free time.

### Conclusion

This year was a very productive year for me. I hope next year will be as productive as this year and I will finish projects that weren't finished this year.
