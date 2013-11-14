---
layout: post
title: "Reduced Instruction Set Editing"
date: 2013-11-13 20:51:02 -0800
categories: ramblings vim
---
[risc]: #
[pnwscala]: #

First off, this is going to be heavily centered around vi/vim. But, the ideas
should apply to any text editor or IDE with extensive key binding facilities.

##Endless possibility

Vim's binding support is supreme. With different editing modes binding are
extremely convenient not need modifier keys. And you can still use the modifier
keys in bindings! The number of binding combinations is fucking crazy. But,
with so many combinations one can quickly achieve _binding-bloat_ at which point
binding more keys decreases productivity. A preferrable configuration utilizes
a kernel of simple key commands which are adequate for any text editing scenario.

##Complexity kills

Some bindings in vim can be prepended with a count or range to repeat the action
or restrict its scope respectively. These are primative concepts in vim and
alone they are quite powerful. Plugins allow allow you to bind complex
functionality however is desired.

These concepts allow for very concise expression of very powerful text
manipulation functions. But these tools are very sharp. Flexibility and power
requires reasoning and precision to apply correctly.

Something I started to do for a while was  bind keys for 'every' possible editing
task I could think of. Generally people this type of thing is done with `<leader>`
combinations. After accumulating too many of these, and not really internallizing
many of them, I had to start thinking about what the right binding was to do the
particular thing I wanted.

When I write code I want to think about the algorithms and data structures. I
don't want to think about manipulating text. When a desired outcome is known
planning the movement between those points should require minimal brain cycles.
It's like walking.

##RISE

The RISC movement in computer architecture was based on the idea that when
restricted to a small simple set of primitive instructions a processor can
perform tasks faster than a contemporary processor which provides a large amount
of more powerful instructions. The reduced instruction set allows for heavier
optimizations in the processor resulting in higher overall throughput. 

This is also reminiscent of a [talk][pnwscala] I saw Paul Philips give at
PNWScala about the incredible hardships faced in development of the scala
compiler trying to be everything for everyone and his desire for a language
with a simpler core.

There is a perfect analogy here with text editing. But I am not talking about
making the editing software fast by reducing the set of operations, I'm talking
about speeding up the processor orchestrating the text manipulation. My fucking
brain! With a small, simple, widely applicable set of functions at my fingertips
I can edit text faster than someone with lots of specialized bindings.

Even if things aren't always faster the use of simple operations reduces the
amount of thought applied to each keystroke. In turn, edits require less of a
context switch. I can keep thinking about bigger-picture stuff while issuing
repetitive key commands.

##Perfection!

There is no such thing. Sorry. I thought about telling you earlier. But thought
this would be more rewarding. My opinion is that the optimal comibination of
edit functions is different for everyone. It depends on the patterns you find
most natural. But I will share a few of the patterns that I find useful.

I do a lot of _visual_ (`v`/`V`) and _substitute_ (`s`/`S`) edits. I use
`A<return>` instead of the specialized `o` key because `A` is more useful,
although `O` is somwhat more cumbersome to produce out of more primative
actions (`kA<return>` in most cases, `I<return><esc>kS` on line 1). I use
`C-c` instead of `<esc>` when in _insert_ mode, although that habit can cause
problems when using "vi-mode" in programs like Bash. I don't use _change_
sequences (`c*`) but I am actuall ywant to try using them more becaus the
pattern is just SO common.

What I'm proposing here is not a rule. It's a guideline. There are times when
a particular compound operation is so common it just makes sense to treat it as
a primitive.
