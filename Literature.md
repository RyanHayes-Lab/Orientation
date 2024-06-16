# Literature

## Foundational Papers

<!-- R712 -->
Hayes, R. L.; Vilseck, J. Z. & Brooks III, C. L.
Approaching Protein Design with Multisite λ Dynamics: Accurate and Scalable Mutational Folding Free Energies in T4 Lysozyme 
Protein Science, 2018, 27, 1910-1922  
[https://onlinelibrary.wiley.com/doi/10.1002/pro.3500](https://onlinelibrary.wiley.com/doi/10.1002/pro.3500)  
This paper is what the T4 lysozyme practice problem is based on. It is the proof of concept paper that began using λ dynamics for protein mutations.

<!-- R612 -->
Hayes, R. L.; Armacost, K. A.; Vilseck, J. Z. & Brooks III, C. L.
Adaptive Landscape Flattening Accelerates Sampling of Alchemical Space in Multisite λ Dynamics 
Journal of Physical Chemistry B, 2017, 121, 3626-3635  
[https://pubs.acs.org/doi/10.1021/acs.jpcb.6b09656](https://pubs.acs.org/doi/10.1021/acs.jpcb.6b09656)  
The ALF algorithm has changed somewhat since this paper was written, but this still gives a good overview of how it works. Every λ dynamics simulation we run starts with ALF, so understanding it is pretty important.

<!-- R960 -->
Braun, E.; Gilmer, J.; Mayes, H. B.; Mobley, D. L.; Monroe, J. I.; Prasad, S. & Zuckerman, D. M.
Best Practices for Foundations in Molecular Simulations 
Living Journal of Computational Molecular Science, 2019, 1, 5957  
[https://livecomsjournal.org/index.php/livecoms/article/view/v1i1e5957](https://livecomsjournal.org/index.php/livecoms/article/view/v1i1e5957)  
A good paper reviewing what molecular dynamics is and how it works. λ dynamics runs on top of molecular dynamics simulations, so understanding molecular dynamics is an important first step to understanding what we do.

<!-- R513 -->
Kong, X. & Brooks III, C. L.
λ-Dynamics: A New Approach to Free Energy Calculations 
Journal of Chemical Physics, 1996, 105, 2414-2423  
[https://doi.org/10.1063/1.472109](https://doi.org/10.1063/1.472109)  
The original λ dynamics paper. A lot has changed, but it’s worth a read.

<!-- R514 -->
Knight, J. L. & Brooks III, C. L.
λ-Dynamics Free Energy Simulation Methods 
Journal of Computational Chemistry, 2009, 30, 1692-1700  
[https://onlinelibrary.wiley.com/doi/10.1002/jcc.21295](https://onlinelibrary.wiley.com/doi/10.1002/jcc.21295)
A good review of how λ dynamics worked in 2009. A lot has changed, but it gives a good overview of the concepts.

<!-- R515 -->
Knight, J. L. & Brooks III, C. L.
Applying Efficient Implicit Nongeometric Constraints in Alchemical Free Energy Simulations 
Journal of Computational Chemistry, 2011, 32, 3423-3432  
[https://onlinelibrary.wiley.com/doi/10.1002/jcc.21921](https://onlinelibrary.wiley.com/doi/10.1002/jcc.21921)  
All modern λ dynamics simulations use implicit constraints. Understanding what they are and how they work is important for understanding how things actually work.

<!-- R516 -->
Knight, J. L. & Brooks III, C. L.
Multisite λ Dynamics for Simulated Structure-Activity Relationship Studies 
Journal of Chemical Theory and Computation, 2011, 7, 2728-2739  
[https://pubs.acs.org/doi/10.1021/ct200444f](https://pubs.acs.org/doi/10.1021/ct200444f)  
This is the paper that first introduced multisite λ dynamics, the idea of running λ dynamics with multiple perturbation sites.

<!-- R881 -->
Hayes, R. L.; Nixon, C. F.; Marqusee, S. & Brooks III, C. L.
Selection Pressures on Evolution of Ribonuclease H Explored with Rigorous Free-Energy-Based Design 
Proceedings of the National Academy of Sciences of the United States of America, 2024, 121, e2312029121  
[https://www.pnas.org/doi/10.1073/pnas.2312029121](https://www.pnas.org/doi/10.1073/pnas.2312029121)  
Most recent application of λ dynamics to a large protein sequence space. Also the first prospective study on protein mutations with lambda dynamics.

WORKING HERE:

https://livecomsjournal.org/index.php/livecoms/article/view/v2i1e18378

## Other λ Dynamics Papers

<!-- R893 -->
Hayes, R. L.; Buckner, J. & Brooks III, C. L.
BLaDE: A Basic Lambda Dynamics Engine for GPU Accelerated Molecular Dynamics Free Energy Calculations
Journal of Chemical Theory and Computation, 2021, 17, 6799-6807  
[https://pubs.acs.org/doi/10.1021/acs.jctc.1c00833](https://pubs.acs.org/doi/10.1021/acs.jctc.1c00833)  
This paper covers the GPU software that makes molecular dynamics simulations run
 fast in CHARMM. It is pretty technical, but gives a tangential glimpse to how m
olecular dynamics works and how GPU programming works.

<!-- R906 -->
Hayes, R. L.; Vilseck, J. Z. & Brooks III, C. L.
Addressing Intersite Coupling Unlocks Large Combinatorial Chemical Spaces for Alchemical Free Energy Methods 
Journal of Chemical Theory and Computation, 2022, 18, 2114-2123  
[https://pubs.acs.org/doi/abs/10.1021/acs.jctc.1c00948](https://pubs.acs.org/doi/abs/10.1021/acs.jctc.1c00948)  
This paper covers how to get good results from λ dynamics with many perturbation sites. ALF that flattens coupling between sites and the Potts model estimator are introduced.

## Other Authors Worth Reading

WORKING HERE:

Charles Brooks

Bernard Brooks

Jonah Vilseck

Jana Shen

Alex Dickson

David Baker

David Mobley, Michael Shirts, John Chodera

Gapsys and de Groot

MacKerrell

Benoit Roux

etc...
