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

## 1.1 System Setup

CHARMM-GUI is the easiest way to set up systems. You upload a pdb file from the pdb or one that you have edited, and it will solvate it for you, adding water and ions. As time goes on, you will learn to setup the system yourself, using other tools like mmtsb, and using propka to determine the ionization state of titratable residues in your protein.

The first thing you’ll need to do is sign up for an account on CHARMM-GUI. Use your UCI email to do so. Once you have an account, you can click on the "Input Generator" link in the grey "CHARMM-GUI" box on the left. On the next page you’ll click on the "Solution Builder" link in the grey "Input Generator" box on the left. Enter the PDB code next to the box that says "Download PDB File:" - you should find the four character PDB code for T4 lysozyme in the manuscript. Then click the "Next Step: Select Model/Chain" in the lower right corner.

Check the boxes for the solution components you want to include. In this case you should include the protein chain "PROA". For the folded ensemble simulation, include all residues 1-162. For the unfolded ensemble simulations, just choose the five residues from two residues before the mutating residue to two residues after. You do not need to include the BME or CL atoms. Whether you choose to include these other atoms in a PDB in the future depends on your system and whether you expect them to be important. It is often good practice to include the crystallographic waters by checking the final box for WATA, because these waters can relax slowly, so it may take a long time for water that you add in the later steps to get into all the pockets it normally occupies in equilibrium. You do not need them for the unfolded ensemble. Once you have selected these two components, click on "Next Step: Manipulate PDB" in the bottom right.

This page allows you to make all sorts of modifications to the PDB file. "Terminal Group Patching:" controls what the N and C termini of your protein look like. In solution, the N terminal generally ends with an -NH3+ group, and the C terminal group generally ends with a -COO– group. These termini have the names NTER and CTER that you can select from the dropdown boxes. For the unfolded pentapeptide, the peptide is cut out of a longer protein, and the termini would not have these charged ends, but would be neutral and continue on to the next residue, so unless the mutating peptide is on one end or the other of the protein, I generally end it with a neutral capping group. The capping groups are ACE and CT3 in the drop down menus.

You won’t need to select anything in the "Mutation:" row, this is used if the native sequence you want to make mutations to is different from the sequence in the PDB. You’ll be adding your mutations on top of the native sequence later in this tutorial. This row is also used to select the protonation state of histidines: on the delta nitrogen (HSD), on the epsilon nitrogen (HSE), or on both nitrogens (HSP). All histidines have the default protonation of HSD in this protein, so no modification... WORKING HERE

The "Protonation:" row is fairly important. On this row you set the protonation state of various titratable residues in your protein (and possibly in your ligands).

WORKING HERE

So I've installed propka in my home directory, you can look at  
`/data/homezvol0/rhayes1/programs/propka/Install.sh`  
for instructions on how to install it yourself.

Then you'll want to run it on your protein. These two lines should let you do that:  
`export PYTHONPATH=/data/homezvol0/rhayes1/programs/propka`  
`/data/homezvol0/rhayes1/programs/propka/bin/propka3 yourpdbhere.pdb`

Comment on adding to  
`IMAGE BYRESID XCEN @xcen YCEN @ycen ZCEN @zcen sele resname TIP3 end`  
`IMAGE BYRESID XCEN @xcen YCEN @ycen ZCEN @zcen sele segid IONS end`

WORKING HERE

Once you’ve set up the system using CHARMM-GUI, you’ll need to relax the water for a while at constant volume using the provided script step4_equilibration.inp, and then relax the volume for a while longer at constant pressure using the provided script step5_production.inp. These scripts use standard CHARMM routines to run dynamics and are very slow, so modify these scripts to use BLaDE on GPUs as follows. Add the line  
`scalar fbeta set 0.1 sele all end`  
to set the friction coefficient fbeta immediately before dynamics in both files, and change the dynamics command in step4_equilibration.inp to  
`DYNA leap start timestep 0.001 nstep @nstep -`  
`        blade -`  
`        nprint 1000 iprfrq 1000 ntrfrq 0 -`  
`        iunread -1 iunwri 12 iuncrd -1 iunvel -1 kunit -1 -`  
`        nsavc 0 nsavv 0 -`  
`        tref @temp firstt @temp`  
and change the dynamics command in step5_production.inp to  
`DYNA CPT leap restart time 0.002 nstep @nstep -`  
`        blade prmc iprs 100 pref 1 prdv 100 -`  
`        nprint 1000 iprfrq 1000 ntrfrq 0 -`  
`        iunread 11 iunwri 12 iuncrd 13 iunvel -1 kunit -1 -`  
`        nsavc 50000 nsavv 0 -`  
`        reft @temp firstt @temp`

