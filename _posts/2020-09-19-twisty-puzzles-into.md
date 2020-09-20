---
layout: post
title: Twisty Puzzles Intro
---

Personal History
================

When I was a teenager, one of my older brothers taught me to solve a Rubik's
cube. He gave me a handwritten sheet of notes on what to do in a bunch of cases
and if I followed the sheet correctly, the cube would always end up solved.
Eventually, I memorized the dozen or so move sequences from the sheet.

Later, my parents bought me a [megaminx](https://wikipedia.org/wiki/Megaminx) -
a puzzle that is a lot like a Rubik's cube except that it's based around a
dodecahedron. I was able to apply some of my knowledge from the Rubik's Cube to
the megaminx but got stuck on the last layer. Even though the puzzles were quite
similar, I didn't understand what the seemingly-magic algorithms from my
brother's notes so I was lost when it came to adapting them to a new situation.
Eventually I ended up looking up a solution online so I could solve the puzzle
and at the time that felt satisfying.

Rediscovery
===========

A few years ago, I saw a Youtube video about a puzzle I'd never seen before. I
think it was a review of a mass produced version of Oskar van Deventer's [Redi
Cube](https://www.youtube.com/watch?v=cjfMzA1u3vM). I bought a copy and quickly
felt the same frustration I'd felt with the megaminx - I didn't have a strategy
and I didn't even know how to come up with a strategy. So again, I went to the
internet and I found a tutorial for how to solve that puzzle.

Looking back, I regret how quickly I gave up. I feel like I deprived myself of
the experience of building an understanding of a puzzle and the joy of coming up
with a solution.

Nonetheless, solving a puzzle was still satisfying and I started to do research
on what other puzzles were out there. It turns out that the Rubik's cube was
only the tip of the iceberg. At the time, I didn't really have the framework to
understand these puzzles but I still liked seeing videos of how they moved. They
were like interactive sculptures! 

 - [Mandala Cube](https://www.youtube.com/watch?v=_x0So1SCWD4)
 - [Bloomy Gears](https://www.youtube.com/watch?v=M6o3fmlJ_6c)
 - [Circle Starminx](https://www.youtube.com/watch?v=Yz7dXv5zM-M)

What's in a Solution
====================

The sheet my brother gave me was one strategy for solving a Rubik's cube, but it
certainly wasn't the only one. In fact, dozens of strategies exist for the
Rubik's cube! Lets zoom in a bit and compare two of them.

### CFOP

Speedsolvers care about solving their puzzles as quickly as possible. This means
their solutions need to optimize for sequences that can be executed with [good
ergonomics](https://www.speedsolving.com/wiki/index.php/Category:Finger_tricks)
and preferably for low move count when possible. They also need to focus on
patterns that can be recognized quickly - because looking around the puzzle for
a piece wastes time.

To get a little more specific, the beginner-friendly version of the CFOP method
has users learn around 15 algorithms for the
[different](https://jperm.net/algs/2lookoll)
[phases](https://jperm.net/algs/2lookpll) of the solve. At the highest level,
CFOP-users learn sets of algorithms like
[ZBLL](https://www.speedsolving.com/wiki/index.php/ZBLL) which consist of
hundreds of move sequences for more optimally handling situations that come up
during a solve.

Other speedsolving methods exist with different tradeoffs but
most have the same property as CFOP that advanced users typically learn large
numbers of algorithms in order to get faster. This isn't to say that the only
way to solve faster is to memorize large sets of algorithms, but the methods
that speedsolvers use aren't opposed to such algorithm sets.

### Low Algorithm Count

In contrast [here](https://www.youtube.com/watch?v=iVYsJK7T7iQ) is a method for
solving the same puzzle using only two algorithms. It is not optimized for
speed, move count, or ergonomics. Instead, it is optimized understandability and
ease of memorization.

Notably, the two algorithms are lengths 4 and 8. This is in sharp contrast with
cfop which contains 15+ move algorithms in even the [intermediate
versions](http://speedcubedb.com/a/3x3/PLL).

### What I value in a solution

I think both of these methods are succeeding at their goals. CFOP is an
effective speedsolving method that lets users start with a moderate number of
algorithms and gives them the option to learn more in order to get faster. The
two-algorithm method from that video is easy to learn and easy to remember.

I am not a speedsolver and I am not such a minimalist that I want a solution
with just 2 algorithms. Ideally I would like my solutions to:

 - Be easy to remember
 - Be easy to execute without errors
 - Be flexible
 - Be understandable or even intuitive
 - Avoid long or tedious algorithms
 - Avoid large numbers of algorithms
 - Have some room for creativity to combine or skip steps
 - Be personal in some way

Techniques for Designing Solutions
==================================

Getting back to the story of my experiences with twisty puzzles, I wanted to
stop learning solutions from others and start coming up with my own. There are
many ways to do this but one common one is based on the group theory idea of a
commutator. There's a lot to say here so instead of writing it all, I'll
link you to [this video](https://www.youtube.com/watch?v=-NL76uQOpI0) from
Mathologer who does a pretty good job.

Assuming you watched that video - you now know how to come up with your own
solutions! So why am I writing a blog post about this? Well Mathologer glosses
over some details that I think are quite interesting.

 - Finding magic sequences to isolate pieces is not trivial, especially on
   complex puzzles
 - The "trick" only applies if the puzzle is doctrinaire - meaning all
   moves are always available. Many interesting puzzles do not meet this
   definition and need additional strategy.
 - The video only covers "pure" commutators, where only one type of piece is
   permuted. When solving puzzles with many piece types, it can be convenient to
   ignore some piece types while focusing other others. In this case, deciding
   which order to address the pieces becomes an interesting part of finding a
   solution.

My Journey of Improvement
=========================

Over the last few years, I've been slowly improving my solving skills and
growing my puzzle collection. At the start of 2020, I decided to push myself so
I scrambled every puzzle in my collection (about 75 puzzles at the time), and
attempted to solve all of them by the end of the year without consulting any
outside help. Embarrassingly, this was the first time I'd ever scrambled a few
of them.

It's currently September and I'm on good pace toward that goal. My puzzle
collection has also grown to about 100 puzzles (new ones were scrambled and
counted toward my 2020 goal) and I only have three left to solve before the end
of the year. I believe that my work so far has had the desired effect of making
me a stronger puzzle solver.

That said, some of my solutions were quite ugly. For several puzzles, like the
bagua cube, and the geared mixup, my solutions involved long (40+ moves) pure
commutators. For some of the crazy 3x3x3 plus series, I got stuck on edge
orientation issues and feel like I got out of them by luck.

In 2020, I thought this was fine, since my goal was to solve all puzzles - not
to solve them *well*. But in 2021, I want to step up my game further. I want to
focus on a smaller set of puzzles and find solutions that I am proud of. I want
to solve each one more than once to make sure my solution covers all possible
edge cases. Also, I want to do this with less use of digital puzzle simulators
than in 2020 as a way to improve my puzzle visualization skills.

I haven't decided whether I'll consult other people's tutorials in 2021. On one
hand, finding the solutions myself feels more meaningful. On the other hand, I
think I'm missing out on ideas that I could be learning from others.

Many of my solutions were written down on scratch paper and were probably lost.
So moving forward, when I find a solution that I'm happy with, I'm going to try
to document it on this blog for posterity. I may also do some smaller posts on
misc puzzle ideas I had that I think are worth recording. These posts will not
be tutorial and will probably be written for an audience that is already
familiar with puzzles.
