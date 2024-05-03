# Using bash and linux to Access the Cluster

## Learn to Use bash and linux

The place you’ll be using bash is on the hpc3 cluster, as well as on your local machine to interact with the cluster. You will be accessing the cluster through a text based interface called the terminal. On Mac, terminal is a separate program, which you should place on your dock. (You can find it in the Utilities folder of Applications). On Windows, you can activate WSL and run commands through the windows terminal, or else you can download and install Putty for terminal access and WinSCP for file transfer. Use the former option (WSL) if possible. If you use Linux, you probably already know what a terminal is.

In order to log in to the hpc3 cluster, you can type  
`ssh panteater@hpc3.rcic.uci.edu`  
into your terminal. Replace `panteater` with your UCI NetID. `hpc3.rcic.uci.edu` is the URL of the cluster.

The hpc3 people have assembled a useful tutorial on how to use bash and shell scripting  
[https://rcic.uci.edu/tutorials.html](https://rcic.uci.edu/tutorials.html)  
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
will tell you how much space we have left. Don't let it get to 0, or jobs will crash. Ask Dr. Hayes to buy more, or clean out some old files. If you are trying to find which directories are taking up all the space, use the `du` command.

There are some issues with copying to and from BeeGFS. Read about how to copy files on BeeGFS from this link  
[https://rcic.uci.edu/storage/dfs.html](https://rcic.uci.edu/storage/dfs.html)  
once you’ve become more familiar with bash and linux and the `scp` command.

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
