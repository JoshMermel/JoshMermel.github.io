---
layout: post
title: Bermuda Cubes
---

Background
==========

I've recently been playing with a series of puzzles called the Bermuda cubes.
They look and behave a like a Rubik's cube but some centers have been replaced
by triangles and others have been rotated 45 degrees. Mechanically, these
new centers don't add anything new. The pieces around them can be interchanged
with regular corners and edges. From a solving perspective, these centers add a
few challenges. 

<img src="/images/bermuda/header_photo.jpg" style="max-height: 400px">

1. The unusually shaped pieces and centers make it harder for me to visualize
   setups.
2. The notation for algorithms is cumbersome which makes it harder for me to
   memorize them.
3. The triangle centers block other faces depending on their position so it's
   harder to come up with broadly reusable algorithms.

One nice feature of this collection of puzzles is that they have a suggested
order. Each one is named after a planet and they (mostly) get harder as the
planets get further from the sun.

Now that I've completed the series, I wanted to write a blog post to reflect on
the solving experiences and to record my solving notes for others (and maybe for
my future self).

First Approach
==============

A pretty standard technique for designing a solution is to solve one kind of
piece at a time. The idea is that you can solve the solve the first piece type
with ignoring the rest of the puzzle. Then you can solve the second piece type
while only paying attention to the first piece type and ignoring the rest, etc.
This sort of solution doesn't require very many algorithms but does require you
to be good at visualizing setups so you can get the most out of your sequences.

For this whole series, it felt most natural to me to solve centers then edges,
then corners. Moving centers after anything else is solved seemed like a pain so
I knew I wanted to do those first. Isolating edges without slice turns seemed
like it would be painful so I chose to do those before corners.

With a bit of planning, I was able to adapt well known 3x3x3 commutators to the
first few puzzles in the Bermuda cubes series. The trick was identifying which
center positions were workable and then finding ways to set up to those
positions. But when I started working on Mars, I found it a lot harder to adapt
ideas I already knew from 3x3x3 Rubik's cubes. That puzzle has three triangle
centers, two of which are opposite each other. That means the middle one is
always blocking at least one of the other triangles and that made it much
trickier to access certain pieces of the puzzle.

I decided to search for sequences that depended on as few faces as possible,
that way the state of the puzzle would matter as little as possible when
executing them. In particular, I looked for algorithms that only depended on
neighboring triangle and square faces.

My big breakthrough centered on the fact that RUR'U' repeated 3 times has no
effect on edges and is a 2,2 swap of corners. I could execute that with a square
face on top and a triangle face on the right in the correct orientation. From
there, I could turn this into a useful algorithm by swapping some of the pieces
of the U face with the unmodified corner/edge/corner on the hypotenuse of the R
face. There were a few nice ways to do that interchange and all of the ones I
tried resulted in useful 3-cycles!

<img src="/images/bermuda/34_algos.png" style="max-height: 400px">

