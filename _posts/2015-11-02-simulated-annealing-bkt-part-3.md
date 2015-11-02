---
layout: post
title:  "Applying lessons from a physics Ph.D. to educational data mining: part 3"
date:   2015-11-02 17:00:00
categories: monte-carlo educational-data-mining
author: Neal Miller
---
This is the third in a series of post in which I'm describing how some of my work in educational data minining (EDM) unexpectedly intersected with my background as a physics Ph.D.  You can find the first two parts here: [part 1]({% post_url 2015-10-19-simulated-annealing-bkt-part-1 %}), [part 2]({% post_url 2015-10-26-simulated-annealing-bkt-part-2 %}).

In part 1, I gave a description of the model behind Bayesian Knowledge Tracing (BKT).  In part 2, I gvae in-depth descriptions of the methods by which BKT parameters are obtained, and decribed why those methods weren't well-suited for my needs.  In this part, I'll describe the method I ended up using.  This work is decribed in reference 1, below.

##Simulated Annealing
The technique I applied for fitting BKT parameters was simulated annealing, defined [on Wikipedia](https://en.wikipedia.org/wiki/Simulated_annealing) as:
> [A] probabilistic technique for approximating the global optimum of a given function. Specifically, it is a metaheuristic for approximate global optimization in a large search space. It is often used when the search space is discrete (e.g., all tours that visit a given set of cities). **For problems where finding the precise global optimum is less important than finding an acceptable global optimum in a fixed amount of time, simulated annealing may be preferable to alternatives such as brute-force search or gradient descent.**

(Emphasis mine).  I'll give a more detailed (and more clear) description of the methof below but note that bold sentence: this may be exactly what we're looking for.

Simulated annealing is a direct analogy to [annealing](https://en.wikipedia.org/wiki/Annealing_(metallurgy) in metallurgy.  The basic process in metallurgy is simply the heating and controlled cooling of a material in order to achieve a more ordered end-product.  If a material at a high temperature is rapidly cooled, the component atoms within the metal can become trapped in local maxima rather than being able to reach a global energy minimum, which will generally be a crystal.

In my Ph.D. work, the analogy was very direct.  I executed and analyzed computer simulations of physical systems, with the goal of understanding the ways in which different systems organized themselves into ordered structures such as crystals.  In almost every case, this meant simulating the system starting at a high temperature and slowly decreasing the temperature until self-organization occured.

### An aside about Monte Carlo simulations
I won't give a comprehensive description of Monte Carlo simulations, but it's worth giving a quick run-down, as the application of simulated annealing to the BKT problem becomes very straightforward.

Consider a system of \\(N\\) particles in a box (I'll leave the term "particles" generic, but there are some qualifications to make this technique applicable).  They are subject some interaction potential between particles -- the details are unimportant.  Assume the particles are completely rotationally symmetric (so particle rotation can be ignored).  Then we have a system with \\(3N\\) degrees of freedom.

For each step of the simulation, a particle is chosen at random, and a trial move is generated, in some random direction and for some random distance (within a defined limit).  The potential energy difference between the particle after this trial move and the particle before the trial move, \\(\Delta E\\), is calculated.  The trial move is then accepted or rejected based on the Metropolis criterion,

$$
P_\mathrm{accept} = \mathrm{min}\left(1, \mathrm{exp}\left(-\frac{\Delta E}{T}\right)\right)
$$

where \\(T\\) is the temperature.  Note a few observations:

1. If \\(\Delta E \leq 0\\) (potential energy decreases), the move is always accepted.
2. if \\(\Delta E >0 0\\), then acceptence is less and less likely as the energy difference grows.
3. For a given positive \\(\Delta E\\), acceptance is more likely the higher the temperature.

The affect of these factors is that the system is generally drawn to lower-energy states, but is allowed to increase in potential energy so that it may explore the phase space, particularly at high energies.

## Application to BKT
Hopefully the application of the above technique to BKT parameter fitting is now obvious.  If we treat each BKT parameter as a degree of freedom, we have between 4 and 10 degrees of freedom in our application.  A random initial guess can be made, and then trial moves of random increases and decreases in the parameters can be made.

In our analogy, the root mean square error (RMSE, as described in part 2) serves as the potential energy to be minimized, and the temperature is simply the denominator in the acceptance criterion.  Start at some "high" value for temperature (we used 0.005), and then decrease is steadily -- for example, halving it every 10,000 of annealing steps.  Fitting is considered complete when overall RMSE doesn't decrease in 10,000 steps after a temperature halving.

## Results
To verify the method, I compared the results of simulated annealing to the results of brute force fitting for two large sample data sets, using traditional BKT (4 parameters).  The result are detailed in the paper,<sup>1</sup>, but the gist is a follows: convergence was achieve in about 2% of the time (i.e. 50 time faster) and goodness-of-fit was essentially equivalent.  An importantly, because the algorithm is no longer exponential in its complexity, extension to more complex algorithms with up to 10 parameters per KC became very feasible.

## Lessons Learned
As I mentioned in part 1, I'm not making any claim to any stroke of genius or even an especially non-obvious insight into the problem solution.  Quite the contrary; I think the solution would be immediately obvious to many people, but it hadn't become obvious to people working in the EDM field.

I'm quite confident that if I'd never worked in this space, someone else would have come along and come to the same realization about the obvious fit of simulated annealing to this problem space that I did.  I just happened to be working on the right problem, at the right time, and with the right (non-obvious and unconventional) background to immediately settle on a workable solution.

### References
1. Miller, W.L., Baker, R. S., Rossi, L.M. Unifying Computer-Based Assessment Across Conceptual Instruction, Problem-Solving, and Digital Games. *Technology, Knowledge and Learning* 19, 165 (2014).