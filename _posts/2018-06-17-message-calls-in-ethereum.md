---
layout:     post
title:      Message calls in Ethereum
date:       2018-06-17
summary:    Summary of existing message calls in Ethereum
categories: ethereum
---

![dolls](https://i.imgur.com/yCliFxX.png)

### Background

Ethereum Virtual Machine (EVM) allows smart contracts to call other contracts triggering their execution so complex contracts may have deep nested message calls working under the hood. This post will try to give description of all currently available message calls in Ethereum.

Firstly, I will give a small summary without going into details of related to message calls section of the [Ethereum's Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf). Then, I'll describe each message call.

### Message call

As described in the Yellow Paper, message call is the function `Θ` that takes 12 arguments.

![theta](https://i.imgur.com/kpxGzGK.png)

Input values:

01. `σ` - world state.
02. `s` - sender's address.
03. `o` - transaction originator's address.
04. `r` - recipient's address.
05. `c` - code owner's address.
06. `g` - available gas.
07. `p` - gas price.
08. `v` - value that will be trasferred from sender to recipient.
09. `v with overline`  - value that is used in the call's execution context.
10. `d` - input data of the call.
11. `e` - present depth of the message-call stack.
12. `w` - the permission to make modifications to the state.

Output values are:

1. `σ` - new state.
2. `g` - remaining gas.
3. `A` - substate.
4. `z` - execution halt status.
5. `o` - output data.

Message call starts with transferring value `v` from sender's balance `s` to recipient's balance `r`. All other input values are used in the execution context of the call.

If the message-stack `e` will exceed 1024 or there is not enough value to transfer `v` from sender's address, the message call is not executed.

### Message call types

EVM has 4 types of message call instructions:

1. `CALL`
2. `CALCODE`
3. `DELEGATECODE`
4. `STATICCALL`

#### `CALL`

#### `CALLCODE`

#### `DELEGATECODE`

#### `STATICCALL`
