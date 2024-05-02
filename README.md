Hayes Group
New Student Orientation
Version 2023-03-09
 
1 Preliminaries
1.1 Expectations
Lab culture, work load, and professionalism WORKING HERE
1.2 Notation
Code and syntax will appear throughout this tutorial using the Courier New font. Commands you should type into the command prompt will begin with “> ”; don’t type that part. If you see panteater, replace that with your username.
1.3 Setup
Have Dr. Hayes email rcic-request@uci.edu to have you added to the rhayes1_lab_share group and the rhayes1_lab account.
 
Email Yi-San to request office keys (y.chang@uci.edu).
1.4 Lab Resources
Your home directory doesn't have much room (50 Mb). My home directory is
/data/homezvol0/rhayes1
if you ever need to look at it. A copy of the CHARMM executable is available in my home directory at
/data/homezvol0/rhayes1/CHARMM_EXE/gnu/charmm
 
Our group has just 1 Tb of CRSP storage. It is located at
/share/crsp/lab/rhayes1
This space is internet accessible, but is inconvenient to use because there are limits on the number of files in addition to the amount of storage. Use BeeGFS instead for most applications. Our group does store git repositories for various softwares we use in CRSP. They are located in
/share/crsp/lab/rhayes1/share/git
 
Our group has 25 Tb of BeeGFS storage. It is located at
/dfs8/rhayes1_lab
You should make a directory here with your username to hold your data.
> cd /dfs8/rhayes1_lab
> mkdir panteater
The command
> dfsquotas rhayes1 dfs8
will tell you how much space we have left. Don't let it get to 0, or jobs will crash. Ask Dr. Hayes to buy more, or clean out some old files.

There are some issues with copying to and from BeeGFS. Read about how to copy files on BeeGFS from this link
https://rcic.uci.edu/storage/beegfs-howtos.html
once you’ve become more familiar with bash and linux and the scp command.
2 Using bash and linux to Access the Cluster
2.1 Learn to Use bash and linux
The place you’ll be using bash is on the hpc3 cluster, as well as on your local machine to interact with the cluster. You will be accessing the cluster through a text based interface called the terminal. On Mac, terminal is a separate program, which you should place on your dock. On Windows, you can activate WSL and run commands through the windows terminal, or else you can download and install Putty for terminal access and WinSCP for file transfer. Use the former option (WSL) if possible. If you use Linux, you probably already know what a terminal is.

In order to log in to the hpc3 cluster, you can type
> ssh panteater@hpc3.rcic.uci.edu
into your terminal. hpc3.rcic.uci.edu is the URL of the cluster.

The hpc3 people have assembled a useful tutorial on how to use bash and shell scripting
https://rcic.uci.edu/tutorials.html
https://swcarpentry.github.io/shell-novice/
These links are useful to learn and read in their entirety, but you will remember them better if you’re trying to use what you learn for a specific task (such as setting up and running a protein simulation) while you’re doing them. When I first started my graduate research, I found a bash tutorial and worked on just that for a week straight until I knew the ins and outs of how to use a cluster.
2.2 Setup ssh Keys
Life is too short to use two factor authentication every time you log into the cluster. Setup ssh keys so you don't have to use two factor authentication for logging into the cluster. hpc3 has an explanation of how this works
https://rcic.uci.edu/hpc3/ssh-guide.html
as well as a link to a tutorial for Windows. See below for Mac.
 
