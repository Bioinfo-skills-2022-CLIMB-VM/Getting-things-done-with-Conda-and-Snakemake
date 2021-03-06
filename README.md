# Getting-things-done-with-Conda-and-Snakemake
This repository contains the presentation for the Conda and Snakemake session for the CLIMB-BD Bioinformatics Skills for Microbial Genomics Workshop on the 2nd February 2022 along with supporting material.

## Getting started with Conda on the CLIMB-BD VM
To get started with conda, you will first need to initialise conda. You only need to do this once. In this workshop two of you are sharing a VM under the same user, so only one of you needs to run this! You can tell if conda has already been initialised by running `conda activate base`. If you see a prompt telling you that your shell needs to be configured by running `conda init`, then run the following
```
conda init bash
```
Running this command will edit your .bashrc. You will then need to reload .bashrc for the changes to take effect
```
source ~/.bashrc
```
You should see the `base` environment activated at the start of the command prompt.

## Working with Conda Environments
The default conda environment called `base`. You should not use `base` for installing new software packages and instead you should create a new environment to keep software packages isolated from each other.

**Short Tutorial:** Creating a new Conda Environment for the bacterial genome assembler [Shovill](https://github.com/tseemann/shovill)

Conda installs software from channels. Most bioinformatics software can be found in the `bioconda` channel. The `bioconda` channel is also dependent on the `defaults` and `conda-forge` channels. To setup these channels, run the following (you only need to do this once)
```
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```
Note the order in which you run these commands is important as this sets the order of priority for the channels (i.e. this sets `conda-forge` as the highest priority). You can check the order of the channels in your config by running `conda config --show channels`

You can then use conda search to see which versions of shovill are available to install. This may take a few minutes to run.
```
conda search shovill
```
Next we'll look at creating a new environment named `shovillenv` containing shovill v1.1.0. Note that as two of you are sharing the same VM, you may want to add your initials to all of the environment names in this tutorial to avoid clashing with the other person's work!, e.g. `conda create --name shovillenv-AP shovill=1.1.0`. If when creating an environment you see the error `CondaValueError: prefix already exists`, then there is already a conda environment using this name, so you will need to pick a different name!

```
conda create --name shovillenv shovill=1.1.0
```
You will have to press `y` to proceed. When conda has finished setting up the environment, run the following to activate the environment
```
conda activate shovillenv
```
You can then run the following to check shovill has installed correctly
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
conda activate $myEnvName
```
To deactivate your current conda environment
```
conda deactivate
```
To install a new software package in your currently activated environment
```
conda install $packageName
```

## Using YAML files to build Conda Environments
Files called `environment.yml` can be used to create conda environments and share them with others.

E.g. Let's create an `environment.yml` file that will install trimgalore. Remember two of you are sharing the VM under the same user, so you should create a directory, e.g. `mkdir myDirectoryName` to save your YAML file, to avoid clashing with the other user. Open `nano environment.yml` and add the following (again you may wish to add your initials to environment name to avoid clashing with the other person your sharing the VM with!)
```
name: trim-galore

channels:
  - bioconda

dependencies:
  - trim-galore==0.6.7
```
To ensure your environment is reproducible, you should always pin the software versions (e.g. `trim-galore==0.6.7`)

To create a conda environment called `trim-galore` from this YAML file
```
conda env create -f environment.yml
```
You can also export an existing conda environment into a YAML file
```
conda env export > environment.yml
```

## Working with Mamba
Mamba is a re-implementation of conda with faster dependency solving (i.e. it's a faster way of building conda environments!). It uses the same command line parser as conda. E.g. to install shovill with mamba, we can take the command from before and simply replace `conda` with `mamba`
```
mamba create --name shovillenv2 shovill=1.1.0
```
You can then use `conda activate shovillenv2` as before.

## Getting Started with Snakemake on the CLIMB-BD VM
To use mamba to create a snakemake environment
```
mamba create --name snakemake snakemake
```
and to activate
```
conda activate snakemake
```
You can find out more about the usage of snakemake by running
```
snakemake -h
```

## Snakemake Rules
A software analysis in Snakemake is defined in a rule. Rules are contained in Snakefiles. E.g. A general template for a Snakemake rule would be the following
```
rule NAME
    input: "path/to/inputfile", "path/to/other/inputfile"
    output: "path/to/outputfile", "path/to/another/outputfile"
    shell: "somecommand {input} {output}"
```
where we define our input and output files, then call these in the shell command.

E.g.  We can write a rule for fastqc like the following, where we feed the rule a list of samples
```
sample_list = ['sampleName', 'sampleName2', 'sampleName3']

rule fastqc
    input:
        expand(["{sample}_1.fq", "{sample}_2.fq"], sample=sample_list)
    output:
        expand(["data/{sample}_1_fastqc.html" , "data/{sample}_2_fastqc.html"], sample=sample_list)
    threads: 4
    shell:
        "fastqc -o data -t {threads} {input}"
```
Notice we have also added a directive for the number of threads.

For information on how to build a workflow from rules see [here](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html)

## Using Conda and Singularity with Snakemake
Conda and Singularity can be used to manage the software dependencies of a Snakemake workflow. A global conda environment or Singularity image can be defined for the entire workflow or conda and Singularity directives can be added on a per rule basis, like so
```
sample_list = ['sampleName', 'sampleName2', 'sampleName3']

rule fastqc
    input:
        expand(["{sample}_1.fq", "{sample}_2.fq"], sample=sample_list)
    output:
        expand(["data/{sample}_1_fastqc.html" , "data/{sample}_2_fastqc.html"], sample=sample_list)
    threads: 4
    conda: "envs/fastqc.yml"
    container: "docker://quay.io/biocontainers/fastqc"
    shell:
        "fastqc -o data -t {threads} {input}"
``` 
On running the workflow, the flag `--use-conda` or `--use-singularity` is used to select either the conda environment(s) or Singularity container(s). Snakemake can also use an ad-hoc combination of conda environments with Singularity containers (i.e. it will build the conda environment in a container). For more on distribution and reproducibility with Snakemake see [here](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html)


## Snakemake Tutorials
Tutorials for Snakemake can be found here: <br />
https://snakemake.readthedocs.io/en/stable/tutorial/basics.html <br />
https://snakemake.readthedocs.io/en/stable/tutorial/advanced.html


