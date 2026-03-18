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

## 1.1 Running CHARMM

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
`/dfs8/rhayes1_lab/bin/CHARMM_EXE/gnu/charmm -i helloworld.inp`  
named HelloWorldCHARMM.sh. You may run this slurm script by executing  
`sbatch HelloWorldCHARMM.sh`

However, this CHARMM script just prints "Hello World", it doesn't do molecular dynamics, so next we'll set up a molecular dynamics simulation and run it with CHARMM.

## 1.2 System Setup

CHARMM-GUI is the easiest way to set up systems. You upload a pdb file from the pdb or one that you have edited, and it will solvate it for you, adding water and ions. As time goes on, you will learn to setup the system yourself, using other tools like mmtsb, and using propka to determine the ionization state of titratable residues in your protein. We'll use CHARMM-GUI to set up a standard molecular dynamics simulation without λ dynamics.

To perform mutations to L99 (pH 3.0), M106 (pH 3.0), V149 (pH 5.4), and F153 (pH 3.0), you will need to set up two copies of the folded state with CHARMM-GUI, one at each pH, and four different pentapeptides to mimic the four unfolded ensembles, each centered on the mutating residue. No mutations are introduced in CHARMM-GUI, that comes later in the tutorial.

The first thing you’ll need to do is sign up for an account on CHARMM-GUI. Use your UCI email to do so. Once you have an account, you can click on the "Input Generator" link in the grey "CHARMM-GUI" box on the left. On the next page you’ll click on the "Solution Builder" link in the grey "Input Generator" box on the left. Enter the PDB code next to the box that says "Download PDB File:" - you should find the four character PDB code for T4 lysozyme in the manuscript. Then click the "Next Step: Select Model/Chain" in the lower right corner.

Check the boxes for the solution components you want to include. In this case you should include the protein chain "PROA". For the folded ensemble simulation, include all residues 1-162. For the unfolded ensemble simulations, just choose the five residues from two residues before the mutating residue to two residues after. You do not need to include the BME or CL atoms. Whether you choose to include these other atoms in a PDB in the future depends on your system and whether you expect them to be important. It is often good practice to include the crystallographic waters by checking the final box for WATA, because these waters can relax slowly, so it may take a long time for water that you add in the later steps to get into all the pockets it normally occupies in equilibrium. You do not need them for the unfolded ensemble. Once you have selected these two components, click on "Next Step: Manipulate PDB" in the bottom right.

This page allows you to make all sorts of modifications to the PDB file. "System pH:" is an easy way to set protonation states of all titratable residues, but we'll use propka because system pH is a little conservative. You'll be setting up systems at the two experimental pHs listed in the paper. Find what those pHs are.

"Terminal Group Patching:" controls what the N and C termini of your protein look like. In solution, the N terminal generally ends with an -NH3+ group, and the C terminal group generally ends with a -COO– group. These termini have the names NTER and CTER that you can select from the dropdown boxes. For the unfolded pentapeptide, the peptide is cut out of a longer protein, and the termini would not have these charged ends, but would be neutral and continue on to the next residue, so unless the mutating peptide is on one end or the other of the protein, I generally end it with a neutral capping group. The capping groups are ACE and CT3 in the drop down menus.

You won’t need to select anything in the "Mutation:" row, this is used if the native sequence you want to make mutations to is different from the sequence in the PDB. You’ll be adding your mutations on top of the native sequence later in this tutorial.

The "Protonation:" row is fairly important. On this row you set the protonation state of various titratable residues in your protein (and possibly in your ligands). Determining protonation states could take an hour or two, and may require you to come back to CHARMM-GUI later.

First you need to determine the protonation states you want. Upload your pdb to the cluster. Then run the following commands on it to analyze it with propka:  
`export PYTHONPATH=/dfs8/rhayes1_lab/bin/propka`  
`/dfs8/rhayes1_lab/bin/propka/bin/propka3 1l63.pdb`  
This will produce a file named `1l63.pka`. If you scroll down to `SUMMARY OF THIS PREDICTION`, you will see a list of pka predictions:  
`   ASP  10 A     3.48       3.80`  
`   ASP  20 A     3.95       3.80`  
`   ASP  47 A     2.72       3.80`  
`   ASP  61 A     3.99       3.80`  
`-- etc --`  
Next we need to choose protonation states based on these pkas. If the pH is below the pka, the residue will be protonated, otherwise it will be deprotonated. At a pH of 3, ASP 10, ASP 20, and ASP 61 will be protonated while ASP 47 will be deprotonated. If you're setting up the unfolded ensemble, it is more appropriate to use the model-pka column, so all residues (or all residues that show up in the pentapeptide) would be protonated. Generally you should check ASP, GLU, HIS, and LYS. ARG, TYR, and CYS are unlikely to titrate. You may see 99 sometimes for CYS, this happens if propka thinks it is forming a disulfide. N+ refers to the N terminus, usually protonated and positively charged as described above.