For Mac: run
> ssh-keygen
on your local machine and enter a passphrase. Use the default file locations. This will generate ~/.ssh/id_rsa (the private key) and ~/.ssh/id_rsa.pub (the public key) on your local machine. ~/.ssh is a hidden directory inside your home directory. (Directories starting with . are hidden.)
Copy ~/.ssh/id_rsa.pub into the end of a file called ~/.ssh/authorized_keys on hpc3. For example, if the file doesn’t exist yet, you can just run
> scp ~/.ssh/id_rsa.pub panteater@hpc3.rcic.uci.edu:~/.ssh/authorized_keys
but if the file already has authorized keys in it this will overwrite them, and you need to do something more careful using what you learned in the bash and linux tutorial.
3 Using CHARMM for Molecular Dynamics
The first step to learning to run multisite λ dynamics (MSλD) is to learn to run normal molecular dynamics (MD). For this exercise, we’ll set up T4 lysozyme with the mutations described in
Hayes, R. L.; Vilseck, J. Z. & Brooks III, C. L.
Approaching Protein Design with Multisite λ Dynamics: Accurate and Scalable Mutational Folding Free Energies in T4 Lysozyme
Protein Science, 2018, 27, 1910-1922
which can be accessed at https://doi.org/10.1002/pro.3500 . Read this paper to get a feel for the methods the group uses and the system to which they are applied. In the upcoming sections we’ll evaluate the free energy for the four systems that did not require VB-REX sampling for converged free energy estimates, i.e. focus on L99, M106, V149, and F153, and ignore A42, A98, and M102. In this section though, the goal is simply to set up the simulation boxes, solvate them, and run molecular dynamics. You’ll be setting up the full folded protein, as well as the 4 different pentapeptides mimicking the unfolded ensemble for each site.

WORKING HERE: Other MD references:
"BLaDE paper"
Gromacs manual, first few chapters
3.1 System Setup
CHARMM-GUI is the easiest way to set up systems. You upload a pdb file from the pdb or one that you have edited, and it will solvate it for you, adding water and ions. As time goes on, you will learn to setup the system yourself, using other tools like mmtsb, and using propka to determine the ionization state of titratable residues in your protein.

The first thing you’ll need to do is sign up for an account on CHARMM-GUI. Use your UCI email to do so. Once you have an account, you can click on the “Input Generator” link in the grey “CHARMM-GUI” box on the left. On the next page you’ll click on the “Solution Builder” link in the grey “Input Generator” box on the left. Enter the PDB code next to the box that says “Download PDB File:” – you should find the four character PDB code for T4 lysozyme in the manuscript. Then click the “Next Step: Select Model/Chain” in the lower right corner.

Check the boxes for the solution components you want to include. In this case you should include the protein chain “PROA”. For the folded ensemble simulation, include all residues 1-162. For the unfolded ensemble simulations, just choose the five residues from two residues before the mutating residue to two residues after. You do not need to include the BME or CL atoms. Whether you choose to include these other atoms in a PDB in the future depends on your system and whether you expect them to be important. It is often good practice to include the crystallographic waters by checking the final box for WATA, because these waters can relax slowly, so it may take a long time for water that you add in the later steps to get into all the pockets it normally occupies in equilibrium. You do not need them for the unfolded ensemble. Once you have selected these two components, click on “Next Step: Manipulate PDB” in the bottom right.

This page allows you to make all sorts of modifications to the PDB file. “Terminal Group Patching:” controls what the N and C termini of your protein look like. In solution, the N terminal generally ends with an -NH3+ group, and the C terminal group generally ends with a -COO– group. These termini have the names NTER and CTER that you can select from the dropdown boxes. For the unfolded pentapeptide, the peptide is cut out of a longer protein, and the termini would not have these charged ends, but would be neutral and continue on to the next residue, so unless the mutating peptide is on one end or the other of the protein, I generally end it with a neutral capping group. The capping groups are ACE and CT3 in the drop down menus.

You won’t need to select anything in the “Mutation:” row, this is used if the native sequence you want to make mutations to is different from the sequence in the PDB. You’ll be adding your mutations on top of the native sequence later in this tutorial.

The “Protonation:” row is fairly important. On this row you set the protonation state of various titratable residues in your protein (and possibly in your ligands). 

WORKING HERE

So I've installed propka in my home directory, you can look at
/data/homezvol0/rhayes1/programs/propka/Install.sh
for instructions on how to install it yourself.

Then you'll want to run it on your protein. These two lines should let you do that:
export PYTHONPATH=/data/homezvol0/rhayes1/programs/propka
/data/homezvol0/rhayes1/programs/propka/bin/propka3 yourpdbhere.pdb

