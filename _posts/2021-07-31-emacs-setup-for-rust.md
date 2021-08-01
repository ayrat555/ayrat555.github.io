---
layout: post
title: Emacs setup for Rust
date: 2021-07-31
summary: My essential package list for Rust
categories: emacs
---

![img](/images/2021-07-31-crab.png)

Recently I was debugging [one issue](https://github.com/racer-rust/emacs-racer/issues/152#issuecomment-881867717) in the [racer](https://github.com/racer-rust/racer) - a completion tool for the rust programming language. I've been using the racer for a couple of years in my emacs setup and it's been very useful. But I noticed the following disclaimer in its Readme a couple of days ago: "Racer is not actively developed now. Please consider using newer software such as rust-analyzer".

Even though the racer still works fine in my text editor, I decided to try `rust-analyzer`. It inspired me to create this short post with all emacs packages that I'm using for rust development.

### Packages

I divided packages into two lists. The first list is packages specific for Rust, packages from the second list can be used with other programming languages.

#### Rust packages

- [rust-mode](https://github.com/rust-lang/rust-mode) - major mode for Rust. Every programming language needs its major mode in Emacs. A major mode handles indentation and syntax highlighting. Rust mode does this job.

- [cargo-mode](https://github.com/ayrat555/cargo-mode) - Emacs minor mode which allows to dynamically select a Cargo command. I wrote it myself. The reasons behind this package can be found in [the post](https://www.badykov.com/emacs/2021/05/29/emacs-cargo-mode/).

- [rust-analyzer](https://github.com/rust-analyzer/rust-analyzer) (through [lsp-mode](https://github.com/emacs-lsp/lsp-mode)). From its Readme, Rust-analyzer is an implementation of Language Server Protocol for the Rust programming language. It provides features like completion and goto definition. Rust analyzer can be used in emacs through Lsp-mode - Emacs client for the Language Server Protocol.

#### General Packages

- More Lsp Packages. You can find more extensions in [lsp-mode's guides](https://emacs-lsp.github.io/lsp-mode/). I use [lsp-ui](https://github.com/emacs-lsp/lsp-ui), [lsp-ivy](https://github.com/emacs-lsp/lsp-ivy), [lsp-treemacs](https://github.com/emacs-lsp/lsp-treemacs).

- [projectile](https://github.com/bbatsov/projectile) - Project interaction library

- [magit](https://github.com/magit/magit) - Interface to the version control system Git

- [treemacs](https://github.com/Alexander-Miller/treemacs) - a tree layout file explorer for Emacs

### More packages

You can find all packages that I use in Emacs in [my emacs configuration](https://github.com/ayrat555/dot-emacs).
