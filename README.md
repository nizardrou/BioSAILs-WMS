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

