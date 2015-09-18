---
layout: post
title: Starting Out With tmux
---

[tmux](https://tmux.github.io/) is a terminal multiplexer. It allows for near
arbitrary layout of terminals in panes and windows (their words for splits and
tabs) and supports detaching and reattaching terminals. I've been curious
about tmux for a while and finally decided to give it a try a few weeks ago. 

Like many powerful tools, tmux has a rich configuration language and a myriad of
commands. In the long run I think these options are beneficial to the user but
starting out they can be overwhelming.  To my dismay, most tmux tutorials were
effectively lists of commands with no sense of progression or order.  I am a big
fan of 
[Learn Vim Progressively](http://yannesposito.com/Scratch/en/blog/Learn-Vim-Progressively/)
and think that many other complicated program can benefit from a similarly well
structured guide.

Unfortunately I am just a beginner at tmux so I don't have the knowledge to
write the advanced part of such a guide. Instead I can relay the path I took to
becoming comfortable in tmux and comment on what went well and what didn't.

## Starting Out
 - Install tmux
 - Launch tmux
 - Use it like a normal terminal

For most purposes, tmux will behave just like your regular terminal except for a
bar on the bottom listing some information like the date and your hostname. For
now, scrollback won't work for you (sorry).

## Using Windows And Panes
A reason that tmux was attractive to me was that it let me clean up my alt-tab
list. When working on projects I usually had a browser, a terminal for building,
a terminal for testing and a vim window. I had to mash alt-tab to get where I
wanted and sometimes pressed it an incorrect number of times. tmux lets me
consolidate all My my terminal applications in one place and provides tools for
organizing them.

Often while coding I would find myself carefully arranging my terminals so each
one took up just the right amount of space on the screen. tmux solves this
problem with panes.

When you open tmux, it will show you a single terminal. This terminal can be 
split with a vertical line down the middle using `C-b %` or split with a
horizontal line through the middle using `C-b "`. Furthermore, these splits
(called panes) can be further split with the same commands to generate
relatively complex layouts. Moving between panes is easy, we press `C-b` (this
sequence lets tmux know that the next character presses is intended as a command
to it) and then the arrow key of the direction we want to move in. 

I tend to use panes to group related tasks. Most of the time, all of my panes in
a given window will be in the same directory. As I write this blog post I have
one pane with vim running where I'm typing and another where I'm running git
commands.

In parallel to panes, tmux has the idea of windows which are analogous to tabs
in most applications. Each window contains an arbitrary number of panes (get
it). When you open tmux there will be one window by default. Its name will be
the name of the program you are running in that window


### Some Commands For Windows

Create a new window

    C-b c

Move to next window

    C-b n 

Move to previous window

    C-b p

Move to window 0 (and so on for other window numbers)

    C-b 0 

    C-b z
`C-b ,`

## Workflows at this point

## Copy mode and scrollback

## Messing With .tmux.conf

## Using Sessions?
