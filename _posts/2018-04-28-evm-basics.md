---
layout:     post
title:      Ethereum Virtual Machine in Elixir
date:       2018-03-25
summary:    EVM execution basics in Elixir
categories: elixir
---

![ethereum](https://i.imgur.com/zaIytOk.jpg)

### Introduction

This week we, at Exthereum (Mana?) project, finally made all official EVM tests pass. I think it's time to write a post describing its inner magic with code examples from our project. So every crypto alchemist<a name="footnote1">1</a> interested in Ethereum can follow along.

### Background

#### EVM

From Ethereum offical web site: Ethereum is a decentralized platform for applications that run exactly as programmed without any chance of fraud, censorship or third-party interference.

So what's Ethereum Virtual Machine? To put it simply, it's virtual machine that executes machine code compiled from high level smart contracts programming languages. There are several languages that are compiled to EVM machine code: Solidity (similar to Javascript), LLL (Lisp Like Language) etc.

EVM is a simple stack machine. Its memory is a word-addressed byte array. The stack has a maximum size of 1024.

EVM can execute 132 operation that are divided into 11 categories:

1. Stop and Arithmetic Operation

   Examples:
     - `ADD` - adds two first items on stack saves result and to first stack item.
     - `STOP` - halts execution.

2. Comparison & Bitwise Logic Operation

   Examples:
     - `LT` - less-than comparison. It compares first two items on stack, saves 0 if the first item is lesser than the second item, saves 1 otherwise.
     - `AND` - bitwise AND operation.

3. SHA3

   Examples:
     - `SHA3` - computes Keccak-256 hash.

4. Environmental Information

   Examples:
     - `ADDRESS` - gets address of currently executing account.
     - `CALLDATACOPY` - copies input data in current environment to memory.

5. Block Information

   Examples:
     - `NUMBER` - gets the block’s number.
     - `TIMESTAMP` - gets the block’s timestamp.

6. Stack, Memory, Storage and Flow Operation

   Examples:
     - `POP` - removes item from stack.
     - `JUMP` - alters program counter.

7. Push Operation

   Examples:
     - `PUSH1` - places 1 byte item on stack.
     - `PUSH17` - places 17 byte item on stack.

8. Duplication Operation

   Examples:
     - `DUP1` - duplicates 1st stack item.
     - `DUP2` - duplicates 2st stack item..

9. Exchange Operations

   Examples:
     - `SWAP1` - exchanges 1st and 2nd stack items.
     - `SWAP16` - exchanges 1st and 17th stack items.

10. Logging Operations

    Examples:
      - `LOG0` - appends log record with no topics
      - `LOG1` - appends log record with one topic.

11. System operations

    Examples:
      - `RETURN` - halts execution returning output data.
      - `CREATE` - Creates a new account with associated code.

Every operation consumes some amount of gas. Gas is the internal pricing for code execution in EVM. If not enough gas was provided than the execution halts with out of gas error.

#### How Ethereum's yellow paper corresponds to programing code

This section is intended for acute readers that read Yellow Paper and want to know how our programming code in Elixir corresponds to execution model described in the paper. If you haven't read Ethereum's yellow paper, you can skip this section or return to it later.

Here's exerpt from Ethereum's yellow paper:

![execution-model](https://i.imgur.com/5xJ9lFl.jpg)

Let's exemine programming code file `lib/evm/vm.ex':

- `EVM.VM.run/2` - Ξ function from the paper.
- `EVM.VM.exec/3` - X function from the paper.
- `EVM.VM.cycle/3` - O function from the paper.
- `EVM.Functions.is_exception_halt?/2` - Z function.

### Examples

#### Prerequisites

To follow along executing code examples you need to clone our [implementation of EVM](https://github.com/exthereum/evm) (Mana repo?) from GitHub:

```bash
git clone https://github.com/exthereum/evm
```

Fetch dependencies:

```bash
mix deps.get
```

And start Elixir REPL:

```bash
iex -S mix
```

#### Code examples

So finally we came to examples.

##### Example 1




# Footnotes

<sup>[1](#footnote1)</sup> Elixir developer interested in blockchain technologies
