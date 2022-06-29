+++
title = "Programmer’s Foray into Making 3D Art"
description = "Thoughts on modelling, texturing and animation in Blender"
date = 2021-02-24T11:25:33+05:30
tags = ["tech", "art", "gamedev", "introspections", "blender"]
draft = true
+++

I’m a graphics programmer by passion.  Not by profession; [most software jobs today are plumbing][sw-plumbing] -- I’m no exception.  I’ve been self-studying graphics and game programming since 2012.  I learned the math, technologies (Direct2D, OpenGL, WebGL, etc.) and non-technical aspects of game making[^extra-credits].

Once I got a hold on all these, I started seeing the elephant in the room; a roadblock.  A _video_ game needs art as much as code; perhaps more.  There’re no-art hits like [Pong][], [Breakout][], [Tetris][], [thomas was alone][], … but they’re mostly 2D.  I haven’t seen 3D games in this category though.  It’s a shame to kill your passion owing to this one issue; admittedly big though.

_I’ll try and convince you, with pictures and some words, that 3D art actually is doable by a programmer with some patience and perseverance_.

**Disclaimer**: If you want to make game art with AAA production values you mostly wouldn’t be needing/reading this; you’ll already have teams of artists at your disposal.  This post isn’t for you.  Even if you’re a small team and buying art is affordable, do it and be happy.  I’ll just be focusing on a programmer-only team’s art needs.

# Should I learn to draw?

I’m not an artist.  I can’t draw; my drawing never graduated from kindergarten.  However, I realize having a methodical mind.  Being a programmer is telling of my _sense of design_.  I can appreciate symmetry, [colour harmony][], [golden ratio][] and so on.  I might not create great art, but I can also call out bad art.  This gave me some confidence that I can create some kind of art.

> Drawing isn’t _essential_ for 3D art.

A huge weight was lifted off my chest when I came to know that drawing isn’t _essential_ for 3D art; it definitely helps but isn’t mandatory.  There are two opposing camps here.  I stumbled upon [Blender Guru][]’s opinion: [Should 3D artists learn to draw?][3d-needs-draw]  Lucky for me, I like 3D more than 2D.  Had I wanted to make a 2D game, I might’ve lamented about being a non-artist and I’m not sure I’d have proceeded further.  Of course, buying art is an option; but not for everyone.

Even if you want to make 2D games with little artistic skills, it’s possible with some work.  I highly recommend going through [How to bootstrap your indie art needs][lost-garden-advice].  You can get some free asset pack like [kenny.nl][]’s, use your minimal [GIMP][] skills and get going.

The other, more harder but rewarding, route: become a good artist.  This will need a lot of time.  If you’ve the patience and tenacity, you’ve lot of resources.  However, for a programmer I highly recommend [DrawABox.com][]: this program is geared at non-naturals to become good artists by methodical exercises; it’s free too.  There’re Reddit communities around it for getting feedback.

# Why Blender?

I’ve had an eye on Blender since 2008.  I still remember seeing [Blender’s Gallery page][blender-gallery] periodically; getting pumped-up, opening Blender only to close it in a few minutes without knowing what to do.  As a programmer, I’ve been exposed to daunting UI -- most IDEs have umpteen buttons and menus.  Blender’s UI[^blender-ui] didn’t threaten me; unknown unknowns did.  I didn’t even know the term _modelling_ back then.  Today I know that [Pareto principle][] applies here too:  _20% of the UI is enough for 80% of your job_.

Why Blender?  More than its enticing price tag, I think what it enabled and the thriving community around it matters.

> Blender opened 3D art to anyone.  An ~~elite~~[^elites] enthusiast could jump in, find direction and enjoy doing their thing.

More importantly, I can’t help but respect this open-source software rivalling paid, proprietary software by a million-dollar company.  [Open movie projects][] like [_Agent 327: Operation Barbershop_][agent-327], [_Spring_][spring], etc. clearly demonstrates its technical prowess.  [Ton Roosendaal][] -- original dev., founder and chairman -- seems to be a great guy.

Features I value

1. **Pure tool**: no membership dialogs, cloudware, nagging or tracking (gets out of your way)
2. **Consistent hotkeys**: different parts understand same combos for similar operations
3. **Open-source and Cross-platform**: first-class Linux support
    1. Improve by code contribution, [bug][my-bug] submission, add-on making, etc.
