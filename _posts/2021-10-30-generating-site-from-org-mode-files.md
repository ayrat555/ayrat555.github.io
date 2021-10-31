---
title: Generating a site from org-mode files
date: 2021-10-31
summary: My org-roam enhancements
categories: emacs
---

![cover](/images/2021-10-31-blog.png)

[One of my recent posts](https://www.badykov.com/common/braindump/) was about [my braindump](https://braindump.badykov.com/) ([repo](https://github.com/ayrat555/braindump)). In this post, I mentioned that I have to find a way to gracefully generate a site from my org-mode files without interfering with the way I'm taking notes.

I found it. And this post I'm going to share with you.

## GitHub actions

The site is built from org-mode files with [github actions](https://github.com/features/actions) and [hugo](https://gohugo.io/). It's CI/CD integrated into GitHub.

Let's take a look at its steps to get an overview of what's going on:

1. Fetch the project and its theme
```yml
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod
```

This step is straightforward. It just fetches the project and its submodule [hugo theme](https://github.com/ayrat555/cortex-dark).

2.  Install sqlite
```yml
   - name: Install org-roam dependencies
        run: |
          sudo apt-get install sqlite3

```

[org-roam](https://www.badykov.com/common/org-roam/) uses the SQLite database to store metadata about notes and provide search features. That's why we have to install SQLite

3. Install emacs

```yml
      - name: Install emacs
        uses: purcell/setup-emacs@master
        with:
          version: '27.1'
```

This step installs emacs and [ox-hugo](https://github.com/kaushalmodi/ox-hugo) which are used to convert org-mode files into hugo markdown files.


4. Convert org-mode files to hugo markdown

```yml
      - name: Convert org files to hugo
        run: make org2hugo
```

This step does the actual conversion. We will take a closer look into it a little bit later.


5. Build and deploy with Hugo
```yml

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.85.0'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/master' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          cname: braindump.badykov.com
```

The steps above are from [the provided hugo actions](https://github.com/peaceiris/actions-hugo) which install hugo, convert markdown file into html files and deploy the project

## Makefile

The makefile below creates the build directory and call elisp script which installs ox-hugo and converts org-mode files into hugo markdown files.

```bash
BASE_DIR=$(shell pwd)
SOURCE_ORG_FILES=$(BASE_DIR)/notes
EMACS_BUILD_DIR=/tmp/notes-home-build/
BUILD_DIR=/tmp/notes-home-build/.cache/org-persist/
HUGO_SECTION=notes

all: org2hugo

.PHONY: org2hugo
org2hugo:
	mkdir -p $(BUILD_DIR)
	cp -r $(BASE_DIR)/init.el $(EMACS_BUILD_DIR)
        # Build temporary minimal EMACS installation separate from the one in the machine.
	HOME=$(EMACS_BUILD_DIR) NOTES_ORG_SRC=$(SOURCE_ORG_FILES) HUGO_SECTION=$(HUGO_SECTION) HUGO_BASE_DIR=$(BASE_DIR) emacs -Q --batch --load $(EMACS_BUILD_DIR)/init.el --execute "(build/export-all)" --kill
```

## Emacs lisp script

```elislp
;;; build.el --- Minimal emacs installation to build the website -*- lexical-binding: t -*-
;; Based on the one from Bruno Henirques: https://github.com/bphenriques/knowledge-base/blob/8fb31838fd3f682d602eeb269ee7d92eecbbb8dc/tools/init.el
;;
;;; Commentary:
;;; - Requires NOTES_BASE_ORG_SOURCE environment variable
;;; Code:

(require 'subr-x)

(toggle-debug-on-error)      ;; Show debug informaton as soon as error occurs.
(setq make-backup-files nil) ;; Disable "<file>~" backups.

(defconst notes-org-files
  (let* ((env-key "NOTES_ORG_SRC")
         (env-value (getenv env-key)))
    (if (and env-value (file-directory-p env-value))
        env-value
      (error (format "%s is not set or is not an existing directory (%s)" env-key env-value)))))

;; Setup packages using straight.el: https://github.com/raxod502/straight.el
(defvar bootstrap-version)
(let ((bootstrap-file
       (expand-file-name "straight/repos/straight.el/bootstrap.el" user-emacs-directory))
      (bootstrap-version 5))
  (unless (file-exists-p bootstrap-file)
    (with-current-buffer
        (url-retrieve-synchronously
         "https://raw.githubusercontent.com/raxod502/straight.el/develop/install.el"
         'silent 'inhibit-cookies)
      (goto-char (point-max))
      (eval-print-last-sexp)))
  (load bootstrap-file nil 'nomessage))

(setq straight-use-package-by-default t)
(straight-use-package 'use-package)

(use-package ox-hugo
  :straight (:type git :host github :repo "kaushalmodi/ox-hugo"))

;;; Public functions
(defun build/export-all ()
  "Export all org-files (including nested) under notes-org-files."

  (setq org-hugo-base-dir
    (let* ((env-key "HUGO_BASE_DIR")
           (env-value (getenv env-key)))
      (if (and env-value (file-directory-p env-value))
          env-value
        (error (format "%s is not set or is not an existing directory (%s)" env-key env-value)))))

  (setq org-hugo-section "notes")

  (dolist (org-file (directory-files-recursively notes-org-files "\.org$"))
    (with-current-buffer (find-file org-file)
      (message (format "[build] Exporting %s" org-file))
      (org-hugo-export-wim-to-md :all-subtrees nil nil nil)))

  (message "Done!"))

(provide 'build/export-all)

;;; init.el ends here

```
## Conclusion

You can check out the original scripts in [my braindump repo](https://github.com/ayrat555/braindump).