HIS is an extra nuissance because it has three protonation states: protonated on the delta nitrogen (HSD), protonated on the epsilon nitrogen (HSE), or protonated on both nitrogens (HSP). If propka says the histidine is protonated, then use HSP. Otherwise you have to decide whether to put the remaining proton on the delta or epsilon nitrogen. In the unfolded state, HSE is marginally more stable, so use it, but in the folded state, you will need to open up VMD or PyMol (see how to install them in [Installation.md](Installation.md)) to see whether there are hydrogen bond donors or acceptors near the delta or epsilon nitrogens that would bias the protonation state one way or the other. If it's too close to tell or the HIS is fully solvent exposed, use HSE. Fortunately in T4 lysozyme, there is only one HIS, and in the folded state at our pH values it is protonated as HSP.

Once you've decided on all the protonation states, note them down somewhere (this information goes in the methods section when you write a paper), and enter them into CHARMM-GUI. CHARMM-GUI only needs you to enter unusual protonations. ASP is deprotonated by default, so you only need to include protonated ASP as ASPP. Likewise GLU is deprotonated by default, so you only need to include protonated GLU as GLUP. The pKa of HIS is close enough to neutral that HIS should always be specified as HSP, HSD, or HSE. LYS is pronated by default, so you oly need to iclude deprotonated LYS as LSN.

"Disulfide Bonds:" is also important, as CYS residues close to eachother in an oxidizing environment will form covalent bonds, but this protein doesn't have any, so you don't need to do anything here.

When ready click on "Next Step: Generate PDB".

Next you'll set up the periodic boundary conditions. Molecular dynamics typically use periodic boundary conditions so that if an atom goes out one side of the box, it comes back in the other side. This is done because boxes are typically small, and the artifacts (errors) introduced by periodic boundary conditions are significantly less than the artifacts from surface effects of having the protein so close to a solvent-vacuum interface. For normal systems, placing 10 Angstroms of water on each side (so 20 Angstroms to the other side of the protein) is considered sufficient, though for strongly charged molecules like RNA, more distance may be necessary. Thus for the folded structure, you can select "Fit Waterbox Size to Protein Size" and choose "Enter Edge Distance:" as 10. The rectangular "Waterbox type:" is the most common. The alternative octahedral is actually better as it has 23% fewer atoms for the same periodic image distance, and thus runs faster. The fastest option rhombic dodecahedron has 29% fewer atoms, but is not available on CHARMM-GUI. If you have a very large system and need the extra speedup, I can share system setup scripts for rhombic dodecahedron. If you are setting up the unnfolded ensemble, it is likely to fluctuate in size, so click on "Specify Waterbox Size:" and set it to 40 Angstroms.

For ions you should generally mimic the experimental conditions if you can find them. Otherwise the salinity of the human body is about 150 mM, with mostly Na+ outside cells and K+ inside cells, so use the salt concentration most appropriate for where the protein generally resides. (The anions in living organisms are generally not Cl-, but it is a reasonable simplifying approximation.) Set your ion conditions appropriately here. The T4 lysozyme paper used specific salt conditions, find and use those conditions mentioned in the paper.

Then click on "Next Step: Solvate Molecule".

