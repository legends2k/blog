+++
title = "Computer Graphics Resouces"
description = "Materials for self-study"
date = 2021-03-08T16:08:36+05:30
tags = ["tech", "gamedev", "bookmark", "learning"]
toc = true
+++

Here’s a list of curated learning resources for self-studying _Computer Graphics_ I usually recommend.

Learning _Computer Graphics_ concepts is not the same as learing a rendering library; the latter is usually not very beneficial since a new library eventually replaces/takes over.  Strive to learn the theory behind the domain, not the tools or their idiosyncrasies.  [Starting with OpenGL, instead of Vulkan, is better to learn graphics programming][learn-opengl-vulkan].  [Another article][which-graphics-lib] also admonitions not to start with Vulkan or D3D 12 and instead go with OpenGL or D3D 11.

As you keep learning, once the basic ideas are internalized, try doing [some CG projects with increasing hardness][cg-prog-projs].

I’ve mostly centred the resources around OpenGL and not other libraries since they’re not cross-platform.

# Tutorials / eBooks

* [Learn OpenGL][learngl] is a very clear and extensive OpenGL tutorial series on various topics ranging from basics to advanced levels.  Its categorization and illustrations are very good too.  Highly recommended for beginners.
* [Learning Modern 3D Graphics Programming][arcsynth-community] is probably the most extensive free, well maintained online/offline (download-able for desktop, tablet and even kindle) book on modern OpenGL programming (i.e. it doesn't use fixed-function pipeline at all).
* [OpenGL-Tutorial][gltut] is also modern and good, but isn't that extensive/detailed as compared to the former; although perspective projections are explained with diagrams extremely well here.
* [OGLdev][ogldev] has close to 40 tutorials on modern OpenGL with nice explanations and rendered outputs.
* [Lighthouse 3D][lighthouse] has tutorials for most of CG ~ math, Open GL, GLSL, GLUT, writing your own light weight library for game development.
* [An Intro to Modern OpenGL][modern_gl] is short and sweet; it's good to understand the rendering pipeline more clearly; once again because it's very illustrative. For more in-depth understanding, read [A trip through the Graphics Pipeline][pipeline_trip].
* [OpenGL on WikiBooks][wikibooks] has lot of hands-on tutorials which seem to be good too.

## Math

To be proficient in computer graphics or even to understand the basics, a decent amount of mathematical concepts needs to be grasped; it requires one to be comfortable in using trigonometry and linear algebra. For this I recommend

* [Vector Math Tutorial][math-vec-tut] to visualize, seal the understanding and get comforable using vectors and matrices
* [Song Ho Ahn's Articles][songho] on CG Math and OpenGL
* [Matrix and Quaternion FAQ][mat_quat_faq] should clarify your doubts on vectors, matrices and quaternions.
* [Linear Algebra Free Text][lin_alg] by Professor Jim Hefferon covers linear algebra from ground up with lot of exercises.
* [3D Game Engine Programming][3dgep] has a bunch of articles covering spaces, vectors, matrices and quaternions

### 3D Math Books

I’d recommend at least one book as tutorials just aren’t enough.  To get intuition and comfortably work in abstract 3D these would help:

* [3D Math Primer for Graphics and Game Development][3d_primer] (both editions are good)
* [Essential Mathematics for Games and Interactive Applications: A Programmer's Guide][essential_math], 2nd/3rd Edition
* [Mathematics for 3D Game Programming and Computer Graphics][skeleton_math], 3rd Edition

Out of these math books, the most intuitive is the first with lot of funny anecdotes in between, the last is for hard core math fanatics (if you're afraid of symbol vomit, steer clear of it), although it's a good book for experienced CG programmers who need a reference. The one in between is really good in that it details out somethings which the other two (or many books for that matter) omit, and in the spectrum of intuitiveness and hard core math it's in between.

## Shaders

* [The Book of Shaders][shader_book] is for those who want to write cool-looking pixel effects using fragment shaders.
* [Shader School][shader_school] is an introduction to GLSL shaders and graphics programming using WebGL; looks promising.

## WebGL

WebGL is attractive option since

* Setup costs are lower[^1]
* Concepts learnt map 1:1 to other graphics programming libraries like D3D 11, OpenGL, etc.
* Very little legacy cruft since WebGL 2 is based on OpenGL **ES** 3.0

However, I’ve seen library and tooling availability _w.r.t. CG_ more in C and C++.

* [WebGL Academy][webgl_academy] is the most interactive tutorial out there teaching 3D computer graphics with step by step code walk-through and showing what changed on screen.  It starts with the basics and goes on to cover advanced topics like PCF, distance mapping, variance shadow mapping, deferred shading, etc.
* [WebGL2 Fundamentals][] is a very good introduction to 2D and 3D computer graphics using WebGL by Gregg Tavares.  He also has
  - [WebGL samples][webgl_samples]
  - [WebGPU Fundamentals][]
  - [WebGL Fundamentals][]

## Ray Tracing

Most aforementioned resources are for real-time or online rendering where 30 to 60 frames are pumped on screen every second by a _rasterizer_.  Another branch of CG is offline rendering: every frame takes minutes (or sometimes hours!) to render on a single computer by a _ray tracer_.  For these, per-frame render time isn’t a problem; they value visual quality.  Once rendered, the output can be viewed at leisure, without any user input business.

* [Ray Tracing in One Weekend][raytrace-weekend] is a free book series by Peter Shirley; covers the ideas from ground-up
  1. In One Weekend
  2. The Next Week
  3. The Rest of Your Life
* [ScratchAPixel.com][scratchapixel] is an excellent resource for in-depth Computer Graphics theory partial to raytracing

## Cheat Sheets

* [3d Maths Cheat Sheet][cheat-anton]
* [Math Cheat Sheet][cheat-opengl-tut]
* [2d and 3d game math cheat sheet][cheat-game-math]

# Online Courses

* [ECS 175 Computer Graphics][ecs_175], UC Davis by [Professor Kenneth Joy][ken_joy]. The math explanations/proofs are intuitive and the professor is very interactive and dynamic. He has good change in pace when explaning difficult concepts differently. All his assignments are still available as PDFs, one can download and work them out.
* [CSE 167 Computer Graphics][cse_167], UC San Diego by [Professor Ravi Ramamoorthi][ravi_ram]
* [Computer Graphics, 2012][wolfgang_lecture] by Professor Wolfgang Huerst was also good for learning varied concepts; his assignments too are available online.
* [Interactive 3D Graphics][eric_cg] by Eric Haines teaches the basics of 3D computer graphics: meshes, transforms, cameras, materials, lighting, and animation using [three.js][] and WebGL.

# Physical Books

* [Computer Graphics: Principles and Practice][cg-bible-3], 3rd Edition ([2nd][cg-bible-2] is also highly regarded) - this book is called *The Bible of CG*
* [Computer Graphics, C Version][cg-c-ver], 2nd Edition (not 3rd or 4th which weren't well received)
* [Fundamentals of Computer Graphics][cg-fundamentals], 4th Edition
* [Computer Graphics using OpenGL][cg-using-gl], 2nd or 3rd Edition*
* [Interactive Computer Graphics: A Top-Down Approach with WebGL][cg-interactive], 7th Edition*
* [3D Computer Graphics: A Mathematical Introduction with OpenGL][cg-buss]*

_*: not OpenGL books; they just use OpenGL to teach rudimentary CG concepts_

Of these, my personal favourites are the last two.  More practical and hence engrossing for the beginner; the explanations aren't very cryptic, unlike the other, more academic books in the list.

[Physically Based Rendering][pbrt] seems to be _the_ book for designing and coding a physically-based ray tracer from scratch.

# Low-level

Serious game development eventually leads to low-level optimisations.  This list might help at that point:

* Modern X86 Assembly, _Daniel Kusswurm_
* [Low-level C and C++ curriculum][low-c++], _Alex Darby_
* [Assembly and the Art of Debugging][mo_dbg], _Mohit_
* Write Great Code: Volume 1, _Randall Hyde_
* 3D Game Engine Programming ~ §Fast 3D Calculus, _Stefan Oliver_
* Fundamentals of Software Engineering for Games ~ Chapter 3, _Jason Gregory_
* Mathematics for Game Developers ~ Part V ~ Optimization, _Christopher Tremblay_
* Vector Game Math Processors, _James C. Leiterman_
* Real-Time Collision Detection ~ Chapter 13 Optimization, _Christer Ericson_
* Instruction-level Parallelism, _Wikipedia_
* GPGPU Programming for Games and Science ~ Chapter 3, _David Eberly_


[Reinventing the Wheel Often][reinvent_often] is natural when learning!  I like this quote from _Assembly Language Step by Step_, 3rd Edition by Jeff Duntemann:

> "When somebody asks you, “Why would you want to do _that_?” what it really means is this: “You’ve asked me how to do something that is either impossible using tools that I favor or completely outside my experience, but I don’t want to lose face by admitting it.” [...] The answer to the _Infamous Question_ is always the same, and if the weasels ever ask it of you, snap back as quickly as possible, “Because I want to know how it works.” That is a completely sufficient answer."


[learn-opengl-vulkan]: https://computergraphics.stackexchange.com/a/3589/1650
[which-graphics-lib]: http://rastertek.com/choosing.html
[cg-prog-projs]: http://graphicscodex.com/projects/projects/index.html
[arcsynthesis]: https://alfonse.bitbucket.io/oldtut/
[arcsynth-community]: https://paroj.github.io/gltut/
[gltut]: http://www.opengl-tutorial.org/
[ogldev]: http://ogldev.atspace.co.uk/
[open.gl]: http://open.gl/
[learngl]: http://www.learnopengl.com/
[lighthouse]: http://www.lighthouse3d.com/tutorials/
[modern_gl]: http://duriansoftware.com/joe/An-intro-to-modern-OpenGL.-Table-of-Contents.html
[pipeline_trip]: http://fgiesen.wordpress.com/2011/07/09/a-trip-through-the-graphics-pipeline-2011-index/
[wikibooks]: http://en.wikibooks.org/wiki/OpenGL_Programming#Modern_OpenGL
[songho]: http://www.songho.ca/
[mat_quat_faq]: http://www.j3d.org/matrix_faq/matrfaq_latest.html
[lin_alg]: http://joshua.smcvt.edu/linearalgebra/
[3dgep]: https://www.3dgep.com/category/math/
[math-vec-tut]: https://chortle.ccsu.edu/VectorLessons/index.html
[WebGL Fundamentals]: http://webglfundamentals.org/
[WebGL2 Fundamentals]: http://webgl2fundamentals.org/
[webgl_samples]: http://github.com/WebGLSamples
[WebGPU Fundamentals]: http://webgpufundamentals.org/
[webgl_academy]: http://www.webglacademy.com
[shader_book]: http://thebookofshaders.com/
[shader_school]: http://github.com/stackgl/shader-school
[raytrace-weekend]: https://raytracing.github.io/
[scratchapixel]: https://www.scratchapixel.com
[cheat-anton]: https://antongerdelan.net/teaching/3dprog1/maths_cheat_sheet.pdf
[cheat-opengl-tut]: http://www.opengl-tutorial.org/miscellaneous/math-cheatsheet/
[cheat-game-math]: https://gist.github.com/xem/99930986c5333125a13b0ea50600391f
[cse_167]: http://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x
[ravi_ram]: http://cseweb.ucsd.edu/~ravir/
[ken_joy]: http://graphics.cs.ucdavis.edu/~joy/ecs175/
[ecs_175]: https://www.youtube.com/playlist?list=PL_w_qWAQZtAZhtzPI5pkAtcUVgmzdAP8g
[wolfgang_lecture]: http://www.youtube.com/playlist?feature=g-user-u&list=PLDFA8FCF0017504DE
[eric_cg]: http://www.udacity.com/course/cs291
[three.js]: http://threejs.org/
[cg-bible-2]: http://a.co/51D3cMq
[cg-bible-3]: http://a.co/21r6vLM
[cg-c-ver]: http://amzn.com/0135309247
[cg-fundamentals]: http://a.co/05TIOFX
[cg-using-gl]: http://amzn.com/0131496700
[cg-interactive]: http://a.co/ePPekt6
[cg-buss]: http://amzn.com/0521821037
[super_bible]: http://amzn.com/0321712617
[pbrt]: http://amzn.com/0123750792
[3d_primer]: http://amzn.com/1568817231
[essential_math]: http://amzn.com/0123742978 
[skeleton_math]: http://amzn.com/1435458869
[low-c++]: https://1drv.ms/u/s!AqkzqQjJ-7EtgS9-oiB28cv_iqIj?e=c2dnKZ
[mo_dbg]: http://mohit.io/blog/category/debugging/
[gamedev_post]: https://gamedev.stackexchange.com/a/46358
[reinvent_often]: https://97-things-every-x-should-know.gitbooks.io/97-things-every-programmer-should-know/content/en/thing_72/


<!-- Markdeep: --><style class="fallback">body{visibility:hidden;white-space:pre;font-family:monospace}</style><script src="markdeep.min.js"></script><script src="https://casual-effects.com/markdeep/latest/markdeep.min.js"></script><script>window.alreadyProcessedMarkdeep||(document.body.style.visibility="visible")</script><script>window.markdeepOptions = { tocStyle: 'medium' };</script>

[^1]: You just need a text editor and a browser (which also has a JS debugger)
