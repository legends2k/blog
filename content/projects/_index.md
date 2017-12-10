+++
date = "2017-02-24T12:34:07+05:30"
title = "Code Lab"
description = "… of a curious hacker"

+++

Open Source
===========

3D Workouts
-----------
Multiple mid-sized projects to understand 3D math and graphics concepts from ground up using no high-level libraries.  Right from primitives to collision detection to spatial data structures to frustum culling everything was implemented manually from scratch.
  
**Highlights**:

* Free-look (6 DoF) camera using matrices, quaternions (independently) and no `gluLookAt`.
* 3D picking by ray casting.
* Multi-textured terrain with lighting and quadtree-based frustum culling.
* Math calculations on paper and book citations [documented](https://bitbucket.org/rmsundaram/tryouts/src/master/CG/Calculations) for future reference.


**Used**: C++, SIMD, shader-based OpenGL 3+, GLSL ♦ Cross-platform  
**Homepage**: https://bitbucket.org/rmsundaram/tryouts/overview  
**Since** 2013

Field of Vision System
----------------------
  An optimized field of view and line of sight system for visibility queries in strategy games; it minimizes intersection tests so that it is usable in a game where multiple AI agents would need their FoV computed every frame.  I worked out the math in designing the broad and narrow-phase culling; made a proof of concept implementation of the system in HTML5 canvas/JavaScript — viewable and debug-able in a browser.  The complete design with illustrations and the geometry involved is documented in excruciating detail; this document shows the level of understanding and intuition I have on such topics.

**Used**: AI, Math, Geometry, HTML5, JavaScript ♦ Cross-platform  
**Link**: https://legends2k.github.io/2d-fov  
**Since** 2016

Geometric Transforms 101
------------------------
  A [GameDev.net featured](http://www.gamedev.net/page/resources/_/technical/math-and-physics/2d-transforms-101-r4212) interactive tutorial on geometric transformations for programmers favouring intuition over mathematical rigour; concepts explained are illustrated with animation.  In addition to elementary transforms on points, it also treats coordinate system and hierarchical transforms; shows mappings from active and passive viewpoints.  It can be viewed on a browser and in any form factor, as it only uses HTML5 and vector graphics.

**Used**: Linear Algebra, Trigonometry, SVG ♦ Cross-platform  
**Link**: https://github.com/legends2k/2d-transforms-101/  
**Since** 2015

Artha
-----
  A handy offline thesaurus originally written on GNU/Linux using WordNet as its database, with distinct features like global shortcut key look-up, passive desktop notification and wild-card search; most distributions' repositories have it.  It was later ported to Windows and Windows Phone; it was rated 5 stars in [AppStore](https://www.microsoft.com/en-US/store/Apps/Artha-The-Open-Thesaurus/9NBLGGH0DBNB).

**Used**: C, C#, GTK+, X11, XAML, GCC, VC++, Autotools ♦ GNU/Linux, Windows, Windows Phone  
**Link**: https://artha.sourceforge.net/  
**Since** 2009

Puyo Puyo
---------
  A tile-matching puzzle game playable on a terminal in any platform; written in modern C++ with portability and minimalism in mind.  The engine code, handling core logic and input, is an independent module that may be plugged to any front-end.  A command-line interface, using ncurses, was later added to make a cross-platform game out of the engine.

**Used**: C++ 14, ncurses ♦ Cross-platform  
**Link**: https://bitbucket.org/rmsundaram/tryouts/src/master/Puyo/  
**Since** 2012

--------------------

# Closed Source

PDF Engine
----------
Authored the annotations component of the PDF rendering engine used by the Edge browser and the store app. Reader.  Additionally I take care of print and parts of the rendering subsystem involving transparency (compositing and blending).

**Used**: C++, Direct2D ♦ Windows 8+

WGLES
-----
Conceptualized and implemented the texturing subsystem on a roadmap project implementing the OpenGL ES specification using Direct3D.

**Used**: C++, Compressed Textures (DDS), OpenGL ES 2, Direct3D 11 ♦ Windows 8


2D Game Engine
--------------
Authored a 2D game engine and associated tools for artists to work on sprites and resource packing.  The engine abstracted differences amongst WindowsMobile devices and exposed a common feature set for game teams to work on.  Its basic features like image drawing to advanced ones like animation and blending were implemented from scratch.  It sported effects like on-the-fly grey scaling and parallax scrolling.  Ported J2ME (Java) game code of Jewel Quest III and Sims 2 Castaway to WindowsMobile (C++) using said engine.

**Used**: C++, Java, C#, Boost, DirectDraw ♦ WindowsMobile, J2ME


DirectShow Filters
------------------
Authored source filters for A/V codecs like A-Law, μ-Law, H.264 and for progressive downloading of (YouTube) videos on Windows Media Player.

**Used**: C++, DirectShow ♦ WindowsMobile

