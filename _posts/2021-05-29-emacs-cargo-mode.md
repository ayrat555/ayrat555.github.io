---
layout: post
title: Cargo mode for Emacs
date: 2021-05-29
summary: Emacs minor mode for the Rust package manager
categories: emacs
---

![frank](/images/2021-05-29-cargo.jpeg)

In this post, I'll describe why and how I created a cargo package for emacs - [cargo-mode](https://github.com/ayrat555/cargo-mode)

## Why

### Rust and Emacs

This year I started working with Rust code more than usual:

- I added a couple of features to my Telegram feed reader bot in Rust -  [El Monitorro](https://github.com/ayrat555/el_monitorro)
- I wrote a telegram API client for Rust - [Frankenstein](https://github.com/ayrat555/frankenstein)
- Currently, I'm investigating background processing using Rust because I'm not completely satisfied with Tokio for this job in el monitorro. By the way, a while ago I wrote [a post](https://www.badykov.com/rust/2020/06/28/you-dont-need-background-job-library/) about Tokio and background jobs. But it turns out it's not suitable for this if you have thousands of jobs (at least in a naive form that I used).

I've been using Emacs as my main and only text editor for four years already. Emacs is not a perfect text editor, but it may be the best extendable text editor out there. It provides users with Emacs Lisp programming language that can be used to add additional features to the editor. Great examples of this are Magit and Org-Mode. Magit may be the best git UI.

As I started digging into Rust code more, I found that my workflow using cargo tasks is far from perfect. Every time I need to execute a cargo task, I'm switching to the command line to execute it. So I decided to create an Emacs package to select and execute a cargo command from Emacs.

### Issues in the existing cargo package

There is already a cargo package for emacs but it has a couple of issues:

- it hardcodes commands into the package and you can not execute a task if it's not defined in the package [the issue](https://github.com/kwrooijen/cargo.el/issues/109)
- it doesn't remember the last executed task ([the issue](https://github.com/kwrooijen/cargo.el/issues/110))

But still, it has some good features around executing tests based on regexp matching. So I just copied them into my package.


## How

More than a year ago I already created a similar package for Elixir - [mix.el](https://github.com/ayrat555/mix.el). For the cargo-mode package, I used the same approach. But in the case of `mix.el`, it took me around 10 days of 1-2 hour sessions to write (mostly because it was my first emacs package), in the case of `cargo-mode` it took me one weekend (2 days) of 2-3 hour sessions because the functionality is similar.

For people familiar with Emacs Lisp (Common Lisp and Clojure will also work :) ), let's briefly go over the main pieces of code.


###  Fetching available commands

```elisp
(defun cargo-mode--fetch-cargo-tasks (project-root)
  "Fetch list of raw commands from shell for project in PROJECT-ROOT."
  (let* ((default-directory (or project-root default-directory))
         (cmd (concat (shell-quote-argument cargo-path-to-bin) " --list"))
         (tasks-string (shell-command-to-string cmd))
         (tasks (butlast (cdr (split-string tasks-string "\n")))))
    (delete-dups tasks)))
```

In this function, commands are fetched using `shell-command-to-string` function which executes the shell command `cargo --list` and returns its output as a string. After fetching a string with commands, commands are formatted in other functions not presented here.


### Selecting cargo task

```elisp
(defun cargo-mode-execute-task (&optional prefix)
  "Select and execute cargo task.
If PREFIX is non-nil, prompt for additional params."
  (interactive "P")
  (let* ((project-root (cargo-mode--project-directory))
         (available-commands (cargo-mode--available-tasks project-root))
         (selected-command (completing-read "select cargo command: " available-commands))
         (command-without-doc (car (split-string selected-command))))
    (cargo-mode--start "execute" command-without-doc project-root prefix)))
```

In this function, a user selects a command that she wants to execute with `completing-read`. After that, the selected command is passed into `cargo-mode--start` which executes the command.


### Executing the command

```elisp
(defun cargo-mode--start (name command project-root &optional prompt)
  "Start the `cargo-mode` process with NAME and return the created process.
Cargo command is COMMAND.
The command is  started from directory PROJECT-ROOT.
If PROMPT is non-nil, modifies the command."
  (let* ((buffer (concat "*cargo-mode " name "*"))
         (path-to-bin (shell-quote-argument cargo-path-to-bin))
         (base-cmd (if (string-match-p path-to-bin command)
                  command
                  (concat path-to-bin " " command)))
         (cmd (cargo-mode--maybe-add-additional-params base-cmd prompt))
         (default-directory (or project-root default-directory)))
    (save-some-buffers (not compilation-ask-about-save)
                       (lambda ()
                         (and project-root
                              buffer-file-name
                              (string-prefix-p project-root (file-truename buffer-file-name)))))
    (setq cargo-mode--last-command (list name cmd project-root))
    (compilation-start cmd 'cargo-mode (lambda(_) buffer))
    (get-buffer-process buffer)))
```


This command executes a cargo command with `compilation-start` function.

### Conclustion

The package is available on [GitHub](https://github.com/ayrat555/cargo-mode). You can find more emacs and rust related posts in my blog
