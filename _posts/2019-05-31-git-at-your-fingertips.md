---
layout: post
title: Git at your fingertips
summary: Magit
categories: common
---

![cover](https://i.imgur.com/OkR7dM2.png)

Effective people optimize routine tasks to save time.

<blockquote>
  <p>
  All that really belongs to us is time; even he who has nothing else has that.
  </p>
  <footer><cite title="Baltasar Gracian">Baltasar Gracian</cite></footer>
</blockquote>

Software developers perform a lot of routine tasks throughout the day. For example, saving and uploading code changes using a version-control system, running ci, deploying an application, etc. People underestimate the amount of time they spend doing these common tasks.

Git is the most used version-control system. There are a lot of GUI interfaces for it but most of them just make it more obscure by adding new levels of abstraction. That's why the majority of git users prefer staying on the terminal.

But let me tell you about a git interface that is not like the others. If you start using it you won't be able to return to the terminal version ever again.

### Magit

Magit is an interface to the version control system Git, implemented as an Emacs package. Unlike other Git interfaces, it does not add any abstractions over git. It just adds shortcuts to Git's command line operations and parameters. So you have to know your git well to effectively use Magit.

Help window (buffer) below shows some commands that you can use::

![magit](https://i.imgur.com/A5Zh3Tu.png)

Let me share a few easy ways to get started with it.

### Creating a new branch

To create a new branch all you have to do is to press `b` and choose a branch to create a new branch from:

![creating-a-branch](https://i.imgur.com/uYHOUtP.png)

### Committing your changes

You can create a new commit by pressing `c` two times. You can stage a file by pressing `s`.

![commit](https://i.imgur.com/9Mj7y3H.png)

### Pushing your branch

Once you committed your changes, you can push them by pressing `p` `p` two times.

![push](https://i.imgur.com/QPq3ePa.png)

### Logging and blaming

To use `git-blame` all you have to do is to call `magit-blame` function on a file you want to inspect

![blaming](https://i.imgur.com/mItP5im.png)



If you’re interested in learning more, the magit guide has a good tutorial. Enjoy!
