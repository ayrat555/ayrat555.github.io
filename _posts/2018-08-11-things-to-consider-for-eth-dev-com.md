---
layout:     post
title:      Things to consider for Ethereum dev community
date:       2018-08-11
summary:    Just some inconveniences I encountered
categories: ethereum
---

![hide-the-pain](/images/2018-08-11-pain.jpg)

Disclaimer: In this post, I describe my own thoughts and concerns I’ve found while working with Ethereum.

### Background

For some time I've been following Ethereum, learning its instruments and reading everything I can, starting from posts and official wikis and ending with the Ethereum yellow paper. Through this process, I've encountered several inconveniences for developers implementing the protocol or just trying to understand how it works.

### Inconveniences

#### Tests

Ethereum has [common tests](https://github.com/ethereum/tests) that protocol level developers should use when implementing the protocol itself. Most of these tests have cases for each of the hard forks starting from Homestead and ending with Byzantium (the latest hard fork at the time of writing this post). There are tons of tests, they test all known corner cases, hard fork changes etc. and without them, it would be impossible to implement a working Ethereum client.

The problem with these tests is that they're not documented at all and it's very hard to understand what exactly any particular test is testing. Also, many of them have very obscure names and typos. For example:
- `vitalikTransactionTest`
- `refund_multimpleSuicide`
- `ShanghaiLove`

There is a field named `comment` that I think it was meant to resolve this problem but currently, it's blank. So to understand what exactly a test is testing you have to debug the working client.

#### Backward Compatability

Most Ethereum clients are backward compatible with older versions of hard forks. They run all tests for all supported hard forks (a couple of thousand tests for each hard fork). This means the ci builds take a long time (~30 minutes depending on the client) to finish. This creates a productivity issue - you must wait for the ci build to finish for every fix, even a small typo.

Moreover, it makes code maintenance difficult. A developer should keep all hard fork changes in the head, and in most of the clients, these changes look like dirty hacks with many conditional clauses.

One solution to the problem would be to support only the latest version of the network. However, there may be a reason to support the older hard fork that I'm unaware of.

#### DevP2P documentation

Some aspectss of Ethereum are poorly documented. One of these is DEVp2p - a protocol used for peer-to-peer node communication. Actually, it's so poorly documented that it may have a different name. It may be called Wire Protocol, RLPx, or DevP2P because it looks like these names are used interchangeably throughout the scanty docs.

Also, there are no common tests for this part of the protocol which results in communication logic differences in different clients.

### Conclusion

Currently, Ethereum is one of the most exciting new technologies in existence. As development progresses, I hope these issues are addressed to help future developers better understand and implement the protocol.
