---
layout: post
title: Testing Ethereum reorgs
date: 2020-09-27
summary: A couple of notes on Ethereum reorgs
categories: ethereum
---

![img](/images/2020-09-28-testing-ethereum-reorgs.jpg)

### Chain reorganizations

<blockquote>
  <p>
The term "blockchain reorganization" is used to refer to the situation where a client discovers a new difficulty wise-longest well-formed blockchain which excludes one or more blocks that the client previously thought were part of the difficulty wise-longest well-formed blockchain. These excluded blocks become orphans.
  </p>
  <footer><cite title="Bitcoin Wiki">Bitcoin Wiki</cite></footer>
</blockquote>

This quote is from Bitcoin wiki but the term is the same for all blockchains including Ethereum. Note that in this post chain and blockchain are interchangeable.

Chain reorganizations can inflict significant damage to your application if the application makes decisions based on the latest blockchain state. By the application I mean an external observer, not a smart contract which state is stored in the chain. Imagine the application released some value to network based on the state of the contract during a reorganization. The required state change happened only in the reorged chain, it didn't happen in the consensus chain. So the application released value without fulfilling conditions.

Let's see how these situations can be mitigated and tested.

### Solution

A chain reorganisation can be any number of blocks. Although recently Ethereum Classic suffered a 3,693-blockchain reorganization, chain reorganization longer than a couple of blocks are rare. To mitigate reorganizations you can just check that your assumptions are true for a reasonable number of blocks. Crypto exchanges use this approach. In their terms, it's called a confirmation. Confirmations are a measure of how many blocks have passed since a transaction was added to a blockchain.

### Testing

The easiest way to trigger reorgs and test your assumptions on a reorged chain is using Docker. One approach includes two connected to each other Ethereum nodes running behind a load balancer. Docker provides an ability to pause/unpause running containers, we would need this feature. Long story short - by pausing/unpausing nodes, you can make state on each node differ so after nodes are connected, a reorg happens.

Let's examine a pseudocode:

```elixir
      # fetch a block before the reorg
      {:ok, block_before_reorg} = Client.get_latest_block_number()

      # pause the first node
      pause_container!(@node1)

      # generate 4 block on the the second node because the first node is paused
      :ok = Client.wait_until_block_number(block_before_reorg + 4)

      # execute the chain change function on the second node
      func.()

      # get the latest block on the second on node after the state change
      {:ok, block_on_the_first_node} = Client.get_latest_block_number()

      # pause the second node, unpause the first node
      pause_container!(@node2)
      unpause_container!(@node1)

      # generate 4 block on the the first node because the seoncd node is paused
      :ok = Client.wait_until_block_number(block_before_reorg + 4)

      # execute the chain change function on the first node
      response = func.()

      # wait 4 more blocks so the chain length is longer
      :ok = Client.wait_until_block_number(block_on_the_first_node + 4)

      # reorg should happen when nodes will find each other,  the second node blocks are reorged
      unpause_container!(@node2)
      unpause_container!(@node1)
```

### Conclusion

There is almost no information about reorg testing for developers. I hope this post is helpful for anyone trying to test assumptions on the reorged chain.
