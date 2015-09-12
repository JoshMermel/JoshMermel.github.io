---
layout: post
title: Corollaries in Siteswap
---

## Lack of Intro
Siteswap, the mathematical notation for juggling is relatively well researched
and well understood. Rather than start this blog post with a few paragraphs
about its history and some simple examples, I'm going to provide some links that
do that and jump right into the meat.

 - [An hour long and comprehensive lecture at Cornell](https://www.youtube.com/watch?v=38rf9FLhl-8)
 - [A comprehensive wiki article](http://juggle.wikia.com/wiki/Siteswap)
 - [A well written paper proving some fundamental results](http://www.math.ucsd.edu/~ronspubs/94_01_juggling.pdf)
 - [A more modern book proving additional theorems](http://www.amazon.com/gp/product/0387955135?psc=1&redirect=true&ref_=od_aui_detailpages00)

## Validation of Vanilla Siteswaps
It is well known that a vanilla siteswap which does not average to an integer is
invalid. What is less well known is why certain vanilla siteswaps that average
to an integer are invalid. 

The premise of siteswap is that discretized time into a number of evenly space
units. Every throw and every catch must occur at one of those units. When we say
that a pattern is valid we really mean that the indegree each time unit (the
number of objects being caught then) is equal to the outdegree (the number of
objects being thrown then).  In vanilla siteswap, that means at every time unit,
at most one ball can be caught. Thus, a pattern is forbidden if there exist two
throws that land at the same time. We can express this formally as follows.

For all siteswaps $$a_1 a_2 a_3 \cdots a_n$$,
$$ \forall x \forall y,  a_x + x \not \equiv a_y + y \pmod n$$

If we wanted to verify these pairwise relations individually, we would need to
do quadratically many checks. A cheaper way to to note that the above statement
is equivalent to saying that the set $$\{a_x + x\} \pmod n $$ is equivalent to a
permutation of the numbers 1 through $$n$$.

## Generating New Siteswaps From Existing Ones

### Add 1 to Every Number

### Add Pattern Length to One Number

### Swap Sites of Two Numbers

### Extending to Multiplex Siteswaps

### Extending to Sync Siteswaps

## Understanding Transitions

### Entry + Exit is a Valid Siteswap

### Computing Transitions Between Vanilla Siteswap

### Extending to Multiplex Siteswaps

### Extending to Sync siteswaps
