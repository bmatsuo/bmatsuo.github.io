---
layout: post
title: "Structured ad-hoc concurrency"
date: 2014-03-20 17:16:39 -0700
categories: go concurrency
---

I've been toying around with a model for structuring concurrent code in Go.

I've enjoyed using the for-select concurrency model discussed in Sameerr Ajmani's
talk, [Advanced Concurrency Patterns in Go](). But occasionally I find the
complexity to be growing more than I would like. I've got lots of channels.
Channels passing channels which need temporary channel variables while some
concurrent process is executing. There are a couple problems that manifest.
Let's look at an example. An interface to an external service that allows
streaming results.

{%highlight go%}
type Query string
type Row struct { data map[string]string; err error }
func execute(q Query, rs chan<- Row, kill <-chan struct{}) {
    defer close(rs)
    resp, err := http.Get("http://example.com/stream?q="+string(q))
    if err != nil {
        rs <- Row{"", err}
        return
    }

    select {
    case <-kill:
        return
    default:
    }

    // decode rows into a stream
    go func(rc io.ReadCloser, decrs chan<- Row, kill <-chan struct{}) {
        defer rc.Close()
        dec := json.NewDecoder(rc)
        defer close(decrs)
        for {
            row := Row{data: make(map[string]string)}
            err := dec.Decode(row.data)
            if err == io.EOF {
                return
            }
            if err != nil {
                row = Row{nil, err}
            }

            select {
            case <-kill:
                return
            case decrs<-row:
            }
        }
    }(resp.Body, rs, kill)
}
{%endhighlight%}

The concurrency control loop (for-select) is pretty tight. Although really the
call to dec.Decode should be async. But this fairly simple example produced a
lot of variables that make things confusing. For instance, `_rs` is a variable
equal to `rs` only when we want to send over `rs` and nil otherwise. This makes
the for-select loop clean. But it is confusing to read.

If there was an accepted practice here things might not be so bad. For instance,
"`_xxx` is equal to `xxx` if it is ready to send/recv and nil otherwise". OK,
then people know what I mean. I know what they mean. That's ok.

But really its pretty gross. What I really don't like is that state transitions
are not obviously labeled. By assigning to `_rs` I enter an 'output' state.

An idea I'm toying around with is using scoped-types to maintain state
transitions.

{%highlight go%}
type Query string
type Row struct { map[string]string; error }
func execute(q Query, rs chan<- Row, kill <-chan struct{}) {
    defer close(rs)
    resp, err := http.Get("http://example.com/stream?q="+string(q))
    if err != nil {
        rs <- Row{"", err}
        return
    }

    decrs := make(chan Row) // decoded rows
    go func() {
        defer resp.Body.Close()
        dec := json.NewDecoder(resp.Body)
        defer close(decrs)
        for {
            row := Row{data: make(map[string]string)}
            err := dec.Decode(row.data)
            if err == io.EOF {
                return
            }
            if err != nil {
                row = Row{nil, err}
            }
            select {
            case <-kill:
                return
            case decrs<-row:
            }
        }
    }()

    type state struct {
        rs chan<- Row
        decrs <-chan Row
    }
    var s state
    var row Row

    ready := func() { s.decrs = decrs }
    killed := func() { kill = nil }
    decoded := func(r Row) { row = r; s = state{ rs: rs } }

    ready()
    for {
        select {
        case <-kill:
            killed()
        case row, ok := <- s.decrs:
            if !ok {
                return
            }
            decoded(row)
        case s.rs<- v.row:
            ready()
        }
    }
}
{%endhighlight%}