WORKING HERE: moving this section 3.1 to after 3.2 and 3.3 probably makes more sense.

## 1.2 CHARMM Scripts

A molecular dynamics engine is a suite of software that performs molecular dynamics simulations. There are many molecular dynamics engines, including CHARMM, OpenMM, Gromacs, AMBER, LAMMPS, BLaDE, and many more. Our lab primarily focuses on CHARMM because MSλD was originally developed in the CHARMM engine. Among these engines, CHARMM is one of the oldest, and has been in development since the 1970s. It is primarily written in FORTRAN. It takes fewer shortcuts than other simulation packages which means it is typically more accurate and slower. It has many more features than most packages and is probably only second to OpenMM in terms of flexibility in what it can do. It is also quite flexible in how calculations are setup, because it has a builtin scripting language that allows different pieces of a calculation to be linked together. (Most other MD engines have many programs whose output must be linked together by an outside program such as a bash or python script.) While this scripting language is very expressive, it is also a little clunky, requiring goto statements to run for loops, and requiring arrays to be defined as a bunch of scalar variables with similar names.

You can learn more about the CHARMM scripting language in the documentation below in section 1.5, but it is also worth noting a few of the most important commands and ideas here. It is also worth noting that the way CHARMM-GUI works is by using your input to create CHARMM scripts that are then executed on the remote server to setup your system.

Most CHARMM files start with a title. The title looks like this  
`* This is a generic title, it starts with an asterisk`  
`* Titles can be multiple lines.`  
`* Titles are different from comments`  
`* Comments are marked with exclamation points`  
`* Everything after an exclamation point is ignored by CHARMM`  
`* A title is over when you come to a line with just an asterisk`  
`*`  
Side note: CHARMM titles make `vi`, my preferred linux text editor, do weird things. You can stop this behavior by adding the line  
`set nofoldenable`  
into the file `~/.vimrc`, where `~` is bash shorthand for your home directory. Either create or add to this file.

Inside a CHARMM script you can define variables like this:  
`set i = 1 ! This is a comment, the spaces are important here`  
You can also perform calculations  
`calc x1 = 4 * 7.3`  
You can access a variable using an `@` symbol. If you want to use a variable name inside a variable name, you can use a double `@@`, which is useful for getting array like behavior  
`calc sqrtx1 = sqrt ( @x@@i ) ! takes square root of x1`  
Sometimes CHARMM also defines some variables, like the number of atoms (`natom`) or energy (`ener`). These are referenced with a `?`  
`echo ?natom ! Prints the number of atoms to output`

In order to run another CHARMM script from inside a CHARMM script, you can use the stream command.  
`stream "fancyfile.str"`  
This is useful in cases where some part of a calculation might be repeated, and you want to use that same file every time. Note that CHARMM is not case sensitive, but linux is, so if you want to open a file whose path is not all lowercase, you’ll need to enclose it in double quotes to make sure CHARMM preserves the case. The conventional ending for stream files is `.str`, but `.inp` is also frequently used.

The `stop` command tells CHARMM to stop running whenever it reaches that point in the script. Often when debugging, placing a stop command can let you focus on working out problems up to that point before trying to move on.

if, goto, etc. Also pdb, crd, par, rtf, psf WORKING HERE

## 1.3 Running CHARMM

CHARMM can be run with a command following the format  
`charmmexecutable -i script`  
where `charmmexecutable` is the path to the CHARMM executable, and `script` is a file containing a CHARMM script. If you put the following hello world CHARMM script (hello world is a term from computer science for a program that just prints output to let you know it’s working)  
`* This is a title`  
`*`  
`echo "Hello World"`  
`stop`  
in a file named `helloworld.inp`, then you can run it using  
`/dfs8/rhayes1_lab/bin/CHARMM_EXE/gnu/charmm -i helloworld.inp`

Remember, you should never run CHARMM from the login nodes ([Cluster.md](Cluster.md)). Review the information in [Cluster.md](Cluster.md) and then submit this hello world script to the queue. You may wish to use a slurm script like the following:  
`#! /bin/bash`  
`#SBATCH -p free`  
`#SBATCH --time=240 #Maximum time in minutes that job will take`  
`#SBATCH --ntasks=1`  
`/data/homezvol0/rhayes1/CHARMM_EXE/gnu/charmm -i helloworld.inp`  
named HelloWorldCHARMM.sh. You may run this slurm script by executing  
`sbatch HelloWorldCHARMM.sh`  

## 1.4 CHARMM Engines

