+++
title = "Code Lab"
description = "of a curious hacker"
date = "2017-02-24T12:34:07+05:30"
+++

# Open Source

[Live 3D Demos in WebGL 2][demos]

## 3D Workouts

Multiple mid-sized projects to understand 3D math and graphics concepts from ground up using no high-level libraries.  Right from primitives to collision detection to spatial data structures to frustum culling everything was implemented manually from scratch.

**Highlights**:

* Free-look (6 DoF) camera using matrices, quaternions (independently) and no `gluLookAt`.
* [3D picking by ray casting][picking].
* Multi-textured terrain with lighting and quadtree-based frustum culling.
* Math calculations on paper and book citations [documented][board-calculations] for future reference.

**Used**: C++, SIMD, shader-based OpenGL 3+, GLSL ♦ Cross-platform

**Link**: https://bitbucket.org/rmsundaram/tryouts/overview

[board-calculations]: https://bitbucket.org/rmsundaram/tryouts/src/master/CG/Calculations
[demos]: https://legends2k.github.io/projects/demos/
[picking]: https://legends2k.github.io/3d-picking

## Field of Vision System

  An optimized field of view and line of sight system for visibility queries in strategy games; it minimizes intersection tests so that it is usable in a game where multiple AI agents would need their FoV computed every frame.  I worked out the math in designing the broad and narrow-phase culling; made a proof of concept implementation of the system in HTML5 canvas/JavaScript — viewable and debug-able in a browser.  The complete design with illustrations and the geometry involved is documented in excruciating detail; [this document][2d-fov-doc] shows the level of understanding and intuition I have on such topics.

**Link**: [Live Demo](https://legends2k.github.io/2d-fov)

**Used**: Geometry, HTML5/Canvas, JavaScript ♦ Cross-platform

[2d-fov-doc]: https://legends2k.github.io/2d-fov/design.html

## Navmesh Pather

  An optimized navigation mesh based path finding system for strategy games.  It uses funnel narrowing (string-pulling) algorithm for optimized, realistic paths.

**Link**: [Live Demo](https://bbcdn.githack.com/rmsundaram/tryouts/raw/dev/CG/WebGL/NavmeshPather/midpoint/level.html)

**Used**: Geometry, Graph, HTML5/Canvas, JavaScript ♦ Cross-platform

## Geometric Transforms 101

  A [GameDev.net featured](http://www.gamedev.net/page/resources/_/technical/math-and-physics/2d-transforms-101-r4212) interactive tutorial on geometric transformations for programmers favouring intuition over mathematical rigour; concepts explained are illustrated with animation.  In addition to elementary transforms on points, it also treats coordinate system and hierarchical transforms; shows mappings from active and passive viewpoints.  It can be viewed on a browser and in any form factor, as it only uses HTML5 and vector graphics.

**Link**: https://github.com/legends2k/2d-transforms-101/

**Used**: Linear Algebra, Trigonometry, SVG, HTML, CSS, JS ♦ Cross-platform

## Spirit of C++

A presentation for the non-C++ programmer to build a healthy C++ mental model. High-level details are emphasized over low-level ones; it takes the approach of demonstrating high-level and simple ideas leading to a healthy model when learning something new.  The larger C++ community [received it well][spirit-reddit] too.

I've trained multiple teams in Microsoft with this in [my C++ bootstrap workshops][inker].

[View it directly in your browser][spirit-of-cpp].

**Link**: https://github.com/legends2k/spirit-of-cpp/

**Used**: C++, HTML, CSS, JS ♦ Cross-platform

[spirit-of-cpp]: https://legends2k.github.io/spirit-of-cpp
[spirit-reddit]: https://www.reddit.com/r/cpp/comments/da4xrd/spirit_of_c/
[inker]: https://github.com/sundaramramaswamy/inker

## Artha

  A handy offline thesaurus originally written on GNU/Linux using WordNet as its database, with distinct features like global shortcut key look-up, passive desktop notification and wild-card search; [most distributions' repositories have it][artha-pkg].  I later ported to Windows and Windows Phone; it was rated 5 stars in [AppStore](https://www.microsoft.com/en-US/store/Apps/Artha-The-Open-Thesaurus/9NBLGGH0DBNB).

**Link**: https://artha.sourceforge.net/

**Used**: C, C#, GTK+, X11, XAML, GCC, VC++, Autotools ♦ GNU/Linux, Windows, Windows Phone

[artha-pkg]: https://pkgs.org/search/?q=artha

## Puyo Puyo

  A tile-matching puzzle game playable on a terminal in any platform; written in modern C++ with portability and minimalism in mind.  The engine code, handling core logic and input, is an independent module that may be plugged to any front-end.  A command-line interface, using ncurses, was later added to make a cross-platform game out of the engine.

**Link**: https://bitbucket.org/rmsundaram/tryouts/src/master/Puyo/

**Used**: C++ 14, ncurses ♦ Cross-platform

--------------------

# Closed Source

## Microsoft

### WGLES

Conceptualized and implemented the texturing subsystem on a roadmap project implementing the OpenGL ES specification using Direct3D.

**Used**: C++, Compressed Textures (DDS), OpenGL ES 2, Direct3D 11 ♦ Windows 8

### Edge on macOS

I owned the technical development of macOS-specific feature design and build.  My team implemented the [tab scrubber and media controller experience for MacBook Pro Touch Bar which was well-received][touchbar-blog] improved the browser [DAU][] numbers by 3%.

**Used**: Objective-C++, Cocoa, AppKit, Core Animation ♦ macOS

[touchbar-blog]: https://techcommunity.microsoft.com/t5/articles/try-the-revamped-touch-bar-experience-on-microsoft-edge-on-mac/m-p/1061372
[dau]: https://en.wikipedia.org/wiki/Active_users

### PDF Engine

Authored the _Annotations_ subsystem of the PDF rendering engine used by the Edge browser.  Parsing, building document elements, rendering and printing of annotations was my responsibility.  Additionally I took care of PDF print, shading patterns and parts of the rendering subsystem involving transparency (compositing and blending).

**Used**: C++, WinAPI, Direct2D, Chromium ♦ Windows

## Electronic Arts

### 2D Game Engine / C++ EA SDK

Authored a 2D game engine and associated tools for artists to work on sprites and resource packing.  The engine abstracted differences amongst WindowsMobile devices and exposed a common feature set for game teams to work on.  Its basic features like image drawing to advanced ones like animation and blending were implemented from scratch.  It sported effects like on-the-fly grey scaling and parallax scrolling.  Ported J2ME (Java) game code of Jewel Quest III and Sims 2 Castaway to WindowsMobile (C++) using said engine.

**Used**: C++, Java, C#, Boost, DirectDraw ♦ WindowsMobile, J2ME

## Aricent

### DirectShow Filters

Authored source filters for A/V codecs like A-Law, μ-Law, H.264 and for progressive downloading of (YouTube) videos on Windows Media Player.

**Used**: C++, DirectShow ♦ WindowsMobile
