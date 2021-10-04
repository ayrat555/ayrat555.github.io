---
title: Deploying a simple Telegram Open Network smart contract
date: 2019-10-20
summary: The first look at TON
categories: ton

redirect_from:
  - /ton/2019/10/20/deploying-simple-ton-smart-contract/
---

![img](/images/2019-10-20-ton.jpg)

### Intro

The last couple of months have been pretty busy for the blockchain community. In June 2019 Facebook announced that they're working on Libra. I have a [post](/blockchain/2019/06/29/libra-facebooks-ethereum-clone/) about it with comparison to Ethereum. Now (the end of September - the beginning of October 2019) Telegram released [technical papers](https://test.ton.org/) and [the project](https://github.com/ton-blockchain/ton) of Telegram Open Network (TON) - a blockchain by Telegram. If you don't know what Telegram is, Telegram is a messaging service that makes emphasis on security.

There are millions of Telegram users worldwide. So if the company releases its blockchain, it will be available for all Telegram users and TON may become the most massively adopted blockchain platform in the world. That's why TON is also a very promising technology to learn for developers.

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

This language is very practical. It won't be hard to understand for any seasoned developer.

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

This example is not very hard to understand. But in reality even relatively simple expressions are very cryptic for an untrained developer. For example:

```
{ <b swap $, b> <s  public_keys_dict@ swap 16  udict!+  drop public_key# 1+  2 'nop does : public_keys_dict } : add_public_key
```

This expression adds a new key-value pair to the dictionary. Fift may be compared to Lisp languages. Although many people hate Lisp for its brackets, they make code much more readable.

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

There are two main issues related to  using  bleeding-edge technologies:
- There are a lot of bugs.
- There are not enough tools for developer happiness (testing, debugging, code highlighting for your favorite editor, etc)

With TON smart contracts you will experience all points described above. But we have to cut some slack to the TON platform developers because they resolve GitHub issues pretty fast.

### Money giver contract

Let's create a simple smart contract called `Money Sender`.

#### Money sender contract

It's a simple wallet smart contract that can be used to store, send and receive grams - currency in the TON network. It's meant to understand the full process of creating a smart contract in Func, deploying it using Fift and writing external messages to it in Fift.

It consists of several files located in `money_sender` folder:
1. `money_sender.fc` - the contract that proxies internal messages. It doesn't have any checks (seqno, signature) because it's used only as an example. It increments its seqno after every request.

```
() recv_external(slice in_msg) impure {
  var cs = in_msg;
  accept_message();
  var ds = get_data().begin_parse();
  var stored_seqno = ds~load_uint(32);
  ds.end_parse();
  cs~touch_slice();
  if (cs.slice_refs()) {               ;; if the message have any internal messages, let's proxy them.
    var mode = cs~load_uint(8);
    send_raw_message(cs~load_ref(), mode);
  }
  cs.end_parse();
  set_data(begin_cell().store_uint(stored_seqno + 1, 32).end_cell());
}
```
2. `money_sender_deploy.fif` - Fift script that deploys `Money Sender` to the specified workchain. It accepts only one parameter `workchain_id`.

```
#!/usr/bin/env fift -s
"TonUtil.fif" include
"Asm.fif" include

{ ."usage: " @' $0 type ." <workchain-id>" cr
  ."Deploys money sender" cr
  1 halt
} : usage

$# 1 < { usage } if

$1 parse-workchain-id =: workchain_id

PROGRAM{
  "money_sender.fif" include
}END>c
// code
<b 0 32 u, b> // data

dup .s
2 boc+>B dup Bx. cr
"init_data.boc" tuck B>file
drop
cr ."After boc" cr
.s

null // no libraries

<b b{0011} s, 3 roll ref, rot ref, swap dict, b>  // create StateInit

dup ."StateInit: " <s csr. cr
dup hash workchain_id swap 2dup 2constant wallet_addr
."new wallet address = " 2dup .addr cr
2dup "sender.addr" save-address-verbose
."Non-bounceable address (for init): " 2dup 7 .Addr cr
."Bounceable address (for later access): " 6 .Addr cr

<b b{1000100} s, wallet_addr addr, b{000010} s, swap <s s, b{0} s,  b>

 dup ."External message for initialization is " <s csr. cr
 2 boc+>B dup Bx. cr
 "create-money-sender-query.boc" tuck B>file
 ."(Saved wallet creating query to file " type .")" cr

```
3. `send_money.fif` - Fift script that creates a request to send grams from `Money Sender` to the specified address. It accepts 2 parameters: amount in grams and receiver address.

```
#!/usr/bin/env fift -s
"TonUtil.fif" include
"Asm.fif" include

{ ."usage: " @' $0 type ."  <amount> <address>" cr
  ."Transfers the specified <amount> to <address> from already deployed contract" cr
  1 halt
} : usage

$# 2 < { usage } if

$1 parse-int =: amount
$2 true parse-load-address =: bounce 2=: dest_addr

// "0QAu6bT9Twd8myIygMNXY9-e2rC0GsINNvQAlnfflcOv4uVb" true parse-load-address =: wallet_bounce 2=: wallet_addr

Masterchain 0x2E932B8BFC23F7CD0455FF0848CFCCB9E95EB0BA62D93883A665F61612F345AC
2constant wallet_addr
 ."Test giver address = " wallet_addr 2dup .addr cr 6 .Addr cr

cr ."Transferring " amount .
  ."to " dest_addr  .addr
  ."from " wallet_addr .addr cr

<b b{01} s, bounce 1 i, b{000100} s, dest_addr addr,
  amount Gram, 0 9 64 32 + + 1+ 1+ u, 0 32 u, "GIFT" $, b>



<b 1 8 u, swap ref, b> .s

dup
2 boc+>B dup Bx. cr
"send_value_message.boc" tuck B>file
drop

<b b{1000100} s, wallet_addr addr, 0 Gram, b{00} s,
   swap <s s, b>

2 boc+>B dup Bx. cr
"send_money.boc" tuck B>file
."(Saved to file " type .")" cr
```

4. `money_sender_test.fif` - tests for contract TVM execution using different input messages. it's used only for testing.

```
#!/usr/bin/env fift -s
"TonUtil.fif" include
"Asm.fif" include

PROGRAM{ "money_sender.fif" include }END>s constant code // code
"init_data.boc" file>B B>boc constant storage // data

//  let's send an empty message and check that seqno is changed in the storage

<b b> <s constant empty_message

empty_message -1 code storage runvm

cr ."Storage after empty message: " <s 32 i@  . drop .s cr

// let's send value transfer message

"send_value_message.boc" file>B B>boc <s constant value_message

value_message -1 code storage runvm .s

cr ."Storage after value transfer message: " <s 32 i@  . drop .s cr

```

How to deploy and use this contract:
1. Compile `money_sender.fc` to fift.
2. Deploy it using `money_sender_deploy.fif` (Run the script and upload `.boc` file).
3. Send some Grams to it.
4. Set deployed `Money Sender` address in `send_money.fif` (line 17).
5. Use `send_money.fif` to send grams.
