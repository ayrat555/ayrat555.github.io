---
title:      Elixir behaviours for blockchain hard fork configurations
date:       2018-12-29
summary:    Hard fork configuration at the Mana Project
categories: ethereum
---

![trees](/images/2018-12-29-trees.png)

### Introduction

In [the Mana-Ethereum](https://github.com/mana-ethereum/mana) project, we're creating Ethereum client implementation in Elixir. Ethereum is like any software needs constant maintenance, improvement and optimization. In this post, I'll describe how hard fork configuration is done in our client using built-in polymorphism mechanisms in Elixir.

### Hard forks

Upgrades in Ethereum are done using hard folks. A hard fork is a rule change such that the software validating according to the old rules will see the blocks produced according to the new rules as invalid. In case of a hard fork, all nodes meant to work in accordance with the new rules need to upgrade their software.

![hardfork](/images/2018-12-29-hardfork.png)

In Ethereum a hard fork is a way to introduce new changes to the chain. But in some cases, hard forks can occur when groups of miners and developers can't agree on updates to the software governing a particular digital token. As a result, one group continues to operate under the same rules, while another group branches off and generates a new blockchain with updated software setup. In the process, a second digital currency is generated. This is how Bitcoin Cash came to life. In mid-2017, a group of developers wanting to increase Bitcoin's block size limit prepared a code change. A hard fork took effect on 1 August 2017. As a result, the bitcoin ledger and the cryptocurrency split in two. At the time of the fork, anyone owning bitcoin was also in possession of the same number of Bitcoin Cash units.

As an example let's examine Constantinople hard fork which is scheduled to take place on the Ethereum Mainnet in mid-January 2019, at block 7,080,000. It has five EIPs (Ethereum Improvement Proposal):

- EIP 145: details a more efficient method of information processing on ethereum known as bitwise shifting.
- EIP 1052: offers a means of optimizing large-scale code execution on ethereum.
- EIP 1283: this proposal mainly benefits smart contract developers by introducing a more equitable pricing method for changes made to data storage.
- EIP 1014: the purpose of this upgrade is to better facilitate a certain type of scaling solution based upon state channels and “off-chain” transactions.
- EIP 1234: this upgrade is the most contentious of the batch, reducing block mining reward issuance from 3 ETH down to 2 ETH, as well as, delaying the difficulty bomb for a period of 12 months.

### Polymorphism in Elixir

In programming languages and type theory, polymorphism is the provision of a single interface to entities of different types or the use of a single symbol to represent multiple different types.

Elixir has two mechanisms for polymorphism:

- Protocols
- Behaviours

Dispatching on a protocol is available to any data type as long as it implements the protocol. You can write a function that behaves differently depending on the type of the first argument to its functions. Protocol implementations can be supplied for one of the built-in supported aliases Atom, BitString, Float, Function, Integer, List, Map, PID, Port, Reference, Tuple, and Any; or a user defined struct.

Many modules share the same public API. Behaviours provide a way to:

- define a set of functions that have to be implemented by a module;
- ensure that a module implements all the functions in that set.

Let's see how we can implement hard fork configurations using protocols and behaviours in the next sections.

### Hard fork configuration with protocols

The first approach to implement a hard fork configuration was an implementation based on protocols:

```elixir

defmodule EVM.Configuration.Frontier do
  defstruct contract_creation_cost: 21_000, has_delegate_call: false

  def new do
    %__MODULE__{}
  end
end

defimpl EVM.Configuration, for: EVM.Configuration.Frontier do
  @spec contract_creation_cost(EVM.Configuration.t()) :: integer()
  def contract_creation_cost(config), do: config.contract_creation_cost

  @spec has_delegate_call?(EVM.Configuration.t()) :: boolean()
  def has_delegate_call?(config), do: config.has_delegate_call

  ...
end

...

defmodule EVM.Configuration.Homestead do
  defstruct contract_creation_cost: 53_000, has_delegate_call: true

  def new do
    %__MODULE__{}
  end
end

defimpl EVM.Configuration, for: EVM.Configuration.Homestead do
  @spec contract_creation_cost(EVM.Configuration.t()) :: integer()
  def contract_creation_cost(config), do: config.contract_creation_cost

  @spec has_delegate_call?(EVM.Configuration.t()) :: boolean()
  def has_delegate_call?(config), do: config.has_delegate_call

  ;;;
end

...

```

But it turned out that approach was wrong. The first sign of it were dialyzer warnings:

```
 :0: Unknown function 'Elixir.EVM.Configuration.Atom':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.BitString':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Float':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Function':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Integer':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.List':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Map':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.PID':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Port':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Reference':'__impl__'/1
 :0: Unknown function 'Elixir.EVM.Configuration.Tuple':'__impl__'/1
 ```

We should define protocols for all Elixir data types to make these warnings go away. It's apparent that this is not quite right. Here's a [quote](https://groups.google.com/d/msg/elixir-lang-talk/S0NlOoc4ThM/tfqdDUSPvvsJ) from Jose Valim about protocols and behaviours:

<blockquote>
  <p>
However, I think you are missing the point of behaviours. Behaviours are extremely useful. For example, a GenServer defines a behaviour. A behaviour is a way to say: give me a module as an argument and I will invoke the following callbacks on it. A more complex example for behaviours besides a GenServer are the Ecto adapters.

However, this does not work if you have a data structure and you want to dispatch based on the data structure. Hence protocols.
  </p>
  <footer><cite title="Jose Valim">Jose Valim</cite></footer>
</blockquote>


So we should use protocols only when we want to invoke methods based on data types.

[Original PR](https://github.com/mana-ethereum/mana/pull/308)

### Hard fork configuration with behaviours

As it was found in the previous section we should use behaviours for defining public API that all hard forks have to implement:

```elixir
defmodule EVM.Configuration do
  @moduledoc """
  Behaviour for hardfork configurations.
  """

  @type t :: struct()

  # EIP2
  @callback contract_creation_cost(t) :: integer()

  # EIP2
  @callback fail_contract_creation_lack_of_gas?(t) :: boolean()

  # EIP2
  @callback max_signature_s(t) :: atom()

  # EIP7
  @callback has_delegate_call?(t) :: boolean()

  # EIP150
  @callback extcodesize_cost(t) :: integer()

  # EIP150
  @callback extcodecopy_cost(t) :: integer()

  # EIP150
  @callback balance_cost(t) :: integer()

  # EIP150
  @callback sload_cost(t) :: integer()

  # EIP150
  @callback call_cost(t) :: integer()

  ...
```

Let's give an example of a hard fork configuration:

```elixir
defmodule EVM.Configuration.Frontier do
  @behaviour EVM.Configuration

  defstruct contract_creation_cost: 21_000,
            has_delegate_call: false,
            fail_contract_creation: false,
            max_signature_s: :secp256k1n,
            extcodesize_cost: 20,
            extcodecopy_cost: 20,
            balance_cost: 20,
            sload_cost: 50,
            ...

  @type t :: %__MODULE__{}

  def new do
    %__MODULE__{}
  end

  @impl true
  def contract_creation_cost(config), do: config.contract_creation_cost

  @impl true
  def has_delegate_call?(config), do: config.has_delegate_call

  @impl true
  def max_signature_s(config), do: config.max_signature_s

  @impl true
  def fail_contract_creation_lack_of_gas?(config), do: config.fail_contract_creation

  @impl true
  def extcodesize_cost(config), do: config.extcodesize_cost

  @impl true
  def extcodecopy_cost(config), do: config.extcodecopy_cost

  @impl true
  def balance_cost(config), do: config.balance_cost

  @impl true
  def sload_cost(config), do: config.sload_cost

  @impl true
  def call_cost(config), do: config.call_cost
```

[Original PR](https://github.com/mana-ethereum/mana/pull/451)

By the way, this implementation was improved further using macros. For those interested, see [this PR](https://github.com/mana-ethereum/mana/pull/536/files).

### Conclusion

Protocols and behaviours are powerful tools for polymorphism in Elixir. I hope this post was helpful in understanding them.
