---
layout: post
title: Solving a puzzle game the easy way
---

An android game I've been really enjoying lately is
[move](https://play.google.com/store/apps/details?id=com.nitako.move) from
Nitako Brain Studios. It is a game with simple mechanics and extremely good
puzzle design.  

Game Mechanics
==============

Their [trailer](https://www.youtube.com/watch?v=up3lf5Rd97k)
shows off pretty much everything you need to know about it.  If you don't
access to youtube then I'll do my best to explain.

A board consists of an n by n square featuring n colored dots, n shaded squares
whose colors match the dots and any number of blocking obstacles. Each move,
you select one cardinal direction and swipe the screen in that direction. All
dots move one square in that direction unless they are blocked by a wall, an
obstacle, or another blocked dot.

See consider the following example:

Here is the initial state of the board <img src="/images/Move-Brute-Force/ex0.png">

We swipe left to move the top and bottom dots one space to the left and
producing this board 

<img src="/images/Move-Brute-Force/ex1.png">

Next we swipe down moving the rightmost dot down 

<img src="/images/Move-Brute-Force/ex2.png"> 

Finally we swipe left to align all dots with shaded square 

<img src="/images/Move-Brute-Force/ex3.png">

For each board, the game tells you how many moves are in the optimal solution.

A Puzzle I Couldn't Solve
=========================
<img src="/images/Move-Brute-Force/move-4-74.png">

I had solved 99/100 of the 4x4, 1 color puzzles with the shortest possible
solution but struggled on puzzle number 74.  My shortest solution, URURDRURLLDU
was two moves too long.

Trying to Brute Force on Paper
==============================
Put information about our strategy and maybe some images of our scratch paper

Solving the game in code
========================
I decided to write some python to settle whether or not this puzzle could be
completed in ten moves.  My implementation has two parts.

1. A python class that more or less implements the entire game
2. A python main that trys possible move sequences to see if they cause victory.

More to come on this when it isn't midnight

Result
======
Turns out there were two solutions.

UURDULLDRU and ULUULDRDDU

Where We Went Wrong
===================
1. Working backward was harder than it seemed.  One move to enter a state may
   mean many possible preceding states.  
2. Seemingly implausible states may be easier to get into than anticipated.

Open Questions
==============
1. How was this puzzle generated?
  * More generally, how were the 1000+ puzzles in the game generated
2. Given the set of solutions to a puzzle, is it possible to reconstruct the initial board?
  * What additional information helps (i.e. board size, distribution of colors...)
3. Would it be worth it to implement some optimizations?
  * Deduping boards with some kind of global hashmap?
  * Treating the whole problem as a graph search problem and using Dijkstra's Algorithm?
  * Using a heuristic to decide which directions to pursue first?
