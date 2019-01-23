---
layout:     post
title:      Ethereum Contract Application Binary Interface (ABI) in Elixir
date:       2019-01-25
summary:    ABI in Elixir
categories: elixir
---

![smart-contract](https://i.imgur.com/JiTJe6W.jpg)

### Background

Nowadays Ethereum is one of the most successful blockchain platforms. Foremost things that made it possible are Ethereum Virtual Machine (EVM) and smart contracts that are executed in EVM.

Smart contracts are little programs written in high-level languages like Solidity or LLL specifically created for Ethereum. Firstly contract's code is compiled to EVM bytecode and only then it can be executed in EVM.

Application Binary Interface (ABI) is the standard way to interact with contracts in the Ethereum ecosystem, both from outside the blockchain and for contract-to-contract interaction. An account wishing to use a smart contract's function uses the ABI to hash the function definition so it can create the EVM bytecode required to call the function.

In this post, I'll describe how a function and its params are encoded to ABI format and how ABI encoding/decoding can be coded in Elixir.

### ABI in Ethereum

### Specification

### Elixir implementation

### Acknowledgement
