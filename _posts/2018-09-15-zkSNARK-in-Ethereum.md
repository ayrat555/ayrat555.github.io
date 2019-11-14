---
layout:     post
title:      zkSNARK in Ethereum
date:       2018-09-15
summary:    Zero-knowledge proofs in Ethereum
categories: ethereum
---

![crypto-lock](/images/2018-09-15-lock.jpg)

### Background

This week at the [Mana Project](https://github.com/poanetwork/mana), we finished implementing the EIPs (Ethereum Improvement Proposals) for the Byzantium hard fork. The hardest EIPs to implement were those related to zero-knowledge proofs, or zkSNARKs:

- [EIP-196](https://eips.ethereum.org/EIPS/eip-196) | [Mana implementation](https://github.com/poanetwork/mana/pull/397)

- [EIP-197](https://eips.ethereum.org/EIPS/eip-197) | [Mana implementation](https://github.com/poanetwork/mana/pull/406)


To mark this milestone, I'll give a short description of what a zero knowledge proof is and describe it's usefulness. I won't go into too much detail because it uses advanced elliptic curve cryptography based on elliptic curves over finite fields. But if you're interested in the implementation details please visit [BN](https://github.com/poanetwork/bn) - our implementation of BN128 elliptic curve operations.

### Zero-knowledge proof

Zero-Knowledge Succinct Non-interactive ARgument of Knowledge, or zkSNARKs, is a form of zero-knowledge proof. A zero-knowledge proof is a type of cryptography that allows one party (the prover) to prove to another party (the verifier) that it possesses some information without revealing any details about information itself.

A zero-knowledge proof must have three properties:
- **Completeness:** If the statement is true, the verifier will be convinced that the prover is honest.
- **Soundness:** If the statement is false, nothing can convince the verifier that the statement is true.
- **Zero-knowledge:** If the statement is true, the verifier can not learn any additional information from the statement.

#### Example

The following example is inspired by the work of Professor Oded Goldeich. In this scenario, you are the prover and your friend is the verifier.

Imagine your friend is colorblind (and you are not). He has a red ball and a green ball, one in each hand. Your task is to prove to him the 2 balls are different colors without giving him any additional information. He takes the balls behind his back and either switches them between hands or keeps them in the same hand. He brings them back out and you inform him whether or not he switched.

Because you can see the colors, you can tell him each time with certainty whether or not he switched. After a number of rounds, your friend will become convinced the balls are different colors because you accurately tell him whether or not he switched each time. Your statements are always true, and your friend gains no knowledge about which ball is red and which is green

This example satisfies all three zero-knowledge proof properties.

#### Use cases

- Authentication without revealing your identity
- Proving that you have some private information (for example, a private key) without revealing any information about the key
- All other NP-class problems (for example mathematical operations).


### Zero-knowledge proof in blockchains

We all know that blockchain technology is based on the idea of a decentralized ledger that records transactions and so forth. But the problem in the naive "ledger" approach is that it's transparent. Everybody can see all running transactions, payment history, balances etc. This is where zero-knowledge proofs come into play.


#### Zcash

Zcash is the first cryptocurrency that based on zero-knowledge proofs. Zcash provides privacy options backed by zkSNARKs. Transactions can be configured to hide the sender, recipient, and value on the blockchain.

#### Ethereum

In the Byzantium hard fork, a zkSNARK functionality was added to Ethereum in the form of precompiled contracts.

It is still early but there are already several DApps for Ethereum that use this technology. One of them is [Open Vote Network](https://github.com/stonecoldpat/anonymousvoting), a 2-round decentralized voting protocol.

### Conclusion

Zero-knowledge proofs allow for cryptographically secure blockchain transactions. The Ethereum Byzantium hard fork incorporated the ability to use zkSNARKs for zero-knowledge proofs. While zkSNARks is a nice addition to the Ethereum toolbox, it’s too early to see if it will be widely adopted by the Ethereum community.

### See also

- [Mana Project](https://github.com/poanetwork/mana)
- [BN](https://github.com/poanetwork/bn)
- [Zk-SNARKs: Under the Hood](https://medium.com/@VitalikButerin/zk-snarks-under-the-hood-b33151a013f6) by Vitalik Buterin
- [What are zk-SNARKs?](https://z.cash/technology/zksnarks.html)
