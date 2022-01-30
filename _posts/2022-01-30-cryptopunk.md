---
title: We’re on the Brink of Cryptopunk
date: 2022-01-30
summary: My elixir HD wallet library
categories: elixir
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/2022-01-30-cover.jpeg
---

Only a few years ago cryptocurrencies were considered "money for nerds", a technology that doesn't have any potential. But now the blockchain became mainstream. Artists are selling their art as NFTs, Elon Musk keeps pumping Dogecoin in his tweets, many countries are in the process of developing regulation rules for cryptocurrencies.

Blockchain technology has many usages, but currently, it's mostly used as a payment platform.

Over my new year holiday, I developed a crypto wallet library for Elixir called [cryptopunk](https://github.com/ayrat555/cryptopunk). And in this post, I'll describe how you can use it to accept crypto payments in your elixir projects.

## Blockchain

To my surprise, even some seasoned software developers have a vague idea of how blockchain operates. So let me give a brief explanation.

Satoshi Nakomoto who developed the reference implementation of bitcoin brought three innovations together:

- a P2P Network - bitcoin network

In a peer-to-peer (P2P) network, each computer acts as both a server and a client—supplying and receiving files—with bandwidth and processing distributed among all members of the network

- a public transaction ledger - blockchain

A blockchain is a growing list of records, called blocks, that are linked together using cryptography. Each block contains a cryptographic hash of the previous block, a timestamp, and transaction data (generally represented as a Merkle tree).

- a mechanism for reaching globally decentralized consensus - proof-of-work algorithm

Proof-of-Work (PoW), requires computational power to solve a difficult but arbitrary puzzle to keep all nodes in the network honest.

## Crypto wallet

People not familiar with cryptocurrencies may think that a crypto wallet stores all your crypto money. But it's not a correct assumption.

Crypto wallet is an application that has the following features:

1. it stores your private keys
2. it generates addresses from public keys derived from private keys to receive payments
3. it tracks balances of your addresses
4. it signs transactions with your private keys and submits them to the network

There are several types of crypto wallets:

- Nondeterministic (Random) Wallets

It's a collection of randomly generated private keys.

In the first version of the Bitcoin core client (main client) this type of wallet was used. But this kind of wallet is hard to manage, back up and import.

- Deterministic Wallets

In deterministic wallets all keys are derived from the same seed, the seed is randomly generated data.

So the seed is sufficient to backup, recover and import all your private keys and their associated addresses calculated from public keys.

### HD Wallet

A special kind of deterministic wallet is an HD wallet. HD wallets allow deriving a tree of keys

![wallet](/images/2022-01-30-hd-wallet.jpeg)

Cryptopunk implements this kind of wallet.

## Cryptopunk

Since bitcoin is the first widely used blockchain, all HD wallet features were suggested through BIPs (Bitcoin Improvement Proposals). Bitcoin Improvement Proposal (BIP) is a design document for introducing features or information to Bitcoin.

Let's go over each of them and see how they can be used in the cryptopunk library.

### [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

This BIP describes the implementation of a mnemonic code or mnemonic sentence - a group of easy to remember words - for the generation of deterministic wallets.

To generate a mnemonic use `Cryptopunk.create_mnemonic/1`:

```elixir
iex> Cryptopunk.create_mnemonic()


"lyrics maid special submit bid unhappy garlic bacon muffin pizza intact chest sunset learn eternal garment hotel shaft twist step around traffic police trophy"
```

It will generate a string with mnemonic words, by default the number of words is 24. But you can also generate a mnemonic with 12, 15, 18, 21 words.

A mnemonic should be kept secret since knowing it gives access to all your private keys and addresses. But as you can see a mnemonic is easy to backup and import into wallet applications.

The sole purpose of mnemonics is seed generation. To generate a seed from a mnemonic use `Cryptopunk.create_seed/2`:

```elixir
iex> Cryptopunk.create_seed("lyrics maid special submit bid unhappy garlic bacon muffin pizza intact chest sunset learn eternal garment hotel shaft twist step around traffic police trophy")

<<97, 111, 20, 241, 162, 255, 166, 244, 70, 152, 71, 159, 244, 110, 196, 236,
  243, 62, 115, 76, 144, 207, 95, 165, 86, 213, 33, 223, 32, 14, 130, 8, 176,
  41, 229, 162, 113, 52, 179, 241, 2, 230, 229, 186, 112, 66, 33, 110, 162, 150,
  78, 141, 6, 144, 49, 157, 222, 204, 223, 90, 14, 73, 62, 34>>
```

You can protect your seed with a password by passing it as a second argument. It will result in the generation of completely different seed data.

To generate a master key from a seed use `Cryptopunk.master_key_from_seed/1`:

```elixir
  iex> Cryptopunk.master_key_from_seed(seed)
  %Cryptopunk.Key{
     chain_code:
       <<153, 249, 145, 92, 65, 77, 50, 249, 120, 90, 178, 30, 41, 27, 73, 128, 74, 201,
         91, 250, 143, 238, 129, 247, 115, 87, 161, 107, 123, 63, 84, 243>>,
     key:
       <<50, 8, 92, 222, 223, 155, 132, 50, 53, 227, 114, 79, 88, 11, 248, 24, 239, 76,
         236, 39, 195, 198, 112, 133, 224, 41, 65, 138, 91, 47, 111, 43>>,
     type: :private,
     depth: 0,
     parent_fingerprint: <<0, 0, 0, 0>>,
     index: 0
   }
```

Although you already can use your master key in your blockchain transactions to accept and send payments, it's not advised to do it. You should derive child keys and use them instead.

### [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) and [BIP-43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)

These BIPs defines a logical hierarchy for deterministic wallets based on an algorithm described in BIP-32. We'll get to it a little bit later.

It defines 5 levels of derivation path - `m / purpose' / coin_type' / account' / change / address_index` :

- the first part is the type of derived key: `m` - private key, `M` - public key
- the second part is the purpose. it's set to 44, indicating that its subtree is created according to BIP44
- the third part is the coin type. so you can use the same master key / seed with different cryptocurrencies

   You can find the registered coin list [here](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)

- the fourth part is the account.

   Users can use these accounts to organize the funds in the same fashion as bank accounts

-  the fifth part is the change

   This field is specific to blockchains that use outputs. Outputs can;t be spent partially so some part of them should be returned to the same wallet as change.

   And on the programmatic level, you'll detect which addresses are payment addresses and which are change addresses based on this field.

- the last part is the index of the address

Apostrophes in the path indicate that hardened derivation is used.

The main purpose of BIP-43 and BIP44 is to make wallets compatible with each other. But of course, if you don't need compatibility, you can use any derivation path you want.

To parse a derivation path string use `Cryptopunk.parse_path/1`:

```elixir
iex> Cryptopunk.parse_path("m / 44' / 0' / 0' / 0 / 0")
{:ok, %Cryptopunk.Derivation.Path{account: 0, address_index: 0, change: 0, coin_type: 0, purpose: 44, type: :private}}
```

### [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)

BIP-32 defines the following child key derivation (CKD) functions:

- Private parent key → private child key
- Public parent key → public child key
- Private parent key → public child key

To derive a child key from a master key use `Cryptopunk.derive_key/2`:

```elixir
  iex> seed = <<98, 235, 236, 246, 19, 205, 197, 254, 187, 41, 62, 19, 189, 20, 24, 73, 206, 187, 198, 83, 160, 138, 77, 155, 195, 97, 140, 111, 133, 102, 241, 26, 176, 95, 206, 198, 71, 251, 118, 115, 134, 215, 226, 194, 62, 106, 255, 94, 15, 142, 227, 186, 152, 88, 218, 220, 184, 63, 242, 30, 162, 59, 32, 229>>
  iex> {:ok, path} = Cryptopunk.parse_path("m / 44' / 0' / 0' / 0 / 0")

  iex> seed |> Cryptopunk.master_key_from_seed() |> Cryptopunk.derive_key(path)
  %Cryptopunk.Key{chain_code: <<166, 125, 2, 213, 77, 88, 124, 145, 241, 251, 83, 163, 21, 11, 20, 34, 158, 157, 179, 147, 162, 212, 148, 89, 28, 92, 68, 126, 215, 79, 147, 159>>, depth: 5, index: 0, key: <<214, 231, 94, 203, 167, 219, 125, 43, 251, 91, 147, 51, 32, 146, 186, 215, 58, 45, 104, 58, 119, 114, 121, 238, 155, 215, 239, 189, 37, 236, 27, 70>>, parent_fingerprint: <<205, 94, 166, 92>>, type: :private}
```

In practice, you won't have to think about what type of derivation you should use because `BIP-44` already defines a standard way that combines hardened and non-hardened derivations.

## Key features of cryptopunk

### Rust Nifs for number-intensive calculations

Cryptopunk extensively uses Rust NIFs:

- [ex_secp256k1](https://github.com/omgnetwork/ex_secp256k1) - secp256k1 ellictic curve functions
- [ex_pbkdf2](https://github.com/ayrat555/ex_pbkdf2) -  Password-Based Key Derivation Function v2 (PBKDF2). It's used for seed creation
- [ex_keccak](https://github.com/tzumby/ex_keccak) - Keccak hash. It's used for Ethereum address creation

The only disadvantage of using Rust Nifs is you have to bring Rust into your project. But on the other hand, C Nif requires a couple of dependencies to be installed.

So Rust can be considered just one more dependency.

### Support of several cryptocurrencies

Currently, cryptopunk supports addresses for Ethereum, Bitcoin and Dogecoin.

More currencies and bitcoin address types will be added soon.

## Conclustion

Hopefully, this post will be useful for anyone interested in blockchain technology or people trying to integrate them into their projects
