+++
title = "Go Setup"
description = "Make Emacs your Go IDE"
date = "2018-06-17T07:00:43-07:00"
tags = ["tech", "ide", "tools", "go", "emacs"]
+++

An IDE at least should do

- Auto-completion (a.k.a Intellisense)
- Jump to definition
- On-the-fly error checks
- Context-sensitive document lookup
    + Show variable and function type
    + Show function description 
- Autorun `gofmt` on save
- **Bonus**: Auto-add/remove used/unused packages on file save

We’ll try to get these going on Emacs.

## External Tools

Though not part of the canonical tool set, these are written by seasoned Go devs

* gocode
* godef
* goflymake
* goimports
* godoc

## Emacs Packages

We’d be needing these too to interface with above tools

* go-mode
* eldoc
* company-go

It's assumed that you already have `company` installed.

## Steps

1. Install Go binary
    + Linux: use your distro’s package manager
    + Windows: download and install 64-bit MSI package
2. Set `$GOPATH`
    + It can have multiple entries like `$PATH`
    + First one’s where packages are downloaded by `go get`
    + Source code is searched in all entries
    + Add entries with a trailing slash
    + Read `go help gopath`
3. Go get these tools.  On Windows add `-ldflags -H=windowsgui` for `gocode`
{{< highlight basic >}}
go get -u github.com/mdempsky/gocode
go get github.com/rogpeppe/godef
go get -u github.com/dougm/goflymake
go get golang.org/x/tools/cmd/goimports
go get golang.org/x/tools/cmd/godoc
{{< /highlight >}}
4. Install aforementioned Emacs packages
5. If `$GOPATH/bin` isn’t part of `$PATH` at least make it part of Emacs' `exec-path`
{{< highlight lisp >}}
(add-to-list 'exec-path (concat (file-name-as-directory (getenv "GOPATH")) "bin") t)
{{< /highlight >}}
6. Append to `.emacs`:
{{< highlight lisp >}}
(add-to-list 'load-path (concat (file-name-as-directory (getenv "GOPATH")) "src/github.com/dougm/goflymake"))
(require 'go-flymake)
; Use goimports instead of go-fmt for formatting with intelligent package addition/removal
(setq gofmt-command "goimports")
(add-hook 'go-mode-hook (lambda ()
                          (set (make-local-variable 'company-backends) '(company-go))
                          (local-set-key (kbd "M-.") 'godef-jump)
                          (go-eldoc-setup)
                          ; call Gofmt before saving
                          (add-hook 'before-save-hook 'gofmt-before-save)))
{{< /highlight >}}
7. Periodically `go get -u all` to keep the downloaded tools updated.

# Usage

* If you’re not open an existing source file, be sure to save your fresh buffer.
    - Without saving you’d get warnings from tools
* Auto-completion
    - Test it by importing `fmt` and type `fmt.`
    - It should list the package’s public members
    - Cycle through the options with `M-n` and `M-p`
    - Additionally it’d also show the function prototype in the mini-buffer
* Jump to definition
    - When point is atop a function name, press `M-.`
    - This should open the file where it’s defined and seek to the definition
    - To return back to original place press `M-,`
* Flymake should work without any intervention
    - It’s too subtle; just highlights erroneous line with `!!` and underscores the problematic token
    - `flymake-show-diagnostics-buffer` should show you a constantly updated buffer with the actual errors/warnings
* Context-sensitive document lookup
    - Positioning point on a variable/function should show its type in the minibuffer
    - To describe variable/expression at point do `C-c C-d`
    - To get a detailed description of a function do `godoc-at-point`
    - You could bind it to a key combo; it’s fairly handy
* Auto-formatting
    - When you save the buffer it auto-formats with `gofmt`
* Auto-include/exclude
    - [Go is strict](https://golang.org/doc/effective_go.html?#blank_unused) on what you import
    - At the top-level, include packages you wouldn’t reference
    - In functions from packages you didn’t import
    - Save the buffer!  Redundant packages are wiped out and needed ones are imported 😉

This setup is light and good enough for now.

# To Do

Play with [`guru`](https://godoc.org/golang.org/x/tools/cmd/guru) -- looks like a sophisticated tool.

# References

1. `go-mode`'s help in Emacs
2. `$GOPATH/src/github.com/mdempsky/gocode/emacs-company/README.md`
3. [Configure Emacs as a Go Editor From Scratch](https://tleyden.github.io/blog/2014/05/27/configure-emacs-as-a-go-editor-from-scratch-part-2/)
4. [goflymake](https://github.com/dougm/goflymake)
5. [gocode](https://github.com/mdempsky/gocode)
6. [ArchWiki’s Go page](https://wiki.archlinux.org/index.php/Go)
