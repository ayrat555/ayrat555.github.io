---
title:      Message calls in Ethereum
date:       2018-06-17
summary:    Summary of existing message calls in Ethereum
categories: ethereum

redirect_from:
  - /ethereum/2018/06/17/message-calls-in-ethereum/
---

![dolls](/images/2018-06-17-dolls.png)

### Background

In the Ethereum Virtual Machine (EVM), smart contracts can call other contracts and trigger their execution. This allows for complex contracts with deeply nested message calls.

Below I provide a short summary related to the message calls section of [Ethereum's Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf). Then, I describe each message call in more detail.

### Message call

As outlined in the Yellow Paper, message call is the function `Θ` that takes 12 arguments. See the paper for details associated with each value.

![theta](images/2018-06-17-theta.png)

**`Θ` Input values:**

01. `σ` - world state
02. `s` - sender's address
03. `o` - transaction originator's address
04. `r` - recipient's address
05. `c` - code owner's address
06. `g` - available gas
07. `p` - gas price
08. `v` - value to be be transferred from sender to recipient
09. `v with overline`  - value used in the call's execution context
10. `d` - input data of the call
11. `e` - present depth of the message call stack
12. `w` - the permission to make modifications to the state

**Output values:**

1. `σ` - new state
2. `g` - remaining gas
3. `A` - substate
4. `z` - execution halt status
5. `o` - output data

Message call starts with transferring value `v` from sender's address `s` to recipient's address `r`. All other input values are used in the execution context of the call.

If the message call stack `e` exceeds 1024 or there is not enough value to transfer `v` from sender's address, the message call is not executed.

### Message call types

EVM has 4 types of message call instructions:

1. `CALL`
2. `CALLCODE`
3. `DELEGATECALL`
4. `STATICCALL`

#### CALL

`CALL` is the most straightforward type of message call. It takes 7 arguments, the last four of which are used for reading input data from the memory and writing the result of the execution to the memory. The first three arguments:

1. `μ0` - gas.
2. `μ1` - recipient's address.
3. `μ2` - value that will be transferred from sender to recipient.

So our message call function `Θ` will look like `Θ(σ, I_a, I_o, μ1, μ1, C_callgas, I_p, μ2, μ2, i, I_e + 1, I_w)` where:

* `I` - current execution environment
* `C_callgas` - cost of the call operation that is calculated using `μ0` and `σ`
* `i` - input data read from memory

We can see the recipient and code owner are the same account, and the same value is used for transferring and execution.

#### CALLCODE

`CALLCODE` is used for a message call into the current account with an alternative account’s code. Arguments of this instruction are identical to `CALL` except for the **forth parameter**, where the recipient is the same as the account.

CALLCODE function:

`Θ(σ, I_a, I_o, I_a, μ1, C_callgas, I_p, μ2, μ2, i, I_e + 1, I_w)`.


#### DELEGATECALL

This instruction was introduced by [Ethereum Improvement Proposal (EIP) №7](https://eips.ethereum.org/EIPS/eip-7). `DELEGATECALL` is used for a message call into the current account with an alternative account’s code, but the current values for sender and value are persistent.

As described in the EIP, propagating the sender and value from the parent scope to the child scope makes it easier for a contract to store another address as a mutable source of code and ‘‘pass through’’ calls to it, as the child code would execute in essentially the same environment as the parent.

This instruction omits a `μ2` argument, otherwise it is the same as `CALL`.

DELEGATECALL function:

`Θ(σ, I_s, I_o, I_a, μ1, μ0, I_p, 0, μ2, i, I_e + 1, I_w)`


#### STATICCALL

This instruction was introduced by [EIP №214](https://eips.ethereum.org/EIPS/eip-214). As stated in the EIP's summary, the opcode can be used to call another contract (or itself) while disallowing any modifications to the state during the call (and its subcalls, if present). Any opcode that attempts to modify the state results in an exception rather than performing the modification.

`STATICCALL` is equivalent to `CALL` except `μ2` is replaced with 0 and the `w` flag is false.

STATICCALL function:

`Θ(σ, I_a, I_o, μ1, μ1, C_callgas, I_p, 0, 0, i, I_e + 1, false)`

**Note:** The `w` flag is referred to as the `STATIC` flag and was introduced specifically for `STATICCALL`.


### See also

- [Mana Project](https://github.com/poanetwork/mana)
- [Ethereum's Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
