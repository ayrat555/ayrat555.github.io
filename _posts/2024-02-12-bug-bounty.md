---
title: Bugs to Riches: My TON Bug Bounty Story
date: 2024-02-12
summary:
categories: blockchain ton
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/2024-02-10-bounty.png
---

In my line of work, I engage with various blockchains, including [TON](https://ton.org/). Working with the TON blockchain has been particularly enjoyable for me. On my blog, you'll find several posts about it:

- [TON](https://www.badykov.com/elixir/blockchain/ton/): This post reflects my general thoughts on TON, which became quite popular and even got translated into [Chinese](https://mp.weixin.qq.com/s/PfwLnv9Kcl8N8xTvMChInw).
- [Deploying a simple Telegram Open Network smart contract](https://www.badykov.com/ton/deploying-simple-ton-smart-contract/]): Here, I share a postmortem of my participation in the TON hackathon in 2019.
- [Submitting transactions in TON with Elixir](https://www.badykov.com/elixir/ton/submitting-ton-transaction/): This post provides a step-by-step guide on submitting transactions using the [TON Elixir library](https://github.com/ayrat555/ton).

The last post in this list introduces the Elixir library I developed for interacting with the TON blockchain - [ton](https://github.com/ayrat555/ton). It took me a couple of months to write and test, as I had to port the functionality from the official library written in another language.

In this post, I'll share a short story about how I received a bug bounty for uncovering an issue in the TON software used in our production system.

## Ton Problems

As mentioned earlier, my job involves handling various blockchains, integrating them into our system, and ensuring smooth transaction processing.

Our integration with TON had been running smoothly without any significant issues until recently. However, we encountered two problems related to the TON blockchain in the last couple of months. [The first issue](https://telegra.ph/7-Dec-2023-12-07) was a significant challenge caused by a surge in transaction volume, leading to the blockchain's inability to accept new transactions. The second issue, although less impactful overall, was trickier to debug and resolve. It resulted in a bug bounty from the TON foundation.

Now, let's delve into the specifics of the problem.

## Bug bounty problem

A user contacted our customer support team about a missing deposit in our system. Initially, I suspected a problem with our application logic, possibly an overlooked edge case. However, after further investigation, I found that the API of [the service](https://github.com/toncenter/ton-indexer) we deployed to synchronize blockchain data was failing to record the user's transaction.

I then raised an [issue](https://github.com/toncenter/ton-indexer/issues/44) on the [ton-indexer's](https://github.com/toncenter/ton-indexer) repository and reached out to one of the developers, whose contact details I had from previous interactions related to the project. This project is maintained by the Ton Foundation and is used by us to synchronize blockchain data. They pointed out that [the official instance of the ton-indexer](https://toncenter.com/api/index/) showed the missing transaction in its response. By comparing the responses from our instance and the official one, I found the difference in five transactions.

Finally, the developer of the indexer service identified and resolved the issue with pagination -  [the fix](https://github.com/toncenter/ton-indexer/pull/45).

## Conclusion

I received a bug bounty of 200 tons (~$400 at the time of writing) for the discovery from the ton foundation. It was a pleasantly unexpected and fulfilling coincidence. It seems my work not only benefits my company but also the TON community.

I want to give a special thanks to Valeria Shamanova for helping finding this issue; the bounty was split with her.

Thank you for reading this post. As a bonus, I'm sharing my hobby of creating NFTs, which combines my love for drawing and blockchain. Here are the collections I've created on the TON blockchain so far. [Collections I created on the ton blockchain so far](https://getgems.io/user/EQADLRBbbfImjN1yaN6fqWPwkO3sN2fCdg8BD_g8LW_8Dj-G#collections)
