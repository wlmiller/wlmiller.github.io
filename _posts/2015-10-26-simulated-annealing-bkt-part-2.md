---
layout: post
title:  "Applying lessons from a physics Ph.D. to educational data mining: part 2"
date:   2015-10-26 17:00:00
categories: monte-carlo educational-data-mining
author: Neal Miller
---
This is a continuation of [part 1]({% post_url 2015-10-19-simulated-annealing-bkt-part-1 %}).  I'd encourage you to read that part first.  When we left off, I'd discussed what BKT is and what it's used for, but I'd left off describing the important step of *how* the relevant parameters in BKT are obtained.

## Fitting BKT Parameters
At a high level, the process for fitting BKT parameters is the follow: collect a bunch of data on students solving series of problems on a KC (remember, the parameters described above are *KC-specific*), and then find the combination of four parameters \\(P(G), P(S), P(L), P(K_0)\\) that best fits the data -- that is, where actual student performance on each problem is closest to that predicted by the model.

Finding this combination of parameters is non-trivial.  We can usually set some limits on the parameters (for example, \\(P(G)\\) and \\(P(S)\\) should both be less than 50%), but we still have a large, four-dimensional search space, and the parameters are all interconnected in a very nontrivial way.  Furthermore, BKT suffers from the *identifiability problem* first demonstrated by Beck & Chang<sup>1</sup>: it is possible for different sets of four parameters to give identical student performance predictions.  (A more troubling problem, which I won't address, is that these different models can give very different estimates of student knowledge).

So given all that, how does one determine the best values to use for these parameters?  There are a couple of standard methods.

## Brute force
This is Ryan Baker's method,<sup>2</sup> and is the one I started out using.  Conceptually, it is almost trivial.  In Baker's BF algorith, you simply loop over the four parameters in a grid; once you've completed the whole grid, you take the region of the grid that gave the best prediction (measured by root mean square error or RMSE between predicted and actual student performance), and run a finer grid search over that region.  The combination in that finer grid that gives the lowest RMSE is the winner.

There are a couple of benefits to this method:

* Conceptually very simple.
* Assuming your grids are not *too* course, you don't need to worry about local minima.

The problems are obvious, though.

* The time complexity of this algorithm is *exponential* both in the number of parameters (this will become an issue below).  The implementation of this algorithm is a quadruply-nested loop; it just happens that for four parameters and an intial grid size of 100 (for a required ~100 million) is doable on a modern PC in a few hours.
* If your grid size is too large (or the result is strongly dependent on the values of the parameters), you could flat-out miss the minimum.  Even if it's not, you're not finding a true local (or global) minimum - you're finding the global minimum among your sampled grid points.

### Expectation Maximization
Expectation Maximization (EM) is the standard method for finding parameter estimates.  The idea behind EM is relatively simple:

1. From an initial guess at the parameters, calculated the expected values of the hidden nodes (the \\(P(K_n)\\)s).
2. Using the hidden node values, calculate new value for the parameters (given values of all nodes, this is easy - it's just counting the various outcomes).
3. Using these calculated parameters, go back to step 1 and repeat until the calculation converges; i.e., until the parameters stop changing

The benefits of this method are:

* The computational complexity is much lower, particularly in the number of features.  Because it's a directed search, much less feature space needs to be covered.
* Because it's not on a grid, you're guaranteed to find a true local minimum.

However, the downsides are important:

* Note the word "local" above.  Expectation Maximization will strictly find local minima, and will easily get stuck in false basins which may be a much worse fit than the true global minimum.  To get around this, *random restarts* (literally running the computation several times with different initial guesses) are often used - this goes a long way to erasing the gain from using a "smarter" algorithm.
* The standard implementation of EM in BKT, the [Bayes Net Toolbox for Student Modeling (BNT-SM)](http://www.cs.cmu.edu/~listen/BNT-SM),<sup>3</sup> requires a MATLAB license and is quite slow -- Baker claims that there is essentially no performance difference between his brute force method and the EM algorithm in BNT-SM.

## Why neither of those methods worked for me

A problem arose as I worked on my first EDM paper.<sup>4</sup>  In that work, we used data from students working through the [Reasoning Mind](http://reasoningmind.org) math education platform.

We divided problems into three types: conceptual problems (which existed explicitly to teach the student), problem-solving items (which served to test the student's learning), and fluency-building games (timed, computationally simply problems).  The question was -- can we better-model students if we don't assume \\(P(G)\\), \\(P(S)\\), and \\(P(L)\\) are idential across all problems for a given KC?  More specifically: if we model student performance using the same nodes and connections as before, but the edge weights are different depending on the type of problem, do we get appreciably lower RMSEs?

As I mentioned above, I started out using Baker's brute force algorithm, and had reservations about switching to BNT-SM (not least of which was the MATLAB license cost of $2,000, but which also included the local minimum problem described above).  However, recall that the brute force algorithm has exponential complexity in the number of parameters.  I wanted to fit up to 10 parameters per KC (\\(P(K_0)\\), plus three each of \\(P(G)\\), \\(P(S)\\), and \\(P(L)\\)) -- we're talking the original algorithm complexity *more than squared*.  It was completely infeasible.

I did try some other ideas before I moved on to the one I'll describe in part 3.  One that I thought was promising was a sort of "ratcheting" algorithm similar to EM -- I would fit each group of parameters (for each of the three problem types) separately, and keep going until convergence.  However, it still took a long time (days to weeks vs. hours) and didn't converge well.  I needed a new method; ideally, one which was not stymied by local minima, but would complete before the heat death of the universe on a reasonable PC.

#### References

1. Beck, J.E., Chang, K. Identifiability: A Fundamental Problem of Student Modeling. *Proceedings of the 11th International Conference on User Modeling* (2007).
2. Baker, R.S.J.d., *et al.* Contextual Slip and Prediction of Student Performance After Use of an Intelligent Tutor. *Proceedings of the 18th Annual Conference on User Modeling, Adaptation, and Personalization* (2010).
3. Chang, K., Beck, J., Mostow, J., Corbett, A. A Bayes Bet Toolkit for Student Modeling in Intelligent Tutoring Systems. *Proceedings of the 8th International Conference on Intelligent Tutoring Systems* (2006).
4. Miller, E.L., Baker, R. S., Rossi, L.M. Unifying Computer-Based Assessment Across Conceptual Instruction, Problem-Solving, and Digital Games. *Technology, Knowledge and Learning* 19, 165 (2014).
