---
layout: post
title: Bandaged Cube Diameter
---

# Background

Readers of my blog may recall a tool I released in 2020 called [bandaged cube
explorer](https://joshmermelstein.com/bandaged-cube-explorer-post/). 

I recently got an email from someone who had used it to look at
[0x2000402849B5E](https://joshmermelstein.com/bandaged-cube-explorer?id=0x2000402849B5E).
They pointed out that this graph has some vertices that are extremely far away
from the starting configuration, and said "I wonder if there are bandaged
puzzles with configurations that take even more moves to reach."

I found this question fascinating and spent some time generating and analyzing
data. I'd like to share some of my findings with you here.

## Why this is interesting

In my 2020 blog post, I wrote "When I first tried to get into bandaged puzzles,
I found a few docs online containing collections of named configurations [...]
What they both lacked was guidance on what kind of experience to expect when
solving each puzzle". To this day, I still don't have a satisfying way to
predict solving experience from bandaged signature.

Looking at the max distance between vertices has the potential to help with
this. I suspect that puzzle where this metric is large contain long, highly
constrained paths through the shape space, and will be a different solving
experience from puzzles where this metric is small. To be clear, this is not
about the difficulty of the solve, it's about the experience of the solve, and
the kinds of tools a solver will need to develop.

Also, I think no one has computed these numbers before, and that appeals to me
as well.

## Definitions

There are two common ways to measure how far apart two configurations are. In
**QTM** (quarter turn metric), every quarter turn counts as one move. In **HTM**
(half turn metric), a double turn of a single face only counts as 1 move. There
are a few other interesting distance metrics but we will ignore those for this
blog post.

In graph theory, we have a few terms to explain how far apart things are. A
vertex's **eccentricity** is the maximum distance from it to any other vertex.
The vertex with the smallest eccentricity is called the **center** and its
eccentricity is called the **radius**. Note that a graph may have more than one
center. The largest eccentricity in a graph is called the **diameter**.

## Techniques

Bandaged cube graphs are pretty tame by graph theory standards. All edges are
bidirectional and have equal weight. This means we can do a breadth first search
from each vertex in a graph and easily filter for the minimum and maximum
eccentricities. As a bonus, we'll get a representative example for each of
those.

One useful optimization I found was to observe that bandaged configurations that
only differ by orientation/mirroring must have the same eccentricity. Checking
whether two cubes are the same modulo orientation is computationally cheap
compared to breadth first search on a larger graph so this improved runtime
dramatically.

I applied this procedure to all 3559 bandaged configurations listed
[here](https://github.com/paulnasca/paul_cube_bandaged_graph/blob/master/all_cubes.csv)
and containing >1 vertex. Both the source code and the raw data are available on
[my
github](https://github.com/JoshMermel/bandaged-cube-explorer/blob/main/diameter_analysis/).

With all that out of the way, I’d like to share a few bandaged 3x3x3
configurations that are special in some way.

# Cubes with very large diameter and radius

## Largest

0x1003FE80000423 has by far the largest diameter, in both QTM and HTM.

<img src="/images/bandaged-cube-diameter/0x1003fe80000423_overview.svg" style="max-height: 400px">

Note that the 'x' over the white center means that this face is never allowed to
turn. In QTM, 0x2003FD00041830 and 0x1003F680104444 are a whopping 167 moves
apart (note that these two are mirror images of one another). This cube also has
the largest diameter in HTM, 127.

<img src="/images/bandaged-cube-diameter/0x1003fe80000423_worst.svg" style="max-height: 400px">

I was quite surprised to see how small the graph for this configuration is. It
has just 3780 vertices. Some configurations have over 20x as many vertices and
yet have dramatically smaller diameters! (note: it's still large enough to lag
bandaged-cube-explorer. In this blog post, I'll only deeplink to graphs that are
small enough to render nicely on mobile).

At first, I found the bandaged white center on this cube a little unsatisfying,
because my usual tools for playing with bandaged 3x3x3 puzzles (a cubetwist
bandaged kit or the [Magic Cube
app](https://play.google.com/store/apps/details?id=org.distorted.magic)) both
can't handle that. Then I checked the diameter of this configuration with the
white center unlocked and found that it does not change!

## Second largest

The configuration with the second largest diameter in both QTM and HTM is
0x83fe04000c21.

<img src="/images/bandaged-cube-diameter/0x83fe04000c21_overview.svg" style="max-height: 400px">

This configuration has a QTM diameter of 120 (example 0x83BE04021182 to
0x188c30c4204441) and an HTM diameter of 96.

<img src="/images/bandaged-cube-diameter/0x83fe04000c21_worst.svg" style="max-height: 400px">

This configuration also has a surprisingly small graph. It has a mere 2704
vertices.

## Top 10 (QTM)

After this, QTM and HTM disagree about which configurations are largest. Here
are the remainder of the top 10 in QTM:

{:start="3"}
3. 0x10000080800C63 (QTM: 117, HTM: 73)
4. 0x21109308C00C46 (QTM: 115, HTM: 90)
5. 0x108430C40007A3 (QTM: 113, HTM: 92)
6. 0x1003DA80000423 (QTM: 110, HTM: 90)
7. 0x1084308620063D (QTM: 107, HTM: 79)
8. 0x10039E80000403 (QTM: 101, HTM: 83)
9. 0x1084308420063D (QTM: 101, HTM: 75)
10. 0x21000102000C43 (QTM: 100, HTM: 59)

<img src="/images/bandaged-cube-diameter/top_10_diameter_qtm.svg" style="max-height: 1600px">

## Top 10 (HTM)

And here is the rest of the top 10 in HTM:

{:start="3"}
3. 0x108430C40007A3 (#5 above)
4. 0x21109308C00C46 (#4 above)
5. 0x1003DA80000423 (#6 above)
6. 0x10039E80000403 (#8 above)
7. 0x1084308620063D (#7 above)
8. 0x72920080007F (QTM: 95, HTM: 77)
9. 0x1084308420063D (#9 above)
10. 0x139200800C1C (QTM: 94, HTM: 75)

<img src="/images/bandaged-cube-diameter/top_10_diameter_htm.svg" style="max-height: 400px">

For more information on these configurations, such as examples of pairs of nodes
that are the maximum distance apart, see the [raw
data](https://github.com/JoshMermel/bandaged-cube-explorer/blob/main/diameter_analysis/analysis.csv).

# Comparing QTM diameter to HTM diameter

I was surprised to see configurations like 0x21000102000C43 where the QTM
diameter was so much larger than the HTM diameter (100 vs 59). Here’s a
scatterplot of the raw data on these two measures:

<img src="/images/bandaged-cube-diameter/diameter_htm_vs_qtm.png" style="max-height: 600px">

## High QTM/HTM

QTM diameter can never be larger than 2x the HTM diameter since each half turn
can be replaced by two quarter turns. Over 50 configurations achieve the maximum
of HTMx2 = QTM. The two that stand out to me are
[0x8000802E3401](https://joshmermelstein.com/bandaged-cube-explorer?id=0x8000802E3401)
and
[0x8000842FB421](https://joshmermelstein.com/bandaged-cube-explorer?id=0x8000842FB421).
Both have 110 vertices and a QTM diameter of 12. I suspect they're actually the
same puzzle in disguise as two different configurations.

<img src="/images/bandaged-cube-diameter/max_qtm_over_htm.svg" style="max-height: 400px">

All the others have either a much smaller vertex count and/or a smaller diameter
(e.g.
[0x18FFC0C03](https://joshmermelstein.com/bandaged-cube-explorer?id=0x18FFC0C03),
[0x100586802C3421](https://joshmermelstein.com/bandaged-cube-explorer?id=0x100586802C3421),
[0x10818085ADB42D](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10818085ADB42D)).

<img src="/images/bandaged-cube-diameter/max_qtm_over_htm2.svg" style="max-height: 800px">

Another group that jumped out at me are {
[0x320084200C63](https://joshmermelstein.com/bandaged-cube-explorer?id=0x320084200C63),
[0x6020084200C63](https://joshmermelstein.com/bandaged-cube-explorer?id=0x6020084200C63),
[0x6009118400C63](https://joshmermelstein.com/bandaged-cube-explorer?id=0x6009118400C63),
[0x10858086200C63](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10858086200C63)
} have QTM diameter=45 and HTM diameter=25. They also all have 96 vertices and
132 edges.

<img src="/images/bandaged-cube-diameter/high_qtm_over_htm.svg" style="max-height: 800px">

## Low QTM/HTM

QTM diameter can never be lower than HTM diameter. Almost 70 configurations have
an equal QTM diameter and HTM diameter. This means that half turns are not
useful when traveling between antipodes.

The largest of these by diameter (17) is a tie between
[0x30138380002D](https://joshmermelstein.com/bandaged-cube-explorer?id=0x30138380002D)
and
[0x80018620002D](https://joshmermelstein.com/bandaged-cube-explorer?id=0x80018620002D).
The largest of these by vertex count are
[0x180200021](https://joshmermelstein.com/bandaged-cube-explorer?id=0x180200021)
(402, and a diameter of 15) and then
[0x10000000000002](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10000000000002),
(240, and a diameter of 5).

<img src="/images/bandaged-cube-diameter/min_qtm_over_htm.svg" style="max-height: 800px">

A few other configurations that stood out to me are:
0x10000084018421 (2540 vertices, QTM diameter 18, HTM diameter 17)
0x108401843DCC21 (1130 vertices, QTM diameter 64, HTM diameter 56)

<img src="/images/bandaged-cube-diameter/low_qtm_over_htm.svg" style="max-height: 400px">

# Is vertex count a good predictor of diameter?

Nope!

There are many examples of graphs with relatively low vertex count but large
diameter, e.g. 0x21000100008C43 (726 vertices, QTM diameter of 95) and
0x100C00808005AD (482 vertices, 73 QTM diameter).

<img src="/images/bandaged-cube-diameter/low_vertex_high_diameter.svg" style="max-height: 400px">

There are also plenty of examples of graphs with a high vertex count but
relatively low diameter, such as 0x8000000804 (a.k.a “3 stripes”) (1296`
vertices, QTM diameter of 8), and 0x20008000000806 (7344 vertices, QTM diameter
of 14).

<img src="/images/bandaged-cube-diameter/high_vertex_low_diameter.svg" style="max-height: 400px">

# Looking at diameter / vertex count

I was hoping that this ratio might help me find some interesting puzzles but the
results were disappointing. Configurations where this metric was high, such as
[0x30080200000603](https://joshmermelstein.com/bandaged-cube-explorer?id=0x30080200000603)
and
[0x824810800C23](https://joshmermelstein.com/bandaged-cube-explorer?id=0x824810800C23)
tended to have extremely constrainted graphs, (click through to see them). Their
graphs have almost no branching and I suspect these puzzles can barely scramble.

<img src="/images/bandaged-cube-diameter/diameter_over_vertex.svg" style="max-height: 400px">

Puzzles were this metric was small were ones with the largest vertex counts,
and didn't have particularly interesting diameters.

# Other interesting puzzles

While I was searching for patterns in the data, I came across a few puzzles with
fun looking graphs. If you're reading this blog post in search of interesting
configurations to solve, here are the ones that stood out to me:

[0x210B0118C00C5A](https://joshmermelstein.com/bandaged-cube-explorer?id=0x210B0118C00C5A)
: Two identical looking subgraphs connected by a forced chain. I wonder if the
solve will require navigating the chain frequently - or even at all.

[0x1084018C7C0C30](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1084018C7C0C30)
: Similar to the above but with a much longer chain connecting the segments.

[0x1080018FFC0C21](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1080018FFC0C21)
: I'm not sure I've ever seen a graph like this. It has a few big loops that are
all connected to each other, plus so not-too-useful looking long dead ends.

[0x104000B5BDB5AD](https://joshmermelstein.com/bandaged-cube-explorer?id=0x104000B5BDB5AD)
: A very small graph with only a few nontrivial loops. It seems like it can
scramble pretty well though and would make for a challenging and highly
constrained solve.

<img src="/images/bandaged-cube-diameter/bonus.svg" style="max-height: 800px">

# Closing thoughts

The bandaged cube with the largest known god's number is the [Maze-300
cube](https://twistypuzzles.com/forum/viewtopic.php?t=35731) by André Kutepow
(Isaev). In QTM, this puzzle has a (stickered) god's number of 301, but it's
(shape space) diameter is a mere 47. This makes me hopeful that some of the
high-diameter puzzles that I mention in this post could be used to find a
configuration with an even larger god's number.

If you find such a configuration, or you find another intersting feature in the
data, please send me an email and let me know!