Comment on adding to
IMAGE BYRESID XCEN @xcen YCEN @ycen ZCEN @zcen sele resname TIP3 end
IMAGE BYRESID XCEN @xcen YCEN @ycen ZCEN @zcen sele segid IONS end

WORKING HERE

Once you’ve set up the system using CHARMM-GUI, you’ll need to relax the water for a while at constant volume using the provided script step4_equilibration.inp, and then relax the volume for a while longer at constant pressure using the provided script step5_production.inp. These scripts use standard CHARMM routines to run dynamics and are very slow, so modify these scripts to use BLaDE on GPUs as follows. Add the line
scalar fbeta set 0.1 sele all end
to set the friction coefficient fbeta immediately before dynamics in both files, and change the dynamics command in step4_equilibration.inp to
DYNA leap start timestep 0.001 nstep @nstep -
 	blade -
 	nprint 1000 iprfrq 1000 ntrfrq 0 -
 	iunread -1 iunwri 12 iuncrd -1 iunvel -1 kunit -1 -
 	nsavc 0 nsavv 0 -
 	tref @temp firstt @temp
and change the dynamics command in step5_production.inp to
DYNA CPT leap restart time 0.002 nstep @nstep -
 	blade prmc iprs 100 pref 1 prdv 100 -
 	nprint 1000 iprfrq 1000 ntrfrq 0 -
 	iunread 11 iunwri 12 iuncrd 13 iunvel -1 kunit -1 -
 	nsavc 50000 nsavv 0 -
 	reft @temp firstt @temp

WORKING HERE: moving this section 3.1 to after 3.2 and 3.3 probably makes more sense.
3.2 CHARMM Scripts
A molecular dynamics engine is a suite of software that performs molecular dynamics simulations. There are many molecular dynamics engines, including CHARMM, OpenMM, Gromacs, AMBER, LAMMPS, BLaDE, and many more. Our lab primarily focuses on CHARMM because MSλD was originally developed in the CHARMM engine. Among these engines, CHARMM is one of the oldest, and has been in development since the 1970s. It is primarily written in FORTRAN. It takes fewer shortcuts than other simulation packages which means it is typically more accurate and slower. It has many more features than most packages and is probably only second to OpenMM in terms of flexibility in what it can do. It is also quite flexible in how calculations are setup, because it has a builtin scripting language that allows different pieces of a calculation to be linked together. (Most other MD engines have many programs whose output must be linked together by an outside program such as a bash or python script.) While this scripting language is very expressive, it is also a little clunky, requiring goto statements to run for loops, and requiring arrays to be defined as a bunch of scalar variables with similar names.

You can learn more about the CHARMM scripting language in the documentation below in section 3.5, but it is also worth noting a few of the most important commands and ideas here. It is also worth noting that the way CHARMM-GUI works is by using your input to create CHARMM scripts that are then executed on the remote server to setup your system.

Most CHARMM files start with a title. The title looks like this
* This is a generic title, it starts with an asterisk
* Titles can be multiple lines.
* Titles are different from comments
* Comments are marked with exclamation points
* Everything after an exclamation point is ignored by CHARMM
* A title is over when you come to a line with just an asterisk
*
Side note: CHARMM titles make vi , my preferred linux text editor, do weird things. You can stop this behavior by adding the line
set nofoldenable
into the file ~/.vimrc , where ~ is bash shorthand for your home directory. Either create or add to this file.

Inside a CHARMM script you can define variables like this:
set i = 1 ! This is a comment, the spaces are important here
You can also perform calculations
calc x1 = 4 * 7.3
You can access a variable using an @ symbol. If you want to use a variable name inside a variable name, you can use a double @@, which is useful for getting array like behavior
calc sqrtx1 = sqrt ( @x@@i ) ! takes square root of x1
Sometimes CHARMM also defines some variables, like the number of atoms (natom) or energy (ener). These are referenced with a ?
echo ?natom ! Prints the number of atoms to output

