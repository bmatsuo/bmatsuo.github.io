---
layout: post
title: "On the Zero Value: Part One"
date: 2013-11-26 19:54:47 -0800
categories: go types zero
---
##The good, the bad, and the panic: runtime error: invalid memory address...

Dave Cheney wrote a really cool [article][1] about the zero value in Go and
showed some clever tricks for using it. The zero value is an interesting
concept. Any type has an intrinsic notion of 'zero' and all unassigned
variables are guaranteed to have the value 'zero'. From a low level imperitive
programming background this makes intuitive sense. In C all variable values can
be `memset(3)` to the integer 0. Go doesn't let you do fucked up unsafe memset
operations easily. But the **exact** same zeroing concept applies because it is
actually useful in practice (as demonstrated by Dave Cheney).

One thing to note is that the zero value can be painful in practice. "All types
must have a zero value." In a language dealing with memory references directly
the zero value of pointer is not well defined (in a mathematical sense, its
behavior is well defined in the language specification). So the type system
forces the pointer type to have a value that is not a pointer and has
disasterous consequences when used as a pointer (when dereferenced). That's
bad. Dave's nil-method-call trick surprized me. But that's also bad. That's not
the kind of thing you want popping up on you (or not). Really the nil method
call makes total sense and I had just never thought about it the right way
before.

The zero value for all primitive types is no doubt a great idea. And it works
really well for slices because it's low level **and** easy to work with. For
maps, the zero value is not very useful because it cannot grow easily like a
slice (though append). In Go maps grow as more items are inserted into them but
inserting a value into a nil map causes a runtime panic. Zero valued maps are
for ever doomed to their cold lonely void of an existence. Nil struct pointers
are also pesky and seemingly of little use. The nil method trip could prove
useful in either case (I see it being more useful for maps). But it's based on
error checking I'm not sure you could ever rely on someone doing in their code.
At this point I think things are just confusing. Do I handle nil checks on the
inside of method calls or does the caller handle them on the outside? Why have
this degree of freedom in a tightly controlled and oppinionated language? I
think it's a design constraint. If a method call on a nil struct pointer blows
up then a method call a nil slice type should blow up to be consistent. Then
the slice zero value is less useful as an 'empty' slice. At that point the zero
value concept is just totally hosed as a grand theory of types. I don't think
nil pointers are a very good idea even if a type system embraces them as
completely as Go.

**However**, regardless of my thoughts on nil values I do like Go. I can deal
with invalid pointers. I've been doing it since I was a baby. The language
itself is simple and easy to read/write for the most part.

[1] http://dave.cheney.net/2013/01/19/what-is-the-zero-value-and-why-is-it-useful "1"
