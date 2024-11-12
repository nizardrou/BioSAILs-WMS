# BioSAILs-WMS
This repo is intended to accompany the guided hands-on tutorial on workflow management systems (WMS) using BioSAILs.

During this tutorial, the participants will learn the basics of WMS, the YAML format, how to read workflows, define them, and execute them.

By the end of this workshops that participants are expected to learn:

 - Know what WMS are, and why they are critical in deploying production level analytical pipelines.
 - Know how workflows are defined in YAML format within the context of BioSAILs, and the differences between GLOBAL and LOCAL parameters.
 - Know what special variables are used in BIOSAILs, and know how to initialize user-defined variables.
 - Know how to organize the data inputs and outputs in an efficient manner within the BioSAILs ecosystem.
 - Know how to use the Biox command to create an executable script.
 - Know how to use the HPCrunner command to submit the executable script that Biox created.
 - Know how to read the logs, and troubleshoot error codes or potential failures.
 - Understand "Best practices" when creating and executing workflows.

We will be aiming to achieve the above learning outcomes through a series of 3 excercises:

1. Submitting a simple quality checking and quality trimming workflow.
2. Submitting a simple alignment workflow of the quality trimmed reads (using BWA) whereby users have to "tweak" the parameters in order to find the samples.
3. Wrtie a simple 3 rule RNAseq workflow whereby users have to write each one of these analysis steps (rules) in the YAML workflow themselves, execute the workflow, find the outputs, and interogate the output logs to find the the overall alignment rates.

**Note**:
If you are participating in this workshop, all of the training will be conducted using the NYUAD HPC cluster (Jubail). If you are following this tutorial independently, and would like to run BioSAILs on your own setup (which you can), you will have to install BioSAILs (and any software modules that you would like to run for your analysis). BioSAILs is available for installation through Bioconda, and instruction on how to deploy BioSAILs and run a test RNAseq workflow on some example data will be provided at the end of this page.

The presentation slides accompanying this workshops are available in this repo, as well as the datasets and the workflows.

## Setting up the environment and copying the data
We will be using the NYUAD High Performance Computing (HPC) cluster for this workshop, however, you can certainly run all of the analysis on any stand alone machine (server, personal laptop/Desktop etc.) provided that you have pre-installed the necessay software packages.


### Connecting to the HPC using a MAC/Linux machine and copying the data.
1. Open the "Terminal" app and type `ssh NetID@jubail.abudhabi.nyu.edu`. Enter your NYU password when prompted and hit enter.
2. Once logged in, navigate to your personal "SCRATCH" directory `cd $SCRATCH`.
3. Create a directory and change into it `mkdir biosails && cd biosails`.
4. Copy the workshop material 'cp /scratch/gencore/nd48/biosails_training/* .'.

### Connecting to the HPC using a Windows machine and copying the data.
1. Open the "Putty" app, and fill out the fields as follows **Host name**=jubail.abudhabi.nyu.edu, **Port**=22, and then click on "Open".
2. Enter your NetId, and your password when prompted.
3. Once logged in, navigate to your personal "SCRATCH" directory `cd $SCRATCH`.
4. Create a directory and change into it `mkdir biosails && cd biosails`.
5. Copy the workshop material 'cp /scratch/gencore/nd48/biosails_training/* .'.


## Execise 1: Submitting a simple QC/QT workflow

For our first excercise, we will be submitting a workflow that has 3 rules, or analysis steps, raw quality checking using FASTQC, quality trimming using FASTP, and rechecking of the quality trimmed reads using FASTQC again.
The data comprises of 3 paired-end short read sequencing E coli samples.

Let's start by loading the BioSAILs software environment,

```
module purge
module load gencore
module load gencore_biosails
```

### The BIOX command
You can now run 'biox --help' to bring up the Biox command help menu.

Our workflow is called **qc-qt.yml**, and to run the first part of our analysis, we need to use the **biox**, which will read this YAML workflow (qc-qt.yml), look at the directory containg the input (the current directory containing the 3 e coli samples), and produce an executable shell script. 

To do this,

```
biox run -w qc-qt.yml -o my-qc.sh
```

So let's breakdown the biox command a little bit,

