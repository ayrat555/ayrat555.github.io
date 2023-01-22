---
title: Submitting transactions in TON with Elixir
date: 2023-01-22
summary: TON transactions with Elixir
categories: elixir ton
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /images/2023-01-22-ton.jpeg
---

In [my previous post](/elixir/blockchain/ton/) I wrote my thoughts on the TON blockchain and briefly mentioned a project I wrote for the Elixir programming language to interact with TON. In this post, I'll give a step by step instructions for submitting transactions using my library.

## TON SDK for Elixir

[ton](https://github.com/ayrat555/ton) has basic functionality for working with user wallets and submitting transactions. Its features:

- Create a key pair from a mnemonic
- Deploy a [v4r2](https://ton.org/docs/participate/wallets/contracts#wallet-v4) wallet contract
- Parse a TON address
- Create transaction data (boc) which can be submitted using TON API

User wallets in Ethereum are regular accounts that don't have code associated with them. But in TON every account is a smart contract. That's why before interacting with it (sending transactions), its code should be deployed. Fortunately, the `ton` does it automatically which we will see in the next section.

Interestingly enough, to receive transactions a contract doesn't have to be deployed.


## Sending transactions

### I. Generate mnemonic

A mnemonic is a combination of 24 easy-to-remember words used to generate a public key and a secret key. The algorithm for mnemonic generation in `ton` is the same as the one used in Bitcoin and Ethereum. It's implemented in [mnemoniac](https://github.com/ayrat555/mnemoniac) which I extracted from [cryptopunk](/elixir/cryptopunk/)

```elixir
mnemonic = Ton.generate_mnemonic()

# word word word word word word word word word word word word word word word word word word word word word word word word
```

Let's imagine that we generated the most unsafe mnemonic possible - `word word word word word word word word word word word word word word word word word word word word word word word word`. We will use it in the next steps.

### II. Generate public and private keys

The sole purpose of a mnemonic is to generate a secret key for signing transactions and a public key to generate an address. Let's generate a keypair from our mnemonic

```elixir
keypair = Ton.mnemonic_to_keypair(mnemonic)

# %Ton.KeyPair{
#   secret_key: <<199, 231, 167, 187, 26, 34, 150, 13, 203, 55, 65, 90, 35, 178,
#     148, 177, 216, 26, 235, 190, 158, 199, 167, 33, 97, 20, 240, 104, 216, 49,
#     165, 229, 121, 79, 112, 203, 126, 240, 207, 119, 93, 172, 171, 86, 251, 99,
#     160, 246, 242, 237, 147, 129, 148, 34, 212, 63, 30, 7, 97, 167, 42, 234, 65,
#     126>>,
#   public_key: <<121, 79, 112, 203, 126, 240, 207, 119, 93, 172, 171, 86, 251,
#     99, 160, 246, 242, 237, 147, 129, 148, 34, 212, 63, 30, 7, 97, 167, 42, 234,
#     65, 126>>
# }
```

### III. Create a wallet

```elixir
wallet = Ton.create_wallet(keypair.public_key)

# %Ton.Wallet{
#   initial_code:
#     %Ton.Cell{
#       refs: _,
#       data: %Ton.Bitstring{},
#       kind: :ordinary
#   },
#   initial_data:
#     %Ton.Cell{
#       refs: [],
#       data: %Ton.Bitstring{},
#       kind: :ordinary
#     },
#   workchain: 0,
#   wallet_id: 698_983_191,
#   public_key: <<121, 79, 112, 203, 126, 240, 207, 119, 93, 172, 171, 86, 251,
#      99, 160, 246, 242, 237, 147, 129, 148, 34, 212, 63, 30, 7, 97, 167, 42, 234,
#      65, 126>>
# }
```

Every contract is defined by its code and its initial data. Under the hood, `create_wallet/1` function initializes a contract by:

- reading the hardcoded `v4r2` wallet contract code into a data structure called `cell` used for data storage
- converting initial data (`public_key`, `workchain` and `wallet_id`) into a `cell`. `workchain` and `wallet_id` have default values but they can be overridden

The wallet is just a structure that will be used in the next steps

### IV. Displaying a friendly address

Let's check how our wallet's address looks

```elixir
Ton.wallet_to_friendly_address(wallet)
# EQD2_2SN1-PCRRfHVbmM5Q0vf680bZAnIGR7EIQsMzQdHG4d
```

The address `EQD2_2SN1-PCRRfHVbmM5Q0vf680bZAnIGR7EIQsMzQdHG4d` can be used to receive transactions. Under the hood, the library hashed cells created in the previous step.

### V. Generate transaction boc

Before sending a transaction, we have to know the wallet's `seqno`. The value used to prevent replay attacks, it's the number of transactions executed by a wallet. If you're familiar with Ethereum, `seqno` is similar to `nonce`.

Since our wallet is brand new (at least at the time of writing this post), it didn't send any transactions. So `seqno` is 0. In the future, to know `seqno` we will have to call TON HTTP API's `/runGetMethod` with parameters

```
{
  address: 'EQD2_2SN1-PCRRfHVbmM5Q0vf680bZAnIGR7EIQsMzQdHG4d',
  method: 'seqno',
  stack: []
}
```


Let's create transaction data (`boc` - bag of cells) that can be used to submit a transaction

```elixir
{:ok, to_address} = Ton.parse_address("EQAHJQ6gs2NYAXsxsfsucpqhpneZaGP0qCdu9lCEzysMGzst")

params = [
  seqno: 0,
  bounce: true,
  secret_key: keypair.secret_key,
  value: 1,
  to_address: to_address,
  timeout: 60,
  comment: "Hello"
]

boc = Ton.create_transfer_boc(wallet, params)
```

Let's go over each of the parameters:

- seqno - the number of transactions sent by the wallet. if `seqno` is 0, when sending a transaction, the code of the contract will be deployed
- bounce - if the destination smart contract does not exist, or if it throws an unhandled exception while processing this message, the message will be "bounced" back carrying the remainder of the original value if the parameter is set to true
- secret_key - the key used to sign the transaction
- value - the value that will be sent. the value 1 = 0.0000000001 Tons, 1_000_0000 = 1 Ton
- to_address - the destination address. it should be a `Ton.Address` struct, that's why we parsed the to_address first
- timeout - relative timeout in seconds starting from the time `create_transfer_boc` was called
- comment - optional comment

Now we can submit a transaction using `/sendBoc` of TON HTTP API.
