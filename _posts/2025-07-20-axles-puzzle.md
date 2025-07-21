---
layout: post
title: Axles puzzle
---

This is a post about https://joshmermelstein.com/axles , a game I put together
with help from Gemini. This post might make more sense if you go play a round
before you read it.

# Background

In 2024, I saw a post on the twisty puzzles forum about the Peterhoff-Smolensk
puzzle - https://twistypuzzles.com/forum/viewtopic.php?t=39375 . I thought it
was a really nice idea, and filed it on my mental "I should think more about
this" list. 

# Simulator in Rust

I didn't know what specifically I wanted to do, just that I wanted to do
*something*. My two main ideas were to make 3d printed version of it, and to
make some kind of interactive digital version of it. I decided to start with
something in between - an analysis tool to help me figure out what sort of
puzzles I wanted to make.

As an experiment, I asked Gemini to write the first version of this simulator in
Rust (but on a square grid instead of a hex grid). The results were better than
I expected! With minimal edits, I was able to input a hardcoded goal
configuration, and then run the code to answer questions like "what start is
furthest from the goal state?"

I modified the code to try random 3x3 starting configuration and print cases
where the furthest node was especially far away. The results were impressive,
the AI generated code found cases like this one, where these two states were 36
moves apart.

    Goal:
    |↑ ^><|  |↑ >v|  |↑ |
    |↑ ^>v|  |↑ v|  |↑ ^>|
    |↑ ><|  |↑ >|  |↑ ^|

    Start:
    |↑ ^><|  |↑ >v|  |→ |
    |← ^><|  |↑ v|  |↑ ^>|
    |→ ^v|  |↑ >|  |↑ ^|

# Interactive Simulator

I cut out some scraps of paper to try to play with this puzzle, but found it a
little annoying to keep everything aligned, so I turned to Gemini again, and
asked if it could make the code interactive.

I was expecting some sort of stdin/stdout interface, where I could give moves to
the rust code one at a time. To my surprise, Gemini rewrote the simulator in
javascript, and opened an interactive gui where I should interact with the js
that it wrote. Parts of it were a little clunky, but it crystallized my goals in
my head. I wanted to polish this javascript version to make it easy for others
to play with these puzzles.

## GUI stuff

Polishing the gui wasn't that hard. The AI had done a lot of the bits that I'm
inexperienced with (css, event handling...), and I just had to edit them to work
the way I wanted. Where possible, I did this using more AI prompting; though
there were a few cases where the AI did quite a bad job and I ended up reverting
its changes and making them manually.

It got fixated on drawing the dots on each cell using css pseudo-elements, and
I'm not sure that approach is viable at all.

## Puzzle generation

When generating the goal state, it's undesirable if there's a loop of blockages,
such that A blocks B blocks C blocks D blocks A. When this happens, none of
those axles can turn, which means that much of the puzzle isn't intractable.

To prevent this problem, I came up with the following:

    assign each axle a random and unique id between 1 and num_axles
    (treat out-of-bounds as having some arbitrary id, like num_axles/4)

    for every position where it's possible to place a dot:
        if a lower id would block a higher id - do nothing
        if a higher id would block a lower id, add a dot with X% probability.
        if any axle has all four blockers, that's boring, remove one blocker.

I'm quite pleased with this approach. It is computationally cheap, and scales
linearly with the number of axles; and it's easy to convince myself that this
never generates those undesirable loops of blockers.

## Additional modes

Once I had that working, I decided to add some additional modes. By tweaking the
probability with which blockers are generated, I could make an "easy" mode with
sparser boards. And then I added support for diagonal blockers, and called that
"Hard" mode.

# Closing thoughts

My favorite part of this project is the speed and output of puzzle generation.
The quality of puzzles is good enough that I don't feel the need to add
human-authored ones.

I'm also quite happy with the usability. I watched a few friends trying it out,
and they seemed to find the GUI intuitive, in a way that let them engage with
the puzzle's rules.

This was my first time using AI in this way and I found the experience
fascinating. It was quite good at some things, but also got hopeless stuck on
others. I think the best part was the speed with which it got me *something* I
could interact with. That helped me refine my goal, and saved me a few hours.

# Future work

The author of Peterhoff-Smolensk recently published a 3D version of their idea,
which they call the "Bomzh Cube" -
https://twistypuzzles.com/forum/viewtopic.php?t=40211

I think this idea is fascinating as well, and that it would be neat to have a
digital simulator for it. I would need to learn quite a lot (or lean even harder
into AI-assisted programming).
