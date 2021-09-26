---
layout: post
title: My Braindump
date: 2021-08-29
summary: Exporting your notes into a static site
categories: common
---

![brain](/images/2021-08-29-brain.png)

Exactly 6 months ago I wrote a [post](/common/2021/03/28/org-roam/) about zettelkasten and [org-roam](https://github.com/org-roam/org-roam).

Now I accumulated a set of useful knowledge nuggets that I'm returning to again and again. I think these notes can be useful not only for myself. So I decided to export them into [my braindump](https://braindump.badykov.com/about) - a static site with my notes.

In this post, I'll describe simple steps to create a static site with your braindump.

## Org-roam

I use org-roam to take my notes. Org-roam uses [org mode](https://www.badykov.com/emacs/2018/08/26/be-productive-with-org-mode/) files for notes. Org-roam is one of the most powerful note-taking tools.

Some static site generators have built-in support for org-mode files but you can easily export them to different formats (markdown, html, etc). The list of projects I've tried:

- [firn](https://github.com/theiceshelf/firn) - Org Mode Static Site Generator. It doesn't support uppercase org properties used by org-roam.
- [zola](https://github.com/getzola/zola) - A fast static site generator in Rust. Being a fan of the Rust language myself, the idea of using it seemed nice. But I couldn't find a good enough theme for my notes :)
- [hugo](https://github.com/gohugoio/hugo) - A static site generator in Go. It's the most popular one on this list. And there are already braindump sites using it. For example, [https://braindump.jethro.dev/](https://braindump.jethro.dev/) and [https://sidhartharya.me/braindump/index.html](https://sidhartharya.me/braindump/index.html).  So I decided to go with it.

The only problem I experienced is that the note export wasn't smooth for me. So for now I exported just a handful of notes, but I'm planning to create some kind of tool to easily export my public notes with a single command.

## Tweaking Hugo theme

I chose [the cortex Hugo theme](https://github.com/jethrokuan/cortex) by Jethro Kuan. But in my opinion, it has a big flaw - it doesn't have search functionality. I think this feature a critical to make your braindump useful not only for people visiting your site but also for yourself. So I decided to add this functionality.

Fortunately, I'm not the first person stumbling upon this problem. There is [a good post](https://victoria.dev/blog/add-search-to-hugo-static-sites-with-lunr/) describing steps to add search with lunr.js. So I used this post to add a search feature to my braindump. Also, I created [a PR](https://github.com/jethrokuan/cortex/pull/8) to Jethro Kuan's cortex theme with my changes.

Another modification I made to the cortex theme is I changed its CSS styles to make my braindump darker. Because everything looks cooler in the dark mode. You can find my version of the cortex theme at [https://github.com/ayrat555/cortex-dark](https://github.com/ayrat555/cortex-dark). I'm by no means a CSS expert so my changes may seem dirty for some of you. But it got the job done.

## Deploying on GitHub pages

The last step is to deploy a braindump. I used github pages for that. You can host your public static sites freely on github and the deployment process is super easy. You can find the source code of my braindump at [https://github.com/ayrat555/braindump](https://github.com/ayrat555/braindump).

## Conclusion

- [https://github.com/ayrat555/cortex-dark](https://github.com/ayrat555/cortex-dark) - dark theme for braindump (hugo)
- [https://github.com/ayrat555/braindump](https://github.com/ayrat555/braindump) - source code for my braindump
- [https://braindump.badykov.com/](https://braindump.badykov.com/about) - my braindump

The task for next weekend is to figure out a way to smoothly export my public notes. ;)
