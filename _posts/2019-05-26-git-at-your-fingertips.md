---
layout: post
title: Git at your fingertips
summary: Magit
categories: common
---

Effective people optimize common tasks to save time.

<blockquote>
  <p>
  All that really belongs to us is time; even he who has nothing else has that.
  </p>
  <footer><cite title="Baltasar Gracian">Baltasar Gracian</cite></footer>
</blockquote>

Solftware developers perform a lot of common tasks throughout the day. For example, saving and uploading code changes using a version-control system, running ci, deploying an application etc. People underestimate the amount of time they spend doing these common tasks.

Git is the most used version-control system. There are a lot of GUI interfaces for it but most of them just make it more obscure by adding new levels of absctraction. That's why the majority of git users prefer staying on the terminal.

But let me tell me about a git interface that is not like the others. If you start using it you won't be able to return to the terminal version ever again.

### Magit

Magit is an interface to the version control system Git, implemented as an Emacs package. Unlike other Git interfaces it does not add any abstractions over git. It just adds shorcuts to Git's command line operations and parameters. So you have to know your git well to effectively use Magit.

Help window below shows some commonds that you can perform:

![magit](https://i.imgur.com/A5Zh3Tu.png)

Let me share a few easy ways to get started with it.

### Creating a new branch

![creating-a-branch](https://i.imgur.com/n87Cdgl.png)

### Committing your changes

![commit](https://i.imgur.com/9Mj7y3H.png)

### Pushing your branch

![push](https://i.imgur.com/xjAIOPo.png)

### Logging and blaming

![blaming](https://i.imgur.com/hopHzvU.png)
