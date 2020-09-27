---
layout:     post
title:      Ethereum Contract Application Binary Interface (ABI) in Elixir
date:       2019-01-26
summary:    ABI in Elixir
categories: elixir
---

![smart-contract](/images/2019-01-26-hands.jpg)

### Background

Nowadays Ethereum is one of the most successful blockchain platforms. Foremost things that made it possible are the Ethereum Virtual Machine (EVM) and smart contracts that are executed in the EVM.

Smart contracts are little programs written in high-level languages like Solidity or LLL specifically created for Ethereum. Firstly a contract's code is compiled to the EVM bytecode and only then it can be executed in the EVM.

Application Binary Interface (ABI) is the standard way to interact with contracts in the Ethereum ecosystem, both from outside the blockchain and for contract-to-contract interaction. An account wishing to use a smart contract's function uses the ABI to hash the function definition so it can create the EVM bytecode required to call the function.

In this post, I'll describe how a function and its parameters are encoded to ABI format and how ABI encoding/decoding can be implemented in Elixir.

### Specification

Let's provide formal definition excerpts from the official [ABI specification](https://solidity.readthedocs.io/en/develop/abi-spec.html).

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

Let's give examples of a couple of type encodings:

- `uint<M>`: `enc(X)` is the big-endian encoding of `X`, padded on the higher-order (left) side with zero-bytes such that the length is 32 bytes.
- `bool`: as in the `uint8` case, where 1 is used for true and 0 for false
- `address`: as in the uint160 case
- `bytes`, of length `k` (which is assumed to be of type uint256): `enc(X) = enc(k) pad_right(X)`, i.e. the number of bytes is encoded as a `uint256` followed by the actual value of X as a byte sequence, followed by the minimum number of zero-bytes such that `len(enc(X))` is a multiple of 32.

#### Function Selector and Argument Encoding

A call to the function `f` with parameters `a_1`, ..., `a_n` is encoded as `function_selector(f) enc((a_1, ..., a_n))` and the return values `v_1`, ..., `v_k` of f are encoded as `enc((v_1, ..., v_k))` i.e. the values are combined into a tuple and encoded.

### Elixir implementation

Let's see how the ABI encoding can be implemented in Elixir. Decoding is done in reverse order.

#### Encoding

As described in the specification we should concatenate a function selector encoding and a parameters encoding.

```elixir
  def encode(data, %ABI.FunctionSelector{types: types} = function_selector) do

    # calculates parameter data encoding base on function selector's types
    {result, []} = encode_type({:tuple, types}, [List.to_tuple(data)])

    # concatenates function selector encoding and paramer encoding
    encode_method_id(function_selector) <> result
  end
```

We calculate a function selector encoding as the first (left, high-order in big-endian) four bytes of the Keccak-256 (SHA-3) hash of the signature of the function.

```elixir
  defp encode_method_id(function_selector) do
    # Encode selector e.g. "baz(uint32,bool)" and take keccak
    kec =
      function_selector
      |> ABI.FunctionSelector.encode()
      |> ExthCrypto.Hash.Keccak.kec()

    # Take first four bytes
    <<init::binary-size(4), _rest::binary>> = kec

    # That's our method id
    init
  end
```

Here comes the hard part. We should traverse all parameters and encode them based on their type:

- If the type is dynamic we have to encode its size and append it to the head and add the element's encoding to the tail.
- If the type is static we have to append its encoding to the head.

```elixir
  defp encode_type({:tuple, types}, [data | rest]) do
    # all head items are 32 bytes in length and there will be exactly
    # `count(types)` of them, so the tail starts at `32 * count(types)`.
    tail_start = (types |> Enum.count()) * 32

    {head, tail, [], _} =
      Enum.reduce(types, {<<>>, <<>>, data |> Tuple.to_list(), tail_start}, fn type,
                                                                               {head, tail, data,
                                                                                tail_position} ->
        {el, rest} = encode_type(type, data)

        if ABI.FunctionSelector.is_dynamic?(type) do
          # If we're a dynamic type, just encoded the length to head and the element to body
          {head <> encode_uint(tail_position, 256), tail <> el, rest, tail_position + byte_size(el)}
        else
          # If we're a static type, simply encode the el to the head
          {head <> el, tail, rest, tail_position}
        end
      end)

    {head <> tail, rest}
  end
```

We can use neat Elixir pattern matching to determine if a type is dynamic or not.

```elixir
  defmodule ABI.FunctionSelector do

    ...


    def is_dynamic?(:bytes), do: true
    def is_dynamic?(:string), do: true

    ...

    def is_dynamic?(_), do: false

    ...

  end
```

Here's a couple of examples of data type encodings.

```elixir
  ...

  defp encode_type(:bool, [data | rest]) do
    value =
      case data do
        true -> encode_uint(1, 8)
        false -> encode_uint(0, 8)
        _ -> raise "Invalid data for bool: #{data}"
      end

    {value, rest}
  end

  defp encode_uint(data, size_in_bits) when rem(size_in_bits, 8) == 0 do
    size_in_bytes = round(size_in_bits / 8)
    bin = maybe_encode_unsigned(data)

    if byte_size(bin) > size_in_bytes,
      do:
        raise(
          "Data overflow encoding uint, data `#{data}` cannot fit in #{size_in_bytes * 8} bits"
        )

     pad(bin, size_in_bytes, :left)
  end

  ...
```

### ex_abi

Complete Elixir implementation is available in [`ex_abi` GitHub repo](https://github.com/poanetwork/ex_abi). In this short overview of ABI, I hope you got a basic understanding of this encoding.


### Acknowledgement

- [Geoff Hayes](https://github.com/hayesgm) - original `ex_abi` contributor
- [`ex_abi` contributors](https://github.com/poanetwork/ex_abi/graphs/contributors)


### Update 2020

I completely re-wrote the encoder and made significant changes to the decoder, so this post is not up to date with the latest `ex_abi` version.
