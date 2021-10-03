---
title: Hacking Elixir tools in Emacs
date: 2020-04-26
summary: Emacs packages I've been developing recently
categories: emacs
---

![cover](/images/2020-04-26-emacs.png)

The last couple of months I've been extensively investing my time into Emacs. It primarily consists of three types of activities:

1. Improving my usage of Emacs to execute my daily activities in it more optimally. This includes reading through the docs of emacs packages that I use, finding new useful packages, creating keybindings for common tasks, etc.
2. Learning Emacs Lisp.
3. Writing Emacs packages. This comes from the first point: I just couldn't find some functionality in existing emacs packages, the functionality was incomplete so I started writing it myself. That's why the second point is on this list. Because you can't write an emacs package without knowing Emacs Lisp.

I spend my days working with Elixir code and that's why I'm constantly trying to optimize common tasks writing Elixir.

I've been using [`Alchemist`](https://github.com/tonini/alchemist.el) for a couple of years. But it seems the package is abandoned and it is not maintained anymore. I wrote two simple packages that improve on the functionality provided by `Alchemist`:

1. [Mix.el](https://github.com/ayrat555/mix.el) - minor mode to work with `mix`, Elixir build tool.
2. [Company-elixir](https://github.com/ayrat555/company-elixir) - Elixir backend for `company` - completion framework for Emacs.


### Mix.el

I have several issues when using Alchemist's mix functions in Emacs.

1. Alchemist uses a custom Elixir application, called `alchemist-server` under the hood. This application is started for every Elixir application that you're currently working with, i.e. every Elixir app you opened in the current Emacs session. This approach has several downsides. Alchemist does not distinguish between the umbrella root application and subprojects in the umbrella application. So either you're working with mix tasks from root or subproject. To work with both you have to restart alchemist server for the required application.
2. You're provided with a list of keybinding for common tasks. But you can't list all available tasks in the project and execute one of them.

`Mix.el` solves these issues:

1. By default, all commands are started at the root of the umbrella app. But you can prefix your command with `C-c` to select a subproject. Also, you can modify the command before execution by prefixing it with `C-u`.
2. To list all available tasks `mix.el` uses `mix help` shell command. As a bonus, you'll get all docs from the shell. Moreover, its implementation much simpler than the implementation of Alchemist because it doesn't need a background process.

You can find more commands and examples in [README](https://github.com/ayrat555/mix.el/blob/master/README.md).

### Company-elixir

Elixir has a very neat autocompletion system in IEx - Elixir's interactive shell. The purpose of `company-elixir` is to provide this functionality when working with Elixir code in Emacs. `company-elixir` is just a backend to the Emacs autocompletion framework called `company`.

If I understand correctly, iex autocompletion modules are not documented anywhere and they are not meant to be used outside of iex. So in a way, I'm just using a hack to provide iex autocompletion for `company-elixir`. `company-elixir` starts iex process in the background which used to fetch autocompletions from. Here's initialization script that executed in the iex:

```elixir
evaluator = IEx.Server.start_evaluator([]) # start evaluator
Process.put(:evaluator, evaluator)  # put evaluator in the current process

defmodule CompanyElixirServer do
  def evaluator do # this function is needed for `IEx.Autocomplete`
    {Process.get(:evaluator), self()}
  end

  def expand(expr) do # Actual autocompletion function
    case IEx.Autocomplete.expand(Enum.reverse(expr), __MODULE__) do
      {:yes, _, result} -> result
      _ -> :not_found
    end
  end
end

IEx.configure(inspect: [limit: :infinity]) # remove limit from output
Logger.remove_backend(:console) # remove console log messages so we only receive autocompletions
```

`company-elixir` just calls `CompanyElixirServer.expand/1` to fetch autcompletions and then parses the result. For example:

```elixir
iex(8)> CompanyElixirServer.expand('String.re')
['replace/3', 'replace/4', 'replace_leading/3', 'replace_prefix/3',
 'replace_suffix/3', 'replace_trailing/3', 'reverse/1']
```

You can find the package at [https://github.com/ayrat555/company-elixir](https://github.com/ayrat555/company-elixir). Note that currently it's in the work-in-progress state: core ideas work, but `company` part is not finished yet. It will be finished in the next couple of weeks.


### Conclusion

The name of this post is not `Developing Elixir tools for emacs` but `Hacking Elixir tools for emacs` because I can't call it traditional software development. Some Emacs features are not documented very well so you have to dig in for the info or use trial and error to find a solution.
