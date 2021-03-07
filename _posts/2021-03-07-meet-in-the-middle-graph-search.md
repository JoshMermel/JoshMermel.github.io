---
layout: post
title: Meet in the Middle Graph Search
---

Background
==========

I've been working on a puzzle game for Android called [Looping
Dice](https://play.google.com/store/apps/details?id=com.joshmermelstein.loopingdice).
It's all about sliding squares around a grid try to match a pattern while using
the fewest number of moves.

As part of creating levels for the game - I wanted to create a solver. This was
useful for double checking that levels were solvable and also for setting the
threshold where I give the use three stars on each level.

Breadth First Search
====================

My first idea was to model this problem as a graph search. We treat each state
that the puzzle can be in as a node - and think of each valid move as an edge.
Then, given a start and end state, we can implement breadth first search as
follow:

    create an empty queue of puzzle states for things we need to explore
    create an empty set of puzzle states so we know what we've explored already
    insert the initial state into both the queue and the set
    while (the queue is not empty){
       compute neighbors of the state at the front of the queue
       check if any of the neighbors are the goal state
       add each neighbor to the set of seen states
       if any neighbors are new to us, also add them to the queue to explore later
    }

I think people who read my blog are probably familiar with BFS but let's take a
second to review why this works. Using the queue to order which nodes get
explored means that this algorithm explores all nodes at distance N from the
start before it visits any nodes of distance N+1. This also means that the first
time we encounter the goal node - we are guaranteed to have done so by a
shortest path.

The downside here is very similar to the upside - we always visit all nodes
distance N from the state before we visit any at distance N+1. When the distance
from the start to the end is large and when the branching factor from each node
is large - this can be a huge number of nodes to explore before we find the
goal.

The puzzle that motivated me to consider a better approach had 16 neighbors from
each state and had an optimal solve length of 8 moves. It took my unoptimized
BFS solver around 90 seconds to solve.

The idea of Meet in the Middle search
=====================================

When humans try to solve puzzles like mine or like a [15
puzzle](https://en.wikipedia.org/wiki/15_puzzle), we use heuristics to get
closer to the goal, even if we don't have a complete plan for how to finish the
solve. To phrase it differently - we have an idea of what states might be near
the end, and instead of aiming for the end directly, we aim for that cloud of
nearby states.

I decided implement this using an approach I'd heard of but never seen in use
called "Meet in the Middle" (MITM). I would start exploring from both the start
and the end, and declare victory when either side found a state that the other
one had previously found. Much like the BFS above, the first time these
flood-fill searches encounter each other ought to be a shortest path, since both
are exploring length N paths before starting on length N+1 paths. 

First Implementation
====================

Skipping over some details again, the main part of my algorithm looked like
this:

    create two queues of states to explore
    create two a set for states seen by the forward exploration and another for backward
    while (either queue is nonempty) {
      explore the state at the front of the forward queue as in BFS
      when you look at new nodes, also check if they are in the set of states seen by the backward exploration
    
      explore the state at the front of the backward queue as in BFS
      when you look at new nodes, also check if they are in the set of states seen by the forward exploration
    }

I tried this new approach on the puzzle above and was shocked to see that it
took 0.01 seconds to find an 8 move solution. This was different from the one
found by the BFS implementation but it was also correct. I added some logging to
both implementations to see how many states each one explored before it arrived
at a solution. The BFS approach's set saw 414764 distinct states before it
arrived at the goal. In contrast the two sets used in the MITM approach had seen
2934 distinct states and 3069 distinct states before they found a shared state.
No wonder it was so much faster!

## Discovering a bug

While making some levels, I found a case where the solver claimed the shortest
solution was in 6 moves - but I'd already found a 5 move solution on my own.
What was going on?

Recall that in BFS, we guarantee that the first path we find is the shortest by
showing that "if there was a shorter path, we would have found it earlier".

Also note that in MITM, when the forward and backward explorations find their
first shared state, it is necessarily on the "frontier" of the both of their
BFS's. By the same logic as above - if it wasn't then we would have found it
earlier.

Finally, consider this reframing of MITM: each time we discover a new node
(let's say distance N), and we compare it to a node on the other BFS's
"frontier" (let's say distance M), we're effectively checking if a particular
solution of length N+M is valid. If the two nodes are equal, the solution is
valid, and if they are different, then the solution doesn't work. (In practice,
this comparison is set-containment so it's O(1) to compare against the whole
frontier but it's useful to imagine that we're comparing new nodes against
existing ones one-by-one.)

So in this bug, both forward and backward search had finished with all distance 2
nodes and were both starting to discover new nodes at distance 3 from the
start/end. Each time one of these nodes was discovered, it was checked against
the "frontier" of the other BFS.

Each comparison was checking for the existence of some length 5 solutions
(self-length of 3, other-length of 2) and length 6 solutions (self-length and
other-length both 3). That's bad because it means we are looking at some length
6 candidate solutions before we've considered all length 5 solutions. This
throws away the shortest-path guarantee of BFS!

This doesn't happen for all searches but that's exactly what happened in this
case. Because of the arbitrary order that nodes are discovered, a length 6
solution showed up before we had the chance to see all length 5 candidates and
note that one is correct.

Second Implementation
=====================

Once I understood the bug, the fix wasn't conceptually hard. I had to keep the
"frontier" of each BFS at a constant distance while I discovered nodes in the
other one. After the refactor, the code was structured like this:


    create two queues of states to explore
    create two a set for states seen by the forward exploration and another for backward
    target_depth = 1
    while (either queue is nonempty) {
      explore forward as in BFS until you've seen all nodes distance target_depth from the start
      when you look at new nodes, also check if they are in the set of states seen by the backward exploration

      explore backward as in BFS until you've seen all nodes distance target_depth from the end
      when you look at new nodes, also check if they are in the set of states seen by the forward exploration

      increment target_length
    }

Future Work
===========

In my last implementation, I always alternate between forward and backward
exploration. I suspect there might be some slight efficiency gains by using some
heuristic (queue-length?) to decide which one to explore on each cycle of the
outer loop. Doing so without benchmarks felt like premature optimization and at
this point, the solver is fast enough that I don't think I care. Maybe at some
point I'll need it to be even faster than it is today and look into that among
other optimization ideas.

Closing Thoughts
================

I ran my revised solver against all the levels I'd made so far (around 100) and
found only a single case where it did better than the non-optimal solver. It's quite
lucky that I stumbled into a case where the previous MITM solver had a
non-optimal solution.
