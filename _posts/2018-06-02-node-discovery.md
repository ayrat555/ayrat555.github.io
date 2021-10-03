---
title:      Ethereum's node discovery in Elixir
date:       2018-06-02
summary:    Mostly Kademlia in Elixir
categories: elixir
---

![net](/images/2018-06-02-discovery.png)

### Introduction

Each second, millions of messages are sent between nodes on the Ethereum platform. Message contents depend on a communication protocol ([swarm](http://swarm-guide.readthedocs.io/en/latest/introduction.html), [eth](https://github.com/ethereum/wiki/wiki/Ethereum-Wire-Protocol), [whisper](https://github.com/ethereum/wiki/wiki/Whisper)). Nodes need a way to find other nodes.

This post provides  a concise description of node discovery implementation in the [Mana Project](https://github.com/poanetwork/mana) - an Ethereum client written in Elixir. Included is a description of the Kademlia algorithm used by Etherium and the methods required to work with the 3 Kademlia data structures: nodes, k-buckets and routing tables.

### Kademlia

Ethereum nodes find each other using a node discovery protocol based on the Kademlia algorithm.

[Kademlia](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf) is a distributed hash table for decentralized peer-to-peer computer networks designed by Petar Maymounkov and David Mazières in 2002. It specifies the structure of the network and the exchange of information through node lookups.

#### Storage

Kademlia stores nodes it knows about in a routing table. A routing table consists of buckets and  nodes are stored in these buckets.

The novelty of the Kademlia algorithm is the XOR metric it uses. Each node is identified by a unique key, the distance between nodes is defined as XOR of their keys.

Each Bucket contains `k` nodes (that's why they're called k-buckets) with a common prefix in relation to a current node. K-buckets are kept sorted by time:
- Head: last seen - least-recently seen node
- Tail: most-recently seen
 `k` is a system wide replication parameter, `k` is chosen such that any given `k` nodes are very unlikely to fail within an hour of each other.

#### Protocol

Kademlia’s paper defines four RPCs (remote procedure calls): PING, STORE, FIND_NODE, FIND_VALUE. Only PING and FIND_NODE are implemented in Ethereum. When a Kademlia node receives any message (request or reply) from another node, it updates the appropriate k-bucket for the sender’s node id.

Ethereum's node discovery consists of four packets:

- `Ping` - pings a node in the network.
- `Pong` - response to ping packet. Returns data from the ping message.
- `FindNeighbours` - finds the closest (by common prefix distance) nodes to specified node id.
- `Neighbours` - response to FindNeighbours packet.

The most important part of Kademlia algorithm is a **node lookup procedure** which is used for **initial population of a routing table**. Node lookup is a recursive procedure. The initiator sends parallel, asynchronous `FindNeighbours` RPCs to the `alpha` nodes it has chosen (`alpha` is a system-wide concurrency parameter). In the recursive step, the initiator resends `FindNeighbours` to `alpha` nodes it has learned about from previous RPCs.

#### Parameter values

- bucket size (`k`) = 16
- concurrenct (`alpha`) = 3
- number of buckets = 256

### Implementation

#### Process architecture

Node discovery consists of two GenServer processess:

- `ExWire.Network` - sends and receives messages.
- `ExWire.Kademlia.Server` - encapsulates Kademlia algorithm's state.

Each of these processes reference each other. When the network process receives messages from other nodes, it redirects them to the Kademlia process in order to use  Kademlia’s logic. Kademlia then sends an async request using the network process.

Both of these processes depend on each other so they are started by a separate supervisor:

```elixir
defmodule ExWire.NodeDiscoverySupervisor do
  use Supervisor

  @moduledoc """
  The Node Discovery Supervisor manages two processes. The first process is kademlia
  algorithm's routing table state - ExWire.Kademlia.Server,  the second one is process
  that sends and receives all messages that are used for node discovery (ping, pong etc)
  """

  ...

  def start_link(params \\ []) do
    supervisor_name = discovery_param(params, :supervisor_name)

    Supervisor.start_link(__MODULE__, params, name: supervisor_name)
  end

  def init(params) do
    {udp_module, udp_process_name} = discovery_param(params, :network_adapter)
    kademlia_name = discovery_param(params, :kademlia_process_name)
    port = discovery_param(params, :port)
    bootnodes = (params[:nodes] || nodes()) |> Enum.map(&Node.new/1)

    children = [
      {KademliaServer,
       [
         name: kademlia_name,
         current_node: current_node(),
         network_client_name: udp_process_name,
         nodes: bootnodes
       ]},
      {udp_module,
       [
         name: udp_process_name,
         network_module: {Network, [kademlia_process_name: kademlia_name]},
         port: port
       ]}
    ]

    Supervisor.init(children, strategy: :rest_for_one)
  end

  ...
```

#### Node Discovery

Node discovery starts on the Kademlia process startup. Node discovery logic is encapsulated in the `ExWire.Kademlia.Discovery` module:

```elixir
defmodule ExWire.Kademlia.Discovery do
  @moduledoc """
  Module that handles node discovery logic.
  """

  alias ExWire.Kademlia.{RoutingTable, Node}
  alias ExWire.Message.FindNeighbours
  alias ExWire.Network

  @doc """
  Starts discovery round.
  """
  @spec start(RoutingTable.t(), [Node.t()]) :: RoutingTable.t()
  def start(table, bootnodes \\ []) do
    table = add_bootnodes(table, bootnodes)

    this_round_nodes = RoutingTable.discovery_nodes(table)

    Enum.each(this_round_nodes, fn node ->
      find_neighbours(table, node)
    end)

    %{
      table
      | discovery_nodes: table.discovery_nodes ++ this_round_nodes,
        discovery_round: table.discovery_round + 1
    }
  end

  @spec add_bootnodes(RoutingTable.t(), [Node.t()]) :: RoutingTable.t()
  defp add_bootnodes(table, nodes) do
    Enum.reduce(nodes, table, fn node, acc ->
      RoutingTable.refresh_node(acc, node)
    end)
  end

  @spec find_neighbours(RoutingTable.t(), Node.t()) :: :ok
  defp find_neighbours(table, node) do
    find_neighbours = FindNeighbours.new(table.current_node.public_key)

    Network.send(find_neighbours, table.network_client_name, node.endpoint)
  end
end
```

Node discovery is a recursive process. So `ExWire.Kademlia.Discovery` describes a single iteration:

- On the first interation our routing table do not have any nodes, so we add bootnodes to routing table, and send them `FindNeighbours` messages.
- On the following iterations we send `FindNeighbours` messages to nodes that we received from previous iterations.


#### Data structures

As described earlier there are three main data structures in Kademlia:
- node
- bucket
- routing table
In our implementation we defined three modules for each of these entities which contain methods directly related to Kademlia along with helper methods.

##### Node

A module for working with nodes:

```elixir
defmodule ExWire.Kademlia.Node do
  @moduledoc """
  Represents a node in Kademlia algorithm; an entity on the network.
  """
  alias ExWire.Crypto

  defstruct [
    :public_key,
    :key,
    :endpoint
  ]

  @type t :: %__MODULE__{
          public_key: binary(),
          key: binary(),
          endpoint: Endpoint.t()
        }

  @doc """
  Constructs a new node.
  """
  @spec new(binary(), Endpoint.t()) :: t()
  def new(public_key, endpoint = %Endpoint{}) do
    key = Crypto.hash(public_key)

    %__MODULE__{
      public_key: public_key,
      key: key,
      endpoint: endpoint
    }
  end

  ...
end
```

SHA-3 hash of a node’s public key is used for the node’s id (defined simply as key in struct definition). Distance is calculated between the hashes - not the public keys.

The common prefix can be calculated easily using pattern matching against the first bits:


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

##### Bucket

Nodes are stored in buckets (k-buckets). A bucket is chosen by the common prefix value to current node. That's why there are 256 buckets; SHA-3 hash returns 256 bit values and common prefix values are in range of [0, 255].

```elixir
defmodule ExWire.Kademlia.Bucket do
  @moduledoc """
  Represents a Kademlia k-bucket.
  """

  alias ExWire.Kademlia.Node

  defstruct [:id, :nodes, :updated_at]

  @type t :: %__MODULE__{
          id: integer(),
          nodes: [Node.t()],
          updated_at: integer()
        }

  ...

end
```

The `k-Bucket` struct consists of three fields:

- `id` - common prefix length with current node.
- `nodes` - nodes it contains.
- `updated_at` - last time it was updated. It is used for updating stale buckets.

The only method we will pay attention to in this module is the `refresh node/3` method. It returns a tuple with an atom describing insertion result, inserted node and updated bucket:

```elixir
  @spec refresh_node(t(), Node.t(), Keyword.t()) :: {atom, t()}
  def refresh_node(bucket = %__MODULE__{}, node, options \\ [time: :actual]) do
    cond do
      member?(bucket, node) -> {:reinsert_node, node, reinsert_node(bucket, node, options)}
      full?(bucket) -> {:full_bucket, last(bucket), bucket}
      true -> {:insert_node, node, insert_node(bucket, node, options)}
    end
  end
```

There are several possibilites when adding a node to a k-bucket:

1. **Sending node already exists in recipients k-bucket**: recipient moves it to the tail of the list.
2. **Node is not already in the appropriate k-bucket and bucket has fewer than k entries**: recipient  inserts the new sender at the tail of the list.
3. **Appropriate k-bucket is full**: the recipient pings the k-bucket’s least-recently seen node to decide what to do.
    1. **least-recently seen node fails to respond**: Node is is evicted from the k-bucket and the new sender inserted at the tail.
    2. **least-recently seen node responds**: node is moved to the tail of the list, and the new sender’s contact is discarded.

The `refresh/3` method handles parts 1 and 2 of the insertion algorithm. It checks if the node is already a member of the bucket, if it is, the node is reinserted (moved to the tail of the list). If the bucket is full, it returns the least-recently seen node. Otherwise, it just inserts the node.

##### Routing Table

`RoutingTable` contains the core of Kademlia algorithm's logic.

```elixir
defmodule ExWire.Kademlia.RoutingTable do
  @moduledoc """
  Module for working with current node's buckets
  """

  alias ExWire.Kademlia.{Bucket, Node}

  defstruct [
    :current_node,
    :buckets,
    :network_client_name,
    :expected_pongs,
    :discovery_nodes,
    :discovery_round
  ]

  @type expected_pongs :: %{required(binary()) => {Node.t(), Node.t(), integer()}}
  @type t :: %__MODULE__{
          current_node: Node.t(),
          buckets: [Bucket.t()],
          network_client_name: pid() | atom(),
          expected_pongs: expected_pongs(),
          discovery_nodes: [Node.t()],
          discovery_round: integer()
        }
  ...
end
```

`RoutingTable` struct field names are self-explanatory except for the last three fields. Let's look closer at `expected_pongs`. `expected_pongs` is a hash with a ping message mdc as key and tuple of removal candidate, insertion candidate and expiration time as a value. This hash is filled by the `ping/3` method:

```elixir
  @spec ping(t(), Node.t()) :: Network.handler_action()
  def ping(
        table = %__MODULE__{
          current_node: %Node{endpoint: current_endpoint},
          network_client_name: network_client_name
        },
        node = %Node{endpoint: remote_endpoint},
        replace_candidate \\ nil
      ) do
    ping = Ping.new(current_endpoint, remote_endpoint)
    {:sent_message, _, encoded_message} = Network.send(ping, network_client_name, remote_endpoint)

    mdc = Protocol.message_mdc(encoded_message)
    updated_pongs = Map.put(table.expected_pongs, mdc, {node, replace_candidate, ping.timestamp})

    %{table | expected_pongs: updated_pongs}
  end
```

`ping/3` accepts three parameters:
- **table** - routing table
- **node** - node that should be pinged
- **replace_candidate** - node that will replace pinged node if it will not respond

Again, in this module the most important method is `refresh_node/2` which adds a node to a routing table:

```elixir
  @spec refresh_node(t(), Node.t()) :: t()
  def refresh_node(table = %__MODULE__{buckets: buckets}, node = %Node{}) do
    node_bucket_id = bucket_id(table, node)

    refresh_node_result =
      buckets
      |> Enum.at(node_bucket_id)
      |> Bucket.refresh_node(node)

    case refresh_node_result do
      {:full_bucket, candidate_for_removal, _bucket} ->
        ping(table, candidate_for_removal, node)

      {_descr, _node, bucket} ->
        replace_bucket(table, node_bucket_id, bucket)
    end
  end

  @spec bucket_id(t(), Node.t()) :: integer()
  def bucket_id(%__MODULE__{current_node: current_node}, node = %Node{}) do
    node |> Node.common_prefix(current_node)
  end
```

 First we calculate a common prefix between the current node and a node that we’re adding - the id of the k-bucket a node will be added to. Then we try to add a node to the bucket. If the bucket is full we ping the least-recently seen node in the bucket. If a node was added to the bucket, we replace the updated bucket in the routing table.

There are several other methods worth mentioning  in this module:


- `neighbours/3` - returns the closest nodes to the specified node by common prefix distance.
- `handle_neighbours/2` - receives requested neighbours from other nodes and pings them.
- `handle_pong/2` - adds a node to the routing table if a pong was found in `expected_pongs`.

### Conclusion

Node discovery and placement is a continouous process as nodes are added to k-buckets and k-buckets fill routing tables. It requires speed and performance of many simultaneous tasks. In the [Mana Project](https://github.com/poanetwork/mana), we are using Elixir to code the discovery process in an elegant, organized and efficient way.

### See also

- [Kademlia Paper](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf)
- [Mana Project](https://github.com/poanetwork/mana)
- [RLPx](https://github.com/ethereum/devp2p/blob/master/rlpx.md)
