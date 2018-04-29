---
layout:     post
title:      Ethereum Virtual Machine in Elixir
date:       2018-04-29
summary:    EVM execution basics in Elixir
categories: elixir
---

![ethereum](https://i.imgur.com/zaIytOk.jpg)

### Introduction

This week we, at Mana project, finally made all official EVM tests pass. I think it's time to write a post describing its inner magic with code examples from our project. So every crypto alchemist<a name="footnote1">1</a> interested in Ethereum can follow along.

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
     - `LT` - less-than comparison.
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

To follow along executing code examples you need to clone our [Ethereum client](https://github.com/poanetwork/mana) from GitHub:

```bash
git clone https://github.com/poanetwork/mana
```

Fetch dependencies:

```bash
mix deps.get
```

We should print debugging information to see how code executes.

Add these lines to `lib/evm/vm.ex` file in `cycle/3` method:

```elixir
...

def cycle(machine_state, sub_state, exec_env) do
    operation = MachineCode.current_operation(machine_state, exec_env)
    inputs = Operation.inputs(operation, machine_state)

    IO.puts("stack:")
    IO.inspect(machine_state.stack)
    IO.puts("operation: #{operation.sym}")

    ...
```

And start Elixir REPL:

```bash
iex -S mix
```

#### Code examples

So finally we came to examples. Examples illustrated here are taken from [Ethereum's official evm tests](https://github.com/ethereum/tests).

##### Example 1

This test is located in `VMTests/vmArithmeticTest/add3.json` in Ethereum tests repo.

Let's create execution environment:

```elixir
iex> env = %EVM.ExecEnv{
  account_interface: %EVM.Interface.Mock.MockAccountInterface{
    account_map: %{},
    contract_result: %{gas: nil, output: nil, sub_state: nil}
  },
  address: 87579061662017136990230301793909925042452127430,
  block_interface: %EVM.Interface.Mock.MockBlockInterface{
    block_header: nil,
    block_map: %{}
  },
  data: "",
  gas_price: <<90, 243, 16, 122, 64, 0>>,
  machine_code: <<96, 0, 96, 0, 1, 96, 0, 85>>,
  originator: <<205, 23, 34, 242, 148, 125, 239, 76, 241, 68, 103, 157, 163,
    156, 76, 50, 189, 195, 86, 129>>,
  sender: 1170859069521887415590932569929099639409724315265,
  stack_depth: 0,
  value_in_wei: <<13, 224, 182, 179, 167, 100, 0, 0>>
}

```

The only field in env variable we are interested in is `machine_code`. It is represented as a binary. Let's decompile it to human readable form:

```elixir
iex> env.machine_code |> EVM.MachineCode.decompile
[:push1, 0, :push1, 0, :add, :push1, 0, :sstore]
```

As we can see it places two zeros to stack, adds them, places another zero to stack and finally stores the second stack item to storage, storage index is the first stack item.
Let's execute it:

```elixir
iex>  EVM.VM.run(10000, env)

stack:
[]
operation: push1
stack:
[0]
operation: push1
stack:
[0, 0]
operation: add
stack:
[0]
operation: push1
stack:
[0, 0]
operation: sstore
stack:
[]


{..., %EVM.SubState{logs: [], refund: 0, suicide_list: []},
 %EVM.ExecEnv{
   account_interface: %EVM.Interface.Mock.MockAccountInterface{
     account_map: %{
       87579061662017136990230301793909925042452127430 => %{
         balance: 0,
         nonce: 0,
         storage: %{0 => 0}
       }
     },
     contract_result: %{gas: nil, output: nil, sub_state: nil}
   },
   address: 87579061662017136990230301793909925042452127430,
   block_interface: %EVM.Interface.Mock.MockBlockInterface{
     block_header: nil,
     block_map: %{}
   },
   data: "",
   gas_price: <<90, 243, 16, 122, 64, 0>>,
   machine_code: <<96, 0, 96, 0, 1, 96, 0, 85>>,
   originator: <<205, 23, 34, 242, 148, 125, 239, 76, 241, 68, 103, 157, 163,
     156, 76, 50, 189, 195, 86, 129>>,
   sender: 1170859069521887415590932569929099639409724315265,
   stack_depth: 0,
   value_in_wei: <<13, 224, 182, 179, 167, 100, 0, 0>>
 }, ""}
```

As you can see it executes as expected printing exact operations described above. Also note account storage now has new value `storage: %{0 => 0}`.

##### Example 2

This test is located in `VMTests/vmBitwiseLogicOperation/and5.json` in Ethereum tests repo.

Again let's create execution environment:

```elixir
iex> env = %EVM.ExecEnv{
  account_interface: %EVM.Interface.Mock.MockAccountInterface{},
  address: 87579061662017136990230301793909925042452127430,
  block_interface: %EVM.Interface.Mock.MockBlockInterface{},
  data: "",
  gas_price: <<90, 243, 16, 122, 64, 0>>,
  machine_code: <<127, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 127, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238,
  238, 238, 238, 239, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238,
  238, 238, 238, 238, 238, 238, 22, 96, 0, 85>>,
  originator: <<205, 23, 34, 242, 148, 125, 239, 76, 241, 68, 103, 157, 163,
    156, 76, 50, 189, 195, 86, 129>>,
  sender: 1170859069521887415590932569929099639409724315265,
  stack_depth: 0,
  value_in_wei: <<13, 224, 182, 179, 167, 100, 0, 0>>
}
```

Decompiled machine code:

```elixir
iex> env.machine_code |> EVM.MachineCode.decompile |> IO.inspect(limit: :infinity)

[:push32, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
 255, 255, :push32, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238,
 238, 238, 239, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238, 238,
 238, 238, 238, 238, :and_, :push1, 0, :sstore]

```

It places 32 bytes to stack two times, then does bitwise `AND` operation with them, and stores result under 0th storage index.

Let's execute it:

```elixir
iex> EVM.VM.run(100000, evm)

stack:
[]
operation: push32
stack:
[115792089237316195423570985008687907853269984665640564039457584007913129639935]
operation: push32
stack:
[108072616621495115728666252674775380750164271619691439750117644576584916463342,
 115792089237316195423570985008687907853269984665640564039457584007913129639935]
operation: and_
stack:
[108072616621495115728666252674775380750164271619691439750117644576584916463342]
operation: push1
stack:
[0,
 108072616621495115728666252674775380750164271619691439750117644576584916463342]
operation: sstore
stack:
[]
operation: stop

{... , %EVM.SubState{logs: [], refund: 0, suicide_list: []},
 %EVM.ExecEnv{
   account_interface: %EVM.Interface.Mock.MockAccountInterface{
     account_map: %{
       87579061662017136990230301793909925042452127430 => %{
         balance: 0,
         nonce: 0,
         storage: %{
           0 => 108072616621495115728666252674775380750164271619691439750117644576584916463342
         }
       }
     },
     contract_result: %{gas: nil, output: nil, sub_state: nil}
   },
   address: 87579061662017136990230301793909925042452127430,
   block_interface: %EVM.Interface.Mock.MockBlockInterface{
     block_header: nil,
     block_map: %{}
   },
   data: "",
   gas_price: <<90, 243, 16, 122, 64, 0>>,
   machine_code: <<127, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
     255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
     255, 255, 255, 255, 255, 255, 127, 238, 238, 238, 238, 238, 238, 238,
     ...>>,
   originator: <<205, 23, 34, 242, 148, 125, 239, 76, 241, 68, 103, 157, 163,
     156, 76, 50, 189, 195, 86, 129>>,
   sender: 1170859069521887415590932569929099639409724315265,
   stack_depth: 0,
   value_in_wei: <<13, 224, 182, 179, 167, 100, 0, 0>>
 }, ""}

```

In debug output we see that after second push32 operation stack items are 108072616621495115728666252674775380750164271619691439750117644576584916463342 and 115792089237316195423570985008687907853269984665640564039457584007913129639935. Let's calculate bitwise `AND` operation between them:

```elixir
iex>  use Bitwise
iex> 108072616621495115728666252674775380750164271619691439750117644576584916463342 &&& 115792089237316195423570985008687907853269984665640564039457584007913129639935
108072616621495115728666252674775380750164271619691439750117644576584916463342
```

Result is equal to storage value after evm execution.

##### Example 3

This test is located in `VMTests/vmPushDupSwapTest/dup9.json` in Ethereum tests repo.

Let's create environment:

```elixir
iex> env = %EVM.ExecEnv{
  account_interface: %EVM.Interface.Mock.MockAccountInterface{},
  address: 87579061662017136990230301793909925042452127430,
  block_interface: %EVM.Interface.Mock.MockBlockInterface{},
  data: "",
  gas_price: <<90, 243, 16, 122, 64, 0>>,
  machine_code: <<96, 9, 96, 8, 96, 7, 96, 6, 96, 5, 96, 4, 96, 3, 96, 2, 96, 1, 136, 96, 3,
  85>>,
  originator: <<205, 23, 34, 242, 148, 125, 239, 76, 241, 68, 103, 157, 163,
    156, 76, 50, 189, 195, 86, 129>>,
  sender: 1170859069521887415590932569929099639409724315265,
  stack_depth: 0,
  value_in_wei: <<13, 224, 182, 179, 167, 100, 0, 0>>
}
```

Decompiled code:

```elixir
iex> env.machine_code |> EVM.MachineCode.decompile
[:push1, 9, :push1, 8, :push1, 7, :push1, 6, :push1, 5, :push1, 4, :push1, 3,
 :push1, 2, :push1, 1, :dup9, :push1, 3, :sstore]
```

It places nine items to stack, then copies ninth stack and places it to stack.

Let's execute it:

```elixir
iex>  EVM.VM.run(10000, env)

stack:
[]
operation: push1
stack:
'\t' # [9]
operation: push1
stack:
'\b\t' # [8, 9]
operation: push1
stack:
'\a\b\t' # [7, 8, 9]
operation: push1
stack:
[6, 7, 8, 9]
operation: push1
stack:
[5, 6, 7, 8, 9]
operation: push1
stack:
[4, 5, 6, 7, 8, 9]
operation: push1
stack:
[3, 4, 5, 6, 7, 8, 9]
operation: push1
stack:
[2, 3, 4, 5, 6, 7, 8, 9]
operation: push1
stack:
[1, 2, 3, 4, 5, 6, 7, 8, 9]
operation: dup9
stack:
[9, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

As you can see it works as expected.

### Conclustion

I hope this post was helpful in understanding Ethereum Virtual Machine. Currently we are working hard our Ethereum client in Elixir.

### See also

- [Mana Project](https://github.com/poanetwork/mana)
- [Ethereum's Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Ethereum's tests](https://github.com/ethereum/tests)

### Footnotes

<sup>[1](#footnote1)</sup> Elixir developer interested in blockchain technologies
