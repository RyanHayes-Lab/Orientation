# Using the Cluster

## Introduction

The cluster is a large group of computers on campus that we use to run calculations in the group. You can't do much in the group until you have access to it and have learned how to use it. bash is the primary computer scripting language you'll be using to interact with the cluster. The cluster has a queue system managed by the slurm software that you can use to run your individual calculations. Beyond the documentation provided here, RCIC maintains extensive documentation on how to use the cluster, including an introduction for beginners which you may wish to read:  
[https://rcic.uci.edu/guides/beginner.html](https://rcic.uci.edu/guides/beginner.html)  
RCIC also maintains an email that you can contact for support with computing problems:  
[rcic-support@uci.edu](rcic-support@uci.edu)

## Learn to Use bash and linux

The place you’ll be using bash is on the hpc3 cluster, as well as on your local machine to interact with the cluster. You will be accessing the cluster through a text based interface called the terminal. On Mac, terminal is a separate program, which you should place on your dock. (You can find it in the Utilities folder of Applications). On Windows, you can activate WSL and run commands through the windows terminal, or else you can download and install Putty for terminal access and WinSCP for file transfer. Use the former option (WSL) if possible. If you use Linux, you probably already know what a terminal is.

In order to log in to the hpc3 cluster, you can type  
`ssh panteater@hpc3.rcic.uci.edu`  
into your terminal. Replace `panteater` with your UCI NetID. `hpc3.rcic.uci.edu` is the URL of the cluster.

The hpc3 people have assembled a useful tutorial on how to use bash and shell scripting  
[https://rcic.uci.edu/guides/tutorials.html](https://rcic.uci.edu/guides/tutorials.html)  
[https://swcarpentry.github.io/shell-novice/](https://swcarpentry.github.io/shell-novice/)  
These links are useful to learn and read in their entirety, but you will remember them better if you’re trying to use what you learn for a specific task (such as setting up and running a protein simulation) while you’re doing them. When I first started my graduate research, I found a bash tutorial and worked on just that for a week straight until I knew the ins and outs of how to use a cluster. If you don't know how to use bash and linux, going through these tutorials is one of the first things you should do in the lab.

## Lab Resources

Your home directory is the first directory you enter when you log into the clusteter. Your home directory doesn't have much room (50 Mb), so it should only store programs, and other small directories that rarely change. My home directory is  
`/data/homezvol0/rhayes1`  
You can find yours by typing `cd` or `cd ~` to get to your home directory, and then `pwd` to list what directory you are in.

Our group has just 1 Tb of CRSP storage. It is located at  
`/share/crsp/lab/rhayes1`  
This space is internet accessible, but is inconvenient to use because there are limits on the number of files in addition to the amount of storage. Use BeeGFS instead for most applications. Our group does store git repositories for various softwares we use in CRSP. They are located in  
`/share/crsp/lab/rhayes1/share/git`

Our group has 40 Tb of BeeGFS storage. It is located at  
`/dfs8/rhayes1_lab`  
You should make a directory here with your username to hold your data.  
`cd /dfs8/rhayes1_lab`  
`mkdir panteater`  
(Where `panteater` is your UCI NetID). The command  
`dfsquotas rhayes1 dfs8`  
will tell you how much space we have left. Don't let it get to 0, or everyone's jobs will crash. Ask Dr. Hayes to buy more, or clean out some old files. If you are trying to find which directories are taking up all the space, use the `du` command, for example:  
`du -m --max-depth=2 /dfs8/rhayes1_lab`

There are some issues with copying to and from BeeGFS. Read about how to copy files on BeeGFS from this link  
[https://rcic.uci.edu/storage/dfs.html](https://rcic.uci.edu/storage/dfs.html)  
once you’ve become more familiar with bash and linux and the `scp` command. As a summary, every file should belong to the rhayes1_lab_share group, and have the sticky bit set. If not, you will get cryptic errors saying you don't have permission to write files. You can change the group of a file named `problematicfile` with the `chgrp` command, for example  
`chgrp rhayes1_lab_share problematicfile`  
You can set the sticky bit of a directory named `problematicdirectory` with `chmod`, for example  
`chmod g+s problematicdirectory`

Several programs commonly used by the group are available in  
`/dfs8/rhayes1_lab/bin`  
including the CHARMM executable  
`/dfs8/rhayes1_lab/bin/CHARMM_EXE/gnu/charmm`  
CHARMM is the software the lab uses to run molecular dynamics simulations, and will be described later.

## Setup ssh Keys

Life is too short to use two factor authentication every time you log into the cluster. Setup ssh keys so you don't have to use two factor authentication for logging into the cluster. hpc3 has an explanation of how this works  
[https://rcic.uci.edu/account/login.html](https://rcic.uci.edu/account/login.html)  
as well as a link to a tutorial for Windows. See below for Mac.

For Mac: run  
`ssh-keygen`  
on your local machine and enter a passphrase. Use the default file locations. This will generate `~/.ssh/id_rsa` (the private key) and `~/.ssh/id_rsa.pub` (the public key) on your local machine. `~/.ssh` is a hidden directory inside your home directory. (Directories starting with . are hidden.)  
Copy `~/.ssh/id_rsa.pub` into the end of a file called `~/.ssh/authorized_keys` on hpc3. For example, if the file doesn’t exist yet, you can just run  
`scp ~/.ssh/id_rsa.pub panteater@hpc3.rcic.uci.edu:~/.ssh/authorized_keys`  
but if the file already has authorized keys in it this will overwrite them, and you need to do something more careful using what you learned in the bash and linux tutorial.

## The SLURM Queue System

Create the following file as a hello world test:  
`#! /bin/bash`  
`echo "hello world from"`  
`hostname`  
and save is as `HelloWorld.sh`. Hello world is a term from computer science for a program that just prints output to let you know it’s working.

You should never run CHARMM (or do much of anything else that takes longer than a minute) from the login nodes (sometimes also called head nodes). When you first log into the cluster with `ssh`, the login node is the computer that runs all your commands. You don’t want the login nodes to be busy, otherwise routine tasks like logging in or making a directory or copying a file take forever for everyone using that login node. Therefore, clusters have many other computers (called compute nodes) to perform calculations. To use a compute node you have to request it, and then wait in line. The line is called a queue (which is what lines are called in Great Britain). Thus you should submit computationally demanding tasks to the queue. Our job scheduler is called slurm, and the hpc3 documentation has good information on how to use it:  
[https://rcic.uci.edu/slurm/slurm.html](https://rcic.uci.edu/slurm/slurm.html)  
This `HelloWorld.sh` script is much faster than a minute, but we'll use it to practice using the queue.

There are two ways to request slurm to give you a compute node: either by submitting an interactive job or by submitting a slurm script with `sbatch`

An interactive job allows you to log into a compute node and execute commands there rather than on the login node. The two nodes share the same file system, so everything you do there will show up back on the login node. To start an interactive job you can type  
`srun --pty /bin/bash -i`  
There will be a delay until the job starts, and then you can continue entering commands. All new commands will run on the compute node until you type `exit`. Before you type `exit` try typing `bash HelloWorld.sh` to run the hello world script. The primary use of interactive jobs is for debugging or trying out new ideas, when you are running heavy calculations, possibly with a GPU, but don't know in advance all the commands you wish to execute.

Alternatively, you can submit a bash script to run on a compute node with the `sbatch` command. The slurm script will run on the compute node and then exit. You can submit `HelloWorld.sh` as a slurm script by typing  
`sbatch HelloWorld.sh`  
Your output will appear by default in a file called `slurm-*.out` where `*` was the slurm job id. `sbatch` and slurm scripts are used when you know exactly what you want to do and can write a script to do it. You can submit as many jobs to the queue as you wish, and many of them can run on different compute nodes at the same time.

Not all jobs are created equal. Some are fast and some are slow, some require GPUs and some do not, some are urgent and others are not. Slurm applies default options to every job, which may or may not be what you need, so in most cases you should tell slurm about your job using options so it gives you only what you need, and leaves remaining resources available for other users. For some set of options `--option1=value1 --option2=value2`, you can give the options to an interactive job as  
`srun --option1=value1 --option2=value2 --pty /bin/bash -i`  
and you can give them to sbatch either as  
`sbatch --option1=value1 --option2=value2 HelloWorld.sh`  
or by placing them immediately after the shebang line (`#! /bin/bash`) in the script  
`#! /bin/bash`  
`#SBATCH --option1=value1`  
`#SBATCH --option2=value2`  
`echo "hello world from"`
`hostname`

There are many slurm options, and most have a long form and a short form. The long form is two dashes followed by a word followed by an equal sign followed by a value (`--partition=free-gpu`). The short form is one dash followed by a letter followed by a space followed by a value (-p free-gpu`). These are some of the most important options:

`-p`  
The `-p` option controls what partition you run on. The main partitions are `standard` and `free` for CPU jobs and `gpu` and `free-gpu` for GPU jobs. The default option is `standard`. Thus to run the job on the `free-gpu` partition, you would use `-p free-gpu` as the option. Each partition is like a separate queue, so you can have jobs waiting in each of them. The free variants `free` and `free-gpu` cost the lab nothing to run, but if a job is submitted to the paid queue and the free queue job is in its way, the free job will be killed, and your `slurm-*.out` file will have a message about the job being preempted. The scheduler chooses which job to kill by whichever job has been running for the shortest time for avoid killing jobs that are almost finished. The paid queues cost the lab $0.01 per CPU hour and $0.32 per GPU hour. You can see how much computing time the lab has remaining per six month period with the `sbank` command:  
`/pub/hpc3/sbank_test/sbank balance statement -a rhayes1_lab_gpu`  
If you run out of time, I can reallocate time between lab members. For deciding between the paid and free queues, you should try to use most of your balance within each six month period, but you should keep a little in the paid queue for running debugging jobs so you don't have to wait for as long. When you are first starting out, you should run things in the paid queue until you get comfortable with the cluster.

`-A`  
The `-A` or `--account` option controls the account a job is charged against. If you are using the `free` or `free-gpu` partitions, you do not need to specify an account, and might get an error if you do. To run on the `standard` partition, you will need to specify the `rhayes1_lab` account, and to run on the `free-gpu` partition, you will need to specify the `rhayes1_lab_gpu` account, e.g. `-A rhayes1_lab_gpu`. You also have personal accounts you can charge for small jobs, and these personal accounts are the default values, so be sure to specify a lab account if you use a paid queue. Everything on hpc3 is paid in advance, so you don't need to worry about accidentally charging your personal accounts. The worst that can happen is you run out of time.

`--time`  
This sets the maximum length of a job so slurm knows when it can schedule the next job. If a single number is given, the time limit is in minutes, e.g. `--time=1440` requests a full day for the job to run. We will only be billed for as much time as the job uses, so if the job finishes after an hour, we don't have to pay for the full day.

`--gres`  
This requests a generalized resource, and is the mechanism for requesting a GPU. Even if you run on the `gpu` or `free-gpu` partitions, you will not receive a GPU unless you request one. To request one GPU (per node), use `--gres=gpu:1`. There are three types of GPUs available on the cluster: A30, A100, and V100. You can request an A30 using `--gres=gpu:A30:1`. Since different GPUs run different speeds, it can be important to request a paticular type of GPU if you're running benchmarks to see how long a program takes.

`--mem`  
This controls how much RAM a job requests. Our jobs typically run on one GPU and one CPU, but sometimes require more RAM than the 3 Gb allocated to one CPU. CHARMM-GUI jobs and postprocessing on long productions simulations are examples that reuire more memory. Typically, you should not request more memory unless your job gets killed with an OOM (out of memory) error message in your `slurm-*.out` file. You can request more, (30 Gb is usually enough) with `--mem=30G`, or if you really want to be safe, you can request all the RAM on a node with `--mem=0`, but this should be avoided because then no one else on the cluster can use the rest of the node.

Other default options  
`--ntasks=1`  
`--tasks-per-node=1`   
`--cpus-per-task=1`  
`--export=ALL`

Other options to look up  
`--job-name` Give the job a different name so it's easier to find when looking at the queue  
`--output` Write the log file somewhere other than `slurm-*.out`  
`--error` Write the error file somewhere other than `slurm-*.out`  
`--dependency` Make a job wait for another job

You can keep an eye on your jobs with the `squeue` command. To see all your jobs run  
`squeue -u panteater`  
(Remember to use your username). If one of your jobs is 2718281, you can get a bunch more information about it with  
`scontrol show jobid=2718281`  
More help on the slurm job scheduler is available at  
[https://rcic.uci.edu/slurm/slurm.html](https://rcic.uci.edu/slurm/slurm.html)
