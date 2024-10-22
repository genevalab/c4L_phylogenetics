# c4L_phylogenetics
C4L Phylogenetics tutorial



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
