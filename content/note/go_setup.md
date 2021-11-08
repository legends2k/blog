+++
title = "Emacs Go Setup"
description = "Make Emacs your Go IDE"
date = "2018-06-17T07:00:43-07:00"
tags = ["tech", "language", "tools", "emacs"]
+++

An IDE should at least aid with

- Auto-completion (a.k.a _Intellisense_)
- Jump to definition
- On-the-fly error checks

Additionally it’d be good to have

- Context-sensitive document lookup
    + Show variable and function type
    + Show function description
- Auto-run `gofmt` on save
- Auto-add/remove used/unused packages on file save

We’ll get these working for Go on Emacs using LSP.  It’s no secret that [Emacs’
LSP support][emacs-lsp] is top notch.

> We assume Go 1.17+ i.e. with full modules support and `GO111MODULE`
> unset[^GO111MODULE-unset].

[emacs-lsp]: https://emacs-lsp.github.io

## Go, please!

Yeah, that’s the name of `gopls`, [Go’s LSP server][gopls] 😉.  If your package
manager doesn’t have it, `go install` it:

{{< highlight bash >}}
go install golang.org/x/tools/gopls@latest
{{< /highlight >}}

## Emacs Setup

Install `go-mode` package from Melpa and hook it to `lsp-mode`:

{{< highlight lisp >}}
(use-package go-mode
  :mode "\\.go\\'"
  :config
  (defun my/go-mode-setup ()
    "Basic Go mode setup."
  (add-hook 'before-save-hook #'lsp-format-buffer t t)
  (add-hook 'before-save-hook #'lsp-organize-imports t t))
  (add-hook 'go-mode-hook #'my/go-mode-setup))

(use-package lsp-mode
  :ensure t
  :commands (lsp lsp-mode lsp-deferred)
  :hook ((rust-mode python-mode go-mode) . lsp-deferred)
  :config
  (setq lsp-prefer-flymake nil
        lsp-enable-indentation nil
        lsp-enable-on-type-formatting nil
        lsp-rust-server 'rust-analyzer)
  ;; for filling args placeholders upon function completion candidate selection
  ;; lsp-enable-snippet and company-lsp-enable-snippet should be nil with
  ;; yas-minor-mode is enabled: https://emacs.stackexchange.com/q/53104
  (lsp-modeline-code-actions-mode)
  (add-hook 'lsp-mode-hook #'lsp-enable-which-key-integration)
  (add-to-list 'lsp-file-watch-ignored "\\.vscode\\'"))
{{< /highlight >}}

[gopls]: https://github.com/golang/tools/tree/master/gopls#installation

## Verify

Make sure `gopls` is in `$PATH`.  If you’ve `go install`-ed, make sure to add
`$GOPATH/bin` to `$PATH`.  Since projects use modules now, I use `$GOPATH` only
to house Go tools which aren’t part of the official toolchain.  Similar to
`${HOME}/.cargo` for Rust, I set `${HOME}/.go` as my `$GOPATH` and fix `$PATH`.

Read `go help gopath` and `go help install` for details.

Add entries with a trailing slash; missing it may lead to issues.

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
* Flycheck, with no setup, does a good job of flagging warnings/errors
    - `flycheck-verify-setup` is useful at times
* Context-sensitive document lookup
    - Positioning point on a variable/function should show its type in the minibuffer
    - To get a detailed description of a function do `godoc-at-point` (needs `godoc`)
    - You could bind it to a key combo; it’s fairly handy
* Auto-formatting
    - When you save the buffer it auto-formats with `gofmt`
* Auto-include/exclude
    - At the top-level, include packages you wouldn’t reference
    - In functions from packages you didn’t import
    - Save the buffer!  Redundant imports are deleted, needed ones are added 😉
    - [Go is strict about unused imports][no-unused-imports] and this helps

This is a stable and light setup.  I’m happy with it for now.

[no-unused-imports]: https://golang.org/doc/effective_go.html?#blank_unused

# References

1. `go help install`
2. `go help gopath`
3. [gopls Documentation][gopls]

[^GO111MODULE-unset]: Go 1.17+ ignores `GO111MODULE` and assumes modules mostly.