In order to run another CHARMM script from inside a CHARMM script, you can use the stream command.
stream "fancyfile.str"
This is useful in cases where some part of a calculation might be repeated, and you want to use that same file every time. Note that CHARMM is not case sensitive, but linux is, so if you want to open a file whose path is not all lowercase, you’ll need to enclose it in double quotes to make sure CHARMM preserves the case. The conventional ending for stream files is .str, but .inp is also frequently used.

The stop command tells CHARMM to stop running whenever it reaches that point in the script. Often when debugging, placing a stop command can let you focus on working out problems up to that point before trying to move on.

if, goto, etc. Also pdb, crd, par, rtf, psf WORKING HERE
3.3 Running CHARMM
CHARMM can be run with a command following the format
> charmmexecutable -i script
where charmmexecutable is the path to the CHARMM executable, and script is a file containing a CHARMM script. If you put the following hello world CHARMM script (hello world is a term from computer science for a program that just prints output to let you know it’s working)
* This is a title
*
echo "Hello World"
stop
in a file named helloworld.inp, then you can run it using
  -i helloworld.inp

You should never run CHARMM (or do much of anything else that takes longer than a minute) from the login nodes (sometimes also called head nodes). When you first log into the cluster with ssh, the login node is the computer that runs all your commands. You don’t want the login nodes to be busy, otherwise routine tasks like logging in or making a directory or copying a file take forever for everyone using that login node. Therefore, clusters have many other computers (called compute nodes) to perform calculations. To use a compute node you have to request it, and then wait in line. The line is called a queue (which is what lines are called in Great Britain). Thus you should submit computationally demanding tasks to the queue. Our job scheduler is called slurm, and the hpc3 documentation has good information on how to use it:
https://rcic.uci.edu/hpc3/slurm.html

The quick summary is that there are two ways to request slurm to give you a compute node: by submitting a slurm script with sbatch or interactively.

A slurm script is a bash script that will run on the compute node and then exit. In the header of the slurm script you can describe details about what your job will need to run with #SBATCH entries, or you can supply those same options to sbatch. For example if you save the following file to HelloWorld.sh
#! /bin/bash
#SBATCH -p free
#SBATCH --time=240 #Maximum time in minutes that job will take
#SBATCH --ntasks=1
/data/homezvol0/rhayes1/CHARMM_EXE/gnu/charmm -i helloworld.inp
and then execute
> sbatch HelloWorld.sh
it will submit this job to the queue, and you should see its output show up once it starts running. If the file HelloWorld.sh lacked all the #SBATCH lines, you could also submit it to the queue with the command
> sbatch -p free --time=240 --ntasks=1 HelloWorld.sh

When you submit a slurm script, it runs and exits. If you want more control over the compute node, you can request an interactive job with the following syntax
srun -p free --time=240 --ntasks=1 --pty /bin/bash -i
and then continue entering commands. All new commands will run on the compute node until you type exit

There are several partitions where you can run your job, which are selected with the -p option. Some jobs just use CPUs, run those on standard or free, some jobs require GPUs, run those on gpu or free-gpu. To request a gpu include a line like
#SBATCH --gres=gpu:1
In your slurm script, which requests one GPU per node. The free version of each partition doesn’t cost the lab anything, but your job can be killed by other jobs in the paid partition, in which case you might need to go back and clean up the mess. The paid partition costs the lab $0.01 per hour for CPUs and $0.32 per hour for GPUs. To use the standard cpu partition, you will need to add an account option
#SBATCH -A rhayes1_lab
to your slurm script. To use the gpu partition, you will need to add a different account option
#SBATCH -A rhayes1_lab_gpu
to your slurm script. If slurm gives you a QOS error, you may need to remind me to add you to the lab accounts. I typically use the free partitions. Especially while you’re learning it’s probably best to use the paid partition so you don’t have to second guess whether your script had an error or whether another job killed it. As you become more comfortable and start using more GPUs, discuss with Dr. Hayes or more senior lab members before you launch thousands of GPU-hours of paid jobs. Eventually you’ll get a feel for how much our lab can afford, and how much free computing vs. wasted time is worth.

