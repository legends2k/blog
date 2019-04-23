+++
title = "3-day game jam experience"
description = "Alakajam Feb 2019"
date = 2019-02-26T20:54:34+05:30
tags = []
draft = true
+++

I had a great experience participating in Alakajam Feb 2019: a three-day code jam.  Here I document it along with the road blocks we hit and how we crossed them.

# Crossed Hurdles

We hit a lot of roadblocks which we short-circuited swiftly

1. Issues in porting FoV code from JS to C++
  + Non-availability of bézier curve drawing in raylib
  + Filled draw needing counter-clockwise ordered points
  + `Array.sort` vs `std::sort`
  + `undefined` replaced with `std::optional`
  + multiple-return-values with [structured bindings from C++17][structured bindings]
2. Visibility-based Path Finding
3. Colour-based Picking
  + Fix issue with inverted Y in reading texture
  + Fix issue with alpha values for buildings
4. Make a game with just walking and scanning; no power-ups or goodies
  + No HUD, just player control
5. Primitive geometry shapes-based level to hand-drawn map/level
6. Build Emscripten version
  + Fix 3 issues in Raylib `emsdk` build
  + Fix issues in heist code
7. Minimal text framework to show help text
8. About screen
9. Balance game to be not too hard / too easy
10. Slimming down on scope
  + Many heros to one
  + No HUD
  + No goodies or power-ups
  + Just basic interesting map
  + Objective of collecting something to just escaping undetected

[structured bindings]: https://skebanga.github.io/structured-bindings/
