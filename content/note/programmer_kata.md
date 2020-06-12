+++
title = "Improving Programmer Kata"
description = "Form through Practise"
date = 2020-06-12T12:59:50+05:30
tags = ["tech", "tips", "learning"]
+++

Becoming a senior programmer/engineer doesn’t happen in a day.

> Like most skills, often [you need grit][grit] than intelligence/smartness.

# Guidelines

* Write small toy programs to prove yourself right / wrong
  - It’s easy to get lost in the details of a large project; however it’s just noise
  - A toy programs is a simple way of filtering signal from noise
* Document what you learn
  - Sprinkle enough comments; they'll help you later
  - Write markdown articles for involved topics
* Keep a repository handy
  - This should have all your workouts, big or small
  - It used be usable any where without installing big tool chains
  - Preferably plain text; searchable readily with omnipresent tools `grep` and `find`
* Write something substantial -- a mini-project
  - Professionally most programmers are just _cogs in a large wheel_
  - Authoring a small project end-to-end leads to much deeper insights
* Scope
  - Start with something small and tangible
  - Not having something decent in a week is a sign
  - You're biting more than you could chew, cut back
* Always [RTFM][]
  - Mark of a good engineer: ability to digest design, API documentation or books
  - Grok documentation and articles written by better programmers
* **Be consistent**
  - Work on your pet project at least an hour a day
  - An hour isn't much -- consider the minutes that go wasted in a day
  - _Trick_: small, incremental changes every day; _don’t lose steam_
* Write cross-platform and cross toolchain code
  - It makes you a better programmer
  - Code adhering to standards has longevity and better portability
  - Widens user base
* Toolchain
  - Use the same set of tools everywhere (across platforms)
  - Improves muscle memory
  - Sticking to a same set of tools lets tools not get in your way
  - _Examples_: [coreutils][], [findutils][], [binutils][], [Emacs][], [Vim][], etc.
* Dependencies
  - Minimize your dependencies; [_dependency hygiene trumps code reuse_][go-dep-hyg]
  - Maintaining dependencies isn't fun
  - Pick one that has decent maintenance and is well-designed
  - Pick on that supports all the platforms you've in mind

## To Depend or Not?
If you’re writing an image viewer, then rolling your own PNG decoder is a project in itself, perhaps you can take a dependency on a library like `libpng`.  However, if your project is something else; you need to read just one field in a PNG’s header, then taking a library to just do that is an overkill.

# Mini-project Suggestions

I’m a systems/middleware programmer, I’m partial to projects of those kinds.  Feel free to pick any project you fancy!

* [Ray tracer][]
  - High-quality free tutorials [\[1\]][weekend-raytrace], [\[2\]][SAP-raytrace]
  - Lots of scope for parallelization
  - Pluggable into 3D packages like [Blender][], Maya, etc.
* Image viewer / editor
  - Exposes you to GUI programming and user settings management
  - [Filters/Effects][img-processing]
  - Lots of scope for parallelization
* Text Viewer / Editor
  - Infinite extensibility
    + Syntax Highlighting
    + Auto-completion, jump-to-definition ([LSP][])
    + Spell checking
  - Find/replace – regex
  - Buffering (when dealing with very large files; [memory mapping][mmap])
  - Ideas like [MVC][], [MVVM][]
* File Explorer
  - nCurses-based ;)
  - Interfacing with the OS
  - File systems
* Arbitrary precision calculator
  - Exposes to big numbers and floating-point intricacies
  - Libraries like [GMP][], [GNU MPFR][MPFR], etc.
* Simple game
  - Exposes to graphics, performance, math, GPU programming (shader code)
  - Great beginner tutorials like [lazyfoo][]’s
  - Free tools/engines: [Tiled][], [Godot][]
  - Free libraries: [SDL][], [Raylib][], etc.
  - Free assets: [Kenny NL][], [OpenGameArt][OGA], etc.
* Media player
  - Exposes to encoding, decoding, streams, audio-Video sync
  - Libraries like [ffmpeg][]
* Video post-processing / filters
  - WebGL video processing
* Graph plotter
* Markdown renderer / viewer
* Procedural content generation
* Language Server Protocol (for your favourite editor)
  - Parser
  - Lexer
  - Compiler-toolchain


[grit]: https://www.bakadesuyo.com/2012/11/secret-success-not-giving-up/
[RTFM]: https://en.wikipedia.org/wiki/RTFM
[coreutils]: https://www.gnu.org/software/coreutils/coreutils.html
[binutils]: https://www.gnu.org/software/binutils
[findutils]: https://www.gnu.org/software/findutils/
[Emacs]: https://www.gnu.org/software/emacs/
[Vim]: https://www.vim.org/
[go-dep-hyg]: https://talks.golang.org/2012/splash.article#TOC_7.
[weekend-raytrace]: http://www.realtimerendering.com/raytracing/Ray%20Tracing%20in%20a%20Weekend.pdf
[Ray tracer]: https://en.wikipedia.org/wiki/Ray_tracing_(graphics)
[SAP-raytrace]: https://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-ray-tracing
[Blender]: https://www.blender.org/
[img-processing]: https://en.wikipedia.org/wiki/Digital_image_processing
[GMP]: https://en.wikipedia.org/wiki/GNU_Multiple_Precision_Arithmetic_Library
[MPFR]: https://en.wikipedia.org/wiki/GNU_MPFR
[LSP]: https://en.wikipedia.org/wiki/Language_Server_Protocol
[MVC]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[MVVM]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel
[mmap]: https://en.wikipedia.org/wiki/Memory-mapped_file
[ffmpeg]: https://ffmpeg.org
[Tiled]: https://www.mapeditor.org/
[Kenny NL]: https://kenney.nl/
[OGA]: https://opengameart.org/
[Godot]: https://godotengine.org/
[Raylib]: https://raylib.com/
[SDL]: https://libsdl.org/
[lazyfoo]: http://lazyfoo.net/articles/article01/index.php
