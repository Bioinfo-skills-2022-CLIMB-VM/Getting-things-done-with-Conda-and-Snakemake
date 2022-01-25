# Getting-things-done-with-Conda-and-Snakemake
This repository contains the presentation for the Conda and Snakemake session for the CLIMB-BD Bioinformatics Skills for Microbial Genomics Workshop on the 2nd February 2022 along with supporting material.

## Getting started with Conda on the CLIMB-BD VM

To initialise Conda
```
conda init bash
```
Running this command will edit your .bashrc. You will need to reload .bashrc for the changes to take effect
```
source ~/.bashrc
```
You should see the `base` environment activated at the start of the command prompt

## Working with Conda Environments
The default conda environment called `base`. You should not use `base` for installing new software packages and instead you should create a new environment to keep software packages isolated from each other.

**Short Tutorial:** Creating a new Conda Environment for the bacterial genome assembler [Shovill](https://github.com/tseemann/shovill)

Conda installs software from channels. Most bioinformatics software can be found in the `bioconda` channel. The `bioconda` channel is also dependent on the `defaults` and `conda-forge` channels. To setup these channels, run the following (you only need to do this once)
```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```
Note the order in which you run these commands is important as this sets the order of priority for the channels (i.e. this sets `conda-forge` as the highest priority).

You can then use conda search to see which versions of shovill are available to install. This may take a few minutes to run.
```
conda search shovill
```
To create a new environment named `shovillenv` containing shovill v1.1.0
```
conda create --name shovillenv shovill=1.1.0
```
Run the following to check shovill has installed correctly
```
shovill --check
```
To see which conda packages are installed in your `shovillenv` environment you can run
```
conda list
```

### More Useful Commands
To see a list of all your environments
```
conda info --envs
```
To activate an environment
```
conda activate myEnvName
```
To deactivate your current conda environment
```
conda deactivate
```
To install a new software package in your currently activated environment
```
conda install packageName
```
