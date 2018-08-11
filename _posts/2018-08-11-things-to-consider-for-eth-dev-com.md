---
layout:     post
title:      Things to consider for Ethereum dev community
date:       2018-08-11
summary:    Just some inconveniences I encountered
categories: ethereum
---

Disclaimer: In this post, I'll describe my own thoughts and concerns and they may be wrong.

### Background

For some time I've been following Ethereum, learning its instruments and reading everything starting from posts and official wikis and ending with the Ethereum's yellow paper. In this process, I encountered a couple of inconveniences for developers implementing the protocol or just trying to understand how it works.

### Inconveniences

#### Tests

Ethereum has [common tests](https://github.com/ethereum/tests) that protocol level developers should use implementing the protocol itself. Most of these tests have cases for each of hard forks starting from Homestead and ending with Byzantium (the latest hard fork at the time of writing this post). The number of these tests is humongous, they test all known corner cases, hard fork changes etc and without them, it would be impossible to implement a working Ethereum client.

But the problem with these test is that they're not documented at all and it's very hard to understand what exactly any particular test is testing. Also, many of them have very obscure names and typos. Let's see a couple of examples:
- `vitalikTransactionTest`
- `refund_multimpleSuicide`
- `ShanghaiLove`

There is a field named `comment` that I think it was meant to resolve this problem but currently, it's blank. So to understand what exactly a test is testing you have to debug the working client.

#### Backward Compatability

Most of Ethereum clients are backward compatible with older versions of hard forks. They run all tests for all hard forks they support which as I mentioned earlier is a large number (a couple of thousands of tests for each of the hard fork). So ci builds take a long time (~30 minutes but this number depends on the client) to finish which is a productivity issue because even to fix a typo in the code you have to wait ci build to finish.

Moreover, it makes code maintenance very hard because a developer should keep in the head all hard fork changes and in most of the clients, these changes look like dirty hacks with many conditional clauses.

One solution to the problem would be to support only the latest version of the network. But maybe there is a logic to support older hard fork which I don't know about.

#### DevP2P documentation

Some parts of Ethereum is poorly documented. One of these parts is DEVp2p - a protocol used for peer-to-peer node communication. Actually, it's so poorly documented that it may have a different name. It may be called Wire Protocol, RLPx, DevP2P because it looks like these names are used interchangeably throughout scanty docs.

Also, there are no common tests for this part of the protocol. So I found differences in communication logic in different clients.

##### Conclusion

Currently, Ethereum is one of the most exciting new technologies. It's being developed and improved by people far smarter than me. But I hope someday they'll look back and address some of the issues I described here.
