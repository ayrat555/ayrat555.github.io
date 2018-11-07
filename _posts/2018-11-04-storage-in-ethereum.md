---
layout:     post
title:      Storage in Ethereum
date:       2018-11-04
summary:    Storage levels in the Mana-Ethereum client
categories: ethereum
---

![blockchain-storage](https://i.imgur.com/zUwpkeN.jpg)

Ethereum blockchain has pretty complex storage concepts at the heart of which is Merkle Patricia trees. Merkle Patricia trees alone are hard to understand but aside from it there are multiple other problems one should keep in mind:

- How to handle storage modifications made by failed Ethereum Virtual Machine (EVM) execution
- How EVM should access Merkle Patricia tree storage
- When and where storage modifications should be committed (after transaction execution, after block finalisation or after multiple blocks/transactions etc)
- How to optimally access permanent storage and commit changes to disk
- How to handle invalid blocks that failed validation

As you can see from the problems above Ethereum storage complexity is not limited only by Merkle Patricia tree data structure and its traversals for node updates. In this post, I'll describe how we're solving these challenges in [the Mana-Ethereum](https://github.com/mana-ethereum/mana) project. Merkle Patricia trees are outside of the scope of this post because they're a lot of implementations and posts in the ethereum community but no information about high-level storage access.

### Naive approach

The most naive approach is to use Merkle Patricia Trie directly in the EVM and to make all changes to the database during its execution. But how do we handle failed EVM executions? One solution can be to not delete any data from Merkle Patricia trie storage. Merkle Patricia tries are defined by its root hash so if we don't delete any data from the permanent storage we can always return to any root hash that has ever existed in the database. In our case, if the evm execution fails we will return to the root hash that existed before failed evm execution.

### Cache storage for EVM execution

We can improve our initial approach by introducing a simple in-memory cache for storage modifications during EVM execution. All data will be read from the permanent Merkle Patricia trie storage but all modifications will be made to in-memory storage. This in-memory cache can be much simpler than Merkle tree data structure so it'll speed up EVM operations that write data to the storage since Merkle Patricia trie updates are quite expensive operations. We can improve the EVM cache read speed by caching data that was ever read during current EVM run since again Merkle Patricia trie reads traverse Merkle tree and it's an expensive operation.

When should we commit modifications stored in the EVM cache? To answer this question we have to understand what transaction receipt is. A transaction receipt is a data structure that contains some information about the executed transaction (used gas etc). It also contains state root hash right after transaction execution (for pre-Byzantium blocks). So as you can see we have to commit changes after transaction execution to get state root hash from the Merkle Patricia trie storage. But this approach has one problem.

### Cache storage for block

The problem with the previous approach is that we should commit cache after every transaction. It's ok if we'll be importing only valid blocks. But in the real network, some peers will share invalid blocks and we shouldn't commit changes made by their transactions.

We can introduce a second level of cache. It will be in-memory trie where transaction changes will be committed from the EVM cache. It will solve the problem with invalid block data because we will be able to simply discard in-memory tries from these blocks.

In-memory trie will read not existing in-memory nodes from permanent disk storage but it will write all changes to memory. So another optimization we will get from in-memory trie is that update/read speed is much faster from memory than disk.

### Optimisations

#### bacth updates

#### Single trie traversal