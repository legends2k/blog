 +++
title = "Emacs C++ auto-complete on Windows"
description = "Using Irony, Clang & Co"
date = "2017-01-24T17:33:27+05:30"
tags = ["tech", "emacs", "language", "tools", "c++"]
+++

This note is a guide to setup auto completion for C and C++ modes in Emacs on Windows with minimal manual work.  We need [Clang][], [Irony][] and some [Company][]; we additionally require [CMake][] and [MinGW][] for building something small.  Although there are many methods of getting this working, this was the most performant and least time-taking solution for me, so I'm documenting it here.

The general installation instructions provided in Irony's `README` works but it asks you to build the complete Clang compiler from source for a single DLL (`libclang.dll`).  I don't know about you but this isn't exactly my idea of fun.  This article skips that and uses Clang's binary directly; this method has some quirks captured below to help others and myself.

[Clang]: http://clang.llvm.org/
[Irony]: https://github.com/Sarcasm/irony-mode
[Company]: https://company-mode.github.io/
[CMake]: http://www.cmake.org/
[MinGW]: http://mingw-w64.org/

# Packages

This article assumes that you are on Emacs 24.5 or later and have the following packages installed:

* `irony`
* `company`
* `company-irony`

I think mine are from [Melpa][].  Irony's homepage is a straight-forward installation guide.

[Melpa]: http://melpa.org/

# Company

*Company* (**Comp**lete **any**thing) is a text completion "UI" for Emacs.  It has support for a variety of backends; Company only shows the choices, while the choices itself come from a backend.  `company-irony` is one of the supported backends that in turn depends on `irony`.  Make `company-irony` the first in the list of backends for it to be picked up.  Enable `irony` and `company` modes in the `c-mode-common-hook` function (that is called for both `c-mode` and `c++-mode`).  Effectively, my `.emacs` looks thus:

{{< highlight lisp >}}
(eval-after-load 'company
  '(add-to-list 'company-backends 'company-irony))

(defun my-c-common-setup ()    ;; function called by c-mode-common-hook
  (irony-mode 1)
  (company-mode))
{{< /highlight >}}

Also fix key bindings to use Irony:

{{< highlight lisp >}}
;; replace the `completion-at-point' and `complete-symbol' bindings in
;; irony-mode's buffers by irony-mode's function
(defun my-irony-mode-hook ()
  (define-key irony-mode-map [remap completion-at-point]
    'irony-completion-at-point-async)
  (define-key irony-mode-map [remap complete-symbol]
    'irony-completion-at-point-async))

(add-hook 'irony-mode-hook 'my-irony-mode-hook)
(add-hook 'irony-mode-hook 'irony-cdb-autosetup-compile-options)
{{< /highlight >}}

This snippet was lifted from the project's homepage.  Make sure you do the Windows performance tweaks; there are [good reasons][irony_windows] to do this.

{{< highlight lisp >}}
(when (boundp 'w32-pipe-read-delay)
  (setq w32-pipe-read-delay 0))
;; Set the buffer size to 64K on Windows (from the original 4K)
(when (boundp 'w32-pipe-buffer-size)
  (setq irony-server-w32-pipe-buffer-size (* 64 1024)))
{{< /highlight >}}

[irony_windows]: https://github.com/Sarcasm/irony-mode/wiki/Setting-up-irony-mode-on-Windows

# Clang

1. Make sure CMake and MinGW are installed
    + I go for 32-bit MinGW
        * 64-bit binaries for libraries aren't common yet
        * 64-bit MinGW is [not truly dual target][mingw64_not_dual].
    + I choose POSIX threading mechanism
        * The Win32 variant [doesn't support C++11 concurrency][mingw_win32_threads].
    + I prefer Dwarf for exception handling
    + I used CMake 3.6.2 and MinGW 6.3.0
2. Install LLVM from a [pre-built binary package][llvm_download]
    + It should match the *bit-ness* of the MinGW toolchain installed
    + Verify that the installed directory has `/bin/libclang.dll` or `/bin/clang.dll`
    + The LLVM binaries (specifically `clang.dll`) should have been built with MinGW
        * The ones built with MSVC didn't seem to work for me
    + I chose [`LLVM-3.9.1-win32.exe`][llvm391]

[mingw64_not_dual]: http://stackoverflow.com/questions/16304804/dual-target-mingw-w64-isnt-really-dual-target
[mingw_win32_threads]: http://stackoverflow.com/q/13741711/183120
[llvm_download]: http://releases.llvm.org/download.html
[llvm391]: http://releases.llvm.org/3.9.1/LLVM-3.9.1-win32.exe

# Irony Server

While you type out code in a buffer with `c++-mode` enabled, `irony-mode` will get enabled and that would run a `irony-server.exe` in the background.  This program will talk to Clang as you type code into the buffer to get completion suggestions in real-time.  But this program doesn't come pre-built; it has to be built manually.  This is the only manual building in our entire procedure:

1. On a _Command Prompt_, `cd` to Irony's package site
    + e.g. `~/.emacs.d/elpa/irony-20161227.348/server`
2. `mkdir build && cd build`
4. `cmake -DLIBCLANG_LIBRARY=C:\PROGRA~2\LLVM\bin\libclang.dll -G "MinGW Makefiles" ..`
    + `LIBCLANG_LIBRARY` is set manually to make sure the right DLL is picked up
    + Same may be needed for the `INCLUDE` directory as well
    + Notice that I have made sure there're no spaces in the path, even when `Program Files (x86)` is part of it.  I have used its short name
        * Do `dir /x` to get a directory's short name
5. `mingw32-make`
6. Open a `.cpp` file and run `M-x irony-install-server`; don't press `Return`
7. Copy the command shown and cancel it (`C-g`).  Run it on `cmd`
    + Before running on `cmd` un-escape what Emacs escaped
    + `\=` becomes `=`, `\:` becomes `:` and path separator would be `\` not `/`
8. CMake should place `irony-server.exe` in `~/.emacs.d/irony/bin`.
9. `cpp -xc++ -Wp,-v < NUL`
10. Save just the (search) list of directories in a file named `.clang_complete`
11. Change it into compiler flags that `clang++` understands (see sample below)
12. Put this file in the project root or your home directory
13. Restart Emacs
14. A buffer with `c++-mode`, `irony-mode` and `company-mode` should display suggestions
    + Save it to a persistent file if it's not backed by one

**Sample `.clang_complete`**

{{< highlight cfg >}}
-std=c++14
-target
i686-w64-windows-gnu
-Wall
-pedantic
-DDEBUG
-IF:/Apps/mingw32/lib/gcc/i686-w64-mingw32/6.3.0/include/c++
-IF:/Apps/mingw32/lib/gcc/i686-w64-mingw32/6.3.0/include/c++/i686-w64-mingw32
-IF:/Apps/mingw32/lib/gcc/i686-w64-mingw32/6.3.0/include/c++/backward
-IF:/Apps/mingw32/lib/gcc/i686-w64-mingw32/6.3.0/include
-IF:/Apps/mingw32/lib/gcc/i686-w64-mingw32/6.3.0/include-fixed
-IF:/Apps/mingw32/i686-w64-mingw32/include
-IF:/Libs/boost_1_57_0
{{< /highlight >}}

**Note**: each argument gets its own line

Giving `-std=c++14` ate hours of my time investigating why C++14 syntax and tokens aren't suggested by Clang.  I then realized that I had installed Clang 3.4 and [the documentation][clang_doc] clearly states that `-std=c++1y` is to be used for versions 3.4 or earlier.

As a verification of things working fine, check if a process of `irony-server.exe` is running smoothly.

[clang_doc]: http://clang.llvm.org/cxx_status.html#cxx14

# Irony Flycheck

As a bonus of using Irony, we can do on-the-fly syntax checking using Clang and know of errors as the code is typed out without actually compiling the file separately.  This quickens the development cycle by avoiding slow compilations; when you have an error-free buffer, you'll only be left with linker errors, if at all, when you do a proper compilation.  Install `flycheck`, `flycheck-irony` and `flycheck-pos-tip`.  Enable it with

{{< highlight Lisp >}}
(eval-after-load 'flycheck
  '(add-hook 'flycheck-mode-hook #'flycheck-irony-setup))
{{< /highlight >}}

where `flycheck-irony-setup` is a function you get with the `flycheck-irony` package.  Make sure you enable Flycheck when needed

{{< highlight Lisp >}}
(defun my-c-common-setup ()
  (flycheck-mode)
  (flycheck-pos-tip-mode))
{{< /highlight >}}

Flycheck shows the errors in a separate buffer which keeps getting update as you type.  But you can see the error in a particular statement without having to open this buffer with `flycheck-pos-tip-mode`.  It will show error tooltips as the cursor is moved to an erroneous token.

# Advantages

Without having a bloated IDE, you get to have both auto-complete (a.k.a. intellisense) and on-the-fly syntax checking.  They are valuable tools when writing code in a project with large code bases, taking lot of time to build.  Also, most IDEs require you to create a project for some single-file, toy, sample code you want to scribble; in our setup you just need to create a new buffer backed by a file on disk i.e. the usual drill for any new file.  Make sure you have the right `.clang_complete` saved in your _Home_ directory (where you have your code lab of dirty files) with the required `include` paths and compiler flags.

The advantage of Irony is that it uses a proper compiler (Clang) to do this job and hence the results are very reliable.

Finally, you can have a look at my [`.emacs`][config_file] file if you've doubts.

[config_file]: https://bitbucket.org/rmsundaram/tryouts/src/master/Misc/config/.emacs
