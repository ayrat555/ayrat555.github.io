---
title:      GraphQL subscriptions using Absinthe
date:       2018-03-25
summary:    GraphQL in Elixir
categories: elixir
---

![absinthe](/images/2018-03-25-absinthe.png)

Recently I've been playing around with GraphQL using [Absinthe](http://absinthe-graphql.org/) - The GraphQL toolkit for Elixir. Here's a quick overview of this data query language and an explanation on how to create filtered GraphQL subscriptions with Absinthe.

### GraphQL

GraphQL is a data query language similar to SQL but instead of quering data in your database you can query data in your server API.

Key concepts of the GraphQL query language are:

* Hierarchical
* Strongly typed
* Client-specified queries

Advantages of GraphQL:

* Declarative integration on client (what data/operations do I need)
* A standard way to expose data and operations
* Support for real-time data (with subscriptions)

Disadvantages of GraphQL:

* Complexity on the server-side

Fortunately, the problem with backend complexity has been solved for us by Absinthe.

There are three main query types in GraphQL schema:

1. Query - a way to fetch data.

   ```
   query {
     allPosts {
       description
       text
     }
   }
   ```

2. Mutation - a way to change data.


   ```
   mutation {
     updatePost(id: 1, text: "text") {
       text
     }
   }
   ```

3. Subscription - a way to subscribe to real-time data.

   ```
   subscription {
     newPost(category: [1]) {
       description
       text
     }
   }
   ```

### Subscriptions using Absinthe

This section assumes that you have a basic knowledge of subscriptions in Absinthe. If you don’t you can check out [official documentation](https://hexdocs.pm/absinthe/subscriptions.html). They use Phoenix channels under the hood.

#### The problem

API clients should be able to subscribe to new records. That can be easily done with subscriptions, you'd think. But clients also should be able to provide multiple filters, so only records that pass these filters should be sent to clients.

Absinthe's DSL describing this subscription and its params:

```elixir
  subscription :new_post_added do
    field :new_post, :news_post do
      arg(:origin, list_of(:origin))
      arg(:category, list_of(:category))

      ...
    end
  end
```

Examples of possible queries:

```
subscription {
  newPost(category: [1, 5], origin: ["USA"]) {
    description
    text
  }
}
```

```
subscription {
  newPost(category: [7], origin: ["USA", "Japan"]) {
    description
    text
  }
}
```

In Absinthe clients subscribe to a topic and when new data comes to this topic, it is sent to client. Topics have to be strings.

My first idea was to somehow normalize input params and create a string from it; that string would be our topic. The downside of this solution is that you'd have to create topics from all combinations of possible filter values and publish to all of them.

Stumped by this problem, I went to Absinthe slack channel and asked about possible solutions.

The reply I got from one of Absinthe creators - Ben Wilson:

![ben-wilson-slack](/images/2018-03-25-wilson.png)

By the way, you can check out his amazing talk ["Live APIs with GraphQL Subscriptions"](https://www.youtube.com/watch?v=PEckzwggd78) on ElixirConf 2017.

#### Solution

I ended up sending all requests to the same topic:

```elixir
  subscription :new_post_added do
    field :new_post, :news_post do
      arg(:origin, list_of(:origin))
      arg(:category, list_of(:category))


      config(fn _args, _ ->
        {:ok, topic: "*"}
      end)

      resolve(&PostAddedResolver.filter_post/3)
    end
  end
```

Filtering of posts occurs in the moment of their dispatch. Clients sometimes receive empty messages.

```elixir
defmodule PostAddedResolver do
  alias Absinthe.Resolution

  @spec filter_post(map(), map(), Resolution.t()) :: {:ok, Post.t() | nil}
  def filter_post(post, args, _resolution) do
    result =
      post
      |> check_category(args)
      |> check_source_type(args)

    {:ok, result}
  end

  ...
end
```

### See also

- [https://github.com/absinthe-graphql/absinthe](https://github.com/absinthe-graphql/absinthe)
- [Craft GraphQL APIs in Elixir with Absinthe](https://pragprog.com/book/wwgraphql/craft-graphql-apis-in-elixir-with-absinthe)
