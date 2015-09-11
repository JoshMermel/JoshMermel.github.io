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

Here is the initial state of the board 

<img src="/images/Move-Brute-Force/ex0.png" style="max-height: 400px">

We swipe left to move the top and bottom dots one space to the left and
producing this board 

<img src="/images/Move-Brute-Force/ex1.png" style="max-height: 400px">

Next we swipe down moving the rightmost dot down 

<img src="/images/Move-Brute-Force/ex2.png" style="max-height: 400px"> 

Finally we swipe left to align all dots with shaded square 

<img src="/images/Move-Brute-Force/ex3.png" style="max-height: 400px">

For each board, the game tells you how many moves are in the optimal solution.

A Puzzle I Couldn't Solve
=========================
<img src="/images/Move-Brute-Force/move-4-74.png" style="max-height: 400px">

I had solved 99/100 of the 4x4, 1 color puzzles with the shortest possible
solution but struggled on puzzle number 74.  My shortest solution, URURDRURLLDU
was two moves too long.

Trying to Brute Force on Paper
==============================
My strategy to solve the game was to work both forward and backward. Working
forward it was easy to see that the first move must be up; all other moves have
no effect.  Working backward it was easy to see that the last move must be up as
well. No state exists such that a left, right or down leads to the goal
state. From there the options became far more numerous. We met in the middle
believing we knew all states that could be reached after 4 moves and all states
that could be reached after 6 moves.  We then used heuristics to pairwise rule
out all the options.  We concluded that the board could be solved in at best 11
moves.

Solving the game in code
========================
I decided to write some python to settle whether or not this puzzle could be
completed in ten moves.  My implementation has two parts.

1. A python class that more or less implements the entire game
2. A python main that tries possible move sequences to see if they cause victory.

I reused the python class with only minor modifications across my version of the
solver. The class treated a board state as a 2D array of ints where 0's
represented empty space, -1's represented unmoving blockers and 1's represented
dots. For puzzles featuring more than one color, number other than 1 can be used
to specify the color of a dot/goal position. It had a function to do the action
of swiping in each direction and not much else.

###V1
My first version of the program optimized for programmer time not runtime. There
are at most 4^10 (roughly one million) possible move sequences.  It isn't that
expensive to compute one move on the board. Lets just simulate each of the one
million sequences and print the first successful solution we find.

The two interesting pieces of code were as follows. The board class exposed a
function that took an integer, treated it as a sequence of moves, and executed
that sequence on its internal board:

    def move_sequence(self, instructions, num_instructions):
        for i in range(num_instructions):
            todo = instructions % 4
            instructions = instructions / 4
            if todo == 0:
                self.move_down()
            elif todo == 1:
                self.move_up()
            elif todo == 2:
                self.move_right()
            else: # todo == 3
                self.move_left()

The main function simply called the move_sequence function on every integer in
the valid range and then compared the result against the desired final board.

    num_moves = 10
    for i in range(4**num_moves):
        my_board.reset()
        my_board.move_sequence(i, num_moves)
        if my_board == goal_board:
            print 'FOUND IT!'
            my_board.move_sequence_str(i, num_moves)
            break

####The Good
 - Very quick to write
 - Potentially parallelizable?
####The Bad
 - Slow
   - 17 second runtime
 - Tons of recomputed data
   - The transitions from the starting state to each of the 4 following states
     were each calculated ~250,000 times.
 - Doesn't abandon fruitless paths
   - Swiping down as a first move has no effect so it can't possibly be the
     first move but this iteration of the program tries ~250k sequences that
     begin with swiping down.
 - Needs to know how long the ideal solution is at compile time

###V2
This version of the program attempted to leverage the fact that early
subsequences were being executed many times by using memoization. It depth first
explored a trie where every node held the state of the board after the sequence
of steps leading to it. This design made it easy to ignore fruitless
subsequences. Whenever I encountered a move that didn't change the board state,
I stopped exploring that branch of the tree.

####The Good
 - Much faster than V1
   - 4 second runtime
 - Relatively memory inexpensive - The memoization stored at most as many boards
   as the length of the solution at a given time.
####The Bad
 - No sense of duplicate state
   - Down-Up on the board resets initial state but this program still tries up
     to 4^8 sequences beginning with that sequence - none of which are viable
   - Not only that but it computes those paths multiple times, once for each way
     of re-entering the starting state.
 - Needs to know how long the ideal solution is at compile time

###V3
This version switched from depth first search to breadth first search. It was
build on the idea that most states are reachable in many ways and we shouldn't
have to keep paying to do the same computations on those states. It maintains
dict of all visited states and the shortest known path to them. Whenever a 
state is computed, the algorithm checks if it is new and ignores if it is not.

####The Good
 - Much faster than V2
   - 0.2 second runtime
   - Does not need to be told how long the ideal solution is
####The Bad
 - Requires more memory

Result and Analysis
===================
It turns out there were two solutions.
 - UURDULLDRU 
 - ULUULDRDDU

V3 was far faster than I expected.  I believe this is because the number of
states reachable in ten moves is a fraction of the number of states that can be
on the board. There are 14 free squares on the board and 4 dots to place so
there are $$14*13*12*11/24 = 924$$ possible board states. However during the
running of V3, only 593 of those are actually visited.

Where We Went Wrong on Paper
============================
1. Working backward was harder than it seemed.  One move to enter a state may
   mean many possible preceding states.
2. Seemingly implausible states may be easier to get into than anticipated.

Open Questions
==============
1. How was this puzzle generated?
  * More generally, how were the 1000+ puzzles in the game generated
2. Given the set of solutions to a puzzle, is it possible to reconstruct the
   initial board?
  * What additional information helps (i.e. board size, distribution of colors...)
