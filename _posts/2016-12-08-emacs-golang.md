---
layout: post
title: Set up Emacs for Golang from scratch - December 2016
---

This is a summary of how I set up Emacs for Go development from
scratch on macOS. It is based on the refs & inspiration mentioned
below, and brought up to date for the start of 2017.

*TLDR: The complete list of `go get` and `M-x package-install`
 commands, and the Emacs `init.el` file are at the top of
 this document, followed by a step-by-step explaination.*

References and inspiration for this:

* Configure Emacs as a Go Editor: [Part 1](http://tleyden.github.io/blog/2014/05/22/configure-emacs-as-a-go-editor-from-scratch/), [Part 2](http://tleyden.github.io/blog/2014/05/27/configure-emacs-as-a-go-editor-from-scratch-part-2/), [Part 3](http://tleyden.github.io/blog/2016/02/07/configure-emacs-as-a-go-editor-from-scratch-part-3/)
* Using Go Guru: [Docs](https://docs.google.com/document/d/1_Y9xCEMj5S-7rv2ooHpZNH15JgRT5iM742gJkw5LtmQ/edit), [Video](https://www.youtube.com/watch?v=ak97oH0D6fI)


# Summary: All Commands Without Blah Blah

### Bash Config
```bash
# file: ~/.bash_profile
export GOPATH=~/Development/gocode
export PATH=$PATH:$GOPATH/bin
```

### Go Get

```bash
$ go get -u golang.org/x/tools/cmd/...
$ go get -u github.com/rogpeppe/godef/...
$ go get -u github.com/nsf/gocode
$ go get -u golang.org/x/tools/cmd/goimports
$ go get -u golang.org/x/tools/cmd/guru
$ go get -u github.com/dougm/goflymake
```

### M-x package-install

```
go-mode
exec-path-from-shell
auto-complete
go-autocomplete
flymake-go
neotree
atom-one-dark-theme
```

### Reminder

Remember to copy `go-guru.el` to your load path.

### Emacs Config (without auto-generated Custom stuff)

```cl
; file: ~/.emacs.d/init.el
;; High level aesthetic stuff
(tool-bar-mode -1)                  ; Disable the button bar atop screen
(scroll-bar-mode -1)                ; Disable scroll bar
(setq inhibit-startup-screen t)     ; Disable startup screen with graphics
(set-default-font "Monaco 12")      ; Set font and size
(setq-default indent-tabs-mode nil) ; Use spaces instead of tabs
(setq tab-width 2)                  ; Four spaces is a tab
(setq visible-bell nil)             ; Disable annoying visual bell graphic
(setq ring-bell-function 'ignore)   ; Disable super annoying audio bell

;; Make keyboard bindings not suck
(setq mac-option-modifier 'super)
(setq mac-command-modifier 'meta)
(global-set-key "\M-c" 'copy-region-as-kill)
(global-set-key "\M-v" 'yank)
(global-set-key "\M-g" 'goto-line)

;; Set up package repositories so M-x package-install works.
(require 'package) 
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/"))
(package-initialize)

(load-theme 'atom-one-dark t)       ; Color theme installed via melpa

;; Add a directory to the load path so we can put extra files there
(add-to-list 'load-path "~/.emacs.d/lisp/")

;; Snag the user's PATH and GOPATH
(when (memq window-system '(mac ns))
  (exec-path-from-shell-initialize)
  (exec-path-from-shell-copy-env "GOPATH"))

;; Define function to call when go-mode loads
(defun my-go-mode-hook ()
  (add-hook 'before-save-hook 'gofmt-before-save) ; gofmt before every save
  (setq gofmt-command "goimports")                ; gofmt uses invokes goimports
  (if (not (string-match "go" compile-command))   ; set compile command default
      (set (make-local-variable 'compile-command)
           "go build -v && go test -v && go vet"))

  ;; guru settings
  (go-guru-hl-identifier-mode)                    ; highlight identifiers
  
  ;; Key bindings specific to go-mode
  (local-set-key (kbd "M-.") 'godef-jump)         ; Go to definition
  (local-set-key (kbd "M-*") 'pop-tag-mark)       ; Return from whence you came
  (local-set-key (kbd "M-p") 'compile)            ; Invoke compiler
  (local-set-key (kbd "M-P") 'recompile)          ; Redo most recent compile cmd
  (local-set-key (kbd "M-]") 'next-error)         ; Go to next error (or msg)
  (local-set-key (kbd "M-[") 'previous-error)     ; Go to previous error or msg

  ;; Misc go stuff
  (auto-complete-mode 1))                         ; Enable auto-complete mode

;; Connect go-mode-hook with the function we just defined
(add-hook 'go-mode-hook 'my-go-mode-hook)

;; Ensure the go specific autocomplete is active in go-mode.
(with-eval-after-load 'go-mode
   (require 'go-autocomplete))

;; If the go-guru.el file is in the load path, this will load it.
(require 'go-guru)
```

-----

# Step-by-step explainations

Now that I've summarized everything above, if you really want to
understand each step, or if you want to be picky about what you
include, read on! 

### Environment Vars

Edit `~/.bash_profile` so it exports the `GOROOT`, `GOPATH`, and has appropriate entries to the executable `PATH`:

```bash
# file: ~/.bash_profile
export GOROOT=/usr/local/go            
export GOPATH=~/Development/gocode
export PATH=$PATH:$GOROOT/bin
export PATH=$PATH:$GOPATH/bin
```
Make sure this takes effect. You can use `source ~/.bash_profile`; I just start a new terminal.

### Get go tools

Get the associated development tools like `godoc`. Since I've installed these things before, I'll use the `-u` flag to update what I already have.

```bash
$ go get -u golang.org/x/tools/cmd/...
```

### Set up Emacs config file

Create a new `~/.emacs.d/init.el` file, or edit an existing one.

### Use the Melpa emacs package repo

```cl
; file: ~/.emacs.d/init.el
(require 'package) ;; You might already have this line
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/"))
(package-initialize) ;; You might already have this line
```

Now if you restart emacs and do `M-x package-list-packages` it will contact the melpa.org server and you'll see a bunch of packages in the list; many from gnu, some others from melpa.

### Install go-mode

Run `M-x package-install` and install `go-mode`. In recent emacs it loads this automatically, so if you start `foo.go` it puts you into Go mode.

### Install + use path helper

`M-x package-install` and install `exec-path-from-shell`.

Now add the following to your emacs config file. It will invoke a shell and have it print out PATH and GOPATH values. It adds a couple seconds to your emacs startup time, unfortunately.

```cl
; file: .emacs.d/init.el
(when (memq window-system '(mac ns))
  (exec-path-from-shell-initialize)
  (exec-path-from-shell-copy-env "GOPATH"))
```

### Install godef

Install the `godef` tool from the cmd line, again using `-u` to update an existing version:

```bash
$ go get -u github.com/rogpeppe/godef/...
```

**Note for `gb` users:** `godef` will not know how to find your project's files until you run a special function to figure out a proper `GOPATH`. You can do it manually or add a hook to run it as needed: `M-x go-set-project`. I ended up using `projectile` (a project-centric emacs plugin) and issue the `go-set-project` in the project switching hook.

### Projectile

I used `package-install` to install `projectile` and `flx-ido` to get around the `gb/godef/GOPATH` problem that was just mentioned. This is totally optional, but instructions are here for posterity. Once installed, edit your config file to contain:

```cl
(projectile-mode)
(defun my-switch-project-hook ()
  (go-set-project))
(add-hook 'projectile-after-switch-project-hook #'my-switch-project-hook)
```

Now, you'll have projectile mode begun automatically when emacs starts, and it runs `go-set-project` every time you switch projects with `C-c p p` (even if the target project is in Javascript or Ook).

### Create Go mode load hook

Emacs will load go-mode when you open a go source file. You can customize how your go-mode behaves by adding stuff to a hook. This is a good place to add other customizations for go-mode as well.

```cl
; file: .emacs.d/init.el
(defun my-go-mode-hook ()
  (add-hook 'before-save-hook 'gofmt-before-save) ; gofmt before every save
  ; Godef jump key binding                                                      
  (local-set-key (kbd "M-.") 'godef-jump)
  (local-set-key (kbd "M-*") 'pop-tag-mark)
  )
(add-hook 'go-mode-hook 'my-go-mode-hook)
```

### Other go mode customizations

These statements appear within the `my-go-mode-hook` definition. This way the effects are localized to go mode.

```cl
(local-set-key (kbd "M-p") 'compile)            ; Invoke compiler
(local-set-key (kbd "M-P") 'recompile)          ; Redo most recent compile cmd
(local-set-key (kbd "M-]") 'next-error)         ; Go to next error (or msg)
(local-set-key (kbd "M-[") 'previous-error)     ; Go to previous error or msg
```

### Autocomplete and go-autocomplete

In Emacs, `M-x package-install` `auto-complete`.

Add this statement to the go mode hook's function definition:

```cl
; file: ~/.emacs.d/init.el
(auto-complete-mode 1)
```

Now when you load a go source file, emacs will say you're in mode `Go AC`.

Next install go-autocomplete:

```bash
$ go get -u github.com/nsf/gocode
```

And `M-x package-install` `go-autocomplete`.

### goimports

Install `goimports`, which organizes the imports at the top of each file. It also adds/removes things intelligently. The `go get` isn't necessary if you installed all the tools above with `golang.org/x/tools/cmd/...`.

```bash
$ go get -u golang.org/x/tools/cmd/goimports
```

And somewhere in your `my-go-mode-hook()`:

```cl
; file: ~/.emacs.d/init.el
(setq gofmt-command "goimports")
```

### guru (replaces the deprecated oracle tool)

`guru` is a source code analysis tool that works with any editor to give you industrial-strength IDE stuff like call graphs (what calls X, find the definition of X, what are the aliases for X, etc).

You've already installed `guru` when you installed the mess of go tools earlier. `guru` is replaces the `oracle` tool, which you might have read about.

Get `go-guru.el` and put it somewhere in your load path. I put it in `~/.emacs.d/lisp`. I'm not sure why this should be necessary, given that go-mode is a thing, but you have to load it yourself.

```cl
; file: ~/.emacs.d/init.el - top level, not in your go hook
(add-to-list 'load-path "~/.emacs.d/lisp/")
(require 'go-guru)
```

And to configure emacs to do nice things with guru, consider adding stuff to your go mode load hook.

```cl
; file: ~/.emacs.d/init.el
(go-guru-hl-identifier-mode)
```

By default all of the commands are invoked via `C-c C-o ?` where `?` is a single letter (so, lift up on the Control key). They designed the keybindings pretty well so they are fairly mnemonic:

* **j** jump to definition
* **r** referrers
* **f** free names (== if a selected block is turned into a function, what params are needed?)
* **d** describe expression (== show function params or type of identifier)
* **i** show implements (== what interfaces does this thing implement?)
* **<** show callers to a function
* **>** show callees to a function
* **s** show call stack (== possible paths from `main()`)
* **p** show points-to
* **e** show which errors
* **c** show channel peers

For some of these you have to set a scope. I'm fuzzy on exactly what this means. I tried `main` (the name of the package I was working in) and it did not work. I then tried `.` and it worked. The [documentation](https://docs.google.com/document/d/1_Y9xCEMj5S-7rv2ooHpZNH15JgRT5iM742gJkw5LtmQ/edit#) says "The scope is typically specified as a comma-separated set of packages, or wildcarded subtrees like github.com/my/dir/...".

### flymake

Flymake will build your source file as you edit it, using a shadow copy of in-memory file, rather than the one on disk. It will show syntax errors as you make them.

`M-x package-install` `flymake-go`

```bash
$ go get -u github.com/dougm/goflymake
```

# Unrelated to Golang

### Neotree

Neotree is a popular sidebar thing to show file structure.

`M-x package-install` `neotree`

Now navigate to a source directory (or whatever) and do `M-x neotree-dir` and you can tell it which directory you want to open as a sidebar.

### Color Themes

The default look is pretty standard but you can personalize it with color themes. First check out this [compendium of emacs themes](https://emacsthemes.com/) and find some you like. I installed one via `M-x package-install` `atom-one-dark-theme`, and added a this line *below* the `(package-initialize)` call in my emacs init:

```cl
; file: ~/.emacs.d/init.el
(load-theme 'atom-one-dark t)
```

