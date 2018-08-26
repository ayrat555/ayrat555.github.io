---
layout:     post
title:      Be productive with Org mode
date:       2018-08-26
summary:    Note manager and organizer
categories: emacs
---

![org-mode-collage](https://i.imgur.com/hgqCyen.jpg)

### Introduction

In my [previous post about emacs](http://www.badykov.com/emacs/2018/07/31/why-emacs-is-a-great-editor/) I mentioned [org mode](https://orgmode.org/), note manager and organizer. Org mode has very extensive functionality and I use only a small subset of it. In this post I'll describe my day-to-day org mode use cases.

### Todo lists

First and foremost, Org mode is a tool for managing notes and todo lists and all work in org mode is centered around writing notes in plain text files. I manage several kinds of notes using org mode.

#### General notes

The most basic org mode use case is writing simple notes about things that you want to remember. For example, here's my notes about things I'm learning right now:

```plain
* Learn
** Emacs LISP
*** Plan

   - [ ] Read best practices
   - [ ] Finish reading Emacs Manual
   - [ ] Finish Exercism Exercises
   - [ ] Write a couple of simple plugins
     - Notification plugin

*** Resources

   https://www.gnu.org/software/emacs/manual/html_node/elisp/index.html
   http://exercism.io/languages/elisp/about
   [[http://batsov.com/articles/2011/11/30/the-ultimate-collection-of-emacs-resources/][The Ultimate Collection of Emacs Resources]]

** Rust gamedev
*** Study [[https://github.com/SergiusIW/gate][gate]] 2d game engine with web assembly support
*** [[ggez][https://github.com/ggez/ggez]]
*** [[https://www.amethyst.rs/blog/release-0-8/][Amethyst 0.8 Relesed]]

** Upgrade Elixir/Erlang Skills
*** Read Erlang in Anger

```

How it looks using [org-bullets](https://github.com/sabof/org-bullets):

![notes](https://i.imgur.com/lGi60Uw.png)

In this simple example you can see some of org mode features:
- nested notes
- links
- lists with checkboxes

#### Project todos

Often  when I'm working on some task I notice things that I can improve or fix. Instead of leaving TODO comment in source code files (bad smell) I use [https://github.com/IvanMalison/org-projectile](org-projectile) whuch allows me to write TODO items with a single shortcut in a separate file. Here's example:


```plain
* [[elisp:(org-projectile-open-project%20"mana")][mana]] [3/9]
  :PROPERTIES:
  :CATEGORY: mana
  :END:
** DONE [[file:~/Development/mana/apps/blockchain/lib/blockchain/contract/create_contract.ex::insufficient_gas_before_homestead%20=][fix this check using evm.configuration]]
   CLOSED: [2018-08-08 Ср 09:14]
  [[https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2.md][eip2]]:
  If contract creation does not have enough gas to pay for the final gas fee for
  adding the contract code to the state, the contract creation fails (i.e. goes out-of-gas)
  rather than leaving an empty contract.
** DONE Upgrade Elixir to 1.7.
   CLOSED: [2018-08-08 Ср 09:14]
** TODO [#A] Difficulty tests
** TODO [#C] Upgrage to OTP 21
** DONE [#A] EIP150
   CLOSED: [2018-08-14 Вт 21:25]
*** DONE operation cost changes
    CLOSED: [2018-08-08 Ср 20:31]
*** DONE 1/64th for call and create
    CLOSED: [2018-08-14 Вт 21:25]
** TODO [#C] Refactor interfaces
** TODO [#B] Caching for storage during execution
** TODO [#B] Removing old merkle trees
** TODO do not calculate cost twice
* [[elisp:(org-projectile-open-project%20".emacs.d")][.emacs.d]] [1/3]
  :PROPERTIES:
  :CATEGORY: .emacs.d
  :END:
** TODO fix flycheck issues (emacs config)
** TODO use-package for fetching dependencies
** DONE clean configuration
   CLOSED: [2018-08-26 Вс 11:48]
```

How it looks in Emacs:

![project-todos](https://i.imgur.com/Hbu8ilX.png)

In this example you can see more Org mode features:

- todo items have state - `TODO`, `DONE`. You can define your own states (`WAITING` etc)
- closed items have `CLOSED` timestamp
- some items have priorities - A, B, C.
- links can be internal (`[[file:~/...]`)

#### Capture templates

As described in org mode's documentation, capture lets you quickly store notes with little interruption of your
work flow.

I configured several capture templates which help me to quickly create notes about things that I want to remember.

```lisp
  (setq org-capture-templates
        '(("t" "Todo" entry (file+headline "~/Dropbox/org/todo.org" "Todo soon")
           "* TODO %? \n  %^t")
          ("i" "Idea" entry (file+headline "~/Dropbox/org/ideas.org" "Ideas")
           "* %? \n %U")
          ("e" "Tweak" entry (file+headline "~/Dropbox/org/tweaks.org" "Tweaks")
           "* %? \n %U")
          ("l" "Learn" entry (file+headline "~/Dropbox/org/learn.org" "Learn")
           "* %? \n")
          ("w" "Work note" entry (file+headline "~/Dropbox/org/work.org" "Work")
           "* %? \n")
          ("m" "Check movie" entry (file+headline "~/Dropbox/org/check.org" "Movies")
           "* %? %^g")
          ("n" "Check book" entry (file+headline "~/Dropbox/org/check.org" "Books")
           "* %^{book name} by %^{author} %^g")))
```

For a book note I should add its name and its author, for a movie template I should add tags etc.

### Schedule

![schedule](https://i.imgur.com/z5HpuB0.png)

#### Habits

![habits](https://i.imgur.com/YJIp3d0.png)

#### Agenda views

![agenda](https://i.imgur.com/CKX9BL9.png)

### More features

- sync

- Integration, exporting

- expenses

### Conclusion

Most of the day in emacs
