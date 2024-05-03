# Literature

## Foundational Papers

Hayes, R. L.; Vilseck, J. Z. & Brooks III, C. L.
Approaching Protein Design with Multisite λ Dynamics: Accurate and Scalable Mutational Folding Free Energies in T4 Lysozyme 
Protein Science, 2018, 27, 1910-1922  
[https://onlinelibrary.wiley.com/doi/10.1002/pro.3500](https://onlinelibrary.wiley.com/doi/10.1002/pro.3500)  
This paper is what the T4 lysozyme practice problem is based on. It is the proof of concept paper that began using λ dynamics for protein mutations.

Hayes, R. L.; Armacost, K. A.; Vilseck, J. Z. & Brooks III, C. L.
Adaptive Landscape Flattening Accelerates Sampling of Alchemical Space in Multisite λ Dynamics 
Journal of Physical Chemistry B, 2017, 121, 3626-3635  
[https://pubs.acs.org/doi/10.1021/acs.jpcb.6b09656](https://pubs.acs.org/doi/10.1021/acs.jpcb.6b09656)  
The ALF algorithm has changed somewhat since this paper was written, but this still gives a good overview of how it works. Every λ dynamics simulation we run starts with ALF, so understanding it is pretty important.

Braun, E.; Gilmer, J.; Mayes, H. B.; Mobley, D. L.; Monroe, J. I.; Prasad, S. & Zuckerman, D. M.
Best Practices for Foundations in Molecular Simulations 
Living Journal of Computational Molecular Science, 2019, 1, 5957  
[https://livecomsjournal.org/index.php/livecoms/article/view/v1i1e5957](https://livecomsjournal.org/index.php/livecoms/article/view/v1i1e5957)  
A good paper reviewing what molecular dynamics is and how it works. lambda dynamics runs on top of molecular dynamics simulations, so understanding molecular dynamics is an important first step to understanding what we do.

Kong, X. & Brooks III, C. L.
λ-Dynamics: A New Approach to Free Energy Calculations 
Journal of Chemical Physics, 1996, 105, 2414-2423  
[https://doi.org/10.1063/1.472109](https://doi.org/10.1063/1.472109)  
The original λ dynamics paper. A lot has changed, but it’s worth a read.

Knight, J. L. & Brooks III, C. L.
λ-Dynamics Free Energy Simulation Methods 
Journal of Computational Chemistry, 2009, 30, 1692-1700  
[https://onlinelibrary.wiley.com/doi/10.1002/jcc.21295](https://onlinelibrary.wiley.com/doi/10.1002/jcc.21295)
A good review of how lambda dynamics worked in 2009. A lot has changed, but it gives a good overview of the concepts.

Knight, J. L. & Brooks III, C. L.
Applying Efficient Implicit Nongeometric Constraints in Alchemical Free Energy Simulations 
Journal of Computational Chemistry, 2011, 32, 3423-3432  
[https://onlinelibrary.wiley.com/doi/10.1002/jcc.21921](https://onlinelibrary.wiley.com/doi/10.1002/jcc.21921)  
All modern lambda dynamics simulations use implicit constraints. Understanding what they are and how they work is important for understanding how things actually work.

Knight, J. L. & Brooks III, C. L.
Multisite λ Dynamics for Simulated Structure-Activity Relationship Studies 
Journal of Chemical Theory and Computation, 2011, 7, 2728-2739  
[https://pubs.acs.org/doi/10.1021/ct200444f](https://pubs.acs.org/doi/10.1021/ct200444f)  
This is the paper that first introduced multisite lambda dynamics, the idea of running lambda dynamics with multiple perturbation sites.

Hayes, R. L.; Nixon, C. F.; Marqusee, S. & Brooks III, C. L.
Selection Pressures on Evolution of Ribonuclease H Explored with Rigorous Free-Energy-Based Design 
Proceedings of the National Academy of Sciences of the United States of America, 2024, 121, e2312029121  
[https://www.pnas.org/doi/10.1073/pnas.2312029121](https://www.pnas.org/doi/10.1073/pnas.2312029121)  
Most recent application of lambda dynamics to a large protein sequence space. Also the first prospective study on protein mutations with lambda dynamics.

WORKING HERE:

https://livecomsjournal.org/index.php/livecoms/article/view/v2i1e18378
