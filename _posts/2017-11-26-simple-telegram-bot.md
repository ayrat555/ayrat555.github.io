---
layout:     post
title:      Simple Telegram bot in Elixir
date:       2017-11-26
summary:    Yet another telegram bot
categories: elixir
---

![robot](https://i.imgur.com/wHB01sq.jpg)

### Background

Recently I got an opportunity to write a simple Telegram bot. The purpose of this bot is very simple: it has to send random quotes of [Victor Pelevin](https://en.wikipedia.org/wiki/Victor_Pelevin) for any user interaction. I decided to write this bot in Elixir. So in this post I will try describe the bot's implementation.

### Telegram bot updates

Telegram API provides two mutually exclusive ways of getting bot updates:
- [webhooks](https://core.telegram.org/bots/api#setwebhook)
- polling via [getUpdates method](https://core.telegram.org/bots/api#getupdates)

I think, the first way is more scalable because webhook requests can be loadbalanced between unlimited instances of your bot apps. But the second way is much easier to implement, because you do not even need a web server.

This bot is dedicated to Pelevin's birthday which is the 22th of November. I started to develop a day before his birthday so I didn’t have much time to be messing around with webhooks so I chose the second variant of getting updates from Telegram.

### The main design

The bot consists of two parts:
- Polling process
- Replying processes

Every 100 ms polling process gets bot updates and then sends them via replying processes.

### Polling process

I divided polling process into two modules:
- `QuotesBot.Polling.Logic` - module with polling logic
- `QuotesBot.Polling.Server` - module with GenServer callbacks

`QuotesBot.Polling.Logic`:

```elixir
defmodule QuotesBot.Polling.Logic do
  alias QuotesBot.Config
  alias QuotesBot.Bot.Server, as: ReplyServer
  alias Nadia.Model.Update
  @telegram_api Config.telegram_api

  @spec poll(integer()) :: integer()
  def poll(offset) do
    offset
    |> updates
    |> send_replies
  end

  @spec updates(integer()) :: [Update.t]
  defp updates(offset) do
    try do
      case @telegram_api.get_updates([offset: offset]) do
        {:ok, new_updates} -> new_updates
        _                  -> []
      end
    catch
      []
    end
  end

  @spec send_replies([Update.t]) :: integer()
  defp send_replies(updates) do
    updates
    |> Enum.reduce(0, fn(%Update{update_id: update_id} = update, _acc)->
      ReplyServer.reply(update)

      update_id
    end)
  end
end
```
To be able to mock `telegram_api` in tests with the awesome [mox library](https://github.com/plataformatec/mox) I'm getting `telegram_api` from config file. `poll/1` method gets updates from Telegram and replies to every update with replying processes (`QuotesBot.Bot.Server`), it returns id of the last update that was processed.

Here's `QuotesBot.Polling.Server`:

```elixir
defmodule QuotesBot.Polling.Server do
  use GenServer
  alias QuotesBot.{Polling.Logic, Config}

  def start_link do
    GenServer.start_link(__MODULE__, 0, name: __MODULE__)
  end

  def init(offset) do
    schedule_polling()

    {:ok, offset}
  end

  def handle_info(:poll, offset) do
    new_offset = poll(offset)
    schedule_polling()

    {:noreply, new_offset + 1}
  end

  @spec poll(integer()) :: integer()
  defp poll(offset) do
    Logic.poll(offset)
  end

  defp schedule_polling do
    Process.send_after(self(), :poll, Config.polling_period)
  end
end
```

Its implementation is straightforward. In `init` callback it schedules update polling. And in `handle_info` callback it polls updates and schedules the next update polling.

### Replying processes

I also divided replying process into two modules:
- `QuotesBot.Bot.Server` - module with GenServer callbacks
- `QuotesBot.Bot.Logic` - module with replying logic

`QuotesBot.Bot.Server`:

```elixir
defmodule QuotesBot.Bot.Server do
  use GenServer
  alias QuotesBot.{Bot.Logic, Config}

  def start_link(_) do
    GenServer.start_link(__MODULE__, nil, [])
  end

  def handle_cast({:reply, update}, _state) do
    update_id = Logic.reply(update)

    {:noreply, update_id}
  end

  ### Client

  def reply(update) do
    :poolboy.transaction(
      :worker,
      fn pid -> GenServer.cast(pid, {:reply, update}) end,
      Config.poolboy_timeout
    )
  end
end
```


Here I'm using [poolboy](https://github.com/devinus/poolboy) library. `reply/1` method sends a telegram update to process from process pool. `:poolboy.transaction/1` gets process from process pool.

`poolboy` is configured on the bot startup:

```elixir
defmodule QuotesBot do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      worker(QuotesBot.Polling.Server, []),
      :poolboy.child_spec(:worker, poolboy_config())
    ]

    opts = [strategy: :one_for_one, name: QuotesBot.Supervisor]
    Supervisor.start_link(children, opts)
  end

  defp poolboy_config do
    [
      {:name, {:local, :worker}},
      {:worker_module, QuotesBot.Bot.Server},
      {:size, 20},
      {:max_overflow, 5}
    ]
  end
end
```

I don't provide the source code of `QuotesBot.Bot.Logic`, because it is the domain logic of getting quotes that is not necessary to understand the bot implementation.

### See also

- the bot is available at [http://telegram.me/pelevin_quotes_bot](http://telegram.me/pelevin_quotes_bot). Notice: the bot is written for russian speaking users so it replies only with russian quotes.

- [https://core.telegram.org/](https://core.telegram.org/)
- [https://elixirschool.com/en/lessons/libraries/poolboy/](https://elixirschool.com/en/lessons/libraries/poolboy/)
