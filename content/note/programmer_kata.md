+++
title = "Improving Programmer Kata"
description = "Form through Practise"
date = 2020-06-12T12:59:50+05:30
tags = ["tech", "tips", "learning"]
+++

Becoming a good programmer doesn’t happen in a day.

> Like most fields, [you need grit][grit] more than intelligence/smartness.

# Guidelines

* Write small toy programs to prove yourself right / wrong
  - It’s easy to get lost in the details of a large project; it’s just noise
  - A toy program is a simple way of filtering signal from noise
* Document what you learn
  - Sprinkle enough comments; they'll help you later
  - Write markdown articles with references for involved topics
* Always [RTFM][]
  - Mark of a good engineer: ability to digest design, API documentation or books
  - Grok documentation and articles written by better programmers
* Keep a repository handy
  - This should have all your workouts, big or small
  - Usable any where without installing big tool chains
  - Preferably plain text; searchable readily with omnipresent tools `grep` and `find`
* Write something substantial -- a mini-project
  - Professionally most programmers are just _cogs in a large wheel_
  - Authoring a small project end-to-end leads to much deeper insights
  - Great tools like [ripgrep][] are hobby projects
* Scope your project
  - Start with something small and tangible
  - Have something working in a week
  - If not, you're biting more than you could chew
    + scope down or burn out
* **Be consistent**
  - Work on your pet project at least an hour a day
  - An hour isn't much -- usually total wasted minutes/day is more
  - _Trick_: small, incremental changes every day; _don’t lose steam_
* Write cross-platform and cross toolchain code
  - Makes you a better programmer
  - Standards-adhering code has longevity and better portability
  - Widens user base
* Have a reliable toolchain setup
  - Use the same set of tools everywhere (across platforms)
  - Improves muscle memory; [kata][]
  - Sticking to a same set of tools lets them not get in your way; productivity
  - _Examples_: [coreutils][], [findutils][], [binutils][], [Emacs][], [Vim][], etc.
* Take dependencies cautiously
  - Minimize dependencies; [_dependency hygiene trumps code reuse_][go-dep-hyg]
  - Maintaining dependencies, in the long run, isn't fun
  - Pick ones supporting all your target platforms
  - Pick libraries having decent maintenance and design

## To Depend or Not?
If you’re writing an image viewer, don’t roll your own PNG decoder; it’s a project in itself.  Perhaps you can take a dependency on a library like `libpng`.  However, if your project is something else; you need to read just one field in a PNG’s header.  Depending on a library to do just that is an overkill.

# Mini-project Suggestions

I’m a systems/middleware programmer, I’m partial to projects of those kinds.  Feel free to pick any project you fancy!

* Partake in code jams
  - Limited scope and explicit timelines are good teachers
  - Recommendations
    + [Advent of Code][aoc] highly recommended!
    + [Game jams][game-jam] or your-domain-specific jams are [fun][alakajam]!
* [Ray tracer][]
  - Lots of good tutorials [\[1\]][weekend-raytrace], [\[2\]][SAP-raytrace]
  - Scope for parallelization
  - Pluggable into 3D packages like [Blender][], Maya, etc.
  - Exposure
    + 3D Math
    + Optimization
    + Image processing/formats
* Image viewer / editor
  - [Filters/Effects][img-processing]
  - Scope for extensibility and parallelization
  - Exposure
    + GUI programming
    + User settings management
    + File handling
* Text Viewer / Editor
  - Infinite extensibility
    + Syntax Highlighting
    + Auto-completion, jump-to-definition ([LSP][])
    + Spell checking
  - Find/replace – regex
  - Exposure
    + Buffering (when dealing with very large files; [memory mapping][mmap])
    + Ideas like [MVC][], [MVVM][]
    + System software concepts
* File Explorer (nCurses-based ;)
  - Exposure
    + OS APIs
    + File systems
* Arbitrary precision calculator
  - Exposure
    + Big numbers
    + Floating-point intricacies
    + Libraries like [GMP][], [GNU MPFR][MPFR], etc.
* Simple game
  - Great beginner tutorials like [lazyfoo][]’s
  - Free tools/engines: [Tiled][], [Godot][]
  - Free libraries: [SDL][], [Raylib][], etc.
  - Free assets: [Kenny NL][], [OpenGameArt][OGA], etc.
  - Exposure
    + Graphics
    + Performance
    + Math
    + GPU programming (shader code)
* Media player
  - Libraries like [ffmpeg][]
  - Exposure
    + Codecs
    + Streams
    + Audio-Video synchronisation
* Video post-processing / filters
* Graph plotter
* Markdown renderer / viewer
  - Exposure
    + Parsing
    + Text document formats
    + [Template engines][templators]
* Procedural content generation
  - Exposure
    + 3D Math
    + Rendering
    + GPGPU
* LSP support for your favourite language (enabling your fav. editor)
  - Exposure
    + Language grammar
    + Parser
    + Lexer
    + Compiler-toolchain

[How to become a Better Developer][better-dev] admonitions to

* Challenge yourself
  - expose yourself to ideas you don’t understand
  - try new languages or idioms
* Study the masters
  - read better works and blogs
* Learn the classics
  - concentrate on ideas/theory not tools (frameworks, OS, etc)
* Practice Mindfully
  - Participate in programming challenges
  - Use katas and quizzes to conciously deeping your understanding
* Mind your body
  - Get fresh air, keep yourself healthy
  - It’s needed not just for programming
  - Know your priorities: without your body there’s no point in pursuing any of these

Again the biggest factor to success isn’t intelligence, it’s grit -- passion and perseverance for long term goals.  Learning and consistently getting better at the face of failures; [making failure your friend][failure-friend].


[grit]: https://www.bakadesuyo.com/2012/11/secret-success-not-giving-up/
[ripgrep]: https://github.com/BurntSushi/ripgrep
[RTFM]: https://en.wikipedia.org/wiki/RTFM
[kata]: https://no-kill-switch.ghost.io/building-dev-muscle-memory-with-code-kata/
[coreutils]: https://www.gnu.org/software/coreutils/coreutils.html
[binutils]: https://www.gnu.org/software/binutils
[findutils]: https://www.gnu.org/software/findutils/
[Emacs]: https://www.gnu.org/software/emacs/
[Vim]: https://www.vim.org/
[go-dep-hyg]: https://talks.golang.org/2012/splash.article#TOC_7.
[aoc]: {{< relref "aoc_2018.md" >}}
[alakajam]: {{< relref "sneak_out/index.md" >}}
[game-jam]: https://letsmakeagame.net/game-jams-calendar/
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
[templators]: https://en.wikipedia.org/wiki/Template_processor
[better-dev]: https://nuclearsquid.com/writings/how-to-become-a-better-developer/#fn:1
[failure-friend]: https://drawabox.com/comic/4
