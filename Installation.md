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
`../dev-release/configure --with-gnu --without-mkl --without-openmm --with-blade -u --repdstr --with-nvtx -p ./inst &> ../config.log`  
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
# `git clone git@github.com:RyanLeeHayes/ALF.git`  
`git clone https://github.com/RyanLeeHayes/ALF.git`  
The `README.md` file in this directory is a useful source of information on how to use the software, and there are several examples inside the examples directory. Put your copy of ALF somewhere in your home or BeeGFS directories. To get ALF ready for use, follow these steps.

1. First you need to setup the environment to compile the code in this directory by running the following two lines:  
`module load cmake/3.20.5 cuda/11.7.1 gcc/11.2.0 openmpi/4.1.2/gcc.11.2.0 fftw/3.3.10/gcc.11.2.0-openmpi.4.1.2`  
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

## VMD

VMD is a software that lets you visualize molecules and simulations. VMD is slightly better than PyMol for watching trajectories. More information and installation instructions are available at  
[https://www.ks.uiuc.edu/Research/vmd/](https://www.ks.uiuc.edu/Research/vmd/)  
For Mac, the most recent version 1.9.4 should work fine, for Windows, you may need to install 1.9.3 for it to work correctly.

## PyMol

PyMol is a powerful molecular visualization software. PyMol is better than VMD for editing structures and for python interoperability. It does require a fancy mouse. You can learn more about how to use it at  
[https://www.youtube.com/watch?v=o4XR-0VTXrY](https://www.youtube.com/watch?v=o4XR-0VTXrY)  
[https://www.youtube.com/watch?v=PH8cnZ5RjkE](https://www.youtube.com/watch?v=PH8cnZ5RjkE)

PyMol is owned by Schrodinger. They give out educational licenses for free, but not for academic research. However, there is an open source, free, and legal version of PyMol available. If you have conda installed on your local machine, you can install it with  
`conda install --channel conda-forge pymol-open-source`  
Note that it is probably only useful to install PyMol on your local machine as forwarding graphics over x11 is a nuissance.

## ParamChem

ParamChem is the software that generates CHARMM CGenFF force field files for simulations including small molecules or ligands. You can email [info@silcsbio.com](info@silcsbio.com) to request a license for ParamChem.
