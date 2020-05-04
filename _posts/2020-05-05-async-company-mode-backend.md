---
layout: post
title: Async autocompletion in Emacs
date: 2020-05-05
summary: Creating async company-mode backend
categories: emacs
---

Recently I wrote `company-mode` backend for Elixir - [`company-elixir`](https://github.com/ayrat555/company-elixir), which uses IEx in the background to fetch completions from.

Writing async backends for `company-mode` is not documented very well. In this post I'll give a brief instructions on writing an async backend.

### Reasons behind `company-elixir`

You can skip this section if you only interested in emacs part.

You may ask why do you need IEx for completions when there are standalone implementations for this? There is one reason:

1. Completions in IEx are very handy and they work perfectly because it's a part of the Elixir language. So it's constantly being improved and maintained. On the other hand, custom implementations that provide completion functionality are being developed by enthusiasts in their free time.

But it also has its disadvantages:

1. Autocompletion modules for IEx are not documeted and it seems they're not meant to be used outside of IEx. But it's not a problem of `company-elixir` users.
2. Elixir application have to be in the correct state, so it can be compiled to run IEx. If your project can not be compiled, obviously completions won't work.
3. Running IEx as a separate process can be overhead if your application starts processes that does heavy work.

But I'll definitely check [`elixir-lsp`](https://github.com/elixir-lsp/elixir-ls) out after I'll be done with `company-elixir`.

### `company-mode` backend function

In order to implement a `company-mode` backend you should define a single function with signature

```
(command &optional arg &rest ignored)
```

In different situations `company-mode` will call it with different `command` and `arg` parameters so you have to define handlers for a specific command if you want to process it.

Let's check how I defined it for `company-elixir`:

```lisp
(defun company-elixir (command &optional arg &rest ignored)
  "Completion backend for company-mode."
  (interactive (list 'interactive))
  (cl-case command
    (interactive (company-begin-backend 'company-elixir))
    (prefix (and (eq major-mode company-elixir-major-mode)
                 (company-elixir--get-prefix)))
    (candidates (cons :async
                      (lambda (callback)
                        (setq company-elixir--callback callback)
                        (setq company-elixir--last-completion arg)
                        (company-elixir--find-candidates arg))))))
```

It defines three handlers:

1. The function has to be interactive and call `company-begin-backend` to init your backend when called interactively.
2. If command is `prefix`, the function should return the expression that should be completed.
3. Finally, `candidates` command should return comletions for the expression returned in `prefix`.

Let's look further into `candidates` handler.

### Async candidates

To define async fetching of completions we have to return cons of `:async` with the handler lambda which will return candidates, lambda should have a single parameter `callback`. If if wanted to define syncronous handler instead of return cons we could just return completions.

Steps to defind async `company-mode` backend:
1.




It seems `company-backends` variable documentation is the only documentation peiece for `company-mode`




Backend structrure

Code


Conclustion
