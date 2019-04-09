---
layout: post
title: Gamedev with Godot
summary: Introduction to Godot
categories: gamedev
---

### You don't have to invent the wheel

Every developer wants to make video games because video games are fun. But few developers try to create them. Why? Maybe it's because looking at triple A games it may seem impossible to create such tremendous high quality games. It's true you'll never create a game like the Witcher 3 with such a great amount of content alone. But I think modern game engines like Unity or Unreal allow developers to create beautiful games without high budgets and large teams.

Another thing that stops/slows down a developer is that most hardcore hackers want to create games from the ground up. It includes all lowlevel code like I/O, networking, rendering, animation, physics, audio etc. Yes, it's very interesting and it may be helpful to fully understand how everything works under the hood in other game engines. But it may take months/years before the engine will be finished and the work on game itself will be started. I fell into the same trap. Multiple times I was trying to work on a game engine but eventually I was abandoning the idea due to the shortage of time. I learned the Rust language to write fast lowlevel code for my engine. I mentioned it in my post called [In Rust I trust](https://www.badykov.com/rust/2018/01/28/in-rust-i-trust/).

### Why Godot?

In the end of 2018 I decided that I should stop inventing the wheel. I started looking into different game engines and chose to go with [Godot](https://godotengine.org/). Factors that helped me to choose it:

- It's mature. The work on the project started in 2007. And there are already a couple of interesting games developed in Godot in stores.
- It has good documentation. The documentation can be your only source of studying the engine. It's thorough and well written.
- It's free and open source.

You can say that there are a couple of game engines like Unity and Unreal Engine that have all these features and much more. But for me key factor was that Godot is open source. I'm a fan of open source software for a long time. I use Linux for many year, my text editor is Emacs etc.

### Features

Godot is the first game engine that I used long enough so I don't have expertise to compare to other engines. But I found quite a few things very useful.

#### Scripting language

Godot has its own programming language - GDScript. It's basic dynamically typed language that used for scripting. In my opionion dynamically typed language are more suitable for scripting than statically typed. Because you can get feedback from your game much faster when you modified your code.

Also, if you like static typing you can you C# or C++ for scripting.

#### Nodes

All objects in Godot are organized in nodes:

- You can have as deep node hierarchies of nodes as you want.
- You can reuse nodes.

Nodes make your scenes more understanble and improve code reusability.

#### Signals

#### Cross-platform

#### Tooling

### Conclustion
