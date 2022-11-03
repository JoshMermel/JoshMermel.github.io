---
layout: post
title: Mix and Match Gear Cubes
---

<img src="/images/mix-and-match-gear-cubes/solved.jpg" style="max-height: 400px">

Background
==========

# Oskar's gear cube

There's a variation on the standard 3x3x3 Rubik's puzzle called the [Gear
Cube](https://en.wikipedia.org/wiki/Gear_Cube). It has the same centers,
corners, and edges as a regular Rubik's cube, but it turns differently. When you
make a 180 degree turn on a face, the slice layer below it turns by 90
degrees. You cannot make 90 degree turns on faces because then the middle slice
would be turned 45 degrees and would block subsequent moves.

Surprisingly, this puzzle is much easier than a Rubik's cube. The gears
dramatically reduce the number of positions that the puzzle can reach. For
example, all edge on the equator of the cube are permanently stuck on the
equator of the cube. According to [Jaap's puzzle
page](https://www.jaapsch.net/puzzles/gearcube.htm), the gear cube can reach
only 41,472 different states. 

# Variations on Oksar's Gear Cube

Oskar van Deventer designed several variations on the Gear Cube which increase
the challenge. One was mass produced under the name [Gear Cube
Extreme](https://youtu.be/0tm-FNfpIQY). This puzzle replaces four of the geared
edges with conventional Rubik's cube edges, arranged along one of the middle
slices of the cube. When you turn a face, its behavior depends on on the middle
slice next to it. If that middle slice is composed of gear edges then the turn
is like a Gear Cube. If that middle slice is composed of traditional Rubik's
Cube edges than the turn is like a Rubik's Cube. The non-geared edges can never
leave their middle layer so the solver never has to react to which turns are
geared turns vs normal turns.

Another variation that Oskar designed was [Even Less
Gears](https://www.youtube.com/watch?v=79p9vfqsquE). This puzzle replaced 8
geared edges with conventional edges. Like the Gear Cube Extreme, if there are
_any_ gears in the middle layer, turns will engage those gears. Unlike the Gear
Cube Extreme, the gears can move around relative to one another. This means the
solver's strategy needs to account for the positions of the gears when executing
sequences.

Puzzle Modding
==============

I saw a [reddit post](https://www.reddit.com/kkkkk5/) where someone had modded a
different style of Gear Cube so it worked like a Gear Cube Extreme. It seemed
relatively straightforward so I decided to copy them, but make an Even Less
Gears instead.

I used a dremel with a sanding band to remove most of the material, then hand
sanded it to get a smoother finish. My initial attempt didn't remove quite
enough material which resulted in bumpy turning but I did a second round of
sanding and was able to get the turning to be extremely smooth.

## More modding

Playing with my modified puzzle got me wondering if there are other gear cube
variations that can be made from a mix of geared and un-geared edges. I realized
that if I made a copy of the gear cube extreme, I could mix and match edges
between the two puzzles to make all Gear/Rubik's cube hybrids. But what hybrids
exist?

The Hybrids
===========

As is often the case, I'm not the first one with this idea. Andreas Nortmann
[enumatered seven
variations](https://twistypuzzles.com/forum/viewtopic.php?p=239636#p239636). 

"There are no more than 7 different variants. These are:

 - [] The cube without gears.
 - [FR] The cube with a single gear.
 - [FL BR]
 - [FL FR BR]
 - [FL FR BR BL] "Even less Gears"
 - [UL UF UR UB DL DF DR DB] Gear Cube Extreme aka Anisotropic Cube
 - [UL UF UR UB FL FR BR BL DL DF DR DB] Gear Cube (I don't dare to say 'normal')

All other variants are identical in behaviour to one of these."

[Another user](https://twistypuzzles.com/forum/viewtopic.php?t=28325) on the
forums actually built all of these.

Extreme vs Ultimate
===================

Oskar's original Gear Cube Extreme had some unstickered parts. Some people chose
to add stickers to those parts and named the results the "Gear Cube Ultimate".
These stickers represent the orientation of a Rubik's cube edge piece,
independent of the orientation of the gear, and add an additional challenge.

The puzzle I'm modifying does not have these parts but I think I can partially
simulate them. For example, I believe I can replace one geared edge on the Gear
Cube Extreme, which will act like the Ultimate stickers without affecting the
behavior of the puzzle. I need to think more about whether the other hybrids can
be modified in the same way.

<img src="/images/mix-and-match-gear-cubes/scrambled.jpg" style="max-height: 400px">
