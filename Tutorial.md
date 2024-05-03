# Tutorial

## 1. Molecular Dynamics in CHARMM

The first step to learning to run λ dynamics is to learn to run normal molecular dynamics (MD). For this exercise, we’ll set up T4 lysozyme with the mutations described in  
Hayes, R. L.; Vilseck, J. Z. & Brooks III, C. L.
Approaching Protein Design with Multisite λ Dynamics: Accurate and Scalable Mutational Folding Free Energies in T4 Lysozyme
Protein Science, 2018, 27, 1910-1922  
which can be accessed at [https://doi.org/10.1002/pro.3500](https://doi.org/10.1002/pro.3500). Read this paper to get a feel for the methods the group uses and the system to which they are applied. In the upcoming sections we’ll evaluate the free energy for the four systems that did not require VB-REX sampling for converged free energy estimates, i.e. focus on L99, M106, V149, and F153, and ignore A42, A98, and M102. In this section though, the goal is simply to set up the simulation boxes, solvate them, and run molecular dynamics. You’ll be setting up the full folded protein, as well as the 4 different pentapeptides mimicking the unfolded ensemble for each site.

To learn more about how molecular dynamics simulations work, check out these papers, some of which are also mentioned in [Literature.md](Literature.md).

<!-- R960 -->
Braun, E.; Gilmer, J.; Mayes, H. B.; Mobley, D. L.; Monroe, J. I.; Prasad, S. & Zuckerman, D. M.
Best Practices for Foundations in Molecular Simulations
Living Journal of Computational Molecular Science, 2019, 1, 5957
[https://livecomsjournal.org/index.php/livecoms/article/view/v1i1e5957](https://livecomsjournal.org/index.php/livecoms/article/view/v1i1e5957)  
A good paper reviewing what molecular dynamics is and how it works. lambda dynamics runs on top of molecular dynamics simulations, so understanding molecular dynamics is an important first step to understanding what we do.

<!-- R893 -->
Hayes, R. L.; Buckner, J. & Brooks III, C. L.
BLaDE: A Basic Lambda Dynamics Engine for GPU Accelerated Molecular Dynamics Free Energy Calculations 
Journal of Chemical Theory and Computation, 2021, 17, 6799-6807  
[https://pubs.acs.org/doi/10.1021/acs.jctc.1c00833](https://pubs.acs.org/doi/10.1021/acs.jctc.1c00833)  
This paper covers the GPU software that makes molecular dynamics simulations run fast in CHARMM. It is pretty technical, but gives a tangential glimpse to how molecular dynamics works and how GPU programming works.

WORKING HERE: Other MD references:  
Gromacs manual, first few chapters  

## 1.1 System Setup

CHARMM-GUI is the easiest way to set up systems. You upload a pdb file from the pdb or one that you have edited, and it will solvate it for you, adding water and ions. As time goes on, you will learn to setup the system yourself, using other tools like mmtsb, and using propka to determine the ionization state of titratable residues in your protein.

The first thing you’ll need to do is sign up for an account on CHARMM-GUI. Use your UCI email to do so. Once you have an account, you can click on the "Input Generator" link in the grey "CHARMM-GUI" box on the left. On the next page you’ll click on the "Solution Builder" link in the grey "Input Generator" box on the left. Enter the PDB code next to the box that says "Download PDB File:" - you should find the four character PDB code for T4 lysozyme in the manuscript. Then click the "Next Step: Select Model/Chain" in the lower right corner.
