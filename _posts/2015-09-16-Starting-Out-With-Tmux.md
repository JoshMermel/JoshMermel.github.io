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

In parallel to panes, tmux has the idea of windows, which are analogous to tabs
in most applications. Each window contains an arbitrary number of panes (get
it). When you open tmux there will be one window by default. Its name will be
the name of the program you are running in that window. New windows are created
with `C-b c`. There is a shortcut to kill a window but I tend to just close all
the panes with `C-d` which closes the window containing them as well.

You can move to a window using its index with  `C-b` and then the index of the
window.  You can also move one window forward with `C-b n` (for next) or one
window backward with `C-b p`.

I admit, I in a way just did the thing I wanted to not do and wrote a bit list
of commands. The nice thing about tmux is that it is perfectly usable with a
subset of even the ones I've listed here. I recommend picking either panes or
windows and trying to use that one without the other. Once you feel comfortable
with that, try incorporating the other one as well.

### Some Additional Window and Pane Commands
Here are some lower priority commands that may make windows and panes slightly
more useful to you. 

Sometimes a small pane contains long lines that are being wrapped and you wish
it were larger. `C-b z` zooms in on just that terminal and makes it take up the
entire tmux window. Repeating this command unzooms. 

Sometimes I forget what is in a window when its title is just "bash" or "vim".
Windows can be manually renamed with `C-b ,`.

Sometimes tmux's pane layout isn't enough for me and I want to manually resize
a pane. You do this by holding `C-b` and then pressing an arrow key.

## Workflows at this point


## Copy mode and scrollback
Remember when scrollback didn't work. Here is a way to scroll back with
limitations. What is it so bad? I should see if I can get mousewheel to work

## Messing With .tmux.conf
warn about not resetting defaults to unspecified fields.

Some stuff I do

 - C-b C-b for last window
 - mouse
 - vim keys for movement
 - swap window position
 - clearing scrollback

Some stuff I don't do

 - remap C-b
 - remap % and " to | and - even if it is cute
 - use any defalt layouts

## Using Sessions?
I don't do this but its a strength of tmux I guess.
I probably should start using these...
