# c4L_phylogenetics
C4L Phylogenetics tutorial, for week 2 material click [here](https://github.com/genevalab/c4L_phylogenetics/blob/main/README.md#meeting-2).

# Running MrBayes on Amarel
First we will need to make a new conda environment for you to install MrBayes. The below lines will walk you through making a new environment called ```mb```

If you have never used anaconda on amarel you will first need to set it up. To test if you have previously set up conda in your environment type ```which conda``` if that does not return anything then follow the directions below, othwewise jump down to to the ["make mrbayes conda env"](https://github.com/genevalab/c4L_phylogenetics/blob/main/README.md#make-mrbayes-conda-env) section below.


# Using conda on Amarel
Instructions for using conda on the Rutgers Amarel HPC

Conda is an open-source, cross-platform, language-agnostic package manager and environment management system. Conda packages are binaries. No compilers needed to install them.  We typically use Anaconda which is a Conda package repository that includes many python packages and extensions, such as Conda, numpy, scipy, ipython notebook, etc..

While Conda packages are binary distribution allowing fast installation, other forms of installation are supported inside Conda environments, including pip, or any other source installation.  Each package installs along with a list of dependent packages by default.

If you have never used anaconda on amarel you will first need to set it up. Instructions below are derived from the [Amarel user manual](https://sites.google.com/view/cluster-user-guide/amarel/applications#h.t375cfil74a7) but the ones here are updated so use mine.

##To start this process first, determine what the most recent anaconda install is on Amarel
```
module use /projects/community/modulefiles
module keyword anaconda
```

As of 22 October 2024 the output is:
```
  anaconda: anaconda/2022.10-bd387, anaconda/2023.07-ts840, anaconda/2023.10-bd387
    Sets up Anaconda 2022.10 for your environment

  miniconda: miniconda/2023.11-bd387
    Sets up miniconda 2023.11 for your environment

  py-bigdata: py-bigdata/2020-bd387, py-bigdata/2021-bd387
    Sets up anaconda in your environment for tensorflow and keras

  py-data-science-stack: py-data-science-stack/5.1.0-kp807
    Sets up anaconda 5.1.0 in your environment

  py-image: py-image/2020-bd387
    Sets up anaconda in your environment for tensorflow and keras

  py-machine-learning: py-machine-learning/intel18/cuda12/pgarias, py-machine-learning/2023-pgarias
    Sets up anaconda in your environment for tensorflow and keras

```
The most recent anaconda module listed is ```anaconda/2023.10-bd387``` so we will use that in the code below. Be sure to run the keyword seach yourself and use the most recent version availble to you.


```
module use /projects/community/modulefiles/
module load anaconda/2023.10-bd387
conda init bash   ##configure your bash shell for conda, auto update your .bashrc file
cd
source .bashrc
mkdir -p .conda/pkgs/cache .conda/envs   ##These are the folders to store your own env you going to build

```
Log out of Amarel and back in. Now when you log in, you should see ```(base)``` to the left of your netid at the command prompt. If not, then make sure you ran all of the steps above again. If you see (base) you are all set with conda set up and you are ready to start making your own environments!

## Create an environment
To create a conda environment you need only supply a name (athough there are lots of additional options. To make an new environment called ```trial``` run the code below.

```
conda create --name trial
````
Once created you will need to activate the enivornment to start using it:
```
conda activate trial
```
The command prompt should now start with ```(trial)```. If so you are now inside your environment and can start your work! When you are done and ready to leave an environment simply run this command to return to the (base) environment:
```
conda deactivate
```


# Useful Conda commands

To see env already installed:
```conda env list```

You can install prebuilt packages or collections of packages from [bioconda](https://bioconda.github.io) or [conda-forge](https://conda-forge.org). First go to the webpages of the repository and search for the name of the package you want and enter it below where ```<THENAMEOFTHEMODULE>```
```
conda install <THENAMEOFTHEMODULE> -c bioconda -c conda-forge
```

# Make a MrBayes Conda environment
Once you have configured conda in your home directory you can proceed to creating an environment for MrBayes. While not required, we are going to first request an interactive session to speed up the time it takees to build your environment. You only need to build this environment once.

```
#start your interactive session
srun --nodes=1 --ntasks-per-node=1 -p cmain --time=01:00:00 --pty bash -i


conda create --name mb python=3.8.8
conda activate mb
conda install mrbayes -c bioconda -c conda-forge

# now test if MrBayes installation was sucessful
mb
```
That should load the program and present a MrBayes prompt. If it doesn't, ask for help before proceeeding. If you are good, exit MrBayes by running:

```
quit
```

Then close your conda environment
```
conda deactivate
```
and exit your interactive session.
```
exit
```

# Running a MrBayes analysis
Every time you plan to run MrBayes you should start an interactive job on Amarel. I know this is repetitive - we just opened and closed an interactive job above, but its imperative you not run MrBayes on the amarel head node, so I including starting an interactive session as part of the code block below.

```
#start a new interactive session
srun --nodes=1 --ntasks-per-node=1 -p cmain --time=01:00:00 --pty bash -i

#load conda env
conda activate mb
```


First, lets create a new directory to perform our analysis:
```
# move to your home directory 
cd ~

# create and then move into new directory inside your home directory
mkdir C4L_phylo
cd C4L_phylo
 
# lets copy the example data provided with MrBayes to this directory so we can later edit it
cp ~/.conda/envs/mb/share/examples/mrbayes/primates.nex .
```

Take a look at your data file by running ```more primates.nex```

As you can see, this nexus file contains flat text with some basic summary information (the number of taxa and the number of characters) followed by one line per sample with the name of the sample and a DNA sequence. These data are homologus gene regions that have been sequenced for each taxon and aligned so that comparisons among taxa will make use of homologous characters.

With everything in place, we can now open MrBayes and load our data file to perform an analysis. First load MrBayes:
```
mb
```
now we want to load our DNA alignment:

```
execute primates.nex
```

This should have loaded your data into MrBayes. No to run an analysis we only need to few more things. First we need assign a model of molecular evolution. Typically the optimal model of evoluion is determined by a preliminary analysis that using Information Theory to identify the best-fitting model for the the present dataset but for the purposes of today's workshop we are going to use the a parameter rich model "GTR + I + G". This model allows for separate rate distributions for each possible nucleotide substition, assumes some portion of sites in the alignemnt will in invariable, and models the distribution of rates for each substitution type as Gamma distributed.

```
lset nst=6 rates=invgamma
```

Next we will set the number of mcmc generations to run. For the purposes of today's lab I am setting this VERY low so that it finishes in a reasonable amount of time
```
mcmcp ngen=100000
```

For the purposes of this workshop we are going to set some manual seed values so everyone's run gives the same results. In a true analysis you would typically not set a seed in this way

```
set seed=12345 swapseed=12345
```

While there are myriad other parameters we might want to set, we will leave those at thier default settings for now. To begin an analysis simply run
```
mcmc
```

The entire run should take about a minute at which time MrBayes will ask if you want to continue. Answer no and the run will complete and print some summaries of the mcmc analysis. In the next workshop meeting we will investigate in detail how to assess for convergence of your MCMC runs, discard as burn-in the MCMC generations prior to convergence, and finally how to summarize a final tree and visualize it using open source software. We'll also review how to submit a MrBayes run to the SLURM queue.

So as to not leave you hanging - you can run this command to see a preliminary view of your tree.

```
sumt
```

# Meeting 2
Our second meeting of the inferring phylogenies workshop will be entirely hands-on. In order to prepare, please download and install [Tracer](https://github.com/beast-dev/tracer/releases/tag/v1.7.2) and [FigTree](https://github.com/rambaut/figtree/releases) on your laptop.

Our learning goals for the second meeting are:

* Submit MrBayes jobs using sbatch
* Diagnose convergence by examining MCMC trace files in Tracer
* Create a consensus tree from the post-burnin posterior distribution trees
* Generate phylogenetic tree figures using FigTree

## Submitting MrBayes jobs using sbatch
Phylogenetic analayses can take a while to run so most of the time we want to submit them as batch jobs. To accomplish this we first need to set up our alignment file to run MrBayes in non-interactive mode. MrBayes recognizes code blocks within nexus files, so we can pass alll of the same commands we used last week in interactive mode by pasting the code block below to the very bottom of your nexus file.

```
BEGIN mrbayes;
set seed=12345 swapseed=12345;
lset  nst=6 rates=invgamma;
mcmc ngen=100000;
END;

```

If you wanted to now run this analysis, you would activate your conda environment and run MrBayes with the name of your alignment as an added argument (eg ```mb YOURNEXUSFILE.nex```). This will load the alignment, set the parameters we selected, and begin the MCMC. While this is great, we would still need to keep an interactive session open until the MCMC finished, so the second set is to write a batch script to submit your job to the SLURM scheduler. 

The bulk of a submission script is pasted below. You can copy it and then create a new file on Amarel called ```run_mrbayes.sh``` using your favorite text editor in the directory that contains your alignment. Next, edit to include the correct vaules for ```YOUREMAILADDRESS```, ```YOURMBCONDAENVIRONMENT```, ```YOURALIGNMENT.nex```. 

```
#!/bin/bash
#SBATCH --partition=cmain   			# which partition to run the job, options are in the Amarel guide
#SBATCH --constraint=oarc			# excludes owned nodes on cmain
#SBATCH --job-name=mrB	 			# job name for listing in queue
#SBATCH --output=slurm-%j-%x.out
#SBATCH --mem=10G				# memory to allocate in Mb
#SBATCH -n 1	 				# number of cores to use
#SBATCH -N 1 					# number of nodes the cores should be on, 1 means all cores on same node
#SBATCH --time=0-02:00:00			# maximum run time days-hours:minutes:seconds
#SBATCH --no-requeue 				# restart and paused or superseeded jobs
#SBATCH --mail-user=YOUREMAILADDRESS@scarletmail.rutgers.edu 		# email address to send status updates
#SBATCH --mail-type=BEGIN,END,FAIL 	# email for the following reasons


echo "load any Amarel modules that script requires"
module purge					# clears out any pre-existing modules

echo "## setup conda environment"
eval "$(conda shell.bash hook)"
conda activate YOURMBCONDAENVIRONMENT

echo "## run mrB through conda environment"
mb YOURALIGNMENT.nex

```
Once your submission script is saved, all it take to submit a MrBayes jobs is to type:

```
sbatch run_mrbayes.sh
```

To check on the status of your job, you can run ```squeue -u YOURNETID```.

## Diagnosing convergence by examining MCMC trace files in Tracer

The output of any MrBayes run should create the following files:
```
primates.nex.mcmc
primates.nex.run1.p
primates.nex.run1.t
primates.nex.run2.p
primates.nex.run2.t
```

These files contain the results of all logged generations of your MCMC run. To assess convergence, we need to examine the .p files. So using SSH or [OnDemand](ondemand.hpc.rutgers.edu), download ```primates.nex.run1.p``` and ```primates.nex.run2.p``` to your local machine. Once everyone has reached this point we will proceed interactively.

## Creating a post-burnin consensus tree

Now that you have diagnosed convergence we have to do second, very short, run on MrBayes to summarize all of your post-burnin trees so that we can create a single tree that represents the entire posterior distribution of trees.

First, we need to edit the MrBayes block at the bottom of your alignement file. We'll comment out the line that issued the MCMC command and add a line to issue the summarize trees command. MrBayes recognizes text in square brackets ```[like this]``` as comments and doesn't parse them, so put square brackets around the mcmc line (put the semicolon inside the brackets too).

Next, we will add a new line with the command ```sumt Burninfrac=0.XX```. We'll determine the fraction to discard in the previous section. If we decided to discard the first 25% of the run, then set Burninfrac=0.25.

Your new MrBayes block should look something like this:

```
... alignment ...
end;

BEGIN mrbayes;
set seed=12345 swapseed=12345;
lset  nst=6 rates=invgamma;
[mcmc ngen=100000;]
sumt Burninfrac=0.25;
END;

```

Save your alignment file and we can execute it again. This will only take a minute or two to run so you can do this via a batch job or as an interactive session (but not on the head node!).

When completed, you should have a set of additional output files that have summarized your posterior distribution. The one we are after for now will be a tree file named ```primates.nex.con.tre```. Use SSH or OnDemand again to transfer that file to your laptop.

## Generating a phylogenetic tree figure

We'll do this part entirely interactively, so once you all have your tree files transferred onto your laptops, we'll proceed together.