- First we used the "run" sub-command, which tells biox that we want to, well, "run" a workflow and create an output.
- The "-w" flag tells biox, which workflow we are executing, and it always expects an input, in this case the qc-qt.yml workflow.
- The "-o" also expects a parameter, which is the name of our output shell script file, in this case we called it "my-qc.sh" but you can give it any name (although you should use the .sh suffix).

But what if we don't want to run on all the samples, and what if we only wanted to run the sample "sub_sample1" for example. In this case, we can overwrite the automatic sample searching by providing the "--samples" flag to biox followed by the name of the sample we are interested in, for example,

```
biox run -w qc-qt.yml --samples sub_sample1 -o my-qc-sub_sample1.sh
```

We can also provide multiple selected samples that are comma-separated, for example,

```
biox run -w qc-qt.yml --samples sub_sample1,sub_sample2 -o my-qc-sub_samples_1_and_2.sh
```

Biox will not only create the executable shell script, but it will also create the output directory structure for us (according to the instructions that were provided in the YAML workflow). In this case, the workflow defined the output (outdir) variable as "data/processed/", and so biox will create this directory, and proceed to create 3 sub-directories, one for each of our samples. You can see that for yourselves,

```
ls -1 data/processed
```

Biox will also create an output directory per rule (analysis step) according to what that rule is called in our workflow by default, and in our case, the 3 rules are called raw_fastqc, fastp, qt_fastqc.

```
ls -1 data/processed/*/*
```

This behaviour cand be overwritten ib biox by defining a local variable "create_outdir: 0" within the rule that we don't want to have an output directory created.

For example, if we did not want the raw_fastqc rule to have an output directory create within the samples directories by default, we will change it from this,

```
    - raw_fastqc:
        local:
            - indir: "{$self->root_in_dir}"
            - outdir: "{$self->raw_fastqc_dir}"
            - OUTPUT: "{$self->outdir}/{$sample}"
            - HPC:
               - walltime: '08:00:00'
               - cpus_per_task: 12
               - mem: '55GB'
```
To this (look at the 3rd line after the "local:"),

```
    - raw_fastqc:
        local:
            - create_outdir: 0
            - indir: "{$self->root_in_dir}"
            - outdir: "{$self->raw_fastqc_dir}"
            - OUTPUT: "{$self->outdir}/{$sample}"
            - HPC:
               - walltime: '08:00:00'
               - cpus_per_task: 12
               - mem: '55GB'
```

### The HPCrunner command

Greate, so now we are ready to submit the executable shell script in the SLURM queue, and it couldn't be simpler than,
```
hpcrunner.pl submit_jobs -i my-qc.sh --project my-qc
```
And in fact, you don't even need the "--project my-qc" flag, we only add it because it will use the name that we provide to "--project" (my-qc in this case) as the name of the submission in SLURM, and the name of the logs output directory. This makes it easier in case you are submitting multiple analyses to figure out, which ones are running, completed pending etc. It also means that it is then easier to navigate the logs folders in case you submitted multiple analyses within the same folder. So in essence, it's just good practice!

If you don't have a scheduler (for example if you are submitting this on your own machine), then you can bypass the hpcrunner.pl stage and just run the executable on the command line. Jus t make sure to change the permissions first `chmod 777 my-qc.sh` and then simply run it `./my-qc.sh`.

Since we submitted to the scheduler, you can check the status of your jobs using 'squeue'. Once the jobs have finished, your analysis is complete.

Everything should be in the output "data/processed/ folder, so "cd" into each directory and see if all the expected results are there.

You can also examine the logs by navigating to the "hpc-runner" folder,
```
cd hpc-runner/*/my-qc/logs
```
In this folder, you will find 1 folder per rule, with individual logs that have the ".md" suffix. Each one of these contains detailed information about the analysis job for each rule and for each sample. So if something went wrong, this is where you should start looking!
The other benefit to this detailed level of logging is reproducible research, because BioSAILs logs absolutely everything including compute nodes, computational requirements, parameters, runtimes, exitcodes, inputs/outputs etc.

Let's spend a few minutes familiarizing ourselves with all of the above before moving to the next excercise.
