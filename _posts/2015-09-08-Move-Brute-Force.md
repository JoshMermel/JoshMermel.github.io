---
layout: post
title: Solving a puzzle game the easy way
---

An android game I've been really enjoying lately is
[move](https://play.google.com/store/apps/details?id=com.nitako.move) from
Nitako Brain Studios. It is a game with simple mechanics and extremely good
puzzle design.  Their [trailer](https://www.youtube.com/watch?v=up3lf5Rd97k)
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
producing this board <img src="/images/Move-Brute-Force/ex1.png"> Next we swipe
down moving only one dot down <img src="/images/Move-Brute-Force/ex2.png"> Finally
we swipe left to align all dots with shaded square <img
src="/images/Move-Brute-Force/ex3.png">
