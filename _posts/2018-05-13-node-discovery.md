---
layout:     post
title:      Ethereum's node discovery in Elixir. Part I.
date:       2018-05-13
summary:    Mostly Kademlia in Elixir
categories: elixir
---

![net](https://i.imgur.com/vAMxgVz.png)

### Introduction

Warning: This post is in early stage.

### Background

#### Kademlia

Ethereum nodes connect with each other using RLPx - node discovery protocol based on Kademlia algorithm. Kademlia stores nodes it knows about in routing table. Routing table consists of buckets, nodes are stored in buckets.

The novely of Kadmlia algorithm is the XOR metric it uses. Each node is identified by unique key, distance between nodes is defined as XOR of their keys. Buckets contain nodes with common prefix with current node so they are closer by XOR metric

Kademlia parameters (that is currently used in implementation):

- `k` - bucket size
- `alpha` - concurrency
- `key size` - 160 bits (as described in paper)

#### RLPx

RLPx parameters:

- `k` = 16
- `alpha` = 3
- `key size` = 256 bits

RLPx uses node's public key as its key.

### Implementation

#### Peer

`Peer` module contains methods to work with nodes. The only field we are interested in `Peer` struct is `remote_id`, it is out node key in binary form. Also there are a couple methods worth mentioning:

- common_prefix - calculates common key prefix between two nodes

    ```elixir
      @spec common_prefix(binary(), binary()) :: integer()
      def common_prefix(id1, id2) do
        prefix(id1, id2)
      end

      @spec prefix(binary(), binary(), integer()) :: integer()
      defp prefix(bin1, bin2, acc \\ 0)

      defp prefix(<<1::1, tail1::bitstring>>, <<1::1, tail2::bitstring>>, acc),
        do: prefix(tail1, tail2, acc + 1)

      defp prefix(<<0::1, tail1::bitstring>>, <<0::1, tail2::bitstring>>, acc),
        do: prefix(tail1, tail2, acc + 1)

      defp prefix(_bin1, _bin2, acc), do: acc
    ```

- distance - calculates distance XOR distance between two nodes

    ```elixir
      @spec distance(binary(), binary()) :: integer()
      def distance(id1, id2) do
        id1_integer = id1 |> :binary.decode_unsigned()
        id2_integer = id2 |> :binary.decode_unsigned()
        xor_result = id1_integer ^^^ id2_integer

        xor_result
        |> :binary.encode_unsigned()
        |> binary_to_bits()
        |> Enum.sum()
      end
    ```

  It converts both keys to integer, calculates bitwise XOR between them, then converts result to binary and finally counts number of bits equal to 1. So the bigger this value the farther distance between nodes.

#### Bucket

`Bucket` module contains methods to work with bucket entity described in the Kademlia paper. `Bucket` struct consists of four fields:

- id - common prefix length with current node.
- nodes - nodes it contains.
- updated_at - last time it was updated

This module has many helpful trivial methods to work with buckets but the most interesting one is `add_node/3` method.

Let's return to the algorithm's paper. When a Kademlia node receives any message (request or reply) from another node, it updates the appropriate k-bucket for the sender’s node ID.

1. If the sending node already exists in the recipient’s k- bucket, the recipient moves it to the tail of the list.
2. If the node is not already in the appropriate k-bucket and the bucket has fewer than k entries, then the recipient just inserts the new sender at the tail of the list.
3. If the appropriate k-bucket is full, however, then the recipient pings the k-bucket’s least-recently seen node to decide what to do.
4. If the least-recently seen node fails to respond, it is evicted from the k-bucket and the new sender inserted at the tail.
Otherwise, if the least-recently seen node responds, it is moved to the tail of the list, and the new sender’s contact is discarded.

Here's implementation:

```elixir
  @spec add_node(t(), Peer.t()) :: {atom, t()}
  def add_node(bucket = %Bucket{}, node, options \\ [time: :actual]) do
    cond do
      member?(bucket, node) -> {:reinsert_node, node, reinsert_node(bucket, node, options)}
      full?(bucket) -> {:full_bucket, tail(bucket), bucket}
      true -> {:insert_node, node, insert_node(bucket, node, options)}
    end
  end
```

This implementation handles 1 and 2 parts of insertion algorithm. It checks if the node is already a member of the bucket, if it is, the node is being reinserted (moved to the tail of the list). If the bucket is full, it returns least-recently seen node. Otherwise, it just inserts the node.

#### RoutingTable

`RoutingTable` contains a core of Kademlia algorithm logic.  Notable methods:

- add_node - adds nodes to the routing table.

```elixir
  @spec add_node(t(), Peer.t()) :: t()
  def add_node(
        table = %__MODULE__{current_node: %Peer{remote_id: current_node_id}},
        %Peer{remote_id: current_node_id}
      ),
      do: table

  def add_node(table = %__MODULE__{}, node = %Peer{}) do
    bucket_idx = table |> bucket_id(node)

    case table.buckets |> Enum.at(bucket_idx) |> Bucket.add_node(node) do
      {:full_bucket, candidate_for_removal, bucket} ->
        handle_full_bucket(table, bucket, candidate_for_removal, node)
        table

      {_descr, _node, bucket} ->
        replace_bucket(table, bucket_idx, bucket)
    end
  end
```

This method should handle the 4th part of insertion algorithm - pinging least-recently seen nodes. This functionality will be implemented after intergration with `Network` module which handles network connection, sends and receives messages from other nodes.

- neigbours - finds neighbours of specified node.

```elixir
  @spec neighbours(t(), Peer.t()) :: [Peer.t()]
  def neighbours(table = %__MODULE__{}, node = %Peer{}) do
    bucket_idx = bucket_id(table, node)
    similar_to_current_node = table |> nodes_at(bucket_idx)
    found_nodes = traverse(table, bucket_idx) ++ similar_to_current_node

    found_nodes
    |> Enum.sort_by(&Peer.distance(&1, node))
    |> Enum.take(bucket_size())
  end

  @spec traverse(t(), integer(), [Peer.t()], integer()) :: [Peer.t()]
  defp traverse(table, bucket_id, acc \\ [], step \\ 1) do
    is_out_of_left_boundary = bucket_id - step < 0
    is_out_of_right_boundary = bucket_id + step > bucket_size() - 1

    left_nodes = if is_out_of_left_boundary, do: [], else: table |> nodes_at(bucket_id - step)
    right_nodes = if is_out_of_right_boundary, do: [], else: table |> nodes_at(bucket_id + step)

    acc = acc ++ left_nodes ++ right_nodes

    if (is_out_of_left_boundary && is_out_of_right_boundary) || Enum.count(acc) >= bucket_size() do
      acc
    else
      traverse(table, bucket_id, acc, step + 1)
    end
  end
```

The most naive way of finding neigbours would be to get all nodes in all buckets and sort them by their distance to specified node but it is very heavy operation. So this method traverses buckets by their common prefix.

```RoutingTable``` module is wrapped in process to store Kademlia algorithm's state.

### Conclustion

// Add conclustion after finishing this post.