4. **Thriving community**: Paid software doesn’t have information as readily available
    1. Matters when you want to learn or improve yourself constantly
5. **Strong but Light**: smooth on laptops, < 200 MiB, multi-GPU support

Being an Emacs fanatic, I felt right at home with Blender’s hotkey system; very consistent and cohesive.  Additionally, being a _StackOverflow.com_ contributor, I was happy to know of the active [blender.stackexchange.com][] in case I’d doubts :)

**Note**: modelling/texturing/animation skills you learn are translatable between packages.  Terms might differ but functionality remains with some variations.  [Which 3D Software Should You Learn][gabbit-why-blender] gives a detailed exposition.

Look, if you want to just make stuff in 3D use Blender; if you’re worried about meta-3D stuff --- industry adoption, mindshare, etc. ---  choose proprietary software.

# Commitment

YouTube is burgeoning with `#blender`.  You’ll find a video to get past most of your roadblocks.  [Some playlists][gabbit] really helped me get a sense of the lay of the land but something was still lacking.  I didn’t have structure.  I joined [CGCookie; an online Blender tutorial/community][cgcookie].  It had different streams:

* **Modelling**: creating a mathematical representation (_model_) of an object/character
* **Texturing**: wrapping the model with images
* **Shading**: creating and applying materials to models
* **Animation**: animating the model by [keyframing][] or [with a skeleton][skeletal-animation]

each having courses of increasing difficulty.  Streams beyond these aren’t strictly needed for game assets.  They’re geared towards non-interactive (movie) production.  Most of their videos were geared at 3D artists, not programmers, understandably.  Though this might be off-putting, it shouldn’t.  When you learn to make highly-detailed models, it helps.  You mean a decimated low-poly model with the high-poly model’s detail as normal maps anyways for game, so the effort isn’t wasted.

> I just skipped overthinking and resolved to blindly complete just one course; no smarts.

[The Fundamentals of Modelling][modelling-fundamentals] was it!  I resolved to watch one episode/day including exercises.

# First Steps

My first exercise; not interesting but very educating:

> Complex shapes are made out of simple, primitive shapes

{{< figure src="images/primitive_house.jpg" alt="House built with just primitives and transforms" caption="House built with just primitives and transforms" >}}

# ~~Drawing~~ Reference Images

If you’re creating an original character you need to sketch from your imagination.  However, for objects modelled after real world counterparts you generally use reference images.  This isn’t just for programmers.  When you’ve a reference, creating a shape from it isn’t hard.  Here’s an example:

{{< figure src="images/tracing.jpg" alt="Tracing out a base mesh of a traditional kitchen blade from a reference" caption="Tracing out a base mesh of a traditional kitchen blade from a reference" >}}

Basically I created a bunch of points (vertices), connected them (edges) and told Blender which ones form faces.  Essentially, it’s adding, removing and transforming a bunch of vertices, edges and faces.  I’ve compensated for the shear in the reference.  Now giving thickness, tapering, etc. are done with tools like _Extrude_, _Bevel_, _Inset_, etc.  I could use them intuitively after a few days of playing.

It gets easier when you’ve a real world object you want to recreate.  Lengths, transforms (move, scale, rotate), etc. can be specified in exact amounts in your preferred units e.g. `cm`, `ft`, `km`, etc.

# Helpful Tools

You have a lot of tools at your disposal to make your life easy.  I frequently used:

## Mirror

There’s a _Mirror Modifier_ which avoids the tedium of making the same kind of change for symmetric models:

{{< figure src="images/mirror_1.jpg" alt="Mirrored across Y axis (green)" caption="Mirrored across Y axis (green)" >}}

Any change I make on one side is reflected on the other.  You can add as many as you want, in different axes with different settings:

{{< figure src="images/mirror_2.jpg" alt="I only modelled a quarter of a handle to get both" caption="I only modelled a quarter of a handle to get both" >}}

Notice that it’s just a quarter of a [torus][] (primitive) with some tapering done in the end.

## Screw

The screw modifier which is super useful!  I can’t tell how much; I better show :)

{{< figure src="images/screw_1.jpg" alt="An object’s outline --> Object" caption="An object’s outline --> Object" >}}

Blender created the bucket on the left, by rotating the basic outline I created on the right by rotating it 360°.  I set the modifier’s rotation to 260 in the image to show you how it works.

You can also specify an offset and [create a screw or spring-like object][mod-screw].

## Spin

There’d be times where you want vertices to be laid out in a perfect sphere; _spin_ to the rescue!

{{< figure src="images/spin_1.jpg" alt="Spin a vertex and create more" caption="Spin a vertex and create more" >}}

## Array

Create many instances of the same entity.  The generated duplicates can be in a row, around a circle or even a path.





- Modelling is quite logical
- Tackling textures and animation
- Low-poly art is for you, the art-impaired artist

In the end, your programmer art might be good enough to get you quite far; it may not be super-beautiful, but that’s not the goal.  At worst, you might end up buying some art still saving you quite a bit of dough e.g. hero character, textures, animation, etc.  Remember:

> Fun, not art, makes or breaks a game.

If your game is fun and has decent art, what stops you from making it?  [Definitely not art](https://gamedev.stackexchange.com/a/12350/14025).


[sw-plumbing]: https://www.karllhughes.com/posts/plumbing
[Pong]: https://en.wikipedia.org/wiki/Pong
[Breakout]: https://en.wikipedia.org/wiki/Breakout_(video_game)
[Tetris]: https://en.wikipedia.org/wiki/Tetris
[thomas was alone]: https://en.wikipedia.org/wiki/Thomas_Was_Alone
[3d-needs-draw]: https://www.youtube.com/watch?v=XQnaebFYv0w
[colour harmony]: https://en.wikipedia.org/wiki/Harmony_(color)
[lost-garden-advice]: http://www.lostgarden.com/2007/12/how-to-bootstrap-your-indie-art-needs.html
[kenny.nl]: https://www.kenney.nl/
[GIMP]: https://www.gimp.org/
[DrawABox.com]: https://www.drawabox.com
[blender.stackexchange.com]: https://blender.stackexchange.com
[blender-gallery]: http://web.archive.org/web/20081217060142/http://www.blender.org/features-gallery/gallery/art-gallery/
[gabbit]: https://www.youtube.com/playlist?list=PLn3ukorJv4vs_eSJUQPxBRaDS8PrVmIri
[gabbit-why-blender]: https://www.youtube.com/watch?v=JG_JCoX9xTQ
[Pareto principle]: https://en.wikipedia.org/wiki/Pareto_principle
[cgcookie]: https://www.cgcookie.com
[keyframing]: https://en.wikipedia.org/wiki/keyframing
[skeletal-animation]: https://en.wikipedia.org/wiki/skeletal_animation
[modelling-fundamentals]: https://cgcookie.com/course/fundamentals-of-3d-mesh-modeling-in-blender
[torus]: https://en.wikipedia.org/wiki/Torus
[first-game-pls]: https://www.youtube.com/playlist?list=PLhyKYa0YJ_5C6QC36h5eApOyXtx98ehGi
[Open movie projects]: https://en.wikipedia.org/wiki/Blender_(software)#Open_projects
[agent-327]: https://www.youtube.com/watch?v=mN0zPOpADL4
[golden ratio]: https://en.wikipedia.org/wiki/Golden_ratio
[Blender Guru]: https://www.youtube.com/user/AndrewPPrice
[Ton Roosendaal]: https://en.wikipedia.org/wiki/Ton_Roosendaal
[spring]: https://www.youtube.com/watch?v=WhWc3b3KhnY
[my-bug]: https://developer.blender.org/T78540
[mod-screw]: https://docs.blender.org/manual/en/latest/modeling/modifiers/generate/screw.html

[^extra-credits]: [Extra Credits: Making Your First Game][first-game-pls] playlist initiates you into game dev., covering vital touches like scoping, creating a minimum viable product, failing faster, etc.  Very useful.

[^blender-ui]: I’m referring to the old pre-2.8 Blender UI; not the nicer revamped UI 2.8+ got.

[^elites]: Those who can afford to buy expensive 3D software or works at a company or studies in a college which have them
