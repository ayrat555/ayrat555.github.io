---
layout: post
title: Deploying a simple Telegram Open Network smart contract
date: 2019-10-20
summary: The first look at TON
categories: ton
---

### Intro

The last couple of months have been pretty busy for blockchain community. In June, 2019 Facebook announced that they're working on Libra. I have a [post](https://www.badykov.com/blockchain/2019/06/29/libra-facebooks-ethereum-clone/) about it with comparision to Ethereum. Now (the end of September, 2019) Telegram released [technical papers](https://test.ton.org/) and [the project](https://github.com/ton-blockchain/ton) of Telegram Open Network (TON) - a blockchain by Telegram. If you don't know Telegram, is a messaging service that makes emphasis on security.

There are millions of Telegram users worldwide. So if the company releases its blockchain, it will be available for all Telegram users and TON may be become the most massively adopted blockchain platform in the world. That's why TON is also very promising technology to learn for developers.

Let's create and deploy a simple TON smart contract so you can get your hands dirty.

### Fift and func

TON smart contracts can be written in three languages which differ in abstraction levels:

- Func - a high-level smart contracts language similar to C++.

 Example:
```
  var cs = in_msg;
  var msg_seqno = cs~load_uint(32);

  var ds = get_data().begin_parse();
```

- Fift - a stack-based prefix notation language.

Example with execution comments:

```
1 1 + 1 -

// Stack 1  ; 1 + 1 -
// Stack 1 1; + 1 -
// Stack 2  ; 1 -
// Stack 2 1; -
// Stack 1;
```

- Fift assembly - a low-level language that Func programs are compiled to.

```
DECLPROC recv_internal
DECLPROC recv_external
recv_internal PROC:<{
  DROP
}>
recv_external PROC:<{
  ACCEPT
  c4 PUSH
```


#### Testing and debugging

### Money giver contract

#### Contract
#### Deploy
#### Send Money Request
