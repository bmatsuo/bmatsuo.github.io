---
layout: post
title: "Reverse Routes in Go"
date: 2014-05-18 17:27:17 -0700
categories: go http
---
**TL;DR** This is a simple pattern for safe reverse routes in Go without
without code generation and without external packages.  It's restricted to URL
paths with a single variable.  I wrote a
[package](https://github.com/bmatsuo/rt).  But really, the takeaway is to use
http.ServeMux.

##HTTP Routing and Go

There is a lot of contention around HTTP routing (and frameworks) in Go.  The
diehards insist that "net/http" is the one true package.  While many others
continue to invent ways of using "net/http" or masking it.

##What does one really need from application level routes?

At the application level support for getting the suffix of a matched prefix
should be adequate for most things.  Implementing something like GitHub's path
strategy would be difficult.  But you can get pretty far (S3 path would be
reasonable to implement).

Reverse routing is pretty import for maintainable, easy-to-use HTTP interfaces.
Constructing links to other resources is very convenient.  And constructing
links using error prone techniques is no fun to deal with.

## Things I don't want in routing

Code generation.  Pretty simple.  For a complex system.  Sure.  You may need a
multi-step build process or a process for checking in generate code.  But I
want to keep things simple and `go get` friend for as long as possible.  Code
generation is not something I want in my routing.

##Problems with reverse routes in Go packages

In terms of runtime packages you basically have to deal with unsafe things and
pass the buck onto the caller.  This is clear looking at one of the few
packages to even provide reverse routing (the only one I know of),
["github.com/gorilla/mux"](https://github.com/gorilla/mux).

First of all, the [Gorilla Web Toolkit](http://www.gorillatoolkit.org) is
fairly popular but the reverse routing code isn't modified often.  Don't
expect it to be the ideal implementation or anything.

Gorilla acheives reverse routing through named routes and named parameters.
Both of these things make it intrinsically error prone when used casually.
Strings are easy to mistype and there's no way of ensuring that a string
literal on one line of code is equal to the string literal on another line at
compile time.  I don't think that the authors did anything wrong, per se.  They
are coping with a language limitation.  But it's my opinion that they are doing
it wrong.

##Roll your own reverse routes

So runtime packages are not adequate to deal with the problem of reverse
routes.  And code generation is out of the question.  The only option left is
to implement reverse routing at the application level.

I hear the gopher crying out, "Oh. Fuck. No." Just hear me out.  Let's just
make simplifying assumptions that make the problem of reverse routes tractable
from the development perspective.

##Throw out all the constraints

As a rule, routes should contain no more than one path parameter.  It's hard to
construct routes like that from a client perspective.

It's probably OK to have 'suffix parameters' that span multiple url segments.
So let's allow those.

Format constraints on path parameters are really crappy too.  Let's just ignore
them and have the individual http.Handlers parse strings and report errors
themselves.

There are even fewer format constraints if we just say path parameters have to
appear at the end of the path.  So let's do that.  And since handlers will be
checking parameter format themselves let's just say that all path parameters
are suffix parameters.

Wow, OK. This is getting very managable. We are now only supporting routes of
the form (using the "github.com/gorilla/mux" syntax)

    /foo/{id:.*}
    /bar

Let's take a hint from "net/http" and clean up the parameter syntax (since
there is at most one).

    /foo/
    /bar

Cool, now lets define functions to work with route parameters.

{%highlight go%}
// Tail returns the suffix of path matched by pat.
// If pat does not match path the empty string is returned.
func Tail(pat, path string) string {
    if !strings.HasSuffix(pat, "/") {
        return ""
    }
    suf := strings.TrimPrefix(path, pat)
    if len(suf) == len(path) {
        return ""
    }
    return suf
}

// Reverse returns the pat URL with parameter param.
// If pat does not accept paramaters, pat is returned.
func Reverse(pat string, param string) string {
    if !strings.HasSuffix(pat, "/") {
        return pat
    }
    return pat + param
}
{%endhighlight%}

I like this.  There is a little runtime unsafety.  But it is possible to
detect an error by checking equality of the return value to pat.  But in a lot
of cases that is probably not really necessary.  Presuming pat does point to an
endpoint in your service then you probably didn't mess up the paramaters.

OK.  OK.  That was a really weak statement I just made about messing up
parameters.  But that's all I have.  I think it's more important to make sure
that pat is what you want it to be.

##Define routes as a struct

So the simplifications made to the problem of reverse routes got rid of any
problems multiple parameters, or named parameters could have introduced.  But
then there is the problem of 'named routes' as they're called in
"github.com/gorilla/mux".

Using strings to store and retreive routes is error prone.  The simple solution
would be to store names in a variable.  But then the name is useless because
the variable has a name.  So just define variables containing the routes
themselves.  That's a lot of variables to have in some external state.  So just
define the routes in a struct.

{%highlight go%}
type ShelterRoutesV1 struct {
    Pets      string
    Adoptions string
}
{%endhighlight%}

##Initialize routes without names

To avoid defining routes without initializing them with patterns always
initialize values using object liternal notation without named fields. 

{%highlight go%}
var routes = ShelterRoutesV1{
    "/v1/pets/"
    "/v1/adoptions/"
}
{%endhighlight%}

##Now you have reverse routes

{%highlight go%}
var Pets = function(w http.ResponseWriter, r http.Request) {
    if r.Method = "POST" {
        id := uuid.New()
        // ... create a pet
        w.Header().Set("Location", Reserve(routes.Pets, id))
        w.WriteHeader(http.StatusCreated)
        return
    }
    id := Tail(routes.Pets)
    if uuid.Parse(id) == nil {
        http.Error(resp, "not found", http.StatusNotFound)
        return
    }
    fmt.Fprintln("pet %s", id)
}

func Adoptions = function(w http.ResponseWriter, r http.Request) {
    http.Error(resp, "not implemented", http.StatusNotImplemented)
}

mux := http.NewServeMux()
mux.HandleFunc(routes.Pets, Pets)
mux.HandleFunc(routes.Adoptions, Adoptions)
{%endhighlight%}

##Problems

This implementation of reverse routes is not perfect.  There is no guarantee
that `mux.HandleFunc` is called for each field in `ShelterRoutesV1`.  I suppose
a helper function could take `mux`, `routes`, and a `map[string]http.Handler`
then return an error if not all fields have a http.Handler defined for their
value. But I'm not sure that would be convenient enough to use.

##Other notes

I've played around with ways to describe routes without a string-based DSL.  I
didn't like the outcome vary much (and reverse routing is hard).  I'm not
saying it can't be done.  But I no longer think it's a good idea.