Inside a CHARMM script, you can call  
`energy`  
to get the energy of the current configuration,  
`mini sd nsteps 100`  
to relax the structure to a lower energy state, in this case with 100 steps of steepest descent, or  
`dyna ! followed by a whole bunch of options I don’t want to cover`  
to run a molecular dynamics simulation. Each of these commands must compute the energy and force and do something with it, but there are more than half a dozen places in the CHARMM source code that do this calculation, and which one you choose will depend on what you’re trying to do, and the computational resources available.

What about GPUs? How can we see the status of our job? Also add vi fix and domdec commands.

BLaDE syntax...  
`blade prmc iprs 100 pref 1 prdv 100 -`

## 1.5 CHARMM Documentation

CHARMM takes an input in the form of a CHARMM script. These are good because they are very flexible, and allow you to do many more things than most molecular dynamics packages. They are bad because the scripting language is archaic, and they are therefore hard to read and write. Documentation for CHARMM scripts is somewhat thorough, but hard to read as well. A few tutorials are available.

The documentation on CHARMM-GUI will tell you a little more about how to use CHARMM scripts  
[https://www.charmm-gui.org/charmmdoc/contents.html](https://www.charmm-gui.org/charmmdoc/contents.html)

Within this link, the following topics may be useful

### 1. Must reads

Input-Output Commands - these cover reading files in and out of CHARMM. It’s useful to know what they are, but I find them difficult to use, and usually have to copy an old command to get the syntax right.  
Miscellaneous Commands - stream, return, stop, echo, bomblev, and the entire run control section are the most relevant commands here  
Residue Topology File  
Atom Selection  
Generation and Manipulation of the Structure  
Test commands: Commands to test various conditions in CHARMM  
How to use CHARMM  

### 2. Model building and manipulation

The Coordinate Manipulation Commands  
Calculation on Crystals using CHARMM  
Construction of hydrogen positions  
Images  
The Internal Coordinate Manipulation Commands  
SCALar : commands to manipulate scalar atom properties

WORKING HERE
```
        Energy and minimization

        CONSTRAINTS
        Energy Manipulations: Minimization and Dynamics
        The Ewald Summation method
        Energy Manipulations: Minimization and Dynamics
        Merck Molecular Force Field (MMFF94)
        Generation of Non-bonded Interactions
        CHARMM Emprical Energy Function Parameters
        Combined Quantum and Molecular Mechanical Hamiltonian

        Implicit solvent models

        Poisson-Bolztmann Equation Module
        Analytical Continuum Solvent (ACS) Potential
        Atomic Solvation Parameter Based Energy
        FACTS: Fast Analytical Continuum Treatment of Solvation
        Generalized Born using Molecular Volume (GBMV) Solvation Energy and Forces Module and Surface Area
        Generalized Born with a simple SWitching (GBSW) (Electrostatic + Nonpolar) Solvation Energy and Forces Module
        The SASA implicit solvation model
        Screened Coulomb Potentials Implicit Solvent Model (SCPISM)

        Dynamics and trajectory

        Correlation Functions
        Dynamics
        4 Dimension dynamics: Description and Discussion
        Monitor commands: Commands to monitor various dynamics properties
        NMR Analysis Module
        Vibration Analysis

        Free energy method

        BLOCK
        Free Energy Perturbation Calculations

        special features

        Generation of Hydrogen Bonds
        Syntax of basic MMFP commands
        The MOLVIB Module of CHARMM
        Replica: Commands which deal with replication of the molecular system
        The Parallel Distributed Replica
        Method and implementation of deformable boundary forces
        TReK: a program for Trajectory REfinement and Kinematics
        Monte Carlo
        Order Parameters
        The Charge and Drude Polarizability Fitting

        system specific

        GRAPHICS

        Compiling, testing and maintaining

        CHARMM Release and Installation
        CHARMM Testcases
        charmm_gen.com script

        For developers

        CHARMM Developer Guide
        CHARMM Developer’s Change Log

        Full list of documentation
```

## 2. Multisite λ Dynamics

WORKING - put stuff on files Initialize.py into lab intro

Information on how to set up MSλD runs with CHARMM-GUI is currently available in  
`/dfs8/rhayes1_lab/rhayes1/75_MSLD/03_ProteinTemplate/version3`  
Check out the mdcharmmgui directory for initial setup of the non-alchemical molecular dynamics section as described in section 1.1 above, then check out the alchemical setup in msldcharmmgui.

## 3. Adaptive Landscape Flattening

Adaptive landscape flattening (ALF) is required to choose optimal biases that allow λ dynamics simulations to converge within a reasonable time scale. Early references on ALF can be found at [dx.doi.org/10.1021/acs.jpcb.6b09656](dx.doi.org/10.1021/acs.jpcb.6b09656) and [dx.doi.org/10.1002/pro.3500](dx.doi.org/10.1002/pro.3500). The supporting information in the second paper has fairly detailed information on how ALF works.

## 3.1 Installing ALF

ALF is available online on github at [https://github.com/ryanleehayes/alf](https://github.com/ryanleehayes/alf), and can be downloaded to the cluster with the command  
`git clone git@github.com:RyanLeeHayes/ALF.git`  
The `README.md` file in this directory is a useful source of information on how to use the software, and there are several examples inside the examples directory. Put your copy of ALF somewhere in your home or BeeGFS directories. To get ALF ready for use, follow these steps.

1. First you need to setup the environment to compile the code in this directory by running the following two lines:  
`module load cmake/3.22.1 cuda/11.7.1 gcc/11.2.0 openmpi/4.1.2/gcc.11.2.0 fftw/3.3.10/gcc.11.2.0-openmpi.4.1.2`  
`export FFTW_HOME=$FFTW_DIR`  
These are the standard modules you can use to compile most programs on hpc3. You can run these two lines directly from the terminal, or you can save them to a file called `modules`, and then run the command `source modules`.

2. Within your copy of the directory, `cd` into the `alf/wham` subdirectory. Within this directory, run the `Clean.sh` script. This will get rid of previous compilations of the programs in this subdirectory. If the directory is already clean, it may display error messages saying it cannot remove files. Finally, you can run the `Compile.sh` script.

3. Going back to your copy of the directory, now `cd` into the `alf/dca` subdirectory. Run `Clean.sh` and `Compile.sh` in this directory too. If you're using the nonlinear ALF code, you'll need to repeat this process in `alf/lmalf` as well.

4. Go back to the copy of the directory. You are now ready to install ALF with python. You will probably need to load a nice copy of python first, so you can create a virtual environment from it, and install ALF as a module into that python virtual environment. You can load a good copy of python with  
`module load anaconda/2022.05`  
Next, run `Setup.sh`. This will create a virtual environment, and then install ALF into it with pip. If anything goes wrong, or you need to change something and install again, you can delete this installation by removing the `alf.egg-info` and `env-alf` subdirectories and the `setupenv` files. Finally run  
`module rm anaconda/2022.05`  
to remove this version of python from your path, as it interferes with MPI applications.

ALF is now installed as a python module inside a python virtual environment. You can load this python virtual environment at any time by sourcing the file `setupenv` that `Setup.sh` created in this directory, or you can directly copy the line contained in the file and put it into your scripts. To use ALF within a python script, you can include the line `import alf` near the top of the python script.

Examples for how to run ALF are located in the `examples` directory of your ALF directory.

## 3.2 ALF Best Practices

Inside the `examples` directory is the `engines` directory, which has example ALF scripts for several platforms. `bladelib` is the fastest of these that runs inside CHARMM, so we will use it. There are three main alf subroutines: `runflat`, `runprod`, and `postprocess`. You should read their documentation (see the alf `README.md`, and the beginning of `alf/runflat.py`, `alf/runprod.py`, and `alf/postprocess.py`). Inside the `examples` directory, these subroutines are housed inside slurm scripts `runflat.sh`, `runprod.sh`, and `postprocess.sh`, which are submitted to slurm by `subsetAll.sh`.

The `runflat` subroutine is used to run several short simulations to optimize the biases, from it's first argument to its second argument, using the third argument as the number of equilibration time steps and the fourth argument as the number of sampling time steps. Previous numerical experiments suggested discarding the first quarter of the data provides optimal sampling, so the fourth argument should be three times the third. Typically 100 or 200 short simulations of 100 ps (25 ps equilibration and 75 ps sampling) are run to get a rough idea of the biases. 100 cycles is sufficient in most cases, but if you're mutating to or from an arginine, you'll need the full 200 cycles. Next `runflat` is run again, this time with about 10 cycles of longer 1 ns simulations to refine the biases. `runflat` is quite fault tolerant. If an error is encountered during a cycle, `runflat` will keep going back and trying it again until it succeeds. If `runflat` crashes and is launched again, it checks to see how far it got the previous time, and begins at the first incomplete cycle.

The `runprod` subroutine runs some portion of a production simulation. Typically these are launched in groups of 5 by another script. Each individual in the group is launched as a slurm job array with a slurm option like `--array=1-4%1`, which in this case would run four copies of the job, one at a time. WORKING HERE this isn't very clear.

ALF Best practices...
