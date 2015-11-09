---
layout: post
title:  "Some thoughts on technical debt"
date:   2015-11-09 17:00:00
categories: software-development technical-debt
author: Neal Miller
---
I've been thinking a lot about technical debt, lately.
There a couple of big reasons for that related to my day job: first, that we have a lot, and second, that I think we're in a strong position to pay it down.

## We have a lot
I don't think I'm saying anything shocking when I say that there is a ton of technical debt built up in our codebase.
For a small shop, we have a huge diversity within our code base: we have critical components in C, Python, JavaScript, [Go](https://golang.org/), and [LabView](http://www.ni.com/labview/), to name a few.
The reason for that is that we're doing something fairly complex: we write the software that controls the entire process by which data is transformed from a 4-20 mA current reading into a display in our desktop software, is replicated to our cloud server, is tranformed and summarized for [analytics](https://cloud.rigminder.com/analytics/XXX/) purposes, and any other service we provide to our clients.

Our codebase has grown fairly organically as our needs have expanded, often under severe time constraints.
While it's very important that our software is reliable and secure, the speed with which we move and our small size (we're way too small to have a dedicated software architect) mean some pieces, especially those that interoperate with many others, are getting more and more difficult to maintain and enhance while maintaining the reliability and security that are so vital to our clients.

## Paying it down
One of the reasons we have so much technical debt in the first place is also a reason we're in a strong position to pay it down: we're small.
We pride ourselves (and indeed consider it a strong piece of our competitive advantage in our industry) on being lower-case-a *agile*.
We don't have a huge bureaucracy, and we don't have egos about our code.

In addition to that, we've come to a place where we have a very solid, feature-complete offering.
Of course, we're always working on new offerings and improving our product, but we're starting to move into a position to spare some of our development capacity for the very important task of improving the quality of our code and its overall architecture.

While not every decision we've made has resulted in the cleanest and most maintainable codebase imaginable (hence, technical debt), there is one big set of decisions that work very strongly in our favor.
From the beginning, our offering has centered around a "plug-and-play" design.
Each system is unique, but we make it easy to use different components, including some from other vendors, as part of a full system.

As a side-effect of this, almost by accident, we have developed a **service-oriented architecture**.
While we have quite a bit of functionality, we don't have monoliths: pretty much everything in our full system (from individual sensors to our analytics website, and everything in between) is a collection of small services, communicating mostly via TCP and HTTP.

That means a couple of things that put is in a strong position to pay down our technical debt.
First, it means that developing comprehensive test suites (our current set of tests suites is *uneven* at best) for different pieces of functionality is relatively simple.
Because we have a service-oriented architecture, different functional units are almost by definition loosely coupled, so isolating functionality for testing is straightforward.
Second, and most importantly, it means that we can (with great care, of course) dramatically refactor difference services, or (and I think this will be an important factor moving forward) scrap and rewrite them altogether.
As long as we're thorough in our testing and don't disrupt our APIs, we can swap out entire pieces of funcionality for more maintable versions with a minimum of disruptions, and as we move through our system making such changes, we can even begin to make the larger, architecture-level changes that are the eventual goal.