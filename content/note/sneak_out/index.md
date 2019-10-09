+++
title = "First game jam experience!"
description = "Alakajam Feb 2019"
date = 2019-02-26T20:54:34+05:30
tags = ["tech", "self", "software"]
+++

I had a great first experience participating in a game jam -- [5th Alakajam][alakajam-unranked].  A three-day game jam where we[^1] wrote a primitive, real-time tactical strategy game: [Sneak Out][sneak-out-alakajam].  The game is playable within a browser, thanks to [raylib][] and WebAssembly.

{{< figure src="images/screenshot.png" link="https://legends2k.itch.io/sneak-out" alt="Sneak Out! within the browser" caption="_Sneak Out!_ from within the browser" >}}

I’m a person with a decent degree of [analysis paralysis][].  A code jam was the perfect antidote to kick me out of my rut!  Here[^2] I document the high points, road blocks and solutions.

[alakajam-unranked]: https://alakajam.com/5th-alakajam/results?sortBy=1&division=unranked
[raylib]: https://www.raylib.com/
[sneak-out-alakajam]: https://alakajam.com/5th-alakajam/642/sneak-out/
[analysis paralysis]: https://en.wikipedia.org/wiki/Analysis_paralysis
[release-post]: https://alakajam.com/post/1060/beginning-of-a-dream-coming-true

# Hurdles Crossed

We hit a lot of roadblocks which we swiftly hurdled.  Normally I’d have spent days or weeks to "solve it perfectly" -- a fool’s errand.  A game jam makes you realize that the clock is constantly ticking and you don’t have that kind of time.

## Scope Minimization

This is the most important step in the entire process.  When you approach a new project, remember _[Gall’s Law][]_:

> A complex system that works is invariably found to have evolved from a simple system that worked. A complex system designed from scratch never works and cannot be patched up to make it work. You have to start over with a working simple system.

1. Make a game with just walking and scanning
    + No power-ups or goodies
    + Objective of just escaping undetected instead of collecting something
2. Further cuts
    + Many heros to one
    + One simple, interesting map
    + No HUD, just player controls needing no tutorial

[Gall’s Law]: https://en.wikipedia.org/wiki/John_Gall_(author)#Gall's_law

## Non-technical Tasks

1. Help text
2. About screen
3. Game balance (tweak b/w too hard vs too easy)

These are absolutely needed to call something a game rather than a tech demo.  *When there’s a good-to-have feature vs showing help for an existing feature, choose the latter*.  I find it odd to even state this, but many developers naively overlook this because it isn’t as _fun_ from a programming viewpoint; the [ROI][] however is inversely proportional 😉.

[ROI]: https://en.wikipedia.org/wiki/Return_on_investment

## Technical Issues

1. **Visibility-based Pathing**
    : Porting my A*-based path finding JavaScript code to C++ looked like a daunting task; some how the idea of visibility-based path finding sprung to mind!
2. **Colour-based Picking**
    + Writing geometry-based picking for a game jam is an overkill!
    + Issues with inverted-Y and alpha (buildings) in reading texture
3. **Drawing**
    + raylib 2.0 doesn’t offer Bézier curves
    + raylib expects filled figure points ordered counter-clockwise
4. **Porting JavaScript to C++** ([FoV][] code)
    + Functions returning `undefined` replaced with `std::optional`
    + Multiple return values with [structured bindings (C++17)][structured bindings]
    + Functor of `Array.sort` vs `std::sort` (`< 0` vs `bool`)
5. **Building raylib for WebAssembly**
    + Issues in [Emscripten] build scripts
    + Exceptions in enabling VAO support and [querying game pads][gamepad_api] without sampling game pad data -- [now fixed][raylib-emsdk-fixes]

[FoV]: https://legends2k.github.io/2d-fov/
[structured bindings]: https://skebanga.github.io/structured-bindings/
[Emscripten]: https://emscripten.org/
[raylib-emsdk-fixes]: https://github.com/raysan5/raylib/commit/0c567cd259285fb33b3e2ab514c48322da0a0000#diff-9fb9786d0a4c67039ae8e8af40475bca
[gamepad_api]: https://emscripten.org/docs/api_reference/html5.h.html#c.emscripten_get_num_gamepads

# Developer Art

Managing to get cohesive, working code was load enough.  We didn’t have time to spend on game art: searching, polishing, fitting, etc.

I’d to quickly create a map using [Krita] and a Huion tablet in half hour; thanks to [Pixabay for the background][map-background].

Characters were drawn with code; cobbling a triangle to a circle did the trick. 👍

# Bottom Line

Game jams and code jams are great exercises -- not just technically but (project) managerially too!  Most of my learnings were in this area.  Of course, depending on your existing skills, you may learn more technically.

Participating in a jam is a very refreshing and rewarding experience.  Highly recommended to break the vicious loop of not [finishing started projects][finish-game].


[Krita]: https://www.krita.org/
[map-background]: https://pixabay.com/illustrations/paper-parchment-frame-worn-file-473630/
[finish-game]: https://makegames.tumblr.com/post/1136623767/finishing-a-game


[^1]: A friend/colleague of mine and me.
[^2]: [Here’s a shorter post][release-post] I wrote immediately after the jam.
