---
layout: post
title:  "First Impressions of Go"
date:   2015-04-30 15:49:00
categories: go python
author: Neal Miller
---
I've been looking for an excuse to use [Go](http://golang.org) in my work for a while now -- it's gotten a lot of hype, and the characterization it's often given of being something like a compiled, statically-typed Python was intriguing to me.
I use Python a great deal and have for years; it's always been my first choice for many tasks.
However, anyone who's worked with a project of any size in a dynamically-typed language comes to appreciate the help the compiler provides in statically-typed languages like Java.

Recently, just such an opportunity arose.
At work, we have a Python script that's responsible for data replication - data stream in from client sites via TCP, and this script is responsible for routing it to its proper location.
It's a relatively simple script comprising a few hundred lines of Python, but it's a critical piece of infrastructure.

When the server gets a TCP connection, it spawns a [thread](http://docs.python.org/2/library/threading.html) to parse the incoming data and add it to a queue to be inserted into the database.
The threading behavior of this script has always been a little screwy, which isn't too shocking; due to the [Global Interpreter Lock](https://wiki.python.org/moin/GlobalInterpreterLock) (GIL), Python can't do true multithreading.
Rather than fighting the GIL for what was always going to be suboptimal, we decided to consider another language -- Python's great, but not the best tool for every job, and in particular, didn't seem like the best tool for this one.

In our minds, the important features of a language to be considered a good fit for this problem were:

* **Concurrency**: This isn't a hard or computationally-intensive problem, but it's important to be able to spawn a thread to handle each incoming connection and *get out of the way*.  The GIL didn't directly cause issues for us in this regard -- were wen't having problems with blocking -- but a new solution should handle this much better.
* **Productivity**: By this I mean leads to high developer productivity.  This is the reason every program's not written in C (or assembly) -- developer time isn't free, and higher-level languages such as Python allow one to exchange computational performance for developer time.  Python is a relatively concise language and there was already plenty of existing Python expertise on our team -- for many languages, those characteristics wash out benefits in the other categories.

Below I'll talk about how Go addressed each of these items, and at the end, will briefly discuss a couple of shortcomings with Go.
 
 <h2>Concurrency</h2>

Concurrency in Go is easy.
That's a big part of the point.
Concurrency works Go using [Goroutines](https://gobyexample.com/goroutines).
Goroutines are simply functions that can run concurrently, and creating one is very simple.  Imagine we have the following code:
{% highlight go %}
func foo() {
    // Long-running task
}

func bar() {
    // Long-running task
}

func main() {
    foo()
    bar()
}
{% endhighlight %}

This is a contrived obvious example where we should be using concurrency - `foo` and `bar` should be running in parallel.  In Go, that's as easy as modifying `main` as follows:
{% highlight go %}
func main() {
    go foo()
    bar()
}
{% endhighlight %}

`foo` is now run as a goroutine, right alongside `bar`.  But what if we want communication with these routines?  Enter [channels](https://gobyexample.com/channels).  Concretely, imagine we want `bar` to send a bunch of numbers to `foo`, which will then print them out.
{% highlight go %}
func foo(ch chan int) {
    for {
        fmt.Println(<-ch)
    }
}

func bar(ch chan int) {
    for i := 0; i < 10; i++ {
        ch <- i
    }
}

func main() {
    var ch = make(chan int)
    go foo()
    bar()
}
{% endhighlight %}

`bar` now sends the numbers 0-9 to `foo`, which prints them to the screen (note that the program won't wait for `foo` to complete - in this case, 9 won't be printed because the program will exit right after `bar` sends it).

With that introduction out of the way, here's a rough sketch of the bones of part of the Python version of the replication script:
{% highlight python linenos %}
class WriteThread(threading.Thread):
    def __init__(self, queue):
        self.queue = queue

    def run(self):
        while 1:
            data = queue.get()
            # handle the data

class HandlerThread(threading.Thread):
    def __init__(self, queue, conn):
        self.queue = queue
        self.conn = conn

    def run(self):
        data = self.socket.recv(n)
        # parse data
        # split into pieces
        for piece in pieces:
            queue.put(piece)

def replicate(socket, queue):
    while 1:
        conn, remote_addr = socket.accept()
        HandlerThread(queue, conn).start()

def main():
    socket = socket.socket(...)
    #set up socket

    global queue
    queue = Queue.Queue()

    WriteThread(queue).start()

    replicate(socket, queue)
{% endhighlight %}

The basic idea is relatively simple: every time a connection comes in, a new `HandlerThread` is spun up to deal with it.
The data are split into pieces and added to a `queue`, which a running `WriteThread` is constantly pulling from to deal with.
Below, essentially the same idea is expressed in Go.
{% highlight go linenos %}
var ch = make(chan string)

func write() {
    for {
        var data <- ch
        // handle the data
    }
}

func handle(conn net.Conn) {
    data = make([]byte, n)
    var len, _ = conn.Read(data)
    // parse data
    // split into pieces
    for _, piece := range(pieces) {
        go func() { ch <- piece }
    }
}

func replicate(listener net.Listener) {
    for {
        var conn, _ = listener.Accept()
        go handle(conn)
    }
}

func main() {
    var listener, _ = net.Listen(...)

    go write()

    replicate(listener)
}
{% endhighlight %}

That this works basically the exact same way as the Python version, but now I'm using a Go channel as my data queue.  Note that on line 16, I spawn a new goroutine using a closure.
By default, Go channels are unbuffered - a routine attempting to write to a channel will block if the channel is not empty.
They can be made buffered, but this solution was sufficient for me - because writing to the channel is in its *own* goroutine, it won't block the main `handle` routine.
In practice, this means there are often thousands of goroutines at a time, which is no problem at all - goroutines are very lightweight.

In my opinion, the Go code is simply clearer.
The two pieces of code work in almost the same way (although the mechanics of a Python `Queue` vs. a Go `chan` are different), but concurrency is such an ingrained part of Go that the Go syntax feels much less awkward.

Of course, that's secondary to the main point - the Go code is truly multithreaded.
Really, Go only had to not be significantly worse to write to win this category since Python completely fails on multithreading -- being nicer to write is a bonus. 
 
<h2>Productivity</h2>

Before starting this conversion, I'd never written a line of Go, but I'd written thousands of lines of Python over more than 10 years.
While I enjoy learning new and wanted to look into Go anyway, it didn't make sense to pick up a huge, complex language (e.g. C++) or one that is dramatically different from the "C tradition" (e.g. Rust, Haskell) -- we wanted to choose a language that worked better for the purpose, but in a similar basic way to Python (or C).
Beyond some very superficial syntactic differences, it's very easy to recognize what a Go program does for anyone with a degree of fluency in C-like language.

Go is also a small language.
There are a small number of keywords, and most functionality is brought in from compact, self-contained modules.
It's much like Python in this regard, but to me it feels even smaller.
The similarity to C and the small size mean that it was a very simple language to pick up - going from a blank editor screen to a working replication program took a day or so.

There's a general perception that statically-typed languages such as Go tend to be more verbose -- so the tradeoff for benefits like speed and type safety is more lines of code.
However, I was surprised to discover, after completing the conversion of the above code from Python to Go, that the two were essentially identical in length.

<h2>Conclusions</h2>

We were able to accomplish what we set out to do using Go.
