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

## Using Windows
A reason that tmux was attractive to me was that it let me clean up my alt-tab
list. When working on projects I usually had a browser, a terminal for building,
a terminal for testing and a vim window. I had to mash alt-tab to get where I
wanted and sometimes pressed it an incorrect number of times. tmux windows solve
this problem for me by letting me consolidate all my terminals under one alt-tab
entry.

'''C-b c''' to create a new window
'''C-b n''' to move to the next window
'''C-b p''' to move to the previous window
'''C-b 0''' to move to window 0 (and so on for other window numbers)
Extra credit: '''C-b ,''' to rename a window

## Using Panes
Each tmux window can contain a number of panes which are vertical and horizontal
splits.
