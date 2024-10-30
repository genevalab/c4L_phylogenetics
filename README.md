# c4L_phylogenetics
C4L Phylogenetics tutorial

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
The command prompt should not start with ```(trial)```. If so you are now inside your environment and can start your work! When you are done and ready to leave an environment simply run this command to return to the (base) environment:
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

# Running a Bayesian Analysis
Now that we have MrBayes working we'll run a simple analysis using data provided inside our conda environment.

First lets create a new directory to perform our analysis
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

With everything in place, we can now open MrBayes and load our data file to perform an analysis. First load MrBayes
```
mb
```
now we want to load our DNA alignment 

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


