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

Windows are analogous to tabs. By default a window contains a single terminal
and its name is whatever process is running there. You can optionally rename a
window with `C-b ,`

### Some Commands For Windows

Create a new window

    C-b c

Move to next window

    C-b n 

Move to previous window

    C-b p

Move to window 0 (and so on for other window numbers)

    C-b 0 

Rename current window (extra credit)

    C-b , 

### Some Commands For Interacting With Panes

Each tmux window can contain a number of panes which are vertical and horizontal
splits. I use these to group highly related tasks. The window I'm in right now
has one pane for vim where I'm writing this post and another for a terminal
where I'm running git commands.

Split horizontally

    C-b %

Split vertically

    C-b "

Move to a split

    C-b [arrow key]

## Workflows at this point

## Copy mode and scrollback

## Messing With .tmux.conf

## Using Sessions?
