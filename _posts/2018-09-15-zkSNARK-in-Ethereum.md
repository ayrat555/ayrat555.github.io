---
layout:     post
title:      zkSNARK in Ethereum
date:       2018-09-15
summary:    Zero-knowledge proofs in Ethereum
categories: ethereum
---

![crypto-lock](https://i.imgur.com/KQEORrb.jpg)

### Background

This week we, at Mana Project, finished implementing Byzantium hard fork's EIPs (The Ethereum Improvement Proposals). The hardest EIPs to implement were EIPs related to zkSNARKs - [EIP-196](https://eips.ethereum.org/EIPS/eip-196) ([implementation in Mana](https://github.com/poanetwork/mana/pull/397)), [EIP-197](https://eips.ethereum.org/EIPS/eip-197)  ([implementation in Mana](https://github.com/poanetwork/mana/pull/406)).

To mark this milestone, In this post, I'll try to give a short summary of a zero-knowledge proof and try to describe why it can be useful for Ethereum without going into too much detail because it uses pretty advanced elliptic curve cryptography based on elliptic curves over finite fields. But if you're interested in implementation details you can visit [BN](https://github.com/poanetwork/bn) - our implementation of BN128 elliptic curve operations.

### Zero-knowledge proof

Zero-Knowledge Succinct Non-interactive ARgument of Knowledge, or zkSNARKs, is a form of zero-knowledge proof. A zero-knowledge proof is a type of cryptography that allows one party (the prover) to prove to another party (the verifier) that it possesses some information without revealing any details about information itself.

Zero-knowledge proof must have three properties:
- Completeness. If the statement is true, the verifier will be convinced that the prover is honest.
- Soundness If the statement is false, nothing can convince the verifier that the statement is true.
- Zero-knowledge. If the statement is true, the verifier can not learn any additional information from the statement.

#### Example

Let's see the easiest and the most understandable example of a zero-knowledge proof that was inspired by the work of Professor Oded Goldreich.

Imagine that your friend is color-blind (by the way, you can't be color-blind in this example, I'm sorry if you are) and you have two identical green and red balls. Your friend shows you these balls and puts them behind his back. He can swap around the balls behind his back and then he shows them to you again. Your job is to convince him that the balls are of a different color. After many switches, it will be obvious to your friend that the balls are of a different color because your statements will be true every time.

This example satisfies all three zero-knowledge proof properties.

#### Use cases

- Authentication without revealing your identity
- Proving that you have some private information (for example, private key) without revealing information itself
- All other NP-class problems

### Zero-knowledge proof in blockchains

We all know that blockchain technology is based on the idea of a decentralized ledger that records transactions and so forth. But the problem in the naive "ledger" approach is that it's transparent. Everybody can see all running transactions, payment history, balances etc. This is where Zero-knowledge proof comes into play.

#### Zcash

Zcash is the first cryptocurrency that based on zero-knowledge proofs. The novelty of Zcash is privacy which is backed by zkSnarks. Its transactions have the option to hide the sender, recipient, and value on the blockchain.

#### Ethereum

In the Byzantium hard fork, a zkSNARK functionality was added to Ethereum in the form of precompiled contracts. It still early but there are already a couple of apps for Ethereum that use this technology. One of them is [Open Vote Network[(https://github.com/stonecoldpat/anonymousvoting) - 2-round decentralized voting protocol.

### Conclusion

zkSNARks is a nice addition to Ethereum toolbox but it's too early to conclude that it will be vastly adopted by Ethereum community.

### See also

- [Mana Project](https://github.com/poanetwork/mana)
- [BN](https://github.com/poanetwork/bn)
- [Zk-SNARKs: Under the Hood](https://medium.com/@VitalikButerin/zk-snarks-under-the-hood-b33151a013f6) by Vitalik Buterin
- [What are zk-SNARKs?](https://z.cash/technology/zksnarks.html)