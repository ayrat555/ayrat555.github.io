---
title: Smart Contract Verification in Ethereum
date: 2019-02-24
summary: How block explorers verify smart contracts under the hood
categories: ethereum

redirect_from:
  - /ethereum/2019/02/24/smart-contract-verification/
---

![byte-smart-code](/images/2019-02-24-smart-contract.jpg)

### Introduction

The most significant feature of Ethereum is Ethereum Virtual Machine (EVM) that is used to run smart contracts. Smart contracts are little programs written in Solidity or other smart contract languages (LLL etc). EVM doesn't run these smart contracts directly. It runs its byte code.

![byte-code](/images/2019-02-24-bytecode.jpg)

Byte code is used inside EVM and it's not readable by a human. Users of smart contracts utilize blockchain explorers ([BlockScout](https://blockscout.com), [Etherscan](https://etherscan.io/)) to validate that they use correct smart contracts because they allow developers to verify their smart contracts and publish initial code written in a human-readable high-level programming language like Solidity. By the process of verification, I mean checking that byte code corresponds to the supposed smart contract.

In this post, I'll describe how smart contracts can be verified. Blockchain explorers use the same concepts described in this post.

### Input parameters

You should have the following data related to a smart contract that you want to verify:

Required:
- byte code
- solidity contract code
- contract name
- compiler version
- optimization

Optional:
- optimizer runs count (if your contract has compiler optimization)
- constructor arguments (if your smart contract has constructor arguments)
- contract library addresses (if your smart contract uses external libraries)

Basically, we need to check:

`byte_code == verify(solidity_contract_code, contract_name, compiler_version, optimization, optimizer_runs, constructor_arguments, contract_library_addresses)`


### Solidity compiler's JSON-input-output interface

The process of verification is trivial. We will use solidity compiler to compile solidity contract code with provided parameters. Then we will check if initial byte code is equal to the obtained byte code.

<blockquote>
  <p>
The recommended way to interface with the Solidity compiler especially for more complex and automated setups is the so-called JSON-input-output interface. The same interface is provided by all distributions of the compiler.
The fields are generally subject to change, some are optional (as noted), but we try to only make backwards compatible changes.
The compiler API expects a JSON formatted input and outputs the compilation result in a JSON formatted output.
  </p>
  <footer><cite title="Solidity compiler docs">Solidity compiler docs</cite></footer>
</blockquote>

Let's generate JSON input parameters for the compiler's JSON-input-output interface:

```
{
  language: 'Solidity',
  sources: {
    [newContractName]: {
      content: sourceCode
    }
  },
  settings: {
    evmVersion: 'byzantium',
    optimizer: {
      enabled: optimize == '1',
      runs: 200
    },
    libraries: {
      [newContractName]: externalLibraries
    },
    outputSelection: {
      '*': {
        '*': ['*']
      }
    }
  }
}
```

- `newContractName` is the name of our contract
- `sourceCode` is the source code of our contract
- `optimize` is equal to `1` if we enabled compiler optimization
- `externalLibraries` is key-value pairs of external libraries that we used in our contract


We pass the version of Solidity compiler directly to `solc.loadRemoteVersion` as the first argument

#### Constructor Arguments

The only thing left to do is to append constructor arguments to the compiled bytecode and check if it equals to the manually compiled byte code.
