---
layout: post
title: Unexpected Vim behavior when repeating indent
---

## Composability of commands in Vim
The reason that Vim is my text editor of choice is because Vim commands are
composable. Each command has a verb and an object so every time I learn a new
verb, I already know how to use it with each object. Whenever I learn a new
object I already have a lexicon of verbs to use with it. This idea is summarized
well in this article.

## The notion of [count]
A feature of many Vim commands is that they optionally accept a count. This
number tells the command how many times it should happen. For example, to go
down five lines, you could hit `j` five time. Or you could hit `5j`. This feature
lets you cut down on key mashing and get right to the meat of text editing.

## >> 
The `>` command is a verb that can be read "indent by one line." It take a motion
and indents everything between the cursor and the destination by one line. Some
examples are `>ip` for "indent all lines of this paragraph" or `gg>G` for "indent
the whole file by one shiftwidth"

## Weirdness with [count]>>
Recently I wanted to indent a line by three shiftwidths. I'd never done this
particular action before but with my understanding of [count] I assumed the
right thing to do was `3>>`.

To my surprise, this command indented the next three lines by one shiftwidth
each. If I had wanted that I would have used 2>j. I tried several variations on
these commands and found that >2j and >3> both did the same thing.

Looking at the Vim help documentation (`:help <<`), I found that this behaviour
was documented. 

`>>      Shift [count] lines one 'shiftwidth' rightward.`

Here I also found some sequences that did I wanted - shift the current line
right by three shift widths.

`{Visual}[count]>  Shift the highlighted lines [count] 'shiftwidth'`

So one solution to my original problem is `v3>>`. This distinction doesn't feel
good to me. It feels like a waste of entering visual mode and there is no
intuition why the effect is different in visual versus normal.

```
:[range]> {count} [flags]
      Shift {count} lines one 'shiftwidth' right, starting
      with [range] (default current line |cmdline-ranges|).
      Repeat '>' for shifting multiple 'shiftwidth's.
```

Another solution is to use command mode. `:>>>` indents the current line by 3.
This sequence also feels bad to me. 

## Similarities to y and closing
The `y` command (copy) has very similar semantics to the `>` command. In normal mode
`[count]yy` yanks [count] lines. In visual mode, `[count]yy` yanks the selected text
(we can think of this as happening [count] times for convenience.) 

However for the `y` command, repeating an action on the same line doesn't do
anything.  With the indent action, repeating several time on the same line makes
sense and is sometimes desirable. The interaction of [count] and `>` in normal
mode is well documented and consistent. I feel that a doccumented design mistake
is still a mistake. In the grand scheme of things it is pretty minor but I think
it would be more intuitive and more in the spirit of Vim of `[count]>>` indented
one line count times.
