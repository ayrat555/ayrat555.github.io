---
layout:     post
title:      Elixir macros are evil (sometimes)
date:       2017-10-18
summary:    Exciting story of dynamic code generation.
categories: elixir
---

![hydra](https://orig00.deviantart.net/557f/f/2014/084/9/f/hydra_by_yoso999-d7bmpqg.jpg)

### Introduction

First, a bit of background. A couple of months ago I got interested in Ethereum. I decided to write a simple Elixir wrapper for its [JSON RPC api](https://github.com/ethereum/wiki/wiki/JSON-RPC) methods. JSON RPC is light-weight remote procedure call protocol.

JSON RPC message must consist of three properties:
- method - the name of the method
- params - object or array of values (In Ethereum's case, an array)
- id - the id of the request

Let's consider Ethereum's web3_sha3 method that calculates a sha3 hash of the input data. It has one parameter - the string to be converted into a sha3 hash. To call this method wrap the parameter info in an array and call it like so:

```bash
// Request
curl -X POST --data '{"jsonrpc":"2.0","method":"web3_sha3","params":["0x68656c6c6f20776f726c64"],"id":64}'

// Result
{
  "id":64,
  "jsonrpc": "2.0",
  "result": "0x47173285a8d7341e5e972fc677286384f802f8ef42a5ec5f03bbfa254cb01fad"
}

```
Ethereum JSON RPC consists of more than 50 methods. Note that parameters can have types other than string (binary). For example, eth_getBlockByHash accepts boolean value as one of its input parameters.

### The first approach

"That's easy.", I thought at the time. "If method requests differ only in their names and input parameters I should write macro that would generate all of these methods for me"

And I wrote this macro. Here's a simplified version:
```elixir
defmodule Ethereumex.Client.Macro do
  defmacro __using__(_) do

    methods = Methods.methods # a lot of methods with format {binary, atom}

    quote location: :keep, bind_quoted: [methods: methods] do

      methods
      |> Enum.each(fn({original_name, formatted_name}) ->
       def unquote(formatted_name)(params) when is_list(params) do
         send_request(unquote(original_name), params)
       end
      end)

      def send_request(method_name, params) when is_list(params) do
        # params preparation
        ...

        request(params)
      end

      def request(params) do
        # request itself
        ...
      end
    end
  end
end
```

I was satisfied with this solution because it was concise, without any boilerplate code. I felt like Hercules when he killed the lernean hydra cutting its heads and searing the headless tendons of the neck but instead of the power of fire I had the power of elixir macros.

But unfortunately, the solution had a couple of problems.

### Problems with the first approach

Soon an issue with the request for implementation of smart contract compilation and interaction was opened in the repository. The requested feature is more high level compared to the original API methods and it used different combinations of them.

I decided to test this feature with the mox library created by our lord and savior Jose Valim. As described in its readme, the mox library follows the principles outlined in ["Mocks and explicit contracts"](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/), summarized below:

- No ad-hoc mocks. You can only create mocks based on behaviours

- No dynamic generation of modules during tests. Mocks are preferably defined in your test_helper.exs or in a setup_all block and not per test

- Concurrency support. Tests using the same mock can still use async: true

- Rely on pattern matching and function clauses for asserting on the input instead of complex expectation rules

As you can see I couldn't mock with mox because all methods are dynamically generated and don't abide any behaviour. So I created an issue in the mox library asking if I should define behaviour with all api methods or if there is some other way.

One minute later I got an answer from Jose:

<blockquote>
  <p>
    One of the guidelines in Elixir is that every time you you generate a function in the user code it should be to abide to some behaviour, so yes, you need behaviours.
  </p>
  <footer><cite title="Jose Valim">Jose Valim</cite></footer>
</blockquote>

Another problem with the current implementation was the method signatures. It's not apparent what parameters every method accepts.

```elixir
# map in list
HttpClient.eth_send_transaction([%{...}])

# list of binaries
HttpClient.eth_get_storage_at([binary1, binary2, binary3])

```

To summarize, there were two problems:
- Methods were hard to mock
- Ugly method signature

So I began refactoring the library to get rid of this problems.

### The second approach

In the second approach I defined behaviour with all JSON API methods

```elixir
defmodule Ethereumex.Client.Behaviour do
  @type error :: {:error, map() | binary() | atom()}

  # API methods

  @callback web3_client_version(keyword()) :: {:ok, binary()} | error
  @callback web3_sha3(binary(), keyword()) :: {:ok, binary()} | error
  @callback net_version(keyword()) :: {:ok, binary()} | error
  @callback net_peer_count(keyword()) :: {:ok, binary()} | error
  ...
```

And then I implemented this behavior in the macro:
```elixir
defmodule Ethereumex.Client.Macro do
  alias Ethereumex.Client.Behaviour

  defmacro __using__(_) do
    quote location: :keep do
      @behaviour Behaviour
      @type error :: Behaviour.error

      @spec web3_client_version(keyword()) :: {:ok, binary()} | error
      def web3_client_version(opts \\ []) do
        "web3_clientVersion" |> request([], opts)
      end

      @spec web3_sha3(binary(), keyword()) :: {:ok, binary()} | error
      def web3_sha3(data, opts \\ []) do
        params = [data]

        "web3_sha3" |> request(params, opts)
      end

      ...
```

So now I can easily mock any method and all method signatures have explicit parameter types.

I implemented the behaviour in the macro because Ethereum JSON RPC has two types of clients:
- ipc clients using unix sockets
- http clients

So to define this client I only needed to include the macro in client modules and override the request/2 method like this:

```elixir
defmodule MyClient do
  use Ethereumex.Client.Macro

  def request(params, opts) do
    ...
  end
end
```

### Conclusion

So Elixir macros as all powerful tools should be used responsibly.

If you have any remarks or thoughts on the matter please leave comments.

The library has a GitHub repository - [https://github.com/exthereum/ethereumex](https://github.com/exthereum/ethereumex)
