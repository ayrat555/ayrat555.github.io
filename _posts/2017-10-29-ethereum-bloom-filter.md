---
layout:     post
title:      Bloom filters in Ethereum
date:       2017-10-29
summary:    Let’s examine Ethereum's bloom filters
categories: ethereum
---

![bloom](https://i.pinimg.com/736x/6c/d6/5d/6cd65d49e6e9bd27700b4d50da8c6760--brothers-grimm-grimm-fairy-tales.jpg)

### Background

Bloom filters are compact data structures for probabilistic representation of a set in order to support membership queries (i.e. queries that ask: “Is element X in set Y?”).

An empty Bloom filter is a bit array of m bits, all set to 0. There must also be k different hash functions defined, each of which maps or hashes some set element to one of the m array positions, generating a uniform random distribution. Typically, k is a constant, much smaller than m, which is proportional to the number of elements to be added; the precise choice of k and the constant of proportionality of m are determined by the intended false positive rate of the filter.

To add an element, feed it to each of the k hash functions to get k array positions. Set the bits at all these positions to 1.

To query for an element (test whether it is in the set), feed it to each of the k hash functions to get k array positions. If any of the bits at these positions is 0, the element is definitely not in the set – if it were, then all the bits would have been set to 1 when it was inserted. If all are 1, then either the element is in the set, or the bits have by chance been set to 1 during the insertion of other elements, resulting in a false positive. In a simple Bloom filter, there is no way to distinguish between the two cases, but more advanced techniques can address this problem

Pros:
- space efficient

Cons:
- can't store an associated object
- deletions are not allowed
- small false positive probability

Applications:
- weak password dictionary
- cache sharing
- query filtering and routing
- blockchain (logs)

### Ethereum bloom filter function

Here's excerpt from the [yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf):
![ethereum-bloom](https://i.imgur.com/IFEyqRU.jpg)

If you didn't understand anything, no worries, I will try to describe its implementation in the next section.

### Elixir implementation

```elixir
  @spec bloom(integer(), binary()) :: integer()
  def bloom(data, number \\ 0) do
    bits =
      data
      |> sha3_hash            \\ (1)
      |> bit_numbers          \\ (2)

    number |> add_bits(bits)  \\ (3)
  end

```

The bloom/2 method accepts 2 aguments:
- data - binary. a new object to add to the bloom filter
- number - integer. the bloom filter. default value is zero (for new filter)

Bloom filter generation can be divided into three parts:
- calculate sha3 hash of an input data (1)
- get three bit indices from this hash (2)
- set these indices to one in the filter (3)

```elixir
  @spec bit_numbers(binary()) :: [integer()]
  defp bit_numbers(hash) do
    {result, _} =
      1..3
      |> Enum.reduce({[], hash}, fn(_, acc) ->
        {bits, <<a1, a2, tail::bitstring>>} = acc
        new_bit = ((a1 <<< 8) + a2) &&& 2047

        {[new_bit|bits], tail}
      end)

    result
  end
```

The bit_numbers/1 method gets indices from the low order 11-bits of the first three double-bytes of the SHA3 hash.

```elixir
  @spec add_bits(integer(), [integer()]) :: integer()
  defp add_bits(bloom_number, bits) do
    bits
    |> Enum.reduce(bloom_number, fn(bit_number, bloom) ->
      bloom ||| (1 <<< bit_number)
    end)
  end
```

Described code has a GitHub repository - [https://github.com/ayrat555/eth_bloom](https://github.com/ayrat555/eth_bloom)

### See also

- https://ethereum.github.io/yellowpaper/paper.pdf
- https://en.m.wikipedia.org/wiki/Bloom_filter
- https://www.coursera.org/learn/algorithms-graphs-data-structures/lecture/riKfa/bloom-filters-the-basics
