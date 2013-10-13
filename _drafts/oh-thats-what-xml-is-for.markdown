---
layout: post
title: "Oh, that's what XML is for"
date: 2013-10-12 19:48:39 -0700
categories: xml xslt xsd
---
I have had to learn some more about XML for my job recently. I was not looking
forward to this assignment at first. But having gained some experience in the
world of XML schemas and stylesheets. I can finally see that there is something
good about this markup language.

##Verbosity

Yeah, yeah, yeah. I know. XML is verbose. It's true. But I've been working with
some JSON structures recently where the presence of multiline strings really
sucks when you want to pretty print something. When you want to allow multi-line
text in a markup/transport language, the presence of a closing tag is probably the
best thing to do.

Furthermore vocabulary designers need to be mindful of element label length.
Brevity goes a long way. The design work is harder but it can improve the
grokability of XML documents.

##XML Editors

I've traditionally editted XML in Vim and without plugins, where I also felt the
burden of XML's verbosity. I always found it so cumbersome to include closing
tags. But this was a stupid gripe with XML. Editors will either have builtin
functionality to close tags, or have plugins which do so. The
[xml.vim plugin](https://github.com/othree/xml.vim) makes this possible.
There are also specialized XML editor programs out there. These days,
I find myself using the free version of Intellij most often for the good schema
support.

##XML Schema

XML Schema (XSD) and other schema languages are pretty nice when hand-editing
XML is necessary (e.g. config files). Programs can validate shitty hand-coded
XML and give (usually) good hints about parts that I messed up. Intellij has
pretty good XML Schema validation built-in with RelaxNG support available
though a plugin. Using the [syntastic plugin](https://github.com/scrooloose/syntastic),
Vim can only validate DTD schemas (which I don't recommend writing). But, the
`xmllint` command line validate XML Schema and RelaxNG schemas. So Vim users
can shell out for validation

    :!xmllint --schema /path/to/schema.xsd %

I don't know how I feel about code generation from XSDs. I haven't fully explored
the area so I can't make a firm judgement. But, the schema structure does not
seem especially condusive to making reasonable class/structure hierarchies. I
still feel like there is value for people building serializers from schema
generated documentation.

XML Schema has flexible documentation constructs that allow for programatic
consumption of documentation. Intellij is able to give you a 'quick
documentation' pop-up for the docs relating to any element in an instance
document. There are also [stylesheets](http://sourceforge.net/projects/xs3p/)
that produce HTML documentation from XSDs.

##XML Stylesheets

XML Stylesheets (XSL) are a pretty cool idea.  In it's most common form,
stylesheets are templates (XSLT) providing functional document manipulation.
The language itself is a decent template language. For XML to XML transformation
(including XML to HTML), it's pretty good. It seems a little awkward to use XSLT
for transformations into other formats (e.g. JSON), but it can be done.

One of the coolest things about XSLT, in my opinion, is that it can be
rendered by all modern web browsers. It's a very neat idea from a web-service
perspective especially. Services can deal with a common interchange format
(i.e. XML) and rendering of HTML is handled entirely client side with static
templates (served from a CDN, cached, all the good stuff).

##XSLT in the browser

In the culmination of my learning I wrote an XSD describing a
[resume](http://bmats.co/resume.xsd).  I then wrote an XSLT that transforms the
resume into a [Bootstrapped HTML5 web page](http://bmats.co/resume.xml).
I'm pretty happy with the results and it was pretty simple. The project has
other benefits in that, with more stylesheets, my XML resume can be transformed
into almost any other format (like one that describes print documents and
can generate a pdf).

I am slightly concerned though. I have read numerous reports that XSLT suffers
from cross-browser compatibility issues. In my experience so far I have not
noticed anything that is too troublesome. The only major problem I've run into
so far (without IE testing `>_<`) is that Firefox wraps the output in a
`<transformiix>` element. That screwed up the doctype declartion in my
template. I was able to find a workaround
[solution](https://developer.mozilla.org/en/docs/XSL_Transformations_in_Mozilla_FAQ#What_about_document.write.3F)
fairly quickly. Things have been pretty golden since then.

Perhaps browser support for XSLT has improved in recent years. Or perhaps
using a common subset of the language will be fine for most things. I'm
not sure. I need to look into things like browser support for date formatting
and more complex string manipulations. But what I've seen so far with my
basic usage has me pretty satisfied.

##Vocabulary overload

A common gripe I hear about XML is overabundance and duplication of vocabularies
(schemas). I can understand this. An astute reader might have realized that for my
XML resume idea I invented a new vocabulary where one almost certainly already
exists. I did look into this and had previously written my resume in an the vocabulary
of an existing project that included templates for web pages and print documents.
This was OK, but I was unsatisfied with a number of things. First, the schema was in
DTD. These are hard to read and puts some pretty bad restrictions on document structure.
Also the template output was crusty and the project is on Source Forge. Instead of building
on top of these problems trying to smooth them over I created a new project with a
modern schema, pleasant document structure, and modern HTML output. The timelpate is not
super flexible now but I hope to improve that over time.

Still, I've just explaining my reasons for poluting the vocabulary space. I didn't propose
a solution or say why things aren't really that bad. It is bad. And I don't know of a
solution. To me, this problem of vocabulary discovery and cooperative improvement feels
very much like the problem of package discovery in the Go community for which, in my opinion,
there has not been a good solution yet.

##Working with XML

So I've been talking alot about XML, but nothing about actually handling it inside
a program. Truthfully, it is generally harder than handling JSON. Where JSON has
structures which map cleanly to arrays and maps available in most languages, the
node tree structure provided with most APIs can be cumbersome.

Some languages provide serialization libraries to map XML to and from native
classes/structures. This can work well but is not always great with handling
dynamic content. There are also cases where allowing any well-formed XML is
desirable, in which case serialization libraries do not help.

For working directly with node trees there is a DSL called XPath. XPaths allow
queries to be run against a node tree. It's a pretty nifty little language.
The simplest form looks like any path in your file system or in a URI. But
the more approriate analogy is with shell command pipes. Each syntatic element
is a process that accepts nodes as input. Each node taken as input generates
some number (possibly zero) of nodes as output. Chaining these processes
together produces a powerful XML syntax for extracting nodes and values from
XML documents. It's worth noting that XPath also defines a function invocation
syntax and a library of node processing functions. But this feature is not
defined in all XPath parsers.

I have also been working with Scala a lot at work. I am not completely sold on the
language yet. But, it has some native support for XML that is pretty convenient
(XML literals... What?!). I'm not sure that XML literal syntax is something that
**should** be in any general purpose programming language. But I'll use it while
it's there. Scala also provides Java interop, and access to the huge amount of XML
library support in the Java universe. If I have future projects that deal heavily
with XML, it's quite possible that I would opt for Scala again.

##Conclusion

Learning about XML and related technologies has been much more rewarding I
thought initially. I'm not exactly ready to claim that there will be a resurgence
of XML in the future. But, I have certainly seen situations in which I feel it's
superior to JSON. I will continue to look for more goodies in the future.

Well, that's all the XML evangelism for now folk. But I'll see you

    <next-time rocket:time="same" rocket:channel="same"/>
