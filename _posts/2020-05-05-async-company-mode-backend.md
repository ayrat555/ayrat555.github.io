---
layout: post
title: Async autocompletion in Emacs
date: 2020-05-05
summary: Creating async company-mode backend
categories: emacs
---

![img](/images/2020-05-05-company.png)

Recently I wrote `company-mode` backend for Elixir - [`company-elixir`](https://github.com/ayrat555/company-elixir), which uses IEx in the background to fetch completions from.

Writing async backends for `company-mode` is not documented very well. In this post, I'll give a brief instruction on writing an async backend.

### Reasons behind `company-elixir`

You can skip this section if you are only interested in the emacs part.

You may ask why do you need IEx for completions when there are standalone implementations for this? There is one reason:

1. Completions in IEx are very handy and they work perfectly because it's a part of the Elixir language. So it's constantly being improved and maintained. On the other hand, custom implementations that provide completion functionality are being developed by enthusiasts in their free time.

But it also has its disadvantages:

1. Autocompletion modules for IEx are not documented and it seems they're not meant to be used outside of IEx. But it's not a problem of `company-elixir` users.
2. Elixir application has to be in the correct state, so it can be compiled to run IEx. If your project can not be compiled, obviously completions won't work.
3. Running IEx as a separate process can be overhead if your application starts processes that do heavy work.

But I'll definitely check [`elixir-lsp`](https://github.com/elixir-lsp/elixir-ls) out after I'll be done with `company-elixir`.

### `company-mode` backend function

In order to implement a `company-mode` backend, you should define a single function with signature

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
2. If the command is `prefix`, the function should return the expression that will be completed.
3. Finally, `candidates` command should return completions for the expression returned in `prefix`.

Let's look further into `candidates` handler.

### Async candidates

Steps to define async `company-mode` backend:

I. To define async fetching of completions we have to return cons of `:async` with the handler lambda which will return candidates, lambda should have a single parameter `callback`. If you wanted to define synchronous handler instead of returning cons you should return just completions.

II. Save the callback that was passed to lambda to a global variable as I did in the example above:

```lisp

(defvar company-elixir--callback nil "Company callback to return candidates to.")

...

(setq company-elixir--callback callback)

...

```


Or you can just pass this callback to the function that will return completions.

I used a global variable because I couldn't pass a callback to it since I'm using a separate process to fetch completions, the output from this process is sent to my process filter.

You can find more about process filters [here](https://www.gnu.org/software/emacs/manual/html_node/elisp/Filter-Functions.html).

III. Return calculated completions by calling the saved callback with your completions

```lisp
(funcall company-elixir--callback completions)
```

### Conclustion

Hopefully, this post gave you a basic understanding of async `company-mode` backends.

You can also check `company-backends` variable documentation. It seems like it is the only documentation piece for `company-mode`.