This page controls how long range electrostatics are computed. You can read
<!-- R360 -->
Essmann, U.; Perera, L.; Berkowitz, M. L.; Darden, T.; Lee, H. & Pedersen, L. G.
A Smooth Particle Mesh Ewald Method 
Journal of Chemical Physics, 1995, 103, 8577-8593  
[http://aip.scitation.org/doi/10.1063/1.470117](http://aip.scitation.org/doi/10.1063/1.470117)  
if you are morbidly curious. You can use defaults for everything on this page. Click on "Next Step: Setup Periodic Boundary Condition".

This page has options for the molecular dynamics simulation you intend to run. You can check boxes to download your files using other force fields (e.g. AMBER) or other MD engines besides CHARMM (e.g. OpenMM). You should set the system temperature to whatever the experimental temperature is, usually 298.15, rather than the default of 303.15. Then click on "Next Step: Generate Equilibration and Dynamics inputs"

On this page click on "download.tgz". Then copy it to the cluster with `scp`, extract it from the tarred, zipped file with  
`tar -xzf charmm-gui.tgz`  
and `cd` into the resulting directory.

Now that you’ve set up the system using CHARMM-GUI, you’ll need to relax the water for a while at constant volume using the provided script `step4_equilibration.inp`, and then relax the volume for a while longer at constant pressure using the provided script `step5_production.inp`. These scripts use standard CHARMM routines to run dynamics and are very slow, so modify these scripts to use BLaDE on GPUs as follows. Add the line  
`scalar fbeta set 0.1 sele all end`  
to set the friction coefficient fbeta immediately before dynamics in both files, and change the dynamics command in `step4_equilibration.inp` to  
`DYNA leap start timestep 0.001 nstep @nstep -`  
`        blade -`  
`        nprint 1000 iprfrq 1000 ntrfrq 0 -`  
`        iunread -1 iunwri 12 iuncrd -1 iunvel -1 kunit -1 -`  
`        nsavc 0 nsavv 0 -`  
`        tref @temp firstt @temp`  
and change the dynamics command in `step5_production.inp` to  
`DYNA CPT leap restart time 0.002 nstep @nstep -`  
`        blade prmc iprs 100 pref 1 prdv 100 -`  
`        nprint 1000 iprfrq 1000 ntrfrq 0 -`  
`        iunread 11 iunwri 12 iuncrd 13 iunvel -1 kunit -1 -`  
`        nsavc 50000 nsavv 0 -`  
`        reft @temp firstt @temp`

At this point you can run these scripts with  
`/dfs8/rhayes1_lab/bin/CHARMM_EXE/gnu/charmm -i step4_equilibration.inp`  
`/dfs8/rhayes1_lab/bin/CHARMM_EXE/gnu/charmm -i step5_production.inp`  
but be sure to do it inside a slurm script, and be sure to request a gpu with `--gres=gpu:1`.

At this point you can download the `step3_pbcsetup.psf` and `step5_1.dcd` files, and use them to watch the trajectory of your molecule jiggling around in VMD. Load the `.psf` file into vmd first, followed by the `.dcd` file, otherwise VMD has no idea what the numbers in the `.dcd` file mean. Congratulations, this is your first molecular dynamics simulation.

## 2. Multisite λ Dynamics

In this step we add the additional atoms necessary for alchemical perturbations with λ dynamics to the system, and create a directory called prep that ALF will use to run λ dynamics in the next step.

Other tutorials for setting up λ dynamics are available. In August 2023 our group helped present a workshop on λ dynamics [https://github.com/BrooksResearchGroup-UM/MSLD-Workshop](https://github.com/BrooksResearchGroup-UM/MSLD-Workshop) which includes recordings. See the sixth lesson [https://github.com/BrooksResearchGroup-UM/MSLD-Workshop/tree/main/6ProteinMutation](https://github.com/BrooksResearchGroup-UM/MSLD-Workshop/tree/main/6ProteinMutation) .

There is also a set of directions for setting up protein mutations on hpc3 at `/share/crsp/lab/rhayes1/share/git/pert_aa` . The `aa_stream` directory contains the most up to date scripts for protein perturbations, so you should use it. Ignore the `mdsetup` and `msldsetup` directories, these are for automating system setup if you have too many systems for CHARMM-GUI. Use the `mdcharmmgui` and `msldcharmmgui` directories. `mdcharmmgui` is largely redundant with the previous section, this section will explain the procedures in `msldcharmmgui` .

Essentially, ALF will follow instructions in a file called `alf_info.py` to run dynamics using its own CHARMM script, and at some point that CHARMM script will call a CHARMM script you create to set up the molecular system. That means your CHARMM script, we'll call it `system.inp` , should closely mimic `step5_production.inp` , but with the parts that run dynamics stripped out, and with a few extra commands to invoke `aa_stream` to set up the alchemical part of the calculation. You should also be sure you have a working installation of ALF by following the instructions in [Install.md](Install.md) , as the scripts in `msldcharmmgui` use ALF to test the setup.

We'll walk through the contents of these files, comparing `step5_production.inp` and its analogue `system.inp` so you understand them, pointing out necessary edits to other files along the way.

`step5_production.inp` begins with a line like `DIMENS CHSIZE 5000000 MAXRES 3000000` . This is removed from `system.inp` , because it must be at the beginning of the first CHARMM script that CHARMM starts reading, and `system.inp` is called by other CHARMM scripts. For larger systems you may need to add this line to the beginning of those other CHARMM scripts: `4SetupBlock.inp` here and `msld_flat.inp` and `msld_prod.inp` later when you run ALF. This sets the size of various CHARMM arrays for larger systems.

Instead, at the beginning of `system.inp` , we place the command  
`stream "prep/alchemical_definitions.inp"`  
This file, `alchemical_definitions.inp` contains information about what mutations you want in your system, and you should edit it. Set `resid1` to indicate which site you are mutating. If you are mutating multiple sites in the same simulation (you are not in this project, run the four sites in different simulations), you may enter the resid of the second mutation site as `resid2` and so on. The first mutation at a site, here `s1seq1` is always listed as `0` for the native residue found in the non-alchemical .psf (see below). If you have a second mutating site, its first mutation `s2seq1` will also be listed as `0`. I often place the identity of the native residue after a comment character `!`, but this is purely optional. Subsequent mutations at each site are listed by their one letter amino acid code, e.g. `a` for alanine, `c` for cysteine, `d` for aspartic acid, and so on. Histidine protonation states are selected here as `h` for HSD, `b` for HSE, and `j` for HSP. Thus the second mutation at site one (the first thing we are mutating to from native) is `s1seq2`, the third mutation is `s1seq3`, and so on. Next, list the segid of the mutating residue for each site. For site 1, this is `segid1`. The segid can be found in the .psf or .crd files, and for CHARMM-GUI output is usually PROA for a single chain protein like T4 lysozyme. Next the terminal information for the N-terminus and C-terminus of each mutated segid is listed. (If a segid is not mutated, you may omit it from these variables.) `nterdel_proa` and `cterdel_proa` define whether you are doing a terminal deletion on the segid PROA. Always leave these as 0. `nterres_proa` and `cterres_proa` contain the first and last resid of the chain. For T4 lysozyme this will be 1 and 162 in the folded state, and two residues before and after your mutation site in the unfolded state. `ntercap_proa` and `ctercap_proa` list the two terminal patches you selected in CHARMM-GUI, and `nterc_proa` and `cterc_proa` contain a single character code for this terminal patch. These terminal variable definitions typically have no effect unless your mutation is near the terminii.

You should edit `alchemical_definitions.inp` to contain the mutations you want for your system.

Next, the simple command `stream toppar.str` load topology files (.rtf files) and parameter files (.prm files). The .rtf files contain information to tell CHARMM what atoms are in any residue, what their atom types and charges are, and how they are connected together. The .prm files tell CHARMM what the equilibrium bond lengths, spring constants, van der waals parameters, and other parameters are that go into the potential energy function. Roughly speaking, the .rtf tells you what terms are in the potential energy function, and the .prm tells you what the constants in those terms are. In `system.inp` this command is replaced with `stream prep/toppar.str` because all files must be placed inside the `prep` directory for ALF to work properly.

The file `toppar.str` refers to a bunch of files inside the `toppar` directory. However, the `toppar` directory must also be placed inside `prep` . Consequently, in order for the `toppar.str` file generated by CHARMM-GUI to work with λ dynamics, you need to either change every file name in `toppar.str` to begin with `prep/` (don't do this), or a more elegant solution is to have the script create a soft link so files can be referenced by their original names. Soft links are files that point to the address of another file, they are like low memory copies, and they are created with `ln -s`. The `system` command can be used to execute a bash command inside a CHARMM script. Thus,  
`system "ln -s prep/toppar toppar"`  
has been placed before `stream prep/toppar.str` to ensure it still works with ALF.

Next,  
`open read unit 10 card name step3_pbcsetup.psf`  
`read psf  unit 10 card`  
sets up the .psf (protein structure file) of the system. Note, these files are moved to `prep` in `system.inp`. Typically, CHARMM scripts read a sequence of residues for your protein (one segid at a time), and use the information in the .rtf files to decide what atoms are in the system based on your sequence of residues, then you generate a .psf, and add patches to it, such as the protonation and disulfide patches mentioned in the previous section. This involves a lot of lines of code to setup, but at the end you can write out a .psf file to save the list of all the atoms, bonds, angles, dihedrals, etc. in your system, and CHARMM-GUI saves this .psf file and reads it in, rather than generating the .psf every time. We'll make modifications to this non-alchemical .psf later, using the variables you defined in `alchemical_definitions.inp`.

At this point CHARMM knows all the atoms in your system, but it doesn't know where they are. The commands  
`open read unit 10 card name step3_pbcsetup.crd`  
`read coor unit 10 card`  
read the coordinates out of a .crd file. You may also read coordinates out of a .pdb file, but .pdb files have annoying limits on the number of atoms, so .crd files cause fewer errors.

`stream step3_pbcsetup.str`  
Reads information about the periodic boundary conditions (the box size, box angles, and fft grid sizes) out of a stream (.str) file.

The coordinates and box sizes read in already were before equilibration during `step5_production.inp`. To read in the new coordinates and box size, we read the restart file `step5_1.rst`  
`    bomlev -5`  
`    open read  unit 11 card name step5_@pcnt.rst`  
`    read coor dynr curr unit 11`  
`    bomlev  0`  
` `  
`    calc A = ?XTLA`  
`    calc B = ?XTLB`  
`    calc C = ?XTLC`  
and overwrite the values of `A`, `B`, and `C` from `step3_pbcsetup.str`.

At this point, the non-alchemical system is set up, so `system.inp` interrupts to stream three important files from `aa_stream` to set up the alchemical portion of the system  
`stream "prep/aa_stream/patchloop.inp"`  
`stream "prep/aa_stream/selectloop.inp"`  
`stream "prep/aa_stream/deleteloop.inp"`  
These files use the variables defined in `alchemical_definitions.inp` and by `variables1.inp` which ALF created from `alf_info.py` to set up the alchemical perturbations. `patchloop.inp` applies patches (modifications) to the .psf to add additional alchemical atoms and their bonds, angles, dihedrals, etc. `selectloop.inp` saves lists of these alchemical atoms as selections, so the appropriate selections can be scaled by λ later. `deleteloop.inp` deletes some extra angles and dihedrals that were accidentally added to the .psf during `patchloop.inp`.

Next, we continue to setting up the periodic boundary conditions with  
`open read unit 10 card name crystal_image.str`  
`CRYSTAL DEFINE @XTLtype @A @B @C @alpha @beta @gamma`  
`CRYSTAL READ UNIT 10 CARD`  
` `  
` `  
`!Image centering by residue`  
`IMAGE BYRESID XCEN @xcen YCEN @ycen ZCEN @zcen sele resname TIP3 end`  
`IMAGE BYRESID XCEN @xcen YCEN @ycen ZCEN @zcen sele segid IONS end`  
Periodic boundary conditions are used because the surface of water behaves differently from the interior, so to prevent the surface of the water from distorting the protein, we get rid of the surface by introducing periodic boundary conditions. The artifacts due to periodic boundary conditions are generally smaller than the artifacts from simulating a tiny droplet of water.

The remainder of `step5_production.inp` is dropped as the covers setting up the nonbonded options (done here by `nbond.str`) and running dynamics, which are performed by the CHARMM scripts that stream `system.inp`.

One still must apply alchemical scaling to the appropriate atoms. This is done in `system.inp` by the command  
`stream "prep/aa_stream/blocksetup.inp"`  
Again, this script is controlled by the variables in `alchemical_definitions.inp`, and you should not need to edit it, unless you are also mutating ligands, your are using modified implicit constraints, or you are using modified biases, as described below.

Next, you should edit `alf_info.py` to reflect your system. Be sure that `'enginepath'` points to the correct CHARMM executable. `'nsubs'` reads the number of substituents at each site from a file called `nsubs`. You should edit this to include the number of substituents at each site. This is one for the native plus one for each mutant. So for V149 with 5 mutations to CIMST, nsubs is 6. (For the 3 site system mentioned later in the paper with one mutation to M at 3 sites, nsubs is `2 2 2`.) If any mutations in your system are charged, you should add a `q` field to `alf_info.py`, which is a list of the charges of each substituent in order. This will apply the discrete solvent charge changing correction.

By default ALF uses the old `linear2018` loss function. You may wish to add `alf_info['loss']='nonlinear2024'` to your `alf_info.py` file to use a more updated loss function.

By default ALF uses the old `bcxs2018` bias. This bias will work fine for this exercise, but if you have significantly more mutations, you may wish to invoke the `bcxstu2026` bias with `alf_info['bias']='bcxstu2026'`. If you do, you will need to modify `aa_stream/blocksetup.inp` to include there biases. Change  
`   calc nbiaspot = 5 * ( @nblocks * ( @nblocks - 1 ) ) / 2`  
`   ldbi @nbiaspot`  
to  
`   calc nbiaspot = 9 * ( @nblocks * ( @nblocks - 1 ) ) / 2`  
`   ldbi @nbiaspot`  
and change  
`            ldbv @ibias @ip1 @jp1 6 0.0 @cs@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @ip1 @jp1 10 -5.56 @xs@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @ip1 @jp1 8 0.017 @ss@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @jp1 @ip1 10 -5.56 @xs@@{sj}s@@{jj}s@@{si}s@@{ii} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @jp1 @ip1 8 0.017 @ss@@{sj}s@@{jj}s@@{si}s@@{ii} 0`  
`            calc ibias = @ibias + 1`  
to  
`            ldbv @ibias @ip1 @jp1 6 0.0 @cs@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @ip1 @jp1 10 -5.56 @xs@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @ip1 @jp1 8 0.012 @ss@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @ip1 @jp1 8 -1.012 @ts@@{si}s@@{ii}s@@{sj}s@@{jj} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @ip1 @jp1 12 0.012 @us@@{si}s@@{ii}s@@{sj}s@@{jj} 2`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @jp1 @ip1 10 -5.56 @xs@@{sj}s@@{jj}s@@{si}s@@{ii} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @jp1 @ip1 8 0.012 @ss@@{sj}s@@{jj}s@@{si}s@@{ii} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @jp1 @ip1 8 -1.012 @ts@@{sj}s@@{jj}s@@{si}s@@{ii} 0`  
`            calc ibias = @ibias + 1`  
`            ldbv @ibias @jp1 @ip1 12 0.012 @us@@{sj}s@@{jj}s@@{si}s@@{ii} 2`  
`            calc ibias = @ibias + 1`  

By default, ALF used old fnex implicit constraints `fnex2011`. These implicit constraints are ineffective at sampling near the end states beyond 8 or 9 substituents, and never sample the exact end states at all, resulting in a tiny error (about 0.1 to 0.2 kcal/mol) in free energy results. To sample more substituents with the old fnex bias, you can add a theta bias by invoking the `impcons` value `fnexdozen2024` in `alf_info.py` and changing  
`   msld @blockassign fnex @fnex`  
`   msma`  
to
`   msld @blockassign fnex @fnex`  
`   msma`  
`   thbv inde all auto`  
You can use piecewise implicit constraints to sample the exact endpoints by adding the `fnpwise2026` value to the `impcons` key in `alf_info.py` with `alf_info['impcons']='fnpwise2026'` and by modifying `aa_stream/blocksetup.inp` from  
`   msld @blockassign fnex @fnex`  
`   msma`  
to  
`   msld @blockassign fnpw`  
`   msma`  
`   thbv flat all 0.35 59.2 ! w=0.35, k=100kT`  

With these described modifications and explanations, now take a look at `RunSetup.sh`. You should now understand why this file is copying the files it is using from the CHARMM-GUI directory and how they are used to set up the alchemical system. Change the environment setup, CHARMM path, ALF path, CHARMM-GUI directory path and aa_stream directory path to point at the right things, and submit this script with  
`bash SubSetup.sh`  
You can check the contents of `outrun` and `errrun` to make sure the script ran correctly.

## 3. Adaptive Landscape Flattening

Adaptive landscape flattening (ALF) is required to choose optimal biases that allow λ dynamics simulations to converge within a reasonable time scale. Early references on ALF can be found at [dx.doi.org/10.1021/acs.jpcb.6b09656](dx.doi.org/10.1021/acs.jpcb.6b09656) and [dx.doi.org/10.1002/pro.3500](dx.doi.org/10.1002/pro.3500). The supporting information in the second paper has fairly detailed information on how ALF works.

## 3.1 Installing ALF

See installation instructions in [Installation.md](Installation.md)

Examples for how to run ALF are located in the `examples` directory of your ALF directory. You can follow the `README` there to test ALF if you don't trust the prep directory you just created.

## 3.2 ALF routines

Inside the `examples` directory is the `engines` directory, which has example ALF scripts for several platforms. `bladelib` is the fastest of these that runs inside CHARMM, so we will use it. This is done by adding the argument `engine='bladelib'` to the end of the following alf subroutines. There are three main alf subroutines: `runflat`, `runprod`, and `postprocess`. You should read their documentation (see the alf `README.md`, and the beginning of `alf/runflat.py`, `alf/runprod.py`, and `alf/postprocess.py`). Inside the `examples` directory, these subroutines are housed inside slurm scripts `runflat.sh`, `runprod.sh`, and `postprocess.sh`, which are submitted to slurm by `subsetAll.sh`.

The `runflat` subroutine is used to run several short simulations to optimize the biases, from its first argument to its second argument, using the third argument as the number of equilibration time steps and the fourth argument as the number of sampling time steps. Previous numerical experiments suggested discarding the first quarter of the data provides optimal sampling, so the fourth argument should be three times the third. Typically 100 or 200 short simulations of 100 ps (25 ps equilibration and 75 ps sampling) are run to get a rough idea of the biases. 100 cycles is sufficient in most cases, but if you're mutating to or from an arginine, you'll need the full 200 cycles. Next `runflat` is run again, this time with about 10 cycles of longer 1 ns simulations to refine the biases. `runflat` is quite fault tolerant. If an error is encountered during a cycle, `runflat` will keep going back and trying it again until it succeeds. If `runflat` crashes and is launched again, it checks to see how far it got the previous time, and begins at the first incomplete cycle.

The `runprod` subroutine runs some portion of a production simulation. Typically these are launched in groups of 5 by another script. Each individual in the group is launched as a slurm job array with a slurm option like `--array=1-4%1`, which in this case would run four copies of the job, one at a time. Each instance of `runprod` is responsible for running some number of ns of simulation. The first argument to `runprod` is the current ALF cycle (one after wherever the previous `runflat` or `postprocess` ended), the second argument is the independent trial token, typically a through e, the third argument is the last ns the previous invokation of `runprod` was responsible for, and the fourth argument is the last ns this invokation of `runprod` is responsible for. These arguments are generally passes in by environment variables from `subsetAll.sh` or calculated from the index within the job array (the first job in the array is responsible for different ns than the second job in the array and so on.)

After `runprod` invokations conclude, the run can be analyzed with `postprocess`. `postprocess` will make new estimates for the optimal biases and will produce a file called `Result.txt` in the corresponding analysis directory containing dG estimates. To get the ddG folding, subtract dG in the unfolded ensemble from dG in the folded ensemble.

## 3.3 ALF best practices

Running a very long production simulation with poor biases will give rather poor results. Thus, it is best to precede long production runs with shorter production runs to refine biases. Starting with a production run of 5 ns, it is best not to extend production runs by more than a factor of 4 or 5 to reach the desired simulation length. Thus, if a 100 ns production run is desired, one should run a 5 ns production, followed by a 20 ns production, followed by a 100 ns production. If the penultimate production run isn't sampling some substituents (i.e. the Result.txt file has some nan values in it), you may wish to run another production of that length before extending to final production, as it may indicate biases are not yet converged.

On our cluster using the free-gpu partition, ALF runs are often interrupted. ALF is rather fault tolerant, but modified versions of `subsetAll.sh` that are more tolerant to faults can be found at `/dfs8/rhayes1_lab/bin/tutorial/alf_preemptable`

If you are using replica exchange, an hpc3 cluster update in fall 2025 messed up communication between mpi processes. The following command needs to be added to ALF scripts (e.g. `subsetAll.sh`)  
`export OMPI_MCA_btl_openib_if_include=mlx5_0`  
Without this, some nodes use different communication methods, which causes MPI to fail, crashing or hanging the job.

As described above, ALF default behavior is to use the linear2018 loss, the bcxs2018 bias, and the fnex2011 implicit constraint. These are no longer best practice. The nonlinear2024 loss, and fnpwise2026 implicit constraint are best practice. The bcxs2018 bias is best for two substituets, the bcxstu2026 bias is best for 20 substituents, so choose depending on your system. Modify your `alf_info.py` and `blocksetup.inp` files to specify your prefered options.

## 3.4 Cleaning up after ALF

ALF generates a lot of extra files. These files make working with and debugging ALF easier, but waste space once a simulation is finished. You can erase these files by running  
`rm -r run+([0-9])`  
`rm -r slurm*`  
`rm -r analysis*/data`  
`rm -r analysis*/Energy`  
`rm -r analysis*/Lambda`  
`rm -r analysis*/mc_*`  
`rm -r analysis*/weight*`  
`rm -r run*/output*`  
`rm -r run*/*.pdb*`  
`rm -r run*/*.psf*`  
`rm -r run*/res/*.res*`  

If you are especially desperate for space, then after publication you can erase the spatial trajectories with  
`rm -r run*/dcd`  
To generate less trajectory data, you can add the `nsavc=50000` argument to the end of your `runprod` command, as this controls how frequently spatial trajectory frames are printed.

You should probably never erase the alchemical trajectories of a published work, but you can erase them with  
`rm -r run*/res`  
if you need to.

## 4 CHARMM Scripts and Documentation

At this point you have completed the tutorial. You may continue reading to learn more about CHARMM and what the scripts you have written actually do.

## 4.1 CHARMM Scripts

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

CHARMM provides the control flow commands `if` and `goto`. These are sufficient to produce if-elseif-else-endif constructs, functions, for loops, while loops, ad many other programming constructs, but they are a bit clunky. Eventually our lab will switch to pyCHARMM, making these CHARMM scripting commands obsolete. For `if`, you may place an if statement followed by a single command on one line, or you can place `if [comparison] then` followed by several lines of code, followed by `endif`. `else` and `else if` are also supported. The `goto` command is invoked as `goto [place]`, and will then search the CHARMM script for `label [place]` an will resume execution there. You can replace [place] with any label, name, or token you want. Together `if` and `goto` statements can create a for loop by saving a variable, entering a loop that begins with a label, incrementing the variable, and then using an if statement to check if the variable is still below the desired number of iterations, and if so using goto to return to the label.

The pdb, crd, par, rtf, and psf commands have been described in more detail above during the λ dynamics setup section.

## 4.2 CHARMM Engines

Inside a CHARMM script, you can call  
`energy`  
to get the energy of the current configuration,  
`mini sd nsteps 100`  
to relax the structure to a lower energy state, in this case with 100 steps of steepest descent, or  
`dyna ! followed by a whole bunch of options I don’t want to cover`  
to run a molecular dynamics simulation. Each of these commands must compute the energy and force and do something with it, but there are more than half a dozen places in the CHARMM source code that do this calculation, and which one you choose will depend on what you’re trying to do, and the computational resources available.

There are several molecular dynamics engines inside of CHARMM, each of which uses different source code to do roughly the same thing. These engines differ primarily in their efficiency. There is the standard engine which uses Fortan code on CPUs and is very slow. There is the domdec engine, which can run on CPUs, CPUs and GPUs, or primarily GPUs. The primarily GPU version of domdec is still about a factor of five slower than BLaDE and OpenMM. BLaDE is an engine I wrote to run lambda dynamics fast on GPUs. OpenMM is a software developed outside of CHARMM to run fast on GPUs, but it is flexible enough to be called as a library from CHARMM, unfortunately it does not yet run λ dynamics within CHARMM.

BLaDE can be invoked by adding the blade keyword to any energy or minimization command. To add BLaDE to a dynamics command add  
`blade prmc iprs 100 pref 1 prdv 100 -`  
for constant pressure simulations or just  
`blade -`  
for constant volume simulations.

## 4.3 CHARMM Documentation

CHARMM takes an input in the form of a CHARMM script. These are good because they are very flexible, and allow you to do many more things than most molecular dynamics packages. They are bad because the scripting language is archaic, and they are therefore hard to read and write. Documentation for CHARMM scripts is somewhat thorough, but hard to read as well. A few tutorials are available.

The documentation on CHARMM-GUI will tell you way more about how to use CHARMM scripts  
[https://www.charmm-gui.org/charmmdoc/contents.html](https://www.charmm-gui.org/charmmdoc/contents.html)

Within this link, the following topics may be useful. (I've removed subsections that are less important for our uses).

1. Must reads  
Input-Output Commands - these cover reading files in and out of CHARMM. It’s useful to know what they are, but I find them difficult to use, and usually have to copy an old command to get the syntax right.  
Miscellaneous Commands - stream, return, stop, echo, bomblev, and the entire run control section are the most relevant commands here  
Residue Topology File - the residue topology files (rtfs) describe how to go from a sequence to a psf and how to apply patches to a psf. Generally CHARMM-GUI builds the psf for you, but you'll need to understant rtf files if you want to build your own psf.  
Atom Selection - atom selection is important for many things, including specifying which atoms are scaled by λ.  
Generation and Manipulation of the Structure - probably interesting  
How to use CHARMM - good information  

2. Model building and manipulation  
Calculation on Crystals using CHARMM - periodic boundary conditions require both a crystal command and an image command  
Images - periodic boundary conditions require both a crystal command and an image command  
The Internal Coordinate Manipulation Commands - internal coordinates (ICs) are important for setting the positions of alchemical atoms in CHARMM scripts  
SCALar : commands to manipulate scalar atom properties - these turn out to be useful surprisingly often when writing your own CHARMM scripts  

3. Energy and minimization  
CONSTRAINTS - useful for both SHAKE (hydrogen bond length constraint that all simulations use) and harmonic restraints (that keep atoms from moving too far from starting positions)  
Energy Manipulations: Minimization and Dynamics - description of how to use energy and dynamics  
The Ewald Summation method - Long range interactions are hard to calculate. These two sections describe a little how CHARMM does it. Read the second one first.  
Generation of Non-bonded Interactions - Long range interactions are hard to calculate. These two sections describe a little how CHARMM does it. Read the second one first.  

6. Free energy method
BLOCK - This is the module that we use for λ dynamics calculations
