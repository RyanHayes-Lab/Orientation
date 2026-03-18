# Installation

Installation instructions may be found here for several programs used in the group.

Remember best practice is to launch an interactive job before compiling code.  
`srun --pty --time=0-04:00:00 --ntasks=1 --tasks-per-node=1 --cpus-per-task=8 -p standard -A rhayes1_lab --cpu_bind=none bash`

## CHARMM (Chemistry at Harvard Molecular Mechanics)

Initially you can just use the lab copy of CHARMM at  
`/dfs8/rhayes1_lab/bin/CHARMM_EXE/gnu/charmm`  
but as you mature in the group, you should compile and use your own copy of CHARMM.

You may obtain the CHARMM source code online for free at  
[https://charmm.chemistry.harvard.edu](https://charmm.chemistry.harvard.edu)  
but this is just the release version and lacks some of the features our group is actively developing. The current developers version is available to our group on hpc3 at  
`/share/crsp/lab/rhayes1/share/git/CHARMM/dev-release/`  
You should copy this directory to where you want to install CHARMM.

To compile CHARMM, you should first launch an interactive job, otherwise you'll run out of memory.  
`srun --pty --time=0-04:00:00 --ntasks=1 --tasks-per-node=1 --cpus-per-task=8 -p standard -A rhayes1_lab --cpu_bind=none bash`

Next, run the following script from the directory that contains the copy of `dev-release`. The module commands set up the environment, (you can also set this up with conda following instructions at [https://github.com/BrooksResearchGroup-UM/MSLD-Workshop/tree/main/0Install_Tools](https://github.com/BrooksResearchGroup-UM/MSLD-Workshop/tree/main/0Install_Tools)). Then `dev-release/configure` uses cmake to set up the files for installation, and `make` installs them. (The `-j10` option says to use 10 threads while installing, which speeds things up a lot.)  
`module load cmake/3.20.5 cuda/11.7.1 gcc/11.2.0 openmpi/4.1.2/gcc.11.2.0 fftw/3.3.10/gcc.11.2.0-openmpi.4.1.2`  
`export FFTW_HOME=$FFTW_DIR`  
`mkdir gnu`  
`cd gnu`  
`../dev-release/configure --with-gnu --without-mkl --without-openmm --with-blade -u --repdstr --with-nvtx --without-pycharmm --without-library -p ./inst &> ../config.log`  
`make -j10 install VERBOSE=1 &> ../make.log`

If you wish to install pyCHARMM instead, (a python interface to CHARMM), you may do so with  
`module load cmake/3.20.5 cuda/11.7.1 gcc/11.2.0 openmpi/4.1.2/gcc.11.2.0 fftw/3.3.10/gcc.11.2.0-openmpi.4.1.2`  
`export FFTW_HOME=$FFTW_DIR`  
`mkdir pycharmm`  
`cd pycharmm`  
`../dev-release/configure --without-openmm --with-blade --without-mpi --as-library -p inst`  
`make -j 10 install`  
`cd ../`  
`module load anaconda/2022.05`  
`python -m venv env-pycharmm`  
`source env-pycharmm/bin/activate`  
`pip install -e dev-release/tool/pycharmm`  
`module rm anaconda/2022.05`  
`` DIR=`pwd` ``  
`echo "export CHARMM_LIB_DIR=$DIR/pycharmm/inst/lib" > setupenv`  
`echo "source $DIR/env-pycharmm/bin/activate" >> setupenv`  
You should source `$DIR/setupenv` every time you want to use pyCHARMM.

To reinstall CHARMM or pyCHARMM, delete the `gnu` or `pycharmm` directory for CHARMM or pyCHARMM respectively.

## ALF (Adaptive Landscape Flattening)

ALF is the software used to choose optimal biases for λ dynamics simulations, and thus is a prerequisite to running almost any λ dynamics simulation.

ALF is available online on github at [https://github.com/ryanleehayes/alf](https://github.com/ryanleehayes/alf), and can be downloaded to the cluster with the command  
`git clone https://github.com/RyanLeeHayes/ALF.git`  
or if you have write permissions to the package at github with  
`git clone git@github.com:RyanLeeHayes/ALF.git`  
The `README.md` file in this directory is a useful source of information on how to use the software, and there are several examples inside the examples directory. Put your copy of ALF somewhere in your home or BeeGFS directories. ALF installation instructions are in the file `INSTALL`. To install ALF for use, follow those steps, customized for our cluster here.

1. First you need to setup the environment to compile the code in this directory by running the following two lines:  
`module load cmake/3.20.5 cuda/11.7.1 gcc/11.2.0 openmpi/4.1.2/gcc.11.2.0 fftw/3.3.10/gcc.11.2.0-openmpi.4.1.2`  
`export FFTW_HOME=$FFTW_DIR`
These are the standard modules you can use to compile most programs on hpc3. You can run these two lines directly from the terminal, or you can save them to a file called `modules`, and then run the command `source modules`.

2. Within your copy of the directory, run the following commands:  
`module load anaconda`  
`rm -r build`  
`mkdir build`  
`cd build`  
`python -m venv env-alf`  
`source env-alf/bin/activate`  
``cmake .. -DCMAKE_INSTALL_PREFIX=`pwd` ``  
`make install`  
The `module load anaconda` uses a working version of anaconda and python from the cluster and uses it as a starting point to make your copy of python that you will add ALF to. This saves you the trouble of installing anaconda. `rm -r build` deletes the previous installation, if there was one. `python -m venv env-alf` makes a new python virtual environment based on the python in the anacoda module you loaded. `source env-alf/bin/activate` activates that virtual environment so you can install ALF in it. Any time you want to use ALF, you have to source that same file to activate that copy of python. The remaining commands setup the installation with cmake and then perform the installation.

3. If you want to use your ALF immediately, you should run  
`module rm anaconda`  
to remove this version of python from your path, as it interferes with MPI applications.

ALF is now installed as a python module inside a python virtual environment. You can load this python virtual environment at any time by loading the modules from step 1, and then sourcing the file `build/env-alf/bin/activate`. (Note you will need the full absolute path to this file.) Do this at the beginning of any script invoking ALF. To use ALF within a python script, you can include the line `import alf` near the top of the python script.

## Conda

Conda is a software and dependency manager that makes installing things in python easier... most of the time. anaconda is the most common version of conda, but I prefer miniconda, as it is smaller and lighter weight. Installation instructions can be found at  
[https://docs.anaconda.com/miniconda/#quick-command-line-install](https://docs.anaconda.com/miniconda/#quick-command-line-install)

In the example below, I downloaded the installer to `~/Miniconda3-latest-Linux-x86_64.sh` and chose to install my copy of miniconda in `/dfs8/rhayes1_lab/rhayes1/miniconda3`, you should modify these choices as is appropriate for your use.  
`wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/Miniconda3-latest-Linux-x86_64.sh`  
`bash ~/Miniconda3-latest-Linux-x86_64.sh -b -p /dfs8/rhayes1_lab/rhayes1/miniconda3`  

After installing miniconda, it is recommended to initialize it using  
`/dfs8/rhayes1_lab/rhayes1/miniconda3/bin/conda init bash`  
but I find this annoying, as this will modify your `~/.bashrc` so that conda is always on, and conda packages sometimes override libraries that I load with module commands (like MPI or CUDA) causing other compilations to fail. So instead I write a file called `~/conda_active` that contains  
`# >>> conda initialize >>>`  
`# !! Contents within this block are managed by 'conda init' !!`  
`__conda_setup="$('/dfs8/rhayes1_lab/rhayes1/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"`  
`if [ $? -eq 0 ]; then`  
`    eval "$__conda_setup"`  
`else`  
`    if [ -f "/dfs8/rhayes1_lab/rhayes1/miniconda3/etc/profile.d/conda.sh" ]; then`  
`        . "/dfs8/rhayes1_lab/rhayes1/miniconda3/etc/profile.d/conda.sh"`  
`    else`  
`        export PATH="/dfs8/rhayes1_lab/rhayes1/miniconda3/bin:$PATH"`  
`    fi`  
`fi`  
`unset __conda_setup`  
`# <<< conda initialize <<<`  
` `  
`# conda deactivate # after conda inserted code`  
`conda deactivate`  
which contains the modifications to `~/.bashrc` that `conda init` makes (you should change the miniconda path in this file to match yours), and I source it with  
`source ~/conda_active`  
whenever I want to use conda. This also allows me to keep multiple copies of conda, which in in some ways contrary to the philosophy of conda as one massive package manager that controls everything, but since the different python virtual environments never seem to be as independent as they are advertised to be, this seems safer to me.

The strength of conda is that you can create separate virtual environements that house different groups of software, and installing a new package X is usually as easy as `conda install X`

## XQuartz

If you have a Mac, you will probably wish to forward graphical user interfaces (aka GUIs, aka windows) from the cluster from time to time. This is called x11 forwarding. The program that enables x11 forwarding on Macs is XQuartz. You should download and install it. On windows machines, there are different hoops you have to jump through for x11 forwarding.

## Jupyter

Jupyter is a useful software that lets you run python code in a notebook format, where you can use a GUI (graphical user interface) to interactively execute python code and visualize the output. Jupyter is fairly easy to install with a simple conda command once you have a working conda installation, and is already installed on several anaconda modules on hpc3. Using it on the cluster it trickier, because you have to log in, check out an interactive compute node, then use x11 forwarding to tunnel the GUI back from the compute node to the login node to your local machine.

A procedure that works for me is:  
`# On the first terminal:`  
`ssh rhayes1@hpc3.rcic.uci.edu`  
`srun --pty --time=0-04:00:00 --ntasks=1 --tasks-per-node=8 --cpus-per-task=1 --gres=gpu:1 -p gpu -A rhayes1_lab_gpu --cpu_bind=none bash`  
`module load anaconda/2022.05`  
``jupyter notebook --port=45678 --no-browser --ip=`hostname -s` &``  
Leave this process running on the interactive node with the terminal open. You can change the port number to suite your needs, and be sure to use your username, not mine. The output will give a token you need to log into the jupyter notebook.

Next:  
`# On another terminal: (-N and -f optional)`  
`ssh -N -f -L 45678:hpc3-gpu-17-04:45678 rhayes1@hpc3.rcic.uci.edu`  
Be sure to use your username, the port number you chose in the previous step, and the compute node that you were granted for the interactive job (not `hpc3-gpu-17-04`)

Finally, open up your favorite internet browser, using the token given in step 1:  
`http://127.0.0.1:45678/?token=1e6ed1461fd5dce44fc10e5154474aa09e1d7f26a678b379`  
At this point, you should see a jupyter interface, and can navigate to open a jupyter notebook of interest.

## VMD

VMD is a software that lets you visualize molecules and simulations. VMD is slightly better than PyMol for watching trajectories. More information and installation instructions are available at  
[https://www.ks.uiuc.edu/Research/vmd/](https://www.ks.uiuc.edu/Research/vmd/)  

## PyMol

PyMol is a powerful molecular visualization software. PyMol is better than VMD for editing structures and for python interoperability. It does require a fancy mouse. You can learn more about how to use it at  
[https://www.youtube.com/watch?v=o4XR-0VTXrY](https://www.youtube.com/watch?v=o4XR-0VTXrY)  
[https://www.youtube.com/watch?v=PH8cnZ5RjkE](https://www.youtube.com/watch?v=PH8cnZ5RjkE)

PyMol is owned by Schrodinger. They give out educational licenses for free, but not for academic research. However, there is an open source, free, and legal version of PyMol available. If you have conda installed on your local machine, you can install it with  
`conda install --channel conda-forge pymol-open-source`  
Note that it is probably only useful to install PyMol on your local machine as forwarding graphics over x11 is a nuissance.

## ParamChem

ParamChem is the software that generates CHARMM CGenFF force field files for simulations including small molecules or ligands. You can email [info@silcsbio.com](info@silcsbio.com) to request a license for ParamChem. The license is expired for the lab copy of ParamChem, so contact Professor Hayes if you need ParamChem.

ParamChem can be run on a .mol2 file using a command like  
`/dfs8/rhayes1_lab/bin/cgenff/silcsbio.2024.1/cgenff/cgenff molecule.mol2 > molecule.str`  
A .mol2 file can be generated from a smiles string using open babel (available with a conda download)  
`obabel molecule.smiles -h --gen3d -O molecule.mol2`  
but there are better ways to do it for ligand alignment. Discuss with the Mobley lab. The `-h` flag adds hydrogens. Hydrogens can also be added with `h_add` in pymol.

## msld-py-prep

This software package sets up ligand perturbations and is available from github at [https://github.com/Vilseck-Lab/msld-py-prep](https://github.com/Vilseck-Lab/msld-py-prep) . For a tutorial on msld-py-prep use, see [https://github.com/BrooksResearchGroup-UM/MSLD-Workshop](https://github.com/BrooksResearchGroup-UM/MSLD-Workshop) .

## mmtsb

This software can be used for setting up CHARMM simulations. It is installed at `/dfs8/rhayes1_lab/bin/mmtsb` . To use it, source the file `/dfs8/rhayes1_lab/bin/mmtsb/activate` .