You can keep an eye on your jobs with the squeue command. To see all your jobs run
> squeue -u panteater
(Remember to use your username). If one of your jobs is 2718281, you can get a bunch more information about it with
> scontrol show jobid=2718281
More help on the slurm job scheduler is available at
https://rcic.uci.edu/hpc3/slurm.html
3.4 CHARMM Engines
Inside a CHARMM script, you can call
energy
to get the energy of the current configuration,
mini sd nsteps 100
to relax the structure to a lower energy state, in this case with 100 steps of steepest descent, or
dyna ! followed by a whole bunch of options I don’t want to cover
to run a molecular dynamics simulation. Each of these commands must compute the energy and force and do something with it, but there are more than half a dozen places in the CHARMM source code that do this calculation, and which one you choose will depend on what you’re trying to do, and the computational resources available.

What about GPUs? How can we see the status of our job? Also add vi fix and domdec commands.

BLaDE syntax…
blade prmc iprs 100 pref 1 prdv 100 -
3.5 CHARMM Documentation
CHARMM takes an input in the form of a CHARMM script. These are good because they are very flexible, and allow you to do many more things than most molecular dynamics packages. They are bad because the scripting language is archaic, and they are therefore hard to read and write. Documentation for CHARMM scripts is somewhat thorough, but hard to read as well. A few tutorials are available.

The documentation on CHARMM-GUI will tell you a little more about how to use CHARMM scripts
https://www.charmm-gui.org/charmmdoc/contents.html

Within this link, the following topics may be useful
1. Must reads
Input-Output Commands - these cover reading files in and out of CHARMM. It’s useful to know what they are, but I find them difficult to use, and usually have to copy an old command to get the syntax right.
Miscellaneous Commands - stream, return, stop, echo, bomblev, and the entire run control section are the most relevant commands here
Residue Topology File
Atom Selection
Generation and Manipulation of the Structure
Test commands: Commands to test various conditions in CHARMM
How to use CHARMM
2. Model building and manipulation
The Coordinate Manipulation Commands
Calculation on Crystals using CHARMM
Construction of hydrogen positions
Images
The Internal Coordinate Manipulation Commands
SCALar : commands to manipulate scalar atom properties

WORKING HERE
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

4 Multisite λ Dynamics
WORKING - put stuff on files Initialize.py into lab intro

Information on how to set up MSλD runs with CHARMM-GUI is currently available in
/dfs8/rhayes1_lab/rhayes1/75_MSLD/03_ProteinTemplate/version3
Check out the mdcharmmgui directory for initial setup of the non-alchemical molecular dynamics section as described in section 3.1 above, then check out the alchemical setup in msldcharmmgui.
5 Adaptive Landscape Flattening
Adaptive landscape flattening (ALF) is required to choose optimal biases that allow λ dynamics simulations to converge within a reasonable time scale. Early references on ALF can be found at dx.doi.org/10.1021/acs.jpcb.6b09656 and dx.doi.org/10.1002/pro.3500 . The supporting information in the second paper has fairly detailed information on how ALF works.
5.1 Installing ALF
ALF is located at
/share/crsp/lab/rhayes1/share/git/ALF/alf
and will be updated there from time to time. You can check back here for corrections or updates. The README.md file in this directory is a useful source of information on how to use the software, and there are several examples inside the examples directory. Make a copy of this directory and put it somewhere in your home or BeeGFS directories. To get ALF ready for use, follow these steps.

1. Within your copy of the directory, cd into the alf/wham subdirectory. Within this directory, run the Clean.sh script. This will get rid of previous compilations of the programs in this subdirectory. If the directory is already clean, it may display error messages saying it cannot remove files. Then edit the file modules to contain the following two lines:
module load cmake/3.22.1 cuda/11.7.1 gcc/11.2.0 openmpi/4.1.2/gcc.11.2.0 fftw/3.3.10/gcc.11.2.0-openmpi.4.1.2
export FFTW_HOME=$FFTW_DIR
These are the standard modules you can use to compile most programs on hpc3. Then run the Compile.sh script.

