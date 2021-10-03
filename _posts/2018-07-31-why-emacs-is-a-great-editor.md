---
title:      Why Emacs is a great text editor
date:       2018-07-31
summary:    My thoughts on Emacs
categories: emacs
---

![swiss-knife](/images/2018-07-31-knife.png)

Often arguments arise from the fact that I use emacs as my main text editor. In this post, I'll try to explain why I think emacs is a great text editor and how it helps me be more productive every day.

### History

Before using emacs I tried different text editors (Atom, Brackets etc) but I primarily used Sublime for a couple of years before switching to emacs. There are two fundamental reasons that motivated me to learn it.

The first thought that I should level up my text editor usage skills came to me when I was pair programming with my colleague solving a task at work. He uses Vim as his text editor. And in his turns he used his time much more productively than me navigating only with shortcuts, also he had shortcuts for every action he used. This experience was mind-blowing for me, he was like code wizard master compared to me navigating using a touchpad.

The reason why I chose to learn emacs and not vim is that in that period of time in Elixir community emacs was very popular (and still is) and people were talking about it in podcasts and blogs. Also, there are some good plugins for Emacs for writing Elixir ([alchemist](https://github.com/tonini/alchemist.el), [elixir-mode](https://github.com/elixir-editors/emacs-elixir)).

Now I've been using emacs for more than 1.5 years.

### Advantages

The most important advantage of using emacs for me is that I became much more productive. Without using mouse or touchpad at all I save seconds on every action that turn into hours every day. I could achieve this in Sublime learning shortcuts but its interface is not motivated me to do so. There is another approach in emacs, you'll sink or swim: without learning shortcuts you simply can't use it.

Let's turn to advantages specific to Emacs. You can customize everything in Emacs: from its look to every action and behavior. Moreover, emacs has its own language (Emacs Lisp) that you can you use to write plugins. Actually, every configuration file is also written in the Emacs Lisp language. Emacs community created some of the best packages that can not be found in other text editors:

- [Magit](https://magit.vc/) - the most usable git interface.
- [Org-Mode](https://orgmode.org/) - a document editing, formatting, and organizing mode, designed for notes, planning, and authoring within the free software text editor Emacs.

Some people call emacs an operating system because there is a lot of functionality built-in and also there are a lot of packages that you can use to enhance it, you can even use emacs as a [window manager](https://github.com/ch11ng/exwm).

Other things worth mentioning:

- [Terminal emulation](https://www.gnu.org/software/emacs/manual/html_node/emacs/Terminal-emulator.html)
- [Server mode](https://www.emacswiki.org/emacs/EmacsAsDaemon)

### Disadvantages

There are a couple of downsides in emacs, especially for new users:

1. Steep learning curve. You should spend a couple of weeks learning emacs to be productive using it.
2. Awkward keyboard shortcuts, especially for Mac users, where you use `ctrl` and `alt` keys for every shortcut. But you get used to it with time.
3. Emacs is single-threaded so a buggy plugin can freeze your emacs process.

### Conclusion

I remember a quote from another of my colleagues, "I learned emacs so I could use a text editor like I'm playing guitar". This quote contains some kind of truth. Like musical instruments for musicians, text editors are the same tools for developers.
