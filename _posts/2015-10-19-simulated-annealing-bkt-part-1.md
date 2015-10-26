---
layout: post
title:  "Applying lessons from a physics Ph.D. to educational data mining: part 1"
date:   2015-10-19 17:00:00
categories: monte-carlo educational-data-mining
author: Neal Miller
---
I've had quite a varied career, and it's not always obvious how my experience in one field directly benefits my work in another.  I'd like to tell the story of how lessons learned during my Ph.D. studies in Chemical Physics became directly applicable to my later work in [educational data mining](http://en.wikipedia.org/wiki/Educational_data_mining) (EDM).  At the outset, I want to make clear that I don't make any claim that my eventual solution to the problem I'll describe was some transcendent stroke of genius -- quite the contrary.  This is a story about how a unique background made a simple solution from other fields immediately obvious to me in a way that it hadn't been to my colleagues in EDM.

After I completed my Ph.D. at Columbia University, I landed working for an education technology company.
After a few months, I found myself doing EDM in collaboration with Associate Professor [Ryan Baker](http://www.columbia.edu/~rsb2162/) at Teachers College Columbia University.

## Bayesian Knowledge Tracing
Bayesian Knowledge tracing is a technique by which a student's knowledge of specific topic (called "knowledge components" or KCs) can be estimated based upon their performance on a series of questions, accounting for the fact that they may miss questions they know how to answer ("slip"), get lucky on questions they don't ("guess"), and that that knowledge changes (they learn, one hopes)

The BKT model can be represented by a directed acyclic graph.

<div style="width:100%;text-align:center">
	<svg height="170" width="320">
		<marker id="triangle"
			viewBox="0 0 10 10" refX="0" refY="5"
			markerUnits="strokeWidth"
			markerWidth="4" markerHeight="3"
			orient="auto">
			<path d="M 0 0 L 10 5 L 0 10 z" />
		</marker>
		<rect x="10" y="10" rx="15" ry="15" width="100" height="50" stroke="black" stroke-width="5" fill="none"/>
		<text x="41" y="40">P(K<tspan dy="6" style="font-size:8pt">0</tspan><tspan dy="-6">)</tspan></text>

		<line x1="110" y1="35" x2="145" y2="35" marker-end="url(#triangle)" stroke="black" stroke-width="5" />
		<text x="116" y="20">P(L)</tspan></text>
		
		<rect x="160" y="10" rx="15" ry="15" width="100" height="50" stroke="black" stroke-width="5" fill="none"/>
		<text x="191" y="40">P(K<tspan dy="6" style="font-size:8pt">1</tspan><tspan dy="-6">)</tspan></text>
			
		<line x1="260" y1="35" x2="295" y2="35" marker-end="url(#triangle)" stroke="black" stroke-width="5" />
		<text x="266" y="20">P(L)</tspan></text>
			
		<ellipse cx="60" cy="135" rx="50" ry="25" stroke="black" stroke-width="5" fill="none" />
		<text x="50" y="140">C<tspan dy="6" style="font-size:8pt">0</tspan><tspan dy="-6"></tspan></text>
			
		<line x1="60" y1="60" x2="60" y2="95" marker-end="url(#triangle)" stroke="black" stroke-width="5" />
		<text x="22" y="80">P(G)</tspan></text>
		<text x="22" y="100">P(S)</tspan></text>
			
		<ellipse cx="210" cy="135" rx="50" ry="25" stroke="black" stroke-width="5" fill="none" />
		<text x="200" y="140">C<tspan dy="6" style="font-size:8pt">1</tspan><tspan dy="-6"></tspan></text>
		
		<line x1="210" y1="60" x2="210" y2="95" marker-end="url(#triangle)" stroke="black" stroke-width="5" />
		<text x="172" y="80">P(G)</tspan></text>
		<text x="172" y="100">P(S)</tspan></text>
	</svg>
</div>

In this graph, there are a couple of parameters that are properties of the student:

* \\(C_n\\) represents whether or not the student answered problem \\(n\\) correctly.  This is an *observed* node -- once the student attempts the problem, we know explicitly the value of \\(C_n\\).
* \\(K_n\\) represents whether or not the student "knows" the KC the problem \\(n\\) tests.  \\(K_n\\) is a *hidden* node, and the one we're interested in estimating -- we cn infer a probability that the student knows the material based upon their performance on the problems.

There are also a few parameters which are considered to properties of the KC (in the most common version of BKT).  These are the edge weights in the graph -- various probabilities relating the values of connected nodes:

* \\(P(G)\\) is the *guess* probability -- the probability that the student answers a question right, igven they *don't* know the material.  That is, it's the probability \\(P(C_n \vert \neg K_n)\\).
* \\(P(S)\\) is the *slip* probability - the probability that the student misses the qustion, given they *do* know the mateiral, or \\(P(\neg C_n \vert K_n)\\).
* \\(P(L)\\) is the probability that the student *learned* the KC in the process of answering the previous question: \\(P(L_n \vert \neg L_{n-1})\\).

Note that (again, in the most common form of BKT), there's no "forgetting" paramater.  Once a student has learned a KC, the model assumes they will know it forever.  Obviously, that's not a great assumption, and forgetting has been incorporated into the BKT model at times, to varying levels of success.

So in principle, if we know a student's performance on a series of problems, we can estimate (in real-time or after-the-fact) the probability that they'd learned the related KC at each step -- assuming, of course, we have values for \\(P(G)\\), \\(P(S)\\), and \\(P(L)\\), as well as the prior \\(P(K_0)\\) -- that is, the probability that the student started out already knowing the material.  So where do we get those?  That's where I'll begin [part 2]({% post_url 2015-10-26-simulated-annealing-bkt-part-2 %}) .
