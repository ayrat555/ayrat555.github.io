---
title: TON
date: 2024-02-12
summary: My thoughts on Ton and TON SDK for Elixir
categories: blockchain ton
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/2023-01-08-ton.jpeg
---

Due to the nature of my work, I work with different blockchains. One of these blockchains is [TON](https://ton.org/). I really enjoyed working with the TON blockchain. You can find several posts about it on my blog:

- [TON](https://www.badykov.com/elixir/blockchain/ton/) - a general thoughts about ton. the post got so popular it got translated to [the chinese language](https://mp.weixin.qq.com/s/PfwLnv9Kcl8N8xTvMChInw)
- [Deploying a simple Telegram Open Network smart contract](https://www.badykov.com/ton/deploying-simple-ton-smart-contract/]) - Postmortem of my participation in the ton hackathon in 2019.
- [Submitting transactions in TON with Elixir](https://www.badykov.com/elixir/ton/submitting-ton-transaction/) - step by step guide how to submit transactions with the [ton](https://github.com/ayrat555/ton) elixir library

The last post in this list gives an intro to the elixir library that I wrote for interacting with the ton blockchain - [ton](https://github.com/ayrat555/ton). It took me a couple of months to write and test it porting the official library written in another language.

In this post I'll share a short story how I got a bug bounty for the issue I discovered in the ton software that we use in our production system.

## Ton Problems

As I said earlier my responsibilites at work include working with different blockchains, integrating them to our system, making sure that transactions are getting processed correctly.

Our ton integration has been working fine without any major issues until recently. Recently we had two issues related to the TON blockchain. [The first issue](https://telegra.ph/7-Dec-2023-12-07) was a big problem related to the increase in the number of transactions which caused the blockchain to stop accepting new transactions. The second problem had a smaller overall impact but actually got a bug bounty for the ton foundation.

Now let's describe the problem.

## Bug bounty problem

A user complained to our customer support team that his deposit didn't appear on our system. Initially I thought the issue is on our applicate logic side, maybe it's an edge case that we didn't consider. Uplon further investigation, I discovered the API of [the service](https://github.com/toncenter/ton-indexer) that we deployed to synchronize the blockchain data was missing the user's transaction.

Then I created an [issue](https://github.com/toncenter/ton-indexer/issues/44) in the [ton-indexer's](https://github.com/toncenter/ton-indexer) repo. And contracted one of the developers of this project since I had his contract from the previous issues/questions about his project. He pointed out that [the offical instance of the ton-indexer](https://toncenter.com/api/index/) has the missing transaction in its response. Comparing the responses from our instance and the official one , I found that 5 transactions are different.

And finally theh developer of the indexer service found the problem in the pagination and [fixed it](https://github.com/toncenter/ton-indexer/pull/45)

## Conclustion

For the discovery, report I received 200 tons (~400$ at the time of writing this post).  It was very unexpected and at the same time fullfilling coincednce. It seems my work in my company not only beneficial to the company but the ton commninuty as well.

Thank you for reading this post. As a bonus, I'm sharing here my hobby creating nfts which combines my love to drawing and blockchain. [Collections I created on the ton blockchain so far](https://getgems.io/user/EQADLRBbbfImjN1yaN6fqWPwkO3sN2fCdg8BD_g8LW_8Dj-G#collections)
