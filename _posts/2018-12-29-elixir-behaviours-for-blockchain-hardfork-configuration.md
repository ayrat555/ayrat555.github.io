---
layout:     post
title:      Elixir protocols for blockchain hard fork configuration
date:       2018-12-25
summary:    Hard fork configuration at Mana Project
categories: ethereum
---

![trees](https://static.vecteezy.com/system/resources/previews/000/102/765/original/free-minimalist-trees-vector.png)

### Introduction

In [the Mana-Ethereum](https://github.com/mana-ethereum/mana) project, we're creating Ethereum client implementation in Elixir. Ethereum is like any software needs constant maintenance, improvement and optimisation. In this post, I'll describe how hard fork configuration done in our client using built-in polymorphism mechanisms in Elixir.

### Hard forks

Upgrades in Ethereum are done using hardforks. A hard fork is a rule change such that the software validating according to the old rules will see the blocks produced according to the new rules as invalid. In case of a hard fork, all nodes meant to work in accordance with the new rules need to upgrade their software.

![hardfork](https://i.imgur.com/GxxyGB0.png)

In Ethereum a hardfork is a way to introduce new changes to the chain. But in some cases hardforks can occur when groups of miners and developers can't agree on updates to the software governing a particular digital token. As a result, one group continues to operate under the same rules, while another branches off and generates a new blockchain with an updated software setup. In the process, a second digital currency is generated. This is how Bitcoin Cash came to life. In mid-2017, a group of developers wanting to increase Bitcoin's block size limit prepared a code change. A hard fork took effect on 1 August 2017. As a result, the bitcoin ledger and the cryptocurrency split in two. At the time of the fork anyone owning bitcoin was also in possession of the same number of Bitcoin Cash units.

As an example let's examine Constantinople hardfork which is scheduled to take place on the Ethereum Mainnet in mid-January 2019, at block 7,080,000. It has five EIPs (Ethereum Improvement Proposal):

- EIP 145: details a more efficient method of information processing on ethereum known as bitwise shifting.
- EIP 1052: offers a means of optimizing large-scale code execution on ethereum.
- EIP 1283: this proposal mainly benefits smart contract developers by introducing a more equitable pricing method for changes made to data storage.
- EIP 1014: the purpose of this upgrade is to better facilitate a certain type of scaling solution based upon state channels and “off-chain” transactions.
- EIP 1234: this upgrade is the most contentious of the batch, reducing block mining reward issuance from 3 ETH down to 2 ETH, as well as, delaying the difficulty bomb for a period of 12 months.

### Polymorphism in Elixir

In programming languages and type theory, polymorphism is the provision of a single interface to entities of different types or the use of a single symbol to represent multiple different types.

Elixir has two mechanisms for polymorphism:

- Protocols
- Behaviours

Dispatching on a protocol is available to any data type as long as it implements the protocol. You can write a function that behaves differently depending on the type of the first argument to it’s functions. Protocol implementations can be supplied for one of the built in supported aliases Atom, BitString, Float, Function, Integer, List, Map, PID, Port, Reference, Tuple, and Any; or a user defined struct.

Many modules share the same public API. Behaviours provide a way to:

- define a set of functions that have to be implemented by a module;
- ensure that a module implements all the functions in that set.

Let's see how we can implement hardfork configurations using protocols and behaviours in the next sections.

### Hard fork configuration with protocols

### Hard fork configuration with behaviours

### Conclustion