- left: [(R U R' U')3, R^-1.5 U2 R^1.5]
- middle: [(R U R' U')3, R^-1.5 U R^1.5]
- right: [(R U R' U')3, R^-1.5 U' R^1.5]

These cubes are tilted a bit to show the outcomes better. These should be
executed with the orange face on U and the white face on R, starting with the
white triangle orientated as shown.

I really like this trio of algorithms. The similarity made them easy to
remember. The fact that the cube is in its normal shape most of the time meant I
could use edges as clues if I ever lost my place in the middle. On top of that,
they were able to easily move corners in and out of the tricky-to-reach spots on
the hypotenuse of the triangular face.

For corner orientation, I also wanted to find a sequence that would work with
only two turnable faces. Starting with a square face on U and a triangle face on
R (orientated the same as my permutation algorithms), I realized I could execute
a [Sune](https://www.speedsolving.com/wiki/index.php/Sune). After a bit of
fiddling, I found that three Sunes in a row had no effect on edges and did a
swapped the corners of the square face with their diagonals while reorienting
just one of those pairs. That meant I could do a U2 and another triple-Sune to
swap the corners back but end up with two of them orientated differently.

<img src="/images/bermuda/34_corner_orientation_algos.png" style="max-height: 400px">

- left: effect after 3 Sunes
- right: effect after [(R U R' U R U2 R')3 , U2]

You might notice that I only wrote about algorithms for corners, not ones for
edges. That's because I found the edges pretty intuitive. All these algorithms
I'm writing about in this post are for using at the end of a solve when pieces
are mostly where they need to be. Before that, I try to use intuition and
blockbuilding as much as possible.

Oops All Triangles
==================

At this point, I thought I'd cracked the whole series and all the was left was
coming up with nice setup sequences. Then I started scrambling Saturn and
realized my mistake. Even though that puzzle has two square-centered faces,
neither of them is able to turn. Hilariously, the edge in between them is
mechanically functional, it's just impossible to unblock either face and
actually move it. This meant all my nice ideas about triangle/square algorithms
wasn't applicable.

I was pretty happy with my centers, edges, corners solve order and I liked the
flexibility that I got from an algorithm that only needed two turnable faces; so
I decided to try to find some new algorithms to use for adjacent triangular
faces.

I came up with some kinda ugly ideas but I wasn't looking forward to executing
those long algorithms on an actual puzzle. Eventually I decided to give up and
write a program to find some shorter sequences.

Writing a solver
================

In order to model the state of the puzzle, I needed to know orientation of each
center. I used an enum where I named each state after the edge of the triangle
that was shared with the other face:

    enum Orientation : int {
      HYPOT = 0,
      LEFT_LEG = 1,
      RIGHT_LEG = 2,
    };

From there, I modeled each state of the puzzle as a vector of edge locations and
a vector of corner locations. This meant each move could be implemented as a
permutation of those vectors.

    struct Node {
      Node(const std::vector<int>& corners,
          const std::vector<int>& edges,
          const Orientation& left_center,
          const Orientation& right_center,
          const std::string& path) :
        corners_(corners),
        edges_(edges),
        left_center_(left_center),
        right_center_(right_center),
        path_(path) {}

      std::vector<int> corners_;
      std::vector<int> edges_;
      Orientation right_center_;
      Orientation left_center_;
      std::string path_;
    };

One unintuitive detail is that the corners vector had one entry per sticker, not
one entry per piece. That's because I wanted to track the orientation of the
corners. This wasn't needed for edges since edges can't change orientation with
just two axes.

<img src="/images/bermuda/two_triangles_labels.jpg" style="max-height: 400px">

An example of a transformation function is as follows:

    Node LhsClockwise(Node in) {
      if (in.left_center_ == Orientation::HYPOT) {
        PermuteVec(in.corners_, {
            {0,9,3,12,6},
            {1,10,4,13,7},
            {2,11,5,14,8},
          });
        in.left_center_ = Orientation::RIGHT_LEG;
      } else if (in.left_center_ == Orientation::LEFT_LEG) {
        PermuteVec(in.corners_, {
            {0,9,3,12,6},
            {1,10,4,13,7},
            {2,11,5,14,8},
          });
        in.left_center_ = Orientation::HYPOT;
      } else if (in.left_center_ == Orientation::RIGHT_LEG) {
        PermuteVec(in.corners_, {
            {0,12,9,6,3},
            {1,13,10,7,4},
            {2,14,11,8,5},
          });
        in.left_center_ = Orientation::LEFT_LEG;
      }
      PermuteVec(in.edges_, {{0,2,1}});
      in.path_ += "L";

      return in;
    }

You might notice that I always add "L" to the path even though the number of
degrees being turned isn't always the same. To simplify notation, for
triangle+triangle algorithms, I used L to mean "rotate the left triangle
clockwise until the right is unblocked" and the same for "R" mutatis mutandis.

Finally, I wrote some code to breadth first search the graph, and print nodes
which only modified a small number of corners. This approach worked well. By
looking for states with 6 modified corner stickers, I could find pure
orientation algorithms. By looking for states with 9 modified stickers, I could
find permutation algorithms. I've included a table of about 400 algorithms that
the solver found at the bottom of this post.

### Aside: C++ and hashing vectors

One thing that was moderately annoying when I was writing my bfs was that I
wanted to make a set of my Node struct so I could tell whether a node had been
visited or not. Because it contains a vector, C++ didn't know how to hash it so
I had to write my own custom hash function. I found that a little annoying.

Second approach to writing a solver
===================================

The results from my first solver were decent but still felt awkward to me. The
2-axes approach generated algorithms that were highly reusable but were kinda
long. I started to wonder if I could generate shorter ones by allowing the use
of a third axis.

This meant I needed to make a few changes to how I was modeling states. First of
all, I need to start modeling edges in a way that lets me see their orientation.
This isn't too bad - we can use the same trick as we did with corners and use
one int for each sticker. Next, we need to model center orientation in a way
that lets us see which faces are blocked. After some thought, I came up with a
solution to this that felt really elegant.

We can treat each face like a regular pentagon (i.e. from a megaminx) but with
some specific edges bandaged to those centers.

<img src="/images/bermuda/three_triangles_labels.jpg" style="max-height: 400px">

I updated my code to BFS using three triangular faces and this new model but the
results were extremely disappointing. All the algorithms it found used just two
out of three available faces! If I didn't make any mistakes in my
implementation, this means that adding a third triangular face doesn't help at
all.

Closing thoughts
================

If you enjoyed this and want to pick up some Bermuda cubes, you probably don't
need to get the whole set. Some, like Jupiter and Uranus feel pretty much the
same with my method. I think my favorites were Mars and Neptune. Maybe start
with Venus if you're looking for something easier and maybe end with one of
Jupiter/Saturn if you're looking for something harder.

Numbering the pieces and figuring out the permutation that went with each turn
was kinda tedious. I wonder if there's a way to automate that. Also, I think
there's room for a nice library that takes the permutations of each move as
input and has utilities for things like BFS written in a general way. There's
certainly some low hanging fruit when it comes to optimizing my current solver -
it would be great to do that optimization on a shared library once and then
benefit from it on future projects.

I think it would be fun to apply this same technique to mixes of 0 and 1 faces
on the circle 3x3x3s. I think there would be lots of fun arrangements to analyze
and I bet some really short algorithms would show up.

I know there is a series of Bermuda megaminxes that were also mass produced but
are hard to find and expensive these days. Maybe I'll solve a few of those in a
simulator next time I'm looking for the Bermuda puzzle kind of experience.

Appendix
========

### triangle-triangle Permutation algorithms:

(see data model above for starting position and label meanings)

| Effect | Alg |
|-------|--------|
|(9,21,16)(10,22,17)(11,23,15) | LRLRL'RL'R'L'R'LR'|
|(9,18,23)(10,19,21)(11,20,22) | LRLR'LR'L'R'L'RL'R|
|(6,14,18)(7,12,19)(8,13,20) | LR'LRLRL'RL'R'L'R'|
|(6,11,20)(7,9,18)(8,10,19) | LR'LR'L'R'L'RL'RLR|
|(9,19,14)(10,20,12)(11,18,13) | L'RL'RLRLR'LR'L'R'|
|(6,18,14)(7,19,12)(8,20,13) | L'RL'R'L'R'LR'LRLR|
|(9,19,17)(10,20,15)(11,18,16) | L'R'L'RL'RLRLR'LR'|
|(9,16,21)(10,17,22)(11,15,23) | L'R'L'R'LR'LRLRL'R|
|(9,14,19)(10,12,20)(11,13,18) | RLRL'RL'R'L'R'LR'L|
|(9,17,19)(10,15,20)(11,16,18) | RL'RL'R'L'R'LR'LRL|
|(9,23,18)(10,21,19)(11,22,20) | R'LR'LRLRL'RL'R'L'|
|(6,20,11)(7,18,9)(8,19,10) | R'L'R'LR'LRLRL'RL'|
|(0,18,23)(1,19,21)(2,20,22) | LRLRLRL'RL'R'L'R'LRL'|
|(12,18,21)(13,19,22)(14,20,23) | LRLR'LRLRL'RL'R'L'RL'|
|(3,7,19)(4,8,20)(5,6,18) | LRL'R'LR'LRLRL'RL'RL'|
|(3,18,21)(4,19,22)(5,20,23) | LRL'R'L'RL'RLRLR'LRL'|
|(12,21,18)(13,22,19)(14,23,20) | LR'LRLR'LR'L'R'L'RL'R'L'|
|(3,19,7)(4,20,8)(5,18,6) | LR'LR'LR'L'R'L'RL'RLR'L'|
|(0,23,18)(1,21,19)(2,22,20) | LR'L'RLRLR'LR'L'R'L'R'L'|
|(3,21,18)(4,22,19)(5,23,20) | LR'L'RL'R'L'R'LR'LRLR'L'|
|(0,16,19)(1,17,20)(2,15,18) | L'RLR'LRLRL'RL'R'L'RL|
|(3,17,19)(4,15,20)(5,16,18) | L'RLR'L'R'L'RL'RLRLRL|
|(0,18,14)(1,19,12)(2,20,13) | L'RL'RL'RLRLR'LR'L'RL|
|(6,16,19)(7,17,20)(8,15,18) | L'RL'R'L'RL'RLRLR'LRL|
|(0,19,16)(1,20,17)(2,18,15) | L'R'LRLR'LR'L'R'L'RL'R'L|
|(0,14,18)(1,12,19)(2,13,20) | L'R'LRL'RL'R'L'R'LR'LR'L|
|(6,19,16)(7,20,17)(8,18,15) | L'R'L'RL'R'L'R'LR'LRLR'L|
|(3,19,17)(4,20,15)(5,18,16) | L'R'L'R'L'R'LR'LRLRL'R'L|
|(3,9,14)(4,10,12)(5,11,13) | RLRLRLR'LR'L'R'L'RLR'|
|(9,12,22)(10,13,23)(11,14,21) | RLRL'RLRLR'LR'L'R'LR'|
|(0,16,9)(1,17,10)(2,15,11) | RLR'L'RL'RLRLR'LR'LR'|
|(0,11,14)(1,9,12)(2,10,13) | RLR'L'R'LR'LRLRL'RLR'|
|(9,22,12)(10,23,13)(11,21,14) | RL'RLRL'RL'R'L'R'LR'L'R'|
|(0,9,16)(1,10,17)(2,11,15) | RL'RL'RL'R'L'R'LR'LRL'R'|
|(3,14,9)(4,12,10)(5,13,11) | RL'R'LRLRL'RL'R'L'R'L'R'|
|(0,14,11)(1,12,9)(2,13,10) | RL'R'LR'L'R'L'RL'RLRL'R'|
|(3,7,10)(4,8,11)(5,6,9) | R'LRL'RLRLR'LR'L'R'LR|
|(0,7,9)(1,8,10)(2,6,11) | R'LRL'R'L'R'LR'LRLRLR|
|(3,9,21)(4,10,22)(5,11,23) | R'LR'LR'LRLRL'RL'R'LR|
|(6,9,15)(7,10,16)(8,11,17) | R'LR'L'R'LR'LRLRL'RLR|
|(3,10,7)(4,11,8)(5,9,6) | R'L'RLRL'RL'R'L'R'LR'L'R|
|(3,21,9)(4,22,10)(5,23,11) | R'L'RLR'LR'L'R'L'RL'RL'R|
|(6,15,9)(7,16,10)(8,17,11) | R'L'R'LR'L'R'L'RL'RLRL'R|
|(0,9,7)(1,10,8)(2,11,6) | R'L'R'L'R'L'RL'RLRLR'L'R|
|(0,21,16)(1,22,17)(2,23,15) | LRLRLRLR'LR'LR'LRL'R'|
|(0,21,12)(1,22,13)(2,23,14) | LRLR'LRLR'L'RL'R'L'RL'R'|
|(3,22,12)(4,23,13)(5,21,14) | LRL'RLRL'RLR'L'R'LR'L'R'|
|(3,22,9)(4,23,10)(5,21,11) | LRL'RLRL'RL'R'LR'L'R'LR'|
|(3,10,19)(4,11,20)(5,9,18) | LRL'RLRL'R'LR'L'R'LR'L'R|
|(3,7,12)(4,8,13)(5,6,14) | LRL'R'LR'LR'LR'L'R'L'R'L'R'|
|(3,9,18)(4,10,19)(5,11,20) | LR'LRLRL'R'L'RL'R'L'R'LR|
|(0,18,12)(1,19,13)(2,20,14) | LR'LRLR'LRL'RL'R'L'RL'R'|
|(0,18,7)(1,19,8)(2,20,6) | LR'LR'LR'LRLRLRLR'L'R|
|(0,23,9)(1,21,10)(2,22,11) | LR'L'RLRLRLRL'RL'RL'R|
|(0,15,9)(1,16,10)(2,17,11) | LR'L'R'LRL'RLR'L'R'LRL'R|
|(0,11,18)(1,9,19)(2,10,20) | LR'L'R'LR'L'RL'RLRL'RLR'|
|(0,15,6)(1,16,7)(2,17,8) | LR'L'R'LR'L'R'L'RLRL'RLR|
|(0,9,19)(1,10,20)(2,11,18) | L'RLR'LRLR'LRL'R'L'RL'R'|
|(3,17,9)(4,15,10)(5,16,11) | L'RLR'L'R'L'R'L'R'LR'LR'LR'|
|(3,19,14)(4,20,12)(5,18,13) | L'RL'RL'RL'R'L'R'L'R'L'RLR'|
|(3,19,6)(4,20,7)(5,18,8) | L'RL'R'LRLR'L'RL'R'LRLR'|
|(0,14,6)(1,12,7)(2,13,8) | L'R'LRL'RL'RL'RLRLRLR|
|(3,16,6)(4,17,7)(5,15,8) | L'R'L'RLR'LRL'R'L'RLR'LR|
|(3,16,21)(4,17,22)(5,15,23) | L'R'L'R'L'R'L'RL'RL'RL'R'LR|
|(3,12,7)(4,13,8)(5,14,6) | RLRLRLRL'RL'RL'RLR'L'|
|(3,12,22)(4,13,23)(5,14,21) | RLRL'RLRL'R'LR'L'R'LR'L'|
|(0,12,21)(1,13,22)(2,14,23) | RLR'LRLR'LRL'R'L'RL'R'L'|
|(0,12,18)(1,13,19)(2,14,20) | RLR'LRLR'LR'L'RL'R'L'RL'|
|(0,19,9)(1,20,10)(2,18,11) | RLR'LRLR'L'RL'R'L'RL'R'L|
|(0,16,21)(1,17,22)(2,15,23) | RLR'L'RL'RL'RL'R'L'R'L'R'L'|
|(0,18,11)(1,19,9)(2,20,10) | RL'RLRLR'L'R'LR'L'R'L'RL|
|(3,9,22)(4,10,23)(5,11,21) | RL'RLRL'RLR'LR'L'R'LR'L'|
|(3,9,17)(4,10,15)(5,11,16) | RL'RL'RL'RLRLRLRL'R'L|
|(3,14,19)(4,12,20)(5,13,18) | RL'R'LRLRLRLR'LR'LR'L|
|(3,6,19)(4,7,20)(5,8,18) | RL'R'L'RLR'LRL'R'L'RLR'L|
|(3,18,9)(4,19,10)(5,20,11) | RL'R'L'RL'R'LR'LRLR'LRL'|
|(3,6,16)(4,7,17)(5,8,15) | RL'R'L'RL'R'L'R'LRLR'LRL|
|(3,19,10)(4,20,11)(5,18,9) | R'LRL'RLRL'RLR'L'R'LR'L'|
|(0,7,18)(1,8,19)(2,6,20) | R'LRL'R'L'R'L'R'L'RL'RL'RL'|
|(0,9,23)(1,10,21)(2,11,22) | R'LR'LR'LR'L'R'L'R'L'R'LRL'|
|(0,9,15)(1,10,16)(2,11,17) | R'LR'L'RLRL'R'LR'L'RLRL'|
|(3,21,16)(4,22,17)(5,23,15) | R'L'RLR'LR'LR'LRLRLRL|
|(0,6,15)(1,7,16)(2,8,17) | R'L'R'LRL'RLR'L'R'LRL'RL|
|(0,6,14)(1,7,12)(2,8,13) | R'L'R'L'R'L'R'LR'LR'LR'L'RL|
|(3,21,17)(4,22,15)(5,23,16) | LRLRL'R'L'R'LR'LRLRLR'L'|
|(3,20,10)(4,18,11)(5,19,9) | LRL'R'LR'LRLR'LRL'RL'R'L|
|(3,17,21)(4,15,22)(5,16,23) | LRL'R'L'R'L'RL'RLRLR'L'R'L'|
|(0,11,20)(1,9,18)(2,10,19) | LR'L'RL'RLR'LRLR'LR'L'RL|
|(3,10,20)(4,11,18)(5,9,19) | L'RLR'LR'L'RL'R'L'RL'RLR'L'|
|(0,23,16)(1,21,17)(2,22,15) | L'R'LRLRLR'LR'L'R'L'RLRL|
|(0,20,11)(1,18,9)(2,19,10) | L'R'LRL'RL'R'L'RL'R'LR'LRL'|
|(0,16,23)(1,17,21)(2,15,22) | L'R'L'R'LRLRL'RL'R'L'R'L'RL|
|(0,14,7)(1,12,8)(2,13,6) | RLRLR'L'R'L'RL'RLRLRL'R'|
|(0,10,19)(1,11,20)(2,9,18) | RLR'L'RL'RLRL'RLR'LR'L'R|
|(0,7,14)(1,8,12)(2,6,13) | RLR'L'R'L'R'LR'LRLRL'R'L'R'|
|(3,18,11)(4,19,9)(5,20,10) | RL'R'LR'LRL'RLRL'RL'R'LR|
|(0,19,10)(1,20,11)(2,18,9) | R'LRL'RL'R'LR'L'R'LR'LRL'R'|
|(3,14,7)(4,12,8)(5,13,6) | R'L'RLRLRL'RL'R'L'R'LRLR|
|(3,11,18)(4,9,19)(5,10,20) | R'L'RLR'LR'L'R'LR'L'RL'RLR'|
|(3,7,14)(4,8,12)(5,6,13) | R'L'R'L'RLRLR'LR'L'R'L'R'LR|
|(0,14,23)(1,12,21)(2,13,22) | LRLRLRL'RLRLRL'R'LR'L'R'|
|(3,23,13)(4,21,14)(5,22,12) | LRLRLRL'R'L'RLRLRLR'L'R'|
|(6,10,22)(7,11,23)(8,9,21) | LRLRL'R'LR'LR'L'R'L'RLR'LR'|
|(3,14,21)(4,12,22)(5,13,23) | LRLR'LRL'R'L'R'L'RL'R'L'R'L'R'|
|(0,22,13)(1,23,14)(2,21,12) | LRLR'L'R'L'R'L'RLRL'R'L'R'L'R'|
|(0,3,22)(1,4,23)(2,5,21) | LRL'RLR'LRLRL'RLR'LRL'R'|
|(0,5,12)(1,3,13)(2,4,14) | LRL'R'LR'L'RL'R'L'R'LR'L'RL'R'|
|(0,18,10)(1,19,11)(2,20,9) | LR'LRLR'L'R'L'R'L'RL'R'L'R'L'R|
|(12,19,15)(13,20,16)(14,18,17) | LR'LR'L'RLRLR'LR'LRL'R'L'R'|
|(0,20,9)(1,18,10)(2,19,11) | LR'L'R'L'R'LR'L'R'L'R'L'RLRL'R|
|(3,20,9)(4,18,10)(5,19,11) | L'RLRLRL'RLRLRLR'L'R'LR'|
|(6,18,22)(7,19,23)(8,20,21) | L'RL'RLR'L'R'L'RL'RL'R'LRLR|
|(3,19,11)(4,20,9)(5,18,10) | L'RL'R'L'RLRLRLR'LRLRLR'|
|(0,7,16)(1,8,17)(2,6,15) | L'R'LRLR'L'RL'RLR'LRLRLR|
|(0,8,5)(1,6,3)(2,7,4) | L'R'LRL'RLR'LRLRL'RLR'LR|
|(0,15,3)(1,16,4)(2,17,5) | L'R'LR'L'RL'R'L'R'LR'L'RL'R'LR|
|(3,15,8)(4,16,6)(5,17,7) | L'R'L'RLRLRLR'L'R'LRLRLR|
|(9,16,13)(10,17,14)(11,15,12) | L'R'L'R'LRL'RL'RLRLR'L'RL'R|
|(0,17,8)(1,15,6)(2,16,7) | L'R'L'R'L'R'LRLR'L'R'L'R'L'RLR|
|(3,7,17)(4,8,15)(5,6,16) | L'R'L'R'L'R'LR'L'RL'RLR'L'R'LR|
|(3,21,14)(4,22,12)(5,23,13) | RLRLRLR'LRLRLR'L'RL'R'L'|
|(0,13,22)(1,14,23)(2,12,21) | RLRLRLR'L'R'LRLRLRL'R'L'|
|(12,15,19)(13,16,20)(14,17,18) | RLRLR'L'RL'RL'R'L'R'LRL'RL'|
|(0,23,14)(1,21,12)(2,22,13) | RLRL'RLR'L'R'L'R'LR'L'R'L'R'L'|
|(3,13,23)(4,14,21)(5,12,22) | RLRL'R'L'R'L'R'LRLR'L'R'L'R'L'|
|(0,12,5)(1,13,3)(2,14,4) | RLR'LRL'RLRLR'LRL'RLR'L'|
|(0,22,3)(1,23,4)(2,21,5) | RLR'L'RL'R'LR'L'R'L'RL'R'LR'L'|
|(3,9,20)(4,10,18)(5,11,19) | RL'RLRL'R'L'R'L'R'LR'L'R'L'R'L|
|(6,22,10)(7,23,11)(8,21,9) | RL'RL'R'LRLRL'RL'RLR'L'R'L'|
|(3,11,19)(4,9,20)(5,10,18) | RL'R'L'R'L'RL'R'L'R'L'R'LRLR'L|
|(0,10,18)(1,11,19)(2,9,20) | R'LRLRLR'LRLRLRL'R'L'RL'|
|(9,13,16)(10,14,17)(11,12,15) | R'LR'LRL'R'L'R'LR'LR'L'RLRL|
|(0,9,20)(1,10,18)(2,11,19) | R'LR'L'R'LRLRLRL'RLRLRL'|
|(3,17,7)(4,15,8)(5,16,6) | R'L'RLRL'R'LR'LRL'RLRLRL|
|(0,3,15)(1,4,16)(2,5,17) | R'L'RLR'LRL'RLRLR'LRL'RL|
|(0,5,8)(1,3,6)(2,4,7) | R'L'RL'R'LR'L'R'L'RL'R'LR'L'RL|
|(0,8,17)(1,6,15)(2,7,16) | R'L'R'LRLRLRL'R'L'RLRLRL|
|(6,22,18)(7,23,19)(8,21,20) | R'L'R'L'RLR'LR'LRLRL'R'LR'L|
|(3,8,15)(4,6,16)(5,7,17) | R'L'R'L'R'L'RLRL'R'L'R'L'R'LRL|
|(0,16,7)(1,17,8)(2,15,6) | R'L'R'L'R'L'RL'R'LR'LRL'R'L'RL|
|(12,21,17)(13,22,15)(14,23,16) | LRLRL'RL'RLRLR'LR'L'RL'R'L'|
|(6,12,16)(7,13,17)(8,14,15) | LRLRL'R'LR'LRLRL'RL'RL'R'L'|
|(12,17,21)(13,15,22)(14,16,23) | LRLR'LRL'RL'R'L'R'LR'LR'L'R'L'|
|(6,16,12)(7,17,13)(8,15,14) | LRLR'LR'LR'L'R'L'RL'RLR'L'R'L'|
|(3,22,8)(4,23,6)(5,21,7) | LRL'RL'R'LR'LRLRL'RL'RLR'L'|
|(3,8,22)(4,6,23)(5,7,21) | LRL'R'LR'LR'L'R'L'RL'RLR'LR'L'|
|(6,14,17)(7,12,15)(8,13,16) | LR'LRLRLRL'RL'R'L'R'LRL'RL'|
|(3,11,20)(4,9,18)(5,10,19) | LR'LRLR'L'R'L'R'L'RLRL'R'L'R'L|
|(6,17,14)(7,15,12)(8,16,13) | LR'LR'L'RLRLR'LR'L'R'L'R'L'RL'|
|(0,17,20)(1,15,18)(2,16,19) | LR'L'RL'RL'RLRLR'LR'L'RLRL'|
|(0,3,20)(1,4,18)(2,5,19) | LR'L'RL'R'L'RL'RLRLR'LRLRL'|
|(0,20,17)(1,18,15)(2,19,16) | LR'L'R'LRL'RL'R'L'R'LR'LR'LRL'|
|(0,20,10)(1,18,11)(2,19,9) | LR'L'R'L'RLRL'R'L'R'L'R'LRLR'L|
|(0,20,3)(1,18,4)(2,19,5) | LR'L'R'L'RL'R'L'R'LR'LRLR'LRL'|
|(3,20,11)(4,18,9)(5,19,10) | L'RLRLR'L'R'LRLRLRL'R'L'RL'|
|(3,20,23)(4,18,21)(5,19,22) | L'RLRL'R'LR'LRLRL'RL'RL'R'L|
|(3,23,20)(4,21,18)(5,22,19) | L'RLR'LR'LR'L'R'L'RL'RLR'L'R'L|
|(6,14,22)(7,12,23)(8,13,21) | L'RL'RLR'L'R'L'RL'RLRLRLR'L|
|(0,10,20)(1,11,18)(2,9,19) | L'RL'R'L'RLRLRLR'L'R'LRLRL'|
|(6,22,14)(7,23,12)(8,21,13) | L'RL'R'L'R'L'R'LR'LRLRL'R'LR'L|
|(0,13,15)(1,14,16)(2,12,17) | L'R'LRL'RL'RLRLR'LR'L'RL'RL|
|(0,15,13)(1,16,14)(2,17,12) | L'R'LR'LRL'RL'R'L'R'LR'LR'L'RL|
|(6,12,21)(7,13,22)(8,14,23) | L'R'L'RL'RL'RLRLR'LR'L'RLRL|
|(6,23,16)(7,21,17)(8,22,15) | L'R'L'RL'R'LR'LRLRL'RL'RLRL|
|(6,21,12)(7,22,13)(8,23,14) | L'R'L'R'LRL'RL'R'L'R'LR'LR'LRL|
|(6,16,23)(7,17,21)(8,15,22) | L'R'L'R'LR'LR'L'R'L'RL'RLR'LRL|
|(6,23,13)(7,21,14)(8,22,12) | RLRLR'LR'LRLRL'RL'R'LR'L'R'|
|(6,15,21)(7,16,22)(8,17,23) | RLRLR'L'RL'RLRLR'LR'LR'L'R'|
|(6,13,23)(7,14,21)(8,12,22) | RLRL'RLR'LR'L'R'L'RL'RL'R'L'R'|
|(6,21,15)(7,22,16)(8,23,17) | RLRL'RL'RL'R'L'R'LR'LRL'R'L'R'|
|(0,12,17)(1,13,15)(2,14,16) | RLR'LR'L'RL'RLRLR'LR'LRL'R'|
|(0,17,12)(1,15,13)(2,16,14) | RLR'L'RL'RL'R'L'R'LR'LRL'RL'R'|
|(6,17,22)(7,15,23)(8,16,21) | RL'RLRLRLR'LR'L'R'L'RLR'LR'|
|(6,22,17)(7,23,15)(8,21,16) | RL'RL'R'LRLRL'RL'R'L'R'L'R'LR'|
|(3,8,11)(4,6,9)(5,7,10) | RL'R'LR'LR'LRLRL'RL'R'LRLR'|
|(0,10,5)(1,11,3)(2,9,4) | RL'R'LR'L'R'LR'LRLRL'RLRLR'|
|(3,11,8)(4,9,6)(5,10,7) | RL'R'L'RLR'LR'L'R'L'RL'RL'RLR'|
|(0,5,10)(1,3,11)(2,4,9) | RL'R'L'R'LR'L'R'L'RL'RLRL'RLR'|
|(0,10,13)(1,11,14)(2,9,12) | R'LRLR'L'RL'RLRLR'LR'LR'L'R|
|(0,13,10)(1,14,11)(2,12,9) | R'LRL'RL'RL'R'L'R'LR'LRL'R'L'R|
|(12,15,23)(13,16,21)(14,17,22) | R'LR'LRL'R'L'R'LR'LRLRLRL'R|
|(12,23,15)(13,21,16)(14,22,17) | R'LR'L'R'L'R'L'RL'RLRLR'L'RL'R|
|(3,23,6)(4,21,7)(5,22,8) | R'L'RLR'LR'LRLRL'RL'R'LR'LR|
|(3,6,23)(4,7,21)(5,8,22) | R'L'RL'RLR'LR'L'R'L'RL'RL'R'LR|
|(12,16,22)(13,17,23)(14,15,21) | R'L'R'LR'LR'LRLRL'RL'R'LRLR|
|(6,15,13)(7,16,14)(8,17,12) | R'L'R'LR'L'RL'RLRLR'LR'LRLR|
|(12,22,16)(13,23,17)(14,21,15) | R'L'R'L'RLR'LR'L'R'L'RL'RL'RLR|
|(6,13,15)(7,14,16)(8,12,17) | R'L'R'L'RL'RL'R'L'R'LR'LRL'RLR|
|(0,11,23)(1,9,21)(2,10,22) | LRLRLRL'RLR'LRLRLRL'RLR'|
|(3,12,18)(4,13,19)(5,14,20) | LRLR'LRLRLRL'RL'R'L'RL'RL'R|
|(3,9,23)(4,10,21)(5,11,22) | LRL'RLR'L'R'LRLR'LR'LRLRLR'|
|(3,23,9)(4,21,10)(5,22,11) | LRL'RL'RLR'L'R'LR'LR'L'RL'RLR'|
|(3,18,10)(4,19,11)(5,20,9) | LRL'R'LR'L'RL'RLR'LR'L'RL'R'LR|
|(3,10,18)(4,11,19)(5,9,20) | LRL'R'LR'L'R'L'R'L'R'LRLR'L'R'LR'|
|(0,11,19)(1,9,20)(2,10,18) | LR'LRLR'L'R'LRLRLRLR'LRL'R'|
|(15,20,21)(16,18,22)(17,19,23) | LR'LRLR'L'R'LRL'R'L'RL'R'LRL'R|
|(6,10,12)(7,11,13)(8,9,14) | LR'LRL'R'LR'L'R'LRL'R'L'RLRL'R|
|(3,10,16)(4,11,17)(5,9,15) | LR'LR'LR'L'R'LR'LRLRLRL'RLR|
|(0,18,13)(1,19,14)(2,20,12) | LR'L'RL'RLR'LR'LRLR'L'RL'RL'R'|
|(3,14,18)(4,12,19)(5,13,20) | LR'L'RL'R'L'R'L'R'LR'L'RL'R'L'R'L'R'|
|(0,13,18)(1,14,19)(2,12,20) | LR'L'R'L'R'LR'LR'L'R'LRLR'L'RL'R'|
|(3,8,19)(4,6,20)(5,7,18) | L'RLRLRLRL'R'L'RLRL'RL'R'LR|
|(0,7,19)(1,8,20)(2,6,18) | L'RLR'LRLRLRL'RLR'LRLRLR|
|(3,19,8)(4,20,6)(5,18,7) | L'RLR'LR'L'RL'RLR'LRL'R'L'R'LR|
|(0,11,21)(1,9,22)(2,10,23) | L'RL'RL'RLRL'RL'R'L'R'L'R'LR'L'R'|
|(6,12,11)(7,13,9)(8,14,10) | L'RL'R'LRL'RLRL'R'LRLR'L'R'LR'|
|(15,21,19)(16,22,20)(17,23,18) | L'RL'R'L'RLRL'R'LRLR'LRL'R'LR'|
|(0,17,9)(1,15,10)(2,16,11) | L'R'LRLRL'R'LR'L'RL'RLR'LR'L'R|
|(0,19,11)(1,20,9)(2,18,10) | L'R'LRL'RLR'LR'L'RL'RLR'LRL'R'|
|(0,9,17)(1,10,15)(2,11,16) | L'R'LRL'RL'R'L'RLRL'R'L'R'L'R'L'R|
|(0,6,19)(1,7,20)(2,8,18) | L'R'L'RL'R'L'R'L'R'LR'LRLR'LR'LR'|
|(3,10,17)(4,11,15)(5,9,16) | L'R'L'R'L'R'LR'L'RL'R'L'R'L'R'LR'L'R|
|(3,18,14)(4,19,12)(5,20,13) | RLRLRLR'LRL'RLRLRLR'LRL'|
|(0,21,11)(1,22,9)(2,23,10) | RLRL'RLRLRLR'LR'L'R'LR'LR'L|
|(6,11,12)(7,9,13)(8,10,14) | RL'RLRL'R'L'RLR'L'R'LR'L'RLR'L|
|(15,19,21)(16,20,22)(17,18,23) | RL'RLR'L'RL'R'L'RLR'L'R'LRLR'L|
|(0,19,6)(1,20,7)(2,18,8) | RL'RL'RL'R'L'RL'RLRLRLR'LRL|
|(0,23,11)(1,21,9)(2,22,10) | RL'R'LR'L'R'L'R'L'RL'R'LR'L'R'L'R'L'|
|(3,17,10)(4,15,11)(5,16,9) | R'LRL'RLRLRLR'LRL'RLRLRL|
|(3,18,12)(4,19,13)(5,20,14) | R'LR'LR'LRLR'LR'L'R'L'R'L'RL'R'L'|
|(15,21,20)(16,22,18)(17,23,19) | R'LR'L'RLR'LRLR'L'RLRL'R'L'RL'|
|(6,12,10)(7,13,11)(8,14,9) | R'LR'L'R'LRLR'L'RLRL'RLR'L'RL'|
|(3,16,10)(4,17,11)(5,15,9) | R'L'R'LR'L'R'L'R'L'RL'RLRL'RL'RL'|
|(0,19,7)(1,20,8)(2,18,6) | R'L'R'L'R'L'RL'R'LR'L'R'L'R'L'RL'R'L|
|(0,8,16)(1,6,17)(2,7,15) | LRLRLRLRLR'LR'L'R'L'RLR'L'R'L'|
|(3,21,10)(4,22,11)(5,23,9) | LRLRLRLR'L'R'LR'L'R'L'RLRLR'L'|
|(0,7,23)(1,8,21)(2,6,22) | LRLRLRL'RL'RLR'L'R'L'RL'RL'RL'|
|(3,21,11)(4,22,9)(5,23,10) | LRLRLRL'R'L'R'L'R'LRLRLR'L'R'L|
|(12,18,17)(13,19,15)(14,20,16) | LRLRL'RL'RLRLR'L'RL'RL'R'L'RL'|
|(0,16,8)(1,17,6)(2,15,7) | LRLRL'R'LRLRL'RL'R'L'R'L'R'L'R'L'|
|(3,18,17)(4,19,15)(5,20,16) | LRLRL'R'L'R'LR'LR'L'RLRLR'LRL'|
|(3,12,17)(4,13,15)(5,14,16) | LRLR'LRL'RL'RL'RLRLRLRLR'L'|
|(3,21,13)(4,22,14)(5,23,12) | LRLR'LR'LRL'RLRL'RL'R'LR'LR'L'|
|(6,21,13)(7,22,14)(8,23,12) | LRLR'LR'LR'LRLRL'RL'R'LRL'R'L'|
|(9,22,14)(10,23,12)(11,21,13) | LRLR'LR'L'R'LR'LRLRL'RLRL'R'L'|
|(6,22,11)(7,23,9)(8,21,10) | LRLR'LR'L'R'L'RLR'LR'LRLRL'RL'|
|(3,12,6)(4,13,7)(5,14,8) | LRLR'L'RLR'LRL'R'L'RLR'LRLR'L'|
|(6,13,21)(7,14,22)(8,12,23) | LRLR'L'RLR'LR'L'R'L'RL'RL'RL'R'L'|
|(3,22,13)(4,23,14)(5,21,12) | LRLR'L'RL'R'LR'LRLR'LRL'RLR'L'|
|(9,14,22)(10,12,23)(11,13,21) | LRLR'L'R'LR'L'R'L'RL'RLRL'RL'R'L'|
|(6,21,14)(7,22,12)(8,23,13) | LRL'RLRL'RLRLR'LR'L'R'LR'LR'L'|
|(9,20,22)(10,18,23)(11,19,21) | LRL'RLRL'RL'RL'R'L'R'LR'L'R'L'R'L|
|(6,23,11)(7,21,9)(8,22,10) | LRL'RLRL'R'L'R'LR'LRLRLR'LR'L'|
|(6,14,21)(7,12,22)(8,13,23) | LRL'RL'RLRL'RL'R'L'R'LR'L'R'LR'L'|
|(3,13,21)(4,14,22)(5,12,23) | LRL'RL'RLR'LR'L'R'LR'L'RL'RL'R'L'|
|(3,10,21)(4,11,22)(5,9,23) | LRL'RL'RLR'L'R'LRL'RLR'L'R'L'R'L'|
|(6,11,23)(7,9,21)(8,10,22) | LRL'RL'R'L'R'L'RL'RLRLR'L'R'LR'L'|
|(3,15,19)(4,16,20)(5,17,18) | LRL'R'LR'LR'L'RLRLR'LR'LRL'RL'|
|(3,13,22)(4,14,23)(5,12,21) | LRL'R'LR'L'RL'R'L'RL'RLR'LRL'R'L'|
|(3,6,12)(4,7,13)(5,8,14) | LRL'R'L'RL'R'LRLR'L'RL'R'LRL'R'L'|
|(3,17,12)(4,15,13)(5,16,14) | LRL'R'L'R'L'R'L'R'LR'LR'LR'L'RL'R'L'|
|(6,12,23)(7,13,21)(8,14,22) | LR'LRLR'LR'LRLRL'RL'R'LR'L'RL'|
|(12,17,18)(13,15,19)(14,16,20) | LR'LRLR'LR'LRL'R'L'R'LR'LR'L'R'L'|
|(3,12,21)(4,13,22)(5,14,23) | LR'LRLR'LR'LR'LRLRLRLR'LRL'|
|(0,12,20)(1,13,18)(2,14,19) | LR'LRLR'LR'L'R'LR'L'RL'RLRLRL'|
|(6,23,12)(7,21,13)(8,22,14) | LR'LRL'RLR'LR'L'R'L'RL'RL'R'L'RL'|
|(0,14,8)(1,12,6)(2,13,7) | LR'LRL'R'L'RL'RLR'LRLR'LR'LRL'|
|(0,23,7)(1,21,8)(2,22,6) | LR'LR'LR'LRLRL'R'LR'LR'L'R'L'R'L'|
|(3,22,16)(4,23,17)(5,21,15) | LR'LR'LR'LR'LRLRL'RL'R'LRL'RL'|
|(3,15,7)(4,16,8)(5,17,6) | LR'LR'LR'L'RL'RLRLR'LR'LRL'RL'|
|(3,16,22)(4,17,23)(5,15,21) | LR'LR'L'RLR'LR'L'R'L'RL'RL'RL'RL'|
|(3,19,15)(4,20,16)(5,18,17) | LR'LR'L'RL'RL'R'L'R'LRL'RL'RLR'L'|
|(3,7,15)(4,8,16)(5,6,17) | LR'LR'L'RL'RL'R'L'R'LR'LRL'RL'RL'|
|(6,11,22)(7,9,23)(8,10,21) | LR'LR'L'R'L'RL'RL'R'LRLRL'RL'R'L'|
|(9,14,16)(10,12,17)(11,13,15) | LR'L'RLRL'R'LR'L'RL'R'L'R'L'RL'RL|
|(0,8,14)(1,6,12)(2,7,13) | LR'L'RL'RL'R'L'RL'R'LR'LRLR'L'RL'|
|(3,17,18)(4,15,19)(5,16,20) | LR'L'RL'R'L'R'LRL'RL'RLRLR'L'R'L'|
|(3,21,12)(4,22,13)(5,23,14) | LR'L'RL'R'L'R'L'R'L'RL'RL'RL'R'L'RL'|
|(12,18,15)(13,19,16)(14,20,17) | LR'L'R'LRLRLR'LR'L'R'L'RLRLRL'|
|(0,10,16)(1,11,17)(2,9,15) | LR'L'R'LRLRLR'L'R'L'R'L'RLRLRL|
|(9,20,15)(10,18,16)(11,19,17) | LR'L'R'LR'L'R'LR'LRLRL'RLRLRL'|
|(6,14,15)(7,12,16)(8,13,17) | LR'L'R'LR'L'R'L'R'L'RL'RLRL'RL'RL|
|(12,15,18)(13,16,19)(14,17,20) | LR'L'R'L'R'LRLRL'RL'R'L'R'L'RLRL'|
|(0,20,12)(1,18,13)(2,19,14) | LR'L'R'L'R'LR'LRL'RLRL'RL'R'L'RL'|
|(9,15,20)(10,16,18)(11,17,19) | LR'L'R'L'R'LR'L'R'L'RL'RLRL'RLRL'|
|(9,22,20)(10,23,18)(11,21,19) | L'RLRLRL'RLRLR'LR'LR'L'R'LR'L'|
|(3,20,6)(4,18,7)(5,19,8) | L'RLRLRL'RL'R'LR'L'R'LR'LRLR'L|
|(6,22,19)(7,23,20)(8,21,18) | L'RLRLRL'R'L'R'LR'LRLRLR'L'R'L|
|(3,11,21)(4,9,22)(5,10,23) | L'RLRL'R'L'R'L'RLRLRLR'L'R'L'R'L'|
|(6,19,22)(7,20,23)(8,18,21) | L'RLRL'R'L'R'L'RL'RLRLR'L'R'L'R'L|
|(0,16,6)(1,17,7)(2,15,8) | L'RLR'LRLRLRLR'LR'LR'LRLR'L|
|(0,23,19)(1,21,20)(2,22,18) | L'RLR'LRLRL'R'LR'LR'L'R'L'RLRL|
|(0,12,6)(1,13,7)(2,14,8) | L'RLR'LRLR'LRLR'L'RL'R'L'RLR'L|
|(3,13,7)(4,14,8)(5,12,6) | L'RLR'LR'LRLR'LRL'RL'R'L'RLR'L|
|(9,17,14)(10,15,12)(11,16,13) | L'RL'RLRLR'LR'LRL'R'L'R'LR'LRL|
|(0,14,22)(1,12,23)(2,13,21) | L'RL'RLR'LR'LRLRL'RL'R'LR'LR'L|
|(0,18,22)(1,19,23)(2,20,21) | L'RL'RLR'LR'LRLRL'R'LR'LR'L'RL|
|(0,21,15)(1,22,16)(2,23,17) | L'RL'RLR'L'RL'RLRLR'LR'LR'LR'L|
|(0,22,14)(1,23,12)(2,21,13) | L'RL'RL'RLR'LR'L'R'L'RL'RL'R'LR'L|
|(0,15,21)(1,16,22)(2,17,23) | L'RL'RL'RL'RL'R'L'R'LR'LRL'R'LR'L|
|(3,17,14)(4,15,12)(5,16,13) | L'RL'RL'RL'R'L'R'LRL'RL'RLRLRL|
|(0,6,12)(1,7,13)(2,8,14) | L'RL'R'LRLR'LRL'R'L'RL'R'L'RL'R'L|
|(3,7,13)(4,8,14)(5,6,12) | L'RL'R'LRLR'LR'L'RL'R'L'RL'RL'R'L|
|(6,12,17)(7,13,15)(8,14,16) | L'RL'R'LR'L'RL'RLRLR'LR'LRLR'L|
|(3,6,20)(4,7,18)(5,8,19) | L'RL'R'L'RL'RLRL'RLR'LR'L'R'L'R'L|
|(0,6,16)(1,7,17)(2,8,15) | L'RL'R'L'RL'RL'RL'R'L'R'L'R'L'RL'R'L|
|(6,23,19)(7,21,20)(8,22,18) | L'RL'R'L'RL'RL'R'LRLRL'RL'RLRL|
|(6,17,12)(7,15,13)(8,16,14) | L'RL'R'L'RL'RL'R'L'R'LR'LRL'RLR'L|
|(0,23,6)(1,21,7)(2,22,8) | L'R'LRLRLRLRL'RL'RL'RLR'LRL|
|(0,11,16)(1,9,17)(2,10,15) | L'R'LRLRL'RLRL'R'LR'L'R'LR'LRL|
|(0,8,15)(1,6,16)(2,7,17) | L'R'LRL'RLR'LRLR'LR'L'RL'R'LRL|
|(0,22,18)(1,23,19)(2,21,20) | L'R'LRL'RL'RLR'L'R'L'RL'RL'R'LR'L|
|(9,16,14)(10,17,12)(11,15,13) | L'R'LR'LRLRLR'LRL'RLR'L'R'LRL'|
|(6,15,14)(7,16,12)(8,17,13) | L'R'LR'LR'L'R'LR'LRLRLRL'RLRL'|
|(6,17,11)(7,15,9)(8,16,10) | L'R'L'RLRL'RLRLR'LR'L'R'LR'LRL|
|(0,15,8)(1,16,6)(2,17,7) | L'R'L'RLR'LRL'RL'R'L'RL'R'LR'L'RL|
|(6,17,13)(7,15,14)(8,16,12) | L'R'L'RLR'L'RL'RLRLR'LR'LR'LRL|
|(9,14,17)(10,12,15)(11,13,16) | L'R'L'RL'RLRLR'L'RL'RL'R'L'R'LR'L|
|(0,16,11)(1,17,9)(2,15,10) | L'R'L'RL'RLRL'RLR'L'R'LR'L'R'L'RL|
|(6,11,17)(7,9,15)(8,10,16) | L'R'L'RL'RLRL'RL'R'L'R'LR'L'R'LRL|
|(6,13,17)(7,14,15)(8,12,16) | L'R'L'RL'RL'RL'R'L'R'LR'LRL'R'LRL|
|(0,6,23)(1,7,21)(2,8,22) | L'R'L'RL'R'LR'LR'LR'L'R'L'R'L'R'L'RL|
|(0,19,23)(1,20,21)(2,18,22) | L'R'L'R'LRLRL'RL'RLR'L'R'L'RL'R'L|
|(6,19,23)(7,20,21)(8,18,22) | L'R'L'R'LR'LR'L'R'L'RLR'LR'LRLR'L|
|(0,16,10)(1,17,11)(2,15,9) | L'R'L'R'L'R'LRLRLRL'R'L'R'L'RLRL'|
|(3,14,17)(4,12,15)(5,13,16) | L'R'L'R'L'R'LR'LR'L'RLRLR'LR'LR'L|
|(0,14,19)(1,12,20)(2,13,18) | RLRLRLRL'R'L'RL'R'L'R'LRLRL'R'|
|(0,14,20)(1,12,18)(2,13,19) | RLRLRLR'L'R'L'R'L'RLRLRL'R'L'R|
|(6,23,10)(7,21,11)(8,22,9) | RLRLR'LR'LRLRL'R'LR'LR'L'R'LR'|
|(0,11,7)(1,9,8)(2,10,6) | RLRLR'L'R'L'RL'RL'R'LRLRL'RLR'|
|(0,21,7)(1,22,8)(2,23,6) | RLRL'RLR'LR'LR'LRLRLRLRL'R'|
|(12,23,16)(13,21,17)(14,22,15) | RLRL'RL'RL'RLRLR'LR'L'RLR'L'R'|
|(12,23,18)(13,21,19)(14,22,20) | RLRL'RL'R'L'RL'RLRLR'LRLR'L'R'|
|(12,20,15)(13,18,16)(14,19,17) | RLRL'RL'R'L'R'LRL'RL'RLRLR'LR'|
|(12,16,23)(13,17,21)(14,15,22) | RLRL'R'LRL'RL'R'L'R'LR'LR'LR'L'R'|
|(0,12,22)(1,13,23)(2,14,21) | RLRL'R'LR'L'RL'RLRL'RLR'LRL'R'|
|(12,18,23)(13,19,21)(14,20,22) | RLRL'R'L'RL'R'L'R'LR'LRLR'LR'L'R'|
|(12,21,16)(13,22,17)(14,23,15) | RLR'LRLR'LRLRL'RL'R'L'RL'RL'R'|
|(9,14,20)(10,12,18)(11,13,19) | RLR'LRLR'LR'LR'L'R'L'RL'R'L'R'L'R|
|(12,19,17)(13,20,15)(14,18,16) | RLR'LRLR'L'R'L'RL'RLRLRL'RL'R'|
|(12,16,21)(13,17,22)(14,15,23) | RLR'LR'LRLR'LR'L'R'L'RL'R'L'RL'R'|
|(0,19,14)(1,20,12)(2,18,13) | RLR'LR'LRL'R'L'RLR'LRL'R'L'R'L'R'|
|(12,17,19)(13,15,20)(14,16,18) | RLR'LR'L'R'L'R'LR'LRLRL'R'L'RL'R'|
|(0,8,9)(1,6,10)(2,7,11) | RLR'L'RL'RL'R'LRLRL'RL'RLR'LR'|
|(0,22,12)(1,23,13)(2,21,14) | RLR'L'RL'R'LR'L'R'LR'LRL'RLR'L'R'|
|(0,7,21)(1,8,22)(2,6,23) | RLR'L'R'L'R'L'R'L'RL'RL'RL'R'LR'L'R'|
|(12,17,23)(13,15,21)(14,16,22) | RL'RLRL'RL'RLRLR'LR'L'RL'R'LR'|
|(6,10,23)(7,11,21)(8,9,22) | RL'RLRL'RL'RLR'L'R'L'RL'RL'R'L'R'|
|(0,21,14)(1,22,12)(2,23,13) | RL'RLRL'RL'RL'RLRLRLRL'RLR'|
|(3,22,11)(4,23,9)(5,21,10) | RL'RLRL'RL'R'L'RL'R'LR'LRLRLR'|
|(12,23,17)(13,21,15)(14,22,16) | RL'RLR'LRL'RL'R'L'R'LR'LR'L'R'LR'|
|(3,21,15)(4,22,16)(5,23,17) | RL'RLR'L'R'LR'LRL'RLRL'RL'RLR'|
|(0,9,8)(1,10,6)(2,11,7) | RL'RL'R'LR'LR'L'R'L'RLR'LR'LRL'R'|
|(12,15,20)(13,16,18)(14,17,19) | RL'RL'R'L'R'LR'LR'L'RLRLR'LR'L'R'|
|(6,18,23)(7,19,21)(8,20,22) | RL'R'LRLR'L'RL'R'LR'L'R'L'R'LR'LR|
|(3,15,21)(4,16,22)(5,17,23) | RL'R'LR'LR'L'R'LR'L'RL'RLRL'R'LR'|
|(0,7,11)(1,8,9)(2,6,10) | RL'R'LR'L'R'L'RLR'LR'LRLRL'R'L'R'|
|(0,14,21)(1,12,22)(2,13,23) | RL'R'LR'L'R'L'R'L'R'LR'LR'LR'L'R'LR'|
|(6,22,9)(7,23,10)(8,21,11) | RL'R'L'RLRLRL'RL'R'L'R'LRLRLR'|
|(3,20,7)(4,18,8)(5,19,6) | RL'R'L'RLRLRL'R'L'R'L'R'LRLRLR|
|(6,19,11)(7,20,9)(8,18,10) | RL'R'L'RL'R'L'RL'RLRLR'LRLRLR'|
|(6,16,21)(7,17,22)(8,15,23) | RL'R'L'RL'R'L'R'L'R'LR'LRLR'LR'LR|
|(6,9,22)(7,10,23)(8,11,21) | RL'R'L'R'L'RLRLR'LR'L'R'L'R'LRLR'|
|(3,11,22)(4,9,23)(5,10,21) | RL'R'L'R'L'RL'RLR'LRLR'LR'L'R'LR'|
|(6,11,19)(7,9,20)(8,10,18) | RL'R'L'R'L'RL'R'L'R'LR'LRLR'LRLR'|
|(9,20,14)(10,18,12)(11,19,13) | R'LRLRLR'LRLRL'RL'RL'R'L'RL'R'|
|(0,10,15)(1,11,16)(2,9,17) | R'LRLRLR'LR'L'RL'R'L'RL'RLRL'R|
|(9,15,12)(10,16,13)(11,17,14) | R'LRLRLR'L'R'L'RL'RLRLRL'R'L'R|
|(0,20,14)(1,18,12)(2,19,13) | R'LRLR'L'R'L'R'LRLRLRL'R'L'R'L'R'|
|(9,12,15)(10,13,16)(11,14,17) | R'LRLR'L'R'L'R'LR'LRLRL'R'L'R'L'R|
|(3,7,16)(4,8,17)(5,6,15) | R'LRL'RLRLRLRL'RL'RL'RLRL'R|
|(3,14,10)(4,12,11)(5,13,9) | R'LRL'RLRLR'L'RL'RL'R'L'R'LRLR|
|(0,22,16)(1,23,17)(2,21,15) | R'LRL'RL'RLRL'RLR'LR'L'R'LRL'R|
|(6,22,20)(7,23,18)(8,21,19) | R'LR'LRLRL'RL'RLR'L'R'L'RL'RLR|
|(3,9,13)(4,10,14)(5,11,12) | R'LR'LRL'RL'RLRLR'L'RL'RL'R'LR|
|(0,16,22)(1,17,23)(2,15,21) | R'LR'L'RLRL'RL'R'LR'L'R'LR'LR'L'R|
|(6,17,23)(7,15,21)(8,16,22) | R'LR'L'RL'R'LR'LRLRL'RL'RLRL'R|
|(0,15,10)(1,16,11)(2,17,9) | R'LR'L'R'LR'LRLR'LRL'RL'R'L'R'L'R|
|(3,16,7)(4,17,8)(5,15,6) | R'LR'L'R'LR'LR'LR'L'R'L'R'L'R'LR'L'R|
|(9,15,13)(10,16,14)(11,17,12) | R'LR'L'R'LR'LR'L'RLRLR'LR'LRLR|
|(6,23,17)(7,21,15)(8,22,16) | R'LR'L'R'LR'LR'L'R'L'RL'RLR'LRL'R|
|(3,14,16)(4,12,17)(5,13,15) | R'L'RLRLRLRLR'LR'LR'LRL'RLR|
|(3,18,7)(4,19,8)(5,20,6) | R'L'RLRLR'LRLR'L'RL'R'L'RL'RLR|
|(3,15,6)(4,16,7)(5,17,8) | R'L'RLR'LRL'RLRL'RL'R'LR'L'RLR|
|(3,13,9)(4,14,10)(5,12,11) | R'L'RLR'LR'LRL'R'L'R'LR'LR'L'RL'R|
|(6,23,18)(7,21,19)(8,22,20) | R'L'RL'RLRLRL'RLR'LRL'R'L'RLR'|
|(6,21,16)(7,22,17)(8,23,15) | R'L'RL'RL'R'L'RL'RLRLRLR'LRLR'|
|(6,19,17)(7,20,15)(8,18,16) | R'L'R'LRLR'LRLRL'RL'R'L'RL'RLR|
|(3,6,15)(4,7,16)(5,8,17) | R'L'R'LRL'RLR'LR'L'R'LR'L'RL'R'LR|
|(6,21,17)(7,22,15)(8,23,16) | R'L'R'LRL'R'LR'LRLRL'RL'RL'RLR|
|(6,20,22)(7,18,23)(8,19,21) | R'L'R'LR'LRLRL'R'LR'LR'L'R'L'RL'R|
|(3,7,18)(4,8,19)(5,6,20) | R'L'R'LR'LRLR'LRL'R'L'RL'R'L'R'LR|
|(6,17,19)(7,15,20)(8,16,18) | R'L'R'LR'LRLR'LR'L'R'L'RL'R'L'RLR|
|(6,17,21)(7,15,22)(8,16,23) | R'L'R'LR'LR'LR'L'R'L'RL'RLR'L'RLR|
|(3,16,14)(4,17,12)(5,15,13) | R'L'R'LR'L'RL'RL'RL'R'L'R'L'R'L'R'LR|
|(3,10,14)(4,11,12)(5,9,13) | R'L'R'L'RLRLR'LR'LRL'R'L'R'LR'L'R|
|(9,13,15)(10,14,16)(11,12,17) | R'L'R'L'RL'RL'R'L'R'LRL'RL'RLRL'R|
|(3,7,20)(4,8,18)(5,6,19) | R'L'R'L'R'L'RLRLRLR'L'R'L'R'LRLR'|
|(0,21,8)(1,22,6)(2,23,7) | LRLRL'R'LRLRL'RL'RLR'LR'LRL'R'|
|(3,10,22)(4,11,23)(5,9,21) | LRLRL'R'L'RLRL'R'LR'L'R'LR'LR'LR'|
|(0,3,10)(1,4,11)(2,5,9) | LRL'RLRL'R'LR'LR'LR'L'R'LR'LRL'R'|
|(0,5,19)(1,3,20)(2,4,18) | LRL'R'LR'LRLR'LR'LR'LRL'R'L'RL'R'|
|(3,15,12)(4,16,13)(5,17,14) | LRL'R'LR'LR'L'RL'RL'R'L'R'LRL'R'L'R'|
|(3,10,13)(4,11,14)(5,9,12) | LR'LRLR'LRL'RLRLRL'RL'R'LR'L'R|
|(9,12,20)(10,13,18)(11,14,19) | LR'LRLR'LR'L'R'LR'LR'L'R'L'RLRL'R|
|(9,17,20)(10,15,18)(11,16,19) | LR'LRLR'L'R'L'RL'RL'R'L'RL'RLRL'R|
|(0,12,19)(1,13,20)(2,14,18) | LR'LR'LR'LRLR'LRL'R'L'RLRL'R'L'R'|
|(3,15,18)(4,16,19)(5,17,20) | LR'L'RL'R'LR'LRLRLR'LRL'RLRL'R|
|(0,15,12)(1,16,13)(2,17,14) | LR'L'R'LRLRLR'LRL'RLRLRL'R'L'R|
|(0,11,3)(1,9,4)(2,10,5) | LR'L'R'LR'L'RL'RL'RL'RLRL'RL'R'LR|
|(3,22,6)(4,23,7)(5,21,8) | L'RLRL'R'L'R'L'RL'R'LR'L'R'L'R'LRLR'|
|(0,22,19)(1,23,20)(2,21,18) | L'RLR'LRL'RL'R'L'R'L'RL'R'LR'L'R'LR'|
|(3,6,18)(4,7,19)(5,8,20) | L'RL'RLRLRL'R'L'RL'R'L'R'L'R'LRLR|
|(9,23,20)(10,21,18)(11,22,19) | L'RL'R'L'RLRLR'LR'LRLR'LR'L'R'LR'|
|(6,20,9)(7,18,10)(8,19,11) | L'RL'R'L'RL'RLRL'RL'RLRLR'L'R'LR'|
|(0,11,8)(1,9,6)(2,10,7) | L'RL'R'L'RL'R'LR'L'R'L'R'LR'LRL'RLR'|
|(0,22,6)(1,23,7)(2,21,8) | L'R'LRL'RL'RLR'LR'LRLRL'R'LRLR|
|(0,20,5)(1,18,3)(2,19,4) | L'R'LRL'RL'R'L'RL'RLRLR'L'RL'R'LR|
|(0,11,15)(1,9,16)(2,10,17) | L'R'L'R'LRLRLRL'RLRL'R'L'R'L'RL'R|
|(3,16,13)(4,17,14)(5,15,12) | L'R'L'R'LRL'R'L'R'LR'LR'L'RL'RL'R'LR|
|(3,12,15)(4,13,16)(5,14,17) | RLRLR'L'RLRLR'LR'LRL'RL'RLR'L'|
|(0,19,12)(1,20,13)(2,18,14) | RLRLR'L'R'LRLR'L'RL'R'L'RL'RL'RL'|
|(0,19,5)(1,20,3)(2,18,4) | RLR'LRLR'L'RL'RL'RL'R'L'RL'RLR'L'|
|(0,10,3)(1,11,4)(2,9,5) | RLR'L'RL'RLRL'RL'RL'RLR'L'R'LR'L'|
|(0,8,21)(1,6,22)(2,7,23) | RLR'L'RL'RL'R'LR'LR'L'R'L'RLR'L'R'L'|
|(0,19,22)(1,20,23)(2,18,21) | RL'RLRL'RLR'LRLRLR'LR'L'RL'R'L|
|(9,20,23)(10,18,21)(11,19,22) | RL'RLRL'RL'R'L'RL'RL'R'L'R'LRLR'L|
|(6,9,20)(7,10,18)(8,11,19) | RL'RLRL'R'L'R'LR'LR'L'R'LR'LRLR'L|
|(3,22,10)(4,23,11)(5,21,9) | RL'RL'RL'RLRL'RLR'L'R'LRLR'L'R'L'|
|(0,8,11)(1,6,9)(2,7,10) | RL'R'LR'L'RL'RLRLRL'RLR'LRLR'L|
|(3,6,22)(4,7,23)(5,8,21) | RL'R'L'RLRLRL'RLR'LRLRLR'L'R'L|
|(0,5,20)(1,3,18)(2,4,19) | RL'R'L'RL'R'LR'LR'LR'LRLR'LR'L'RL|
|(0,12,15)(1,13,16)(2,14,17) | R'LRLR'L'R'L'R'LR'L'RL'R'L'R'L'RLRL'|
|(3,13,10)(4,14,11)(5,12,9) | R'LRL'RLR'LR'L'R'L'R'LR'L'RL'R'L'RL'|
|(0,15,11)(1,16,9)(2,17,10) | R'LR'LRLRLR'L'R'LR'L'R'L'R'L'RLRL|
|(9,20,12)(10,18,13)(11,19,14) | R'LR'L'R'LRLRL'RL'RLRL'RL'R'L'RL'|
|(9,20,17)(10,18,15)(11,19,16) | R'LR'L'R'LR'LRLR'LR'LRLRL'R'L'RL'|
|(3,18,15)(4,19,16)(5,20,17) | R'LR'L'R'LR'L'RL'R'L'R'L'RL'RLR'LRL'|
|(3,13,16)(4,14,17)(5,12,15) | R'L'RLR'LR'LRL'RL'RLRLR'L'RLRL|
|(0,3,11)(1,4,9)(2,5,10) | R'L'RLR'LR'L'R'LR'LRLRL'R'LR'L'RL|
|(3,18,6)(4,19,7)(5,20,8) | R'L'R'L'RLRLRLR'LRLR'L'R'L'R'LR'L|
|(0,6,22)(1,7,23)(2,8,21) | R'L'R'L'RLR'L'R'L'RL'RL'R'LR'LR'L'RL|
|(0,12,16)(1,13,17)(2,14,15) | LRLRLRLR'LRLRL'R'L'RL'R'L'RL'R'L'|
|(0,20,15)(1,18,16)(2,19,17) | LRLRLRL'RL'RLRLR'LR'L'RL'R'L'R'L'|
|(0,18,9)(1,19,10)(2,20,11) | LRLRLRL'RL'RL'RLRLRLRL'RL'R'L'|
|(3,23,12)(4,21,13)(5,22,14) | LRLRLRL'R'L'RLR'LRL'R'L'RLRL'R'L'|
|(3,14,23)(4,12,21)(5,13,22) | LRLRLRL'R'L'R'L'R'LR'L'R'L'R'LRLR'L'|
|(0,15,20)(1,16,18)(2,17,19) | LRLRLR'LRL'RL'R'L'R'LR'LR'L'R'L'R'L'|
|(3,16,9)(4,17,10)(5,15,11) | LRLRL'RLR'LRL'R'L'RLR'LRL'RL'R'L'|
|(6,9,19)(7,10,20)(8,11,18) | LRLRL'RL'RL'RLRLR'LR'L'RLR'L'R'L'|
|(6,17,10)(7,15,11)(8,16,9) | LRLRL'RL'R'LR'LRLRL'RL'RLR'L'R'L'|
|(6,19,9)(7,20,10)(8,18,11) | LRLRL'R'LRL'RL'R'L'R'LR'LR'LR'L'R'L'|
|(6,10,17)(7,11,15)(8,9,16) | LRLRL'R'LR'LR'L'R'L'RL'RLR'LR'L'R'L'|
|(0,16,12)(1,17,13)(2,15,14) | LRLR'LRLR'LRLR'L'R'L'RL'R'L'R'L'R'L'|
|(3,9,16)(4,10,17)(5,11,15) | LRLR'LRL'R'L'RLRL'RL'R'L'RLRLR'L'|
|(0,9,18)(1,10,19)(2,11,20) | LRLR'LR'L'R'L'R'L'R'LR'LR'LR'L'R'L'R'L'|
|(0,23,8)(1,21,6)(2,22,7) | LRLR'L'RLRLRL'RL'R'L'R'LRLRL'R'L'|
|(3,12,23)(4,13,21)(5,14,22) | LRLR'L'R'LRLR'L'RL'R'LRLR'L'R'L'R'L'|
|(3,14,22)(4,12,23)(5,13,21) | LRLR'L'R'LRLR'L'R'L'R'L'RLRL'R'LR'L'|
|(0,8,23)(1,6,21)(2,7,22) | LRLR'L'R'L'RLRLR'LR'L'R'L'R'LRL'R'L'|
|(0,22,10)(1,23,11)(2,21,9) | LRL'RLRL'R'LR'LRLRL'RL'RL'R'LR'L'|
|(3,10,8)(4,11,6)(5,9,7) | LRL'RLRL'R'L'R'L'R'L'RL'RL'RL'RLR'L'|
|(0,10,22)(1,11,23)(2,9,21) | LRL'RLR'LR'LR'L'R'L'RL'RLR'L'R'LR'L'|
|(9,21,15)(10,22,16)(11,23,17) | LRL'RLR'L'RLRL'R'LR'L'RLRL'RLR'L'|
|(3,22,14)(4,23,12)(5,21,13) | LRL'RLR'L'R'LRLRLRL'R'L'RLRL'R'L'|
|(6,14,20)(7,12,18)(8,13,19) | LRL'RL'RLRLRL'RL'R'L'R'LRLR'LR'L'|
|(6,9,18)(7,10,19)(8,11,20) | LRL'RL'RLR'L'R'L'RL'RLRLRLR'LR'L'|
|(6,20,14)(7,18,12)(8,19,13) | LRL'RL'R'L'RLRLR'LR'L'R'L'R'LR'LR'L'|
|(6,18,9)(7,19,10)(8,20,11) | LRL'RL'R'L'R'L'R'LR'LRLRL'R'LR'LR'L'|
|(3,23,10)(4,21,11)(5,22,9) | LRL'R'LR'LRLR'L'R'L'R'L'RL'RLR'L'R'L|
|(3,8,10)(4,6,11)(5,7,9) | LRL'R'LR'LR'LR'LRLRLRLR'L'R'LR'L'|
|(3,13,19)(4,14,20)(5,12,18) | LRL'R'LR'L'RL'R'L'R'L'R'LR'L'RL'R'L'RL'|
|(9,15,21)(10,16,22)(11,17,23) | LRL'R'LR'L'R'LRL'RLR'L'R'LRL'R'LR'L'|
|(3,23,14)(4,21,12)(5,22,13) | LRL'R'L'RLRLRL'RLRLRLR'L'R'L'R'L'|
|(0,7,12)(1,8,13)(2,6,14) | LR'LRLRLR'LRLR'L'RL'R'L'RL'RL'RL'|
|(3,12,19)(4,13,20)(5,14,18) | LR'LRLRL'RL'RL'RLRLRLRL'RL'RL'|
|(3,6,14)(4,7,12)(5,8,13) | LR'LRLRL'R'LR'LRL'RLRLRLR'LRL'|
|(3,9,7)(4,10,8)(5,11,6) | LR'LRLRL'R'L'RLR'LRL'R'L'RLRL'RL'|
|(3,19,13)(4,20,14)(5,18,12) | LR'LRLR'LRL'RLRLRLR'LRL'RLR'L'|
|(0,5,11)(1,3,9)(2,4,10) | LR'LRLR'L'RL'R'LR'L'R'L'RL'R'LR'LRL'|
|(3,8,13)(4,6,14)(5,7,12) | LR'LRL'RL'R'LR'LRLR'LRL'RL'RL'RL'|
|(3,6,21)(4,7,22)(5,8,23) | LR'LRL'RL'R'L'RL'RLRLR'LRLR'L'RL'|

### Triangle triangle pure corner orientation algorithms

(see data model above for starting position and label meanings)

| Effect | Alg |
|-------|--------|
|(12,14,13)(21,22,23) | LRLR'L'R'LRLR'L'R'LRLR'L'R'|
|(6,7,8)(15,17,16) | L'R'L'RLRL'R'L'RLRL'R'L'RLR|
|(12,13,14)(21,23,22) | RLRL'R'L'RLRL'R'L'RLRL'R'L'|
|(6,8,7)(15,16,17) | R'L'R'LRLR'L'R'LRLR'L'R'LRL|
|(9,11,10)(18,19,20) | LR'LRLR'L'R'LRLR'L'R'LRLR'L|
|(9,10,11)(18,20,19) | L'RL'R'L'RLRL'R'L'RLRL'R'L'RL'|
|(3,4,5)(9,11,10) | LRL'R'LR'L'R'L'RLRL'R'L'RLRLR'|
|(0,2,1)(18,19,20) | LR'L'R'L'RLRL'R'L'RLRLR'LRL'R'|
|(3,4,5)(18,20,19) | L'RLRLR'L'R'LRLR'L'R'L'RL'R'LR|
|(0,2,1)(9,10,11) | L'R'LRL'RLRLR'L'R'LRLR'L'R'L'R|
|(0,1,2)(18,20,19) | RLR'L'RL'R'L'R'LRLR'L'R'LRLRL'|
|(3,5,4)(9,10,11) | RL'R'L'R'LRLR'L'R'LRLRL'RLR'L'|
|(0,1,2)(9,11,10) | R'LRLRL'R'L'RLRL'R'L'R'LR'L'RL|
|(3,5,4)(18,19,20) | R'L'RLR'LRLRL'R'L'RLRL'R'L'R'L|
|(3,5,4)(21,22,23) | LRLRLRL'R'L'RLRL'R'L'RLRL'R'LR'L'|
|(3,4,5)(21,23,22) | LRL'RLR'L'R'LRLR'L'R'LRLR'L'R'L'R'L'|
