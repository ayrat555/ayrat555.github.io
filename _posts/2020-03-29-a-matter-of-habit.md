---
title: A Matter of Habit
date: 2020-03-29
summary: Consistency with org-habit
categories: common

redirect_from:
   - /common/2020/03/29/a-matter-of-habit/
---

![cover](/images/2020-03-29-org-habit.png)

I think everyone gets these inspirational moments when you promise to yourself that from this day on you will start doing something consistently every day or you want to learn a new thing.  Put in other words, you want to develop a habit. And maybe you even start doing your activity for a couple of days or weeks but eventually, you drop it because life gets in the way: you are too exhausted too do it, you have other plans or you're just procrastinating I've also experienced this kind of situations in my life.

Those of you who are using emacs know about org-mode. Org-mode is a note manager and organizer (you can check out my [post about it](/emacs/2018/08/26/be-productive-with-org-mode). Org-mode ships with the additional package called `org-habit`. In this post, I will share my tips for `org-habit`. But mostly this post will be about my experience using it and what I could achieve.

Please keep in mind that I do activities described in this post in my free time in the evenings and weekends because I spend most of my days working like the rest of us. People underestimate what can be done if you have a couple of spare hours.

### My emacs agenda

`Org-habit` is an extension of `org-mode` that helps you track your habits. The only thing you have to add to your regular org-mode TODO item is `STYLE` property with `habit` value and repeat interval. For example:

```org
** TODO Improve emacs/org skills
   SCHEDULED: <2020-03-29 Sun .+1d>
   :PROPERTIES:
   :STYLE:    habit
   :END:
```

After that you'll get a consistency graph in your agenda view with coloured cells:
- Blue. If the task was not to be done yet on that day.
- Green. If the task could have been done on that day.
- Yellow. If the task was going to be overdue the next day.
- Red. If the task was overdue on that day.

That's all there's to `org-habit`. Pretty simple, right?

Let's take a look at my agenda view.:

![agenda](/images/2020-03-29-org-agenda.png)

At the end of the previous month I gave myself the resolution to do four things every day:
1. Learn emacs lisp.
2. Learn Rust
3. Read programming books to broaden my professional skills.
4. Learn drawing.

Let's go step by step and see what I could achieve for each point of my list.

### Emacs Lisp: mix.el - Emacs minor mode for Elixir

It's my second try to learn Emacs Lisp. I started studying it the first time in the summer of 2018. I read through the half of Emacs Lisp Manual and finished all exercises on Exercism that were available at the time. But life got in the way and I abandoned it.

This time the half of the month I was refreshing my Elisp knowledge and then I started writing Emacs package for Mix - a build tool that ships with Elixir. I've been using `alchemist` and its built-in functions to work with mix tasks from emacs: execute a specific test, execute tests in a file, list all tasks etc. They work ok if you don't work with umbrella apps. Umbrella project is an application split into multiple subprojects. `alchemist` uses custom Elixir app under the hood. This Elixir app is initialized only for a specific application. So it's either initialized for a subproject or an umbrella project. Maybe there are customizable variables that allow you to work with umbrella projects and subprojects at the same time but I couldn't find any.

I spend my days writing Elixir code and sometimes I want to execute a Mix task from the root directory for the whole project (for example, to run a credo check) and sometimes I want to execute a task from a subproject (for example, to fetch deps only for this subproject). So I decided to write a package that can do it. The package is almost finished.  Let's check some examples.

You can list all available tasks and as a bonus, you'll get a documentation string because my package directly parses shell output from `mix help`:

![tasks](/images/2020-03-29-tasks.png)

By default, all tasks are executed from umbrella root directory but you can prefix your command with `C-c` to select a subproject to execute the task from:

![subprojects](/images/2020-03-29-projects.png)

Also, you can choose `MIX_ENV` variable and additional parameters for mix task if you prefix your command with `C-u`.

I'll add documentation to the package soon after I add more keybindings for common mix tasks. But the package is already in the working state. I use it every day in my Elixir development. You can find it [here](https://github.com/ayrat555/mix.el).

### Rust: el_monitorro - a web service written in Rust that can subscribe to data feeds and notify about new data.

Again, it's my second try learning Rust. The first time I tried to learn Rust at the end of 2017 - the beginning of 2018. I have a couple of posts published in that period on this blog (you can find it) and one small library in Rust that I wrote - [crontab parser](https://github.com/ayrat555/cronenberg). I read through the Rust book and finished almost all [Exercsim exercises](https://github.com/ayrat-playground/exercism_rust) (Somebody starred it, lol).

This time again I started re-reading Rust book. At the same time, I started writing a web app using Rocket web framework and Diesel ORM. The purpose of the app is to subscribe to data feeds (rss, twitter, instagram etc) and notify you about new entries. For now, I finished synchronization of rss feeds and started working on implementing Telegram bot to work with the app (subscribe to feeds, notifications). You can find the application [here](https://github.com/ayrat555/el_monitorro).

### Book reading

I started reading both fiction and non-fiction (programming) books. Every day after breakfast I read a  programming book for one hour. I already finished re-reading Pragmatic Programmer by Dave Thomas (actually, it stirred me into emacs) and I'm halfway through Design Patterns: Elements of Reusable Object-Oriented Software by Erich Gamma, John Vlissides, Richard Helm, Ralph Johnson (I spend my days writing functional languages but some patterns from this book still are very helpful).

Before going to bed I read a fiction book to improve my English. As you may have guessed, English is not my native language.

### Painting

As I mentioned in this blog several times I worked on a [mobile game](https://thoughtkraken.com/life_balance) last year. I bought a painting tablet to draw art for it. Since it stays idle anyway, I decided why not to learn to paint. Of course, my quality of drawing will never be as good as the quality of drawings from professional artists. Here's some example of my drawings:

![cosmocat](/images/2020-03-29-cosmocat.jpeg)
![mask](/images/2020-03-29-mask.jpg)
![dove](/images/2020-03-29-dove.png)

You can find more art in my [deviantart page](https://www.deviantart.com/ayratb).

### Conclusion

Org-habit (and org-mode in general) is not a magic pill that can make you consistent. It's only a tool that can help you keep track of activities that you want to develop into habits. Your determination is the key that can help you reach your goals and fight with procrastination.

Also, you may not have as much free time as I do if you, for example, have children or have some hobbies. You can choose just one activity to develop it into a habit.
