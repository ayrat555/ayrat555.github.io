---
layout: post
title: Libra: Facebook's Ethereum clone
summary: My thoughts on Facebook's blockchain
categories: blockchain
---

Less than a month ago Facebook announced its new blockchain called Libra. A couple of days after the announcement they released a prototype written in Rust on GitHub. At first glance, it looks like Facebook is trying to catch up with the hyped technology. Let's be clear the hype around blockchains was much bigger at the end of 2018 and at the beginning of 2019. I think the development is started at this period of time.

With the number of resources Facebook has they can work on any technology they want by hiring as many specialists in this technology's area as they can. Even looking at the number of authors of the Libra's technical paper we can see the list of more than 50 people.

I've been in crypto community (mostly Ethereum) for some time ([Mana Project](https://github.com/mana-ethereum), [Blockscout](https://github.com/poanetwork/blockscout)). I took a look at Libra's technical paper. I'll try to describe my thoughts on Libra comparing it with Ethereum in this blog post.

### Libra vs Ethereum

Let's compare some points described in Libra's technical paper that I found important. Note, that I'll compare fundamental things without comparing technical details. Of course, Libra tries to improve on Ethereum but so does Ethereum 2.0 and other blockchains (Pokadot, POA Network, etc).

#### Specification

As it was said above Libra has a [technical paper](https://developers.libra.org/docs/assets/papers/the-libra-blockchain.pdf). Comparing it with Ethereum's [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) it seems rather shallow.

Considering there are fifty people who wrote this paper, they could provide more implementation details. You can say there is an implementation that you can look for the required details. But with Ethereum's Yellow Paper you can implement your own client only by studying it, it includes all the required equations and formulas. Also, the Yellow Paper has only one main author - Gavin Wood.

#### Smart Contracts

Initially, Turing complete programming language was Ethereum's novelty. It was what got me interested in Ethereum in the first place. From Ethereum official website: Ethereum is a decentralized platform for applications that run exactly as programmed without any chance of fraud, censorship or third-party interference.

Ethereum has a virtual machine that executes machine code compiled from high-level smart contracts programming languages. There are several languages that are compiled to EVM machine code: Solidity (similar to Javascript), LLL (Lisp Like Language) etc.

It doesn't look like Libra improves on Ethereum here. It also has a virtual machine that executes smart contracts compiled to bytecode. The smart contract language is called Move. It's not finished yet.

#### Transactions

Again, there are a lot of similarities in Libra and Ethereum in transactions and their execution.

Gas is the internal pricing for running a transaction or a contract in Ethereum. Transaction sender sets gas price when he executes a transaction. From Libra's technical paper: Validators prioritize executing transactions with higher gas prices and may drop transactions with low prices when the system is congested. So gas is exactly the same in Libra.

Receipt in Ethereum is a structure that contains useful information about transaction execution (gas used, status etc). In Libra, receipts are called events.

#### Custom coins

In the Libra paper, they emphasize that multiple different currencies can be implemented using Libra. But it's not innovative since ERC-20 and ERC-721 tokens exist in Ethereum ecosystem for a long time and they are already widely used.

#### Consensus algotithm

I didn't go deep into the Libra's consensus algorithms. It's some kind of Proof of Authority (POA) type of consensus since there are special nodes called validators which validate and publish new transactions. Ethereum ecosystem already has working POA solutions (POA Network, xDai etc). So nothing new here.

### Potential consequences

I don't believe that Facebook is building Libra "so people everywhere can live better lives" as its landing page says. Big corporations always try to have a bigger piece of the pie in technologies of great promise to enrich themselves. A couple of years ago the most hyped technology was VR and Facebook bought Oculus - the most promising company in that field at the time. Today Facebook decided to build Ethereum clone because they just can't buy the Ethereum foundation.
