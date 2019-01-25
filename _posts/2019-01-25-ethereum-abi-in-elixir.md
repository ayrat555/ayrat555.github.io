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

### Specification

Let's provide formal definition excerpts from official [ABI specification](https://solidity.readthedocs.io/en/develop/abi-spec.html).

#### Function selector encoding

The first four bytes of the call data for a function call specifies the function to be called. It is the first (left, high-order in big-endian) four bytes of the Keccak-256 (SHA-3) hash of the signature of the function.

#### Argument encoding

Starting from the fifth byte, the encoded arguments follow.

Types can be static and dynamic. Dynamic types are:
- `bytes`
- `string`
- `T[]` for any `T`
- `T[k]` for any dynamic `T` and any `k >= 0`
- `(T1,...,Tk)` if Ti is dynamic for some `1 <= i <= k`

Examples of static types:

- `uint<M>`: unsigned integer type of `M` bits, `0 < M <= 256`, `M % 8 == 0`. e.g. `uint32`, `uint8`, `uint256`.
- `bool`: equivalent to `uint8` restricted to the values `0` and `1`. For computing the function selector, `bool` is used
- `address`: equivalent to `uint160`, except for the assumed interpretation and language typing. For computing the function selector, `address` is used.
- `function`: an address (20 bytes) followed by a function selector (4 bytes). Encoded identical to bytes24.

Definition: For any ABI value `X`, we recursively define `enc(X)`, depending on the type of `X` being

- `(T1,...,Tk)` for `k >= 0` and any types `T1`, …, `Tk`
- `enc(X) = head(X(1)) ... head(X(k)) tail(X(1)) ... tail(X(k))` where `X = (X(1), ..., X(k))` and `head` and `tail` are defined for `Ti` being a static type as `head(X(i)) = enc(X(i))` and `tail(X(i)) = ""` (the empty string) and as
`head(X(i)) = enc(len(head(X(1)) ... head(X(k)) tail(X(1)) ... tail(X(i-1)) ))` `tail(X(i)) = enc(X(i))` otherwise, i.e. if Ti is a dynamic type.

Let's give examples of a couple of types encoding:

- `uint<M>`: `enc(X)` is the big-endian encoding of `X`, padded on the higher-order (left) side with zero-bytes such that the length is 32 bytes.
- `bool`: as in the `uint8` case, where 1 is used for true and 0 for false
- `address`: as in the uint160 case
- `bytes`, of length `k` (which is assumed to be of type uint256): `enc(X) = enc(k) pad_right(X)`, i.e. the number of bytes is encoded as a `uint256` followed by the actual value of X as a byte sequence, followed by the minimum number of zero-bytes such that `len(enc(X))` is a multiple of 32.

#### Function Selector and Argument Encoding

A call to the function `f` with parameters `a_1`, ..., `a_n` is encoded as `function_selector(f) enc((a_1, ..., a_n))` and the return values `v_1`, ..., `v_k` of f are encoded as `enc((v_1, ..., v_k))` i.e. the values are combined into a tuple and encoded.

### Elixir implementation



### Acknowledgement
