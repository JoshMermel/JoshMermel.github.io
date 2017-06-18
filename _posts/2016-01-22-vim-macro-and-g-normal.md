---
layout: post
title: Vim macro and g/pattern/normal
---

If you find yourself doing the same thing over and over in Vim, there is probably a better way. Here is an example of how to use macros and regex matching to avoid repetitive but mindless work.

Lets say you have a config file of the form:

~~~
foo = bar
baz = "nort"
qux = qat
~~~

where all of the string on the right hand sides of each equals sign are supposed to have quotes around them but many do not. We can fix this pretty easily without needing to leave Vim.

First we go to an offending line and record a macro:

`qqf=wi"<esc>A"<esc>q`

Lets break that down:

 - `qq` - begin recording a macro in register q
 - `f=` - go to the first equals sign on the line
 - `w` - go forward one word
 - `i"<esc>` - insert a quotation mark here
 - `A"<esc>` - insert a quotation mark at the end of the line

We could manually execute this macro with `@q` on each necessary line. But there is an easier way:

`:g/= [^"]/normal @q`

Lets break this down too:

 - `:g` - begin executing an `Ex` command (by default on every line of the file)
 - `= [^"]` - a regex to match meaning "an equals sign followed by a space, followed by something other than a quotation mark"
 - `normal @q` - execute the normal mode vim command stored in macro q
 
 This runs the macro we recorded above on every line that needes it. Voila!