2. Going back to your copy of the directory, now cd into the alf/dca subdirectory. Run Clean.sh, edit the modules file in the same way, and run Compile.sh in this directory too.

3. Go back to the copy of the directory. You are now ready to install ALF with python. To do so, run Setup.sh . If anything goes wrong, or you need to change something and install again, you can delete this installation by removing the alf.egg-info and env-alf subdirectories.

ALF is now installed as a python module inside a python virtual environment. You can load this python virtual environment at any time by sourcing the file setupenv that Setup.sh created in this directory, or you can directly copy the line contained in the file and put it into your scripts. To use ALF within a python script, you can include the line import alf near the top of the python script.

Examples for how to run ALF are located in the test_beta directory of your ALF directory.

6 Other Lab Information
6.1 Conferences Worth Considering Attending
ACS - The American Chemical Society holds spring and fall meetings. These are very large conferences of around 10,000 chemists, so it’s very easy to get lost, but they almost always have interesting talks going if you can find them.

AIChE - The chemical engineering society has a similar meeting annually in November; I haven’t attended yet, so I can’t tell if it’s useful

Biophysical society - This is a medium size conference, maybe a couple thousand, usually in February, the topics are quite diverse, but I learned a lot about biophysics the first time I went here. It covers everything from molecular dynamics to structural determination to lipid rafts and liquid-liquid phase separation.

Protein society - This is a medium size conference of about 1200 scientists focused on proteins, everything from simulation to design to aggregation to folding, characterization, intrinsically disordered proteins and more. There are typically two sessions running at a time to choose between, and it runs at the end of June / beginning of July.

Gordon Research Conference on Protein Folding and Dynamics - There are many Gordon Research Conferences for different topics, held every other year with an attendance limit of 150 participants. They are very focussed on a narrow field, and this has been one of my favorite conferences since I first attended in 2010.

Free Energy Meeting - This conference occurs every other year and is focussed on computing free energies more accurately for big pharma applications in industry. Consequently, about half of the attendees are from industry. It’s capped around 200. On the off years they are trying to organize a similar conference in Europe.

6.2 Publications Worth Reading
Hayes, R. L. et al. Approaching Protein Design with Multisite λ Dynamics: Accurate and Scalable Mutational Folding Free Energies in T4 Lysozyme. Protein Science, 2018, 27, 1910-1922
https://onlinelibrary.wiley.com/doi/10.1002/pro.3500
This paper is what the T4 lysozyme practice problem is based on. It is the proof of concept paper that began using λ dynamics for protein mutations.

Hayes, R. L. et al. Adaptive Landscape Flattening Accelerates Sampling of Alchemical Space in Multisite λ Dynamics. Journal of Physical Chemistry B, 2017, 121, 3626-3635
https://pubs.acs.org/doi/10.1021/acs.jpcb.6b09656
The ALF algorithm has changed somewhat since this paper was written, but this still gives a good overview of how it works. Every λ dynamics simulation we run starts with ALF, so understanding it is pretty important.

Kong, X. and Brooks, C. L. III. λ‐dynamics: A new approach to free energy calculations. Journal of Chemical Physics, 1996, 105, 2414–2423
https://doi.org/10.1063/1.472109
The original λ dynamics paper. A lot has changed, but it’s worth a read.

WORKING HERE:
Add 2009 review
https://onlinelibrary.wiley.com/doi/10.1002/jcc.21295
And maybe MSLD paper? Also implicit constraints, and RNase H paper.

6.3 Homeless
Link to group meeting directions
https://docs.google.com/document/d/1AGdWI6pEpkVlZ0vJh8WUR1ay7AzAAS7NqA18qJkE0q8/edit?usp=sharing
