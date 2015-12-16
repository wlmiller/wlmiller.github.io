---
layout: post
title:  "Developing a simple Scheme interpreter using Rust"
date:   2015-12-15 17:00:00
categories: rust scheme interpreter
author: Neal Miller
---

I've always been interested in developing simple compilers and interpreters -- not because I have a desire to create my own language, but because it strikes me as an interesting challenge and a good way to get a better understanding of general principles of programming languages.
I've also recently been interested in the [Rust Programming Language](https://www.rust-lang.org/).
Combining the two interests and using Rust to develop some simple interpreters/compilers seemed like a fun way to spend some time.

## About Rust
Rust is a systems programming language developed by Mozilla Research.
It's compiled, strongly typed, and very fast; it's intended as a competitor to C++ for systems applications.
While the syntax of Rust is C-like, it has a few features borrowed from modern especially functional) languages which make it nice to program in; for example:

* Pattern maching
* A powerful type system including sum types (aka [tagged unions](https://en.wikipedia.org/wiki/Tagged_union))
* Type inference

However, there's one very unique feature about Rust that particularly interested me.
I'm generally of the opinion that the best reason to learn a language for fun is that it does something fairly unique (e.g. the strict functional purity of Haskell).
That feature in Rust is its guaranteed memory safety without garbage collection.

In Rust, manual memory management is unnecessary in the vast majority of cases -- instead, Rust keeps strict track of object lifetimes, using "move symantics" to ensure that it knows exactly when an object goes otu of scope and can be deallocated.
The details of this are their own, long, blog post, but the result is that once you are able to wrap your head around Rust's system of moving and borrowing variables, you can write efficient code that is memory- and thread-safe.

If you're interested in Rust, I'd encourage you to check out the [*The Rust Programming Language* book](https://doc.rust-lang.org/book/).

## Scheme Interpreter
Developing an interpreter was, as expected, interesting and enlightening; it also got me a lot of experience with fighting Rust's borrow checker.
You can find the code for my simple Scheme interpreter [here](https://github.com/wlmiller/rust-toys/tree/master/rscheme).

### Parsing
Parsing Scheme is very easy.
I won't dedicate much space to discussing it, except to say that I first tokenized the source, then parsed the resulting list of tokens into an abstract syntax tree.
The result of the parsing step is a recursive tree of `Node`s, which is an `enum` type defined to cover all the semantically distinct elements; for example, the integer 4 would be represented as a `Node::Int(4)`, and the complext number 2+3i would be represented as a `Node::Complex(3,2)`.

### Interpreting
This part was much more complex, and I wasn't thrilled with every decision I'd made as I got further into its implementation.
Conceptually, it's fairly simple - evaluate each `Node`, with component `Node`s being evaluated recursively.
Built-in keywords (such as `+`, `lambda`, or `and`) are implemented as methods on the `Interpreter` object and act on lists of `Node`, resulting in `Value`s.
The `Value` enum is very similar to the `Node`s resulting from parsing.

There are a couple of wrinkles which made the implementation of the interpreter trickier than what I've described so far; they're definitely lessons I'll take with me for future project of this type.

First is lexical scoping.
This is actually not too bad; essentially, when you enter a `lambda`, that lambda has its own variable scope, which is interior to the environment from which it was called.
Variable lookup involved starting the in the `lambda`'s scope and climbing up the hierarchy until you either find its value or reach the top.
That part is relatively straightforward; however, I found myself battling odd problems with getting the scopes right for lambdas defined within other lambdas.
I ended up essentially inlining as much as possible when the lambda is defined; this mostly worked, but I'm not convinced it will be completely flexible.

The second is tail-call optimization.
Without going into too much detail, tall-call optimization is a process by which the interpreter can detect when a recursive functionc all is the last statement in a function call; when that's the case, the interpreter doesn't need to hold the current call's stack in memory.
Again, implementation is relatively straightforward: essentially, at any point, you only evaluate the `Node` if you absolutely have to.
For must functions, you need to evaluate immediately; however, for a couple of functions (`if` and `lambda` calls come to mind) you may not.

Basically, for those situations, just return the unevaluated `Node` (actually, the `Node` appropriately wrapped as a `Value`).
In the interpreter, instead of a single `eval` call on the top `Node` in the tree, you then have a loop which terminates when you get anything besides a wrapped `Node` from the `eval` call.
The result is that, for a tail-recursive function, the interpreter essentially evaluates is iteratively -- the stack space for each call of the function is released before the next recursive call.