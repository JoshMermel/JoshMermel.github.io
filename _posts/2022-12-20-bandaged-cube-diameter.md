---
layout: post
title: Bandaged Cube Diameter
---

# Background

Readers of my blog may recall a tool I released in 2020 called [bandaged cube
explorer](https://joshmermelstein.com/bandaged-cube-explorer-post/). It used
javascript to visualize the state space of bandaged 3x3x3 puzzles. 

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
(half turn metric), a double turn of a single face only counts as one move.
There are a few other interesting distance metrics but we will ignore those for
this blog post.

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
compared to breadth first search on most graphs so this improved runtime
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

The vertex counts of all these graphs are lower than I expected. The largest one
is 0x1003DA80000423 with 7586 but most are well below 5000. To get a better
sense of the data, I made this scatter plot of diameter (QTM) vs vertex count.

<img src="/images/bandaged-cube-diameter/diameter_vs_vertices.svg" style="max-height: 600px">

For more information on these configurations, such as examples of pairs of
vertices that are the maximum distance apart, see the [raw
data](https://github.com/JoshMermel/bandaged-cube-explorer/blob/main/diameter_analysis/analysis.csv).

# Comparing QTM diameter to HTM diameter

Another thing I found surprising, was the different between QTM diameter and HTM
diameter in configurations like 0x21000102000C43 (100 vs 59). Here’s a
scatterplot of the raw data on these two measures:

<img src="/images/bandaged-cube-diameter/diameter_htm_vs_qtm.svg" style="max-height: 600px">

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

# Diameter/Radius

In a normal circle, the diameter is definitionally 2x the radius. In undirected
graphs like ours, the diameter is capped at 2x the radius but can be smaller
than that. Here's a scatterplot of the overall relationship between the two:

<img src="/images/bandaged-cube-diameter/radius_vs_diameter.svg" style="max-height: 400px">

## Equal diameter and radius

There are ~70 puzzles where diameter and radius equal in QTM. By radius, the
three largest ones are
[0x30000010048883](https://joshmermelstein.com/bandaged-cube-explorer?id=0x30000010048883),
[0x20108000000C03](https://joshmermelstein.com/bandaged-cube-explorer?id=0x20108000000C03),
and
[0x30000000040203](https://joshmermelstein.com/bandaged-cube-explorer?id=0x30000000040203).
All three have a QTM radius and diameter of 15. Amazingly, all three of these
are familiar to me. The first one is the subject of a [previous blog post of
mine](https://joshmermelstein.com/solving-cofortress/). The other two are both
mentioned at the end of that blog post as potentially related configurations! 

<img src="/images/bandaged-cube-diameter/equal_radius_diameter.svg" style="max-height: 800px">

## Nearly equal diameter and radius

If we expand our search to puzzles where the diameter is only slightly larger
than the radius then we can find some other puzzles whose graph forms one big
loop.

One that jumped out at me was
[0x80408B40DC7A1](https://joshmermelstein.com/bandaged-cube-explorer?id=0x80408B40DC7A1)
(diameter 47 QTM, radius 38 QTM). This puzzle turns out to be the [Maze-300
cube](https://twistypuzzles.com/forum/viewtopic.php?t=35731) by André Kutepow
(Isaev). It is the bandaged 3x3x3 with the largest known god's number (301 QTM).

A few others that seem potentially interesting are
[0x1108100440843](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1108100440843)
(diameter 48 QTM, radius 39 QTM) which has a beautiful spread out graph with
lots of loops. Also 0x10843086218C21 (diameter 53 QTM, radius 42 QTM) and
0x139E0090002D (diameter 62 QTM, radius 46 QTM).

<img src="/images/bandaged-cube-diameter/nearly_equal_radius_diameter.svg" style="max-height: 800px">

# Is vertex count a good predictor of diameter?

Nope!

There are many examples of graphs with relatively low vertex count but large
diameter, e.g. 0x21000100008C43 (726 vertices, QTM diameter of 95) and
0x100C00808005AD (482 vertices, 73 QTM diameter).

<img src="/images/bandaged-cube-diameter/low_vertex_high_diameter.svg" style="max-height: 400px">

There are also plenty of examples of graphs with a high vertex count but
relatively low diameter, such as 0x8000000804 (a.k.a “3 stripes”) (1296
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

# Closing thoughts

The maze 300 cube has a QTM diameter of only 48, many of the puzzles discussed
in the blog post have a dramatically larger diameter. I wonder if one of these
will be able to take the record for the configuration with the largest known
god's number.

If you find such a configuration, or you find another intersting feature in the
data, please send me an email and let me know!

# More thoughts - cube orientation

(This update was published on Jan 14, 2023.)

A user on the twisty puzzles forum [raised an interesting
point](https://twistypuzzles.com/forum/viewtopic.php?p=420978#p420978). The way
my code computes graphs' properties assumes that the orientation of the puzzle
is fixed. This does not map nicely to how people visualize and solve puzzles. If
two states differ only by oreintation, then a solver can use the same approach
to both by just rotating the cube.

I re-ran my analysis but modified to consider two configurations the same if
they only differ by orientation (or mirroring). I'll start by looking at the
results in isolation, and then do some comparison between these results and my
previous results where cube orientation was considered.

When we ignore orientation, a few graphs shrink down to a single vertex and a
diameter of 0. In case where dividing by 0 would be inconvenient, I'll be
ignoring these configurations instead.

If you'd like to play with the raw data, I made it [available on my
github](https://github.com/JoshMermel/bandaged-cube-explorer/blob/main/diameter_analysis/analysis_ignoring_orientation.csv). Putting all of it in one .csv felt unwieldly so I split this data into its own file. If you'd like to merge them, the rows are in the same order in both.

## Largest vertex count

This has nothing to do with graph diameters but I've never seen these results
posted before so I might as well mention them here. These are the four
configurations with the largest vertex counts if we ignore orientation.

1. 0x10840080800423 (21490 vertices) [3rd most vertices when considering orientation]
2. 0x10041080200421 (18551 vertices) [7th most vertices when considering orientation]
3. 0x3DA00000C08 (16894 vertices) [10th most vertices when considering orientation]
4. 0x39E00000420 (15134 vertices) [13th most vertices when considering orientation]

<img src="/images/bandaged-cube-diameter/high_vertex_count_ignore_orientation.svg" style="max-height: 800px">

## Largest Diameter (ignoring cube orientation)

Unsurprisingly, many of the puzzles with a large diameter are familiar to us
from earlier in this post. The puzzles with the largest diameter in QTM are:

1.	0x108430C40007A3 (113 QTM, 92 HTM) [5th largest QTM diameter with orientation considered]
2.	0x1003FE80000423 (103 QTM, 76 HTM) [1st largest QTM diameter with orientation considered]
3.	0x1003DA80000423 (99 QTM, 89 HTM) [6th largest QTM diameter with orientation considered]
4.	0x500180800C3D (91 QTM, 72 HTM)
5.	0x210B9100000863 (91 QTM, 71 HTM)
6.	0x10039A800005AD (86 QTM, 69 HTM)
7.	0x73920080007F (86 QTM, 68 HTM)
8.	0x1084308620063D (84 QTM, 64 HTM) [7th largest QTM diameter with orientation considered]
9.	0x18C400C7A00401 (84 QTM, 66 HTM)
10.	0x3DA00000C60 (83 QTM, 66 HTM)

<img src="/images/bandaged-cube-diameter/top_10_diameter_ignore_orientation.svg" style="max-height: 800px">

Several puzzles on this list (#1, #4, #5, #6, #7, #9) have no self-symmetries,
so their graphs are the same whether or not we consider orientation. The
remaining puzzles all had 2x as many vertices before we stopped considering
orientation.

0x10039A800005AD (#6) appeals to me for some reason. It's graph is slightly too
large to link but the shape is pretty fun; one very big loop and then a complex
appendage coming off of it. It has quite a few "normal" edges which makes me
think the solve will have some fun subproblems.

0x73920080007F (#7), has by far the smallest vertex count (518) of these
puzzles. Its
[graph](https://joshmermelstein.com/bandaged-cube-explorer?id=0x73920080007F)
seems to contain no non-trivial loops. I suspect that when this puzzle is solved
by shape, it is also solved by color.

Interestingly, the top 10 by HTM are the same but in a slightly different order.

## QTM vs HTM diameter (ignoring cube orientation)

### High QTM/HTM diameter

Again, several configurations achieve a the upper bound where QTM diameter is
double HTM diameter. Most of these have a small diameter or seem otherwise
boring. A few configurations get close and seem more interesting to me. Two that
stand out are
[0x1084008C238C63](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1084008C238C63&ignore_orientation=true)
(19 QTM, 10 HTM) and
[0x10818086200C23](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10818086200C23&ignore_orientation=true)
(39 QTM, 21 HTM). The former looks a lot like the cube with the largest vertex
count but with a few bandaged centers.

<img src="/images/bandaged-cube-diameter/high_qtm_over_htm_ignore_orientation.svg" style="max-height: 400px">

### Low QTM/HTM diameter

Many configurations achieve the lower bound where QTM and HTM diameters are
equal.
[0x80018620002D](https://joshmermelstein.com/bandaged-cube-explorer?id=0x80018620002D&highlight=rotations) (discussed above)
does not have any self symmetries so it is still the largest of these by
diameter (17). It is also the largest by vertex count (66).

## Diameter/Radius (ignoring cube orientation)

### Low Diameter/Radius

There are 37 configurations where QTM radius is the same as QTM diameter. The
largest of these by diameter is
[0x10F4018C7C0C63](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10F4018C7C0C63&ignore_orientation=true)
(QTM diameter = 5), which seems like it can barely scramble. Another one that
jumps out at me is
[0x30000080200C11](https://joshmermelstein.com/bandaged-cube-explorer?id=0x30000080200C11&ignore_orientation=true)
(39 QTM diameter, 30 QTM radius). Among the puzzles where QTM radius is close to
QTM diameter, it has an unusually high diameter.

<img src="/images/bandaged-cube-diameter/nearly_equal_radius_diameter_ignore_orientation.svg" style="max-height: 400px">

# Comparing graphs with and without considering orientation

## Change in vertex count

When we stop considering orientation, the graphs for most configurations get
smaller. The factor they shrink by roughly represents how many rotated/mirrored
copies of each state we expect to find in the graph.

This quotient has an lower bound of 1 - if a cube has no self-similar states
then its graph will have the same number of vertices whether or not we consider
orientation. The upper bound is 48, since the cube has 24 orientations and each
of those has a mirror image.

### Large change

Exactly one puzzle has 48x as many vertices when orientation is considered -
[0x28A00000414](https://joshmermelstein.com/bandaged-cube-explorer?id=0x28A00000414&ignore_orientation=true).
When considering orientation it can reach 2544 configurations, and when ignoring
orientation it can only reach 53 configurations.

<img src="/images/bandaged-cube-diameter/0x28A00000414.svg" style="max-height: 400px">

Achieving this bound is quite impressive. It means that for any reachable
configuration of this puzzle, all 48 orientations/mirrors are reachable and
distinct from one another.

A few other puzzle get quite close to a 48x reduction. The largest of these by
diameter is
[0x20088000000C06](https://joshmermelstein.com/bandaged-cube-explorer?id=0x20088000000C06&ignore_orientation=true)
(27 QTM when considering orientation, 16 QTM when ignoring it). The largest of
these by vertex count is
[0x28000000C14](https://joshmermelstein.com/bandaged-cube-explorer?id=0x28000000C14&ignore_orientation=true)
(7344 when considering orientation, 157 when ignoring it). This cube is also
notable for having the largest vertex count if we limit outself to cubes with 0
modified center pieces (9528 when considering orientation, 206 when ignoring
it).

<img src="/images/bandaged-cube-diameter/large_vertex_change.svg" style="max-height: 400px">

### No change

Nearly 1000 puzzles have the same number of vertices whether or not orientation
is considered.

The largest by diameter are 0x108430C40007A3 (seen above), then 0x500180800C3D
(seen above). The largest by vertex count are 0x10840084018423 (12647) and
x100400B4200631 (11589).

<img src="/images/bandaged-cube-diameter/high_vertex_low_symmetry.svg" style="max-height: 400px">

### Large graphs despite large changes

Some configurations still have comparatively large graphs, even after losing a
big portion of their vertices when we ignored orientation.

0x1003DE80000403 (now 18th largest) lost 3/4th of its vertices. 0x3FE0010002D
(now 60th largest), lost ~7/8th of its vertices.

<img src="/images/bandaged-cube-diameter/large_diameter_despite_shrinking.svg" style="max-height: 400px">

## Change in diameter

### Large change

Most of the graphs with large percentage drops in diameter end up with ~2
vertices and don't seem too interesting to me. A few that do seem interesting
are
[0x10525A02800C06](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10525A02800C06&ignore_orientation=true)
(14 QTM -> 4 QTM) and
[0x300000B40D85A3](https://joshmermelstein.com/bandaged-cube-explorer?id=0x300000B40D85A3&ignore_orientation=true)
(14 QTM -> 2 QTM), Both drop from 5 HTM to 1 HTM. Another notable drop is from
[0x53A40280007F](https://joshmermelstein.com/bandaged-cube-explorer?id=0x53A40280007F&ignore_orientation=true)
(84 QTM -> 34 QTM).

<img src="/images/bandaged-cube-diameter/large_diameter_change.svg" style="max-height: 800px">

### Small change

There are some interesting examples of graphs whose vertex count drops
significantly but whose diameter does not change.

A few standouts are
[0x10840000000C03](https://joshmermelstein.com/bandaged-cube-explorer?id=0x10840000000C03&ignore_orientation=true)
(904->127 vertices, 23 diameter QTM), 0x3F6000005AD (2304->576 vertices, 49 HTM
diameter).

<img src="/images/bandaged-cube-diameter/small_diameter_change.svg" style="max-height: 400px">

# Bonus - other stuff that seemed interesting

## Banning "normal" corners

If we limit ourselves to puzzles where every corner is bandaged to an edge, we
still have 81 puzzles to look at. If we care about orientation,
[0x39e00100414](https://joshmermelstein.com/bandaged-cube-explorer?id=0x39e00100414)
has the largest diamter (36 QTM) and
[0x2da00000c14](https://joshmermelstein.com/bandaged-cube-explorer?id=0x2da00000c14)
has the largest vertex count (384). If we stop caring about orientation then
[0x21088300000883](https://joshmermelstein.com/bandaged-cube-explorer?id=0x21088300000883&ignore_orientation=true)
has the largest diameter (30 QTM) and the largest vertex count (257).

<img src="/images/bandaged-cube-diameter/no_normal_corners.svg" style="max-height: 800px">

## Banning "normal" edges

If we limit ourselves to puzzles where every edge is bandaged to another piece,
we still have 132 puzzles to look at. Of these,
[0x21081308440862](https://joshmermelstein.com/bandaged-cube-explorer?id=0x21081308440862)
has the largest diameter (28 QTM) whether or not we care about orientation.
0x3fe0000041c has the largest vertex count (1092) when considering orientations
and
[0x1003fe8000043d](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1003fe8000043d&ignore_orientation=true)
has the largest vertex count (321) when ignoring orientations.

<img src="/images/bandaged-cube-diameter/no_normal_edges.svg" style="max-height: 800px">

## Banning "normal" corners and edges

If we restrict ourselves to puzzles with no "normal" corners and no "normal"
edges, there are still 12 distinct puzzles to look at. These puzzles are all
extremely constrainted but some seem like they can still meaningfully scramble.

[0x3FE00000C1C](https://joshmermelstein.com/bandaged-cube-explorer?id=0x3FE00000C1C)
has the largest vertex count (156) if we care consider orientation.
[0x21189308C00C46](https://joshmermelstein.com/bandaged-cube-explorer?id=0x21189308C00C46&ignore_orientation=true)
has the largest vertex count (25) if we do not consider about orientation. It
also has the largest diameter whether or not we care about orientation.
[0x83FE04000631](https://joshmermelstein.com/bandaged-cube-explorer?id=0x83FE04000631&highlight=rotations)
is in third place for vertex count whether or not we care about orientation.

<img src="/images/bandaged-cube-diameter/no_normal_corners_edges.svg" style="max-height: 800px">

## Banning "abnormal" centers

If we restrict ourselves to puzzles where all centers are "normal" meaning no
bandaging to the core or to edges, we have 26 puzzles to look at.

0x20088000000C06 has the largest diameter (27 QTM) of these puzzles if we care
about orientation. This was was noted above for have losing a large fraction of
vertices when we start caring about orientation.
[0x20088000000C03](https://joshmermelstein.com/bandaged-cube-explorer?id=0x20088000000C03&ignore_orientation=true)
has the largest diamter (20 QTM) if we do not care about orientation.

<img src="/images/bandaged-cube-diameter/no_abnormal_centers.svg" style="max-height: 400px">

## Fun graph shapes

Here are a bunch of puzzles where the shape of their graph seemed interesting to
me:

[0x210B0118C00C5A](https://joshmermelstein.com/bandaged-cube-explorer?id=0x210B0118C00C5A)
has two identical looking subgraphs connected by a forced chain. I wonder if the
solve will require navigating the chain frequently - or even at all.

[0x1084018C7C0C30](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1084018C7C0C30)
is imilar to the above but with a much longer chain connecting the segments.

[0x1080018FFC0C21](https://joshmermelstein.com/bandaged-cube-explorer?id=0x1080018FFC0C21)
has a very unusual graph. It has a few big loops that are
all connected to each other, plus so not-too-useful looking long dead ends.

[0x104000B5BDB5AD](https://joshmermelstein.com/bandaged-cube-explorer?id=0x104000B5BDB5AD)
has very small graph with only a few nontrivial loops. It seems like it can
scramble pretty well though and would make for a challenging and highly
constrained solve.

[0x30018080002D](https://joshmermelstein.com/bandaged-cube-explorer?id=0x30018080002D)
has an intersting graph shape. It is sparse but hard to untangle.

<img src="/images/bandaged-cube-diameter/bonus.svg" style="max-height: 1200px">

# Closing thoughts (again)

I'd like to re-emphasize that diameter is not a good way to predict difficulty.
I'm currently fighting with
[0xC00B40D85A0](https://joshmermelstein.com/bandaged-cube-explorer?id=0xC00B40D85A0)
which a QTM diameter of 3 and am finding it pretty difficult. That said, I think
it's difficulty is quite different from the high-diameter puzzles discussed in
those post.

I'm probably done looking at the diameter data for now, but I'm certainly not
done analyzing bandaged cubes. One idea I had was a web tool to help find paths
between different configurations. Another idea I'm considering is building a
tool that finds non-trivial loops in a puzzle's state space; for the purpose of
finding useful algorithms. If I do either of these I'll be sure to write a blog
post too.
