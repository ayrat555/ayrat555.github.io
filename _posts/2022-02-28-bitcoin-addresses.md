---
title: Bitcoin address types
date: 2022-03-06
summary: My elixir HD wallet library
categories: bitcoin
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/2022-03-06-btc.jpeg
---

Recently I started diving into bitcoin more. One of the interesting things I discovered is bitcoin has several types of address formats.

For example:

- `1JfhAmwWjbGJ3RW2hjoRdmpKaKXgCjSwEL` - starts with 1 (`mainnet`)
- `397Y4wveZFbdEo8rTzXSPHWYuamfKs2GWd` - starts with 3 (`mainnet`)
- `bc1qc89hn5kmwxl804yfqmd97st3trarqr24y2hpqh` - starts with bc1 (`mainnet`)
- `tb1qx3ppj0smkuy3d6g525sh9n2w9k7fm7q3vh5ssm` - starts with tb1 (`testnet`)

In this post, I'll give a brief overview of common bitcoin address types that are used by regular users to receive and send bitcoins.

## Addresses

Addresses are created from the user wallet's public keys which are derived from the associated private keys. You can find out more about crypto wallets in my [previous post](/elixir/cryptopunk/).

Public keys are just points on the elliptic curve. They can be represented in two formats:

- uncompressed - both x and y coordinates are present
- compressed - only x coordinate is present in public key (y coordinate can be easily calculated from this)

Both compressed and uncompressed public keys can be used to create an address but these addresses will be different. In the initial bitcoin implementation, only uncompressed keys were used but now it's advised to use compressed public keys for all types of addresses.

## Outputs

Crypto wallet applications abstract the complexity of sending and receiving bitcoins. And they create the illusion that bitcoins are sent to addresses directly. But in reality, the process is more complex.

A person sending funds creates a new output that can be unlocked only with keys that are associated with the receiver address. A BTC output can be considered a bank bill that can't be spent partially. So addresses don't own bitcoin directly but own a set of outputs that they can unlock.

Bitcoin has a simple language that can be used to program outputs. For example, you can use it to create multisig outputs that can't be unlocked by a single user but only with private keys that belong to several users.

The same programming language is used to send and receive funds. The sender attaches a locking script to newly created outputs and the receiver has to provide an unlocking script to be able to spend these outputs.

Depending on a locking script bitcoin node recognizes several [standard output types](https://github.com/bitcoin/bitcoin/blob/bada9636d7f2efbc620fd89107baa2bf3e64a6b8/src/script/standard.cpp#L49). I suggest taking a look at [this reddit post](https://www.reddit.com/r/Bitcoin/comments/jmiko9/a_breakdown_of_bitcoin_standard_script_types/) which describes most of the script types.

## Types of addresses

Let's review the main address types used in user wallets. Hash160 used below is RIPEMD160 hash of the SHA256 hash of the provided data.

### P2PKH

Examples:

- `1JfhAmwWjbGJ3RW2hjoRdmpKaKXgCjSwEL` - mainnet
- `mkHGce7dctSxHgaWSSbmmrRWsZfzz7MxMk` - testnet

It's calculated as hash160 of the public key encoded in base58 format. It's considered legacy address type and it's used for receiving P2PKH (pay to public key hash) outputs.

Fees sending P2PKH transactions are higher compared to newer segwit compatible address types because transactions are larger for legacy addresses.

## Segwit (Segregated Witness) types

It's a protocol upgrade that was activated on 23 August 2017.

The unlocking part of the transaction (witness) now can be stored in a separate data structure. It allows to include more transactions per block.

### P2WPKH-P2SH

Examples:

- `397Y4wveZFbdEo8rTzXSPHWYuamfKs2GWd` - mainnet
- `2NFNttcoWjE7WUcByBqpPKkcjg8wzgnU5HE` - testnet

This type of address is meant to be used to generate SegWit transactions for wallets without native SegWit support as long as they support P2SH transactions.

Pay to script hash (P2SH) transactions were standardised in BIP 16. They allow transactions to be sent to a script hash (address starting with 3) instead of a public key hash.

It's calculated as hash160 of redeem script (`0014` + `hash160(public_key)`) - `hash160(0014 + hash160(public_key))`


### P2WPKH

Examples:

- `bc1qc89hn5kmwxl804yfqmd97st3trarqr24y2hpqh` - mainnet
- `tb1qx3ppj0smkuy3d6g525sh9n2w9k7fm7q3vh5ssm` - testnet

`P2WPKH` (pay to witness public key) addresses is similar to `P2PKH` addresses and they lock outputs to the hash of the public key. But this type of address supports SegWit.


It's calculated as bech32 encoded hash160 of the public key.


## Conclusion

This post only mentioned basic user wallet addresses. You may want to check out additional address types:

- P2SH - pay to script hash
- P2WSH - pay to witness script hash. it's similar to P2SH but supports SegWit
- P2TR - pay to the taproot
