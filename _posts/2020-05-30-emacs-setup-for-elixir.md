---
layout: post
title: Emacs setup for Elixir
date: 2020-05-30
summary: My essential package list for Emacs
categories: emacs
---

![img](/images/2020-05-30-bicycle.jpg)

I've been using Emacs with different languages for multiple years for all my programming work. For the last couple of years, I've been working with the Elixir lang as my main programming language. Over time I accumulated a set of Emacs tools that I constantly use when writing code with Elixir.

In this post, I'll list the packages I use with Elixir.

### Packages

Majority of these packages can be used with any programming language. So I divided them into three sections: Elixir specific packages, first-tier common packages and second-tier common packages.

#### Elixir specific packages

- [elixir-mode](https://github.com/elixir-editors/emacs-elixir) - Font-locking and indentation for Elixir
- [mix.el](https://github.com/ayrat555/mix.el) - Emacs Minor Mode for Mix, a build tool that ships with Elixir
- [flycheck-credo](https://github.com/aaronjensen/flycheck-credo) - Flycheck checker for Credo - A static code analysis tool for the Elixir language. Flycheck is a modern on-the-fly syntax checking extension

#### First-tier common packages.

- [projectile](https://github.com/bbatsov/projectile) - Project interaction library
- [company-mode](https://github.com/company-mode/company-mode) - Text completion framework for Emacs
- [magit](https://github.com/magit/magit) - Interface to the version control system Git

#### Second-tier common packages.

- [yasnippet](https://github.com/joaotavora/yasnippet) - Template system
- [web-mode](https://github.com/fxbois/web-mode)  - Web template editing mode
- [dumb-jump](https://github.com/jacktasia/dumb-jump) - "Jump to definition" package
- [treemacs](https://github.com/Alexander-Miller/treemacs) - Tree layout file explorer
- [ag](https://github.com/Wilfred/ag.el) - frontend to The Silver Searcher
