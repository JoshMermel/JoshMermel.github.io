---
layout: post
title: Numbering lines of pseudocode in Vim
---

Another short one today. When I write pseudocode that I want to share with
others, I like to number the lines so that I can easily refer to them.
In Vim this is easy thanks to the `nl` command.

Select the desired line to number and then type `:!nl`. By default
`nl` uses tabs. I prefer spaces so I use `:!nl -s "  "` instead.
`man nl` details the other twiddling options that you may find useful.

Piping text blocks in vim through bash is a powerful method of text interaction.
Some other times I use it are to sort my #inclues and to evaluate simple math
expressions with `bc`.
