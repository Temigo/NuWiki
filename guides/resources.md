# Overview
Computing resources are made available at SLAC Data Facility (SDF). The [official documentation of SDF](https://ondemand-dev.slac.stanford.edu/public/doc/#/getting-started) is available though it is also under development still as of September 2020. This page provides a complementary (and a bit of duplicate) documentation to the official SDF one with an aim to guide neutrino users. In particular this covers 4 categories...

* **Gateway**: how can I access computing servers for doing my work?
* **Storage**: where is my space? where to keep my data files?
* **Softwares**: how can I set up a software stuck? how can I add more softwares?
* **Computing**: how to submit batch (=unattended, automated) job to run your computing script?

## Gateway / Access
You can access to SDF either from a web-browser or a terminal. These methods are complementary to each other in strengthe, and you are encouraged to try both methods at least once.

### Interactive access via `ssh` (a terminal-based method). 
  * From your laptop/desktop terminal, you can access SDF via [secure shell (SSH)](https://en.wikipedia.org/wiki/Secure_Shell). 
```
ssh $USER@sdf-login.slac.stanford.edu
```

**If you read to the end of this guide**, you will also learn how to access computing resources within a native web-browser on your laptop or desktop :) Keep reading!


### Data transfer to SDF from outside
Data transfer to SDF should be done using an appropriate data transfer nodes. 
  - **Please do not transfer data via `scp` through `sdf-login.slac.stanford.edu`**.
  - If you want to use `scp`, use `sdf-dtn.slac.stanford.edu` (dtn stands for data transfer node).
  - You can also use `globus` to transfer data files (that would correctly use the right nodes underneath).

## Storage space 
On each server machine, you typically have 3 types of space to work.
* `$HOME` ... 25GB space by default.
    * Network mounted = accessible from different machines.

* `/sdf/group/neutrino` ... global (network mounted) group space.
  * Network mounted = accessible from different machines
  * No purging policy. Keep your usage < 1TB (system does not enforce this limit) 
	* This is `$GROUP` in [official documentation](https://ondemand-dev.slac.stanford.edu/public/doc/#/getting-started?id=disk).
  * Create "your group space" and feel free to store files underneath.
  ```
  mkdir /sdf/group/neutrino/$USER
  ``` 

* `/scratch` ... global (network mounted) scratch space.
  * Network mounted = accessible from different machines
  * Purged approximately once a month. 
	* This is `$SCRATCH` in [official documentation](https://ondemand-dev.slac.stanford.edu/public/doc/#/getting-started?id=disk).
  * Create "your global scratch space" and feel free to store files underneath.
  ```
  mkdir /scratch/$USER
  ``` 
  
* `/lscratch/$USER` ... local RAID0 SSD space for fast read/write (no networking involved) **only on a batch worker node**.
  * Local disk = not accessible from different machines. **`sdf-login` nodes are not batch worker node.**
  * Files written in this space will be purged after you log out (i.e. per job).
  * This is `$LSCRATCH` in [official documentation](https://ondemand-dev.slac.stanford.edu/public/doc/#/getting-started?id=disk).
  


## Software stacks

* **Singularity**
  * At SLAC we recommend the use of a [Singularity container](https://sylabs.io/singularity/) to carry around and setup a software environment. Note if you are using `Open On-Demand`, your session lives inside a singularity container and you should not be following the instructions below. If you logged in using `ssh` or launching `slurm` job, this instruction is relevant.
  * Shared `Singularity` container images can be found at `/sdf/group/neutrino/images`.
  * If you aren't sure about `Singularity` or which container to use, you can just try:
  ```
  $> singularity exec --nv --bind /gpfs,/sdf,/scratch /sdf/group/neutrino/images/latest.sif bash
  ```
  * See the official documentation or [here](https://github.com/DeepLearnPhysics/playground-singularity/wiki) for getting started with `Singularity`.

* **CernVM File System (CVMFS)**
  * [CVMFS](https://cernvm.cern.ch/portal/filesystem) is also available at SLAC. Anyone who uses it should help documenting here.

## Computing
Any computing, no matter how light it is, should be done using a dedicated computing server.
NOPE! `sdf-login.slac.stanford.edu` is not a computing server :) so it's good not to work on that node.
This section briefly explains how to use [slurm](https://slurm.schedmd.com/documentation.html) with which you can access computing servers.

There are 3 ways to use slurm:
1. Interactive access through a terminal
2. Batch job submission
3. Interactive access through a web-browser (Jupyter)

The following guide demonstrates how you can play in all three ways.

### Computing resources
Before submitting a computing job, it is important to understand accessible resources at SLAC.
There are different sets of resources and they are called **partitions**.
Access a particular set of resources is controlled by specifying a unique partition name.
For neutrino group members, there are three relevant partitions:

* **"shared"**
  * a partition that includes ALL computing resources at SLAC and provides opportunistic (as opposed to "dedicated") access. Being opportunistic means that your job can run if there is unused computing resource that matches with the resource requested by your job. Since it is not a dedicated access, if someone else submits a dedicated job (i.e. non-shared partition) and if the `slurm` decides to run this new job on the server your job is running, your job may be killed in the middle.
* **"neutrino"**
  * a partition that provides dedicated access to specialized hardware that is designed for machine learning work and owned by the Neutrino group. 
* **"ml"**
  * a partition that provides dedicated access to specialized hardware that is designed for machine learning work and owned by the Machine Learning Initiatives.
  
Here is a summary of the total dedicated resources for the two partitions.  

| Partition   | RAM total  | CPU total  | GPU - A100 | GPU - V100 | GPU - P100 | GPU - 2080Ti | GPU - 1080Ti |
| :---        |   :----:   |   :----:   |   :----:   |   :----:   |   :----:   |    :----:    |    :----:    |
| neutrino    | 4864GB | 664 | 16 | 10 | 0 | 10 | 5 |
| ml          | 9280GB | 1424| 28 | 0  | 0 | 110 | 0 |

Getting a computing account (with a unix group `nu`) does not automatically give an access to `neutrino` and `ml` partitions. If you need access to those, please [contact Kazu](mailto:kterao@slac.stanford.edu).

### How much CPU, GPU, and RAM should I use?
* If you are using GPUs...
  * most likely you only need 1 GPU. Below is a recommended configuration (you may alter):
  * for A100, request 16 physical CPU cores (= 32 hyperthreads) + 96 GB RAM memory
  * for V100, request 4 physical CPU cores (= 8 hyperthreads) + 40 GB RAM memory
  * for 2080Ti, request 2 physical CPU cores (= 4 hyperthreads) + 16 GB RAM memory
  * for 1080Ti, request 2 physical CPU cores (= 4 hyperthreads) + 20 GB RAM memory
* If you are not using GPUs (CPU-only jobs)...
  * Request N hyperthreads where N is the number of parallel computation performed by your software (via multithreading or multiprocessing). If you are not sure, most likely N=1 for neutrino software workflow. The recommended RAM memory if 4GB per CPU, but this is not a hard limit. 
  
If you have a concern and want to a consultation on submitting a large number of jobs, please [contact Kazu](mailto:kterao@slac.stanford.edu).


### Interactive access through a terminal

One basic option is to access a computing resource interactively on a terminal. 
This would look literally like you are _logging (or ssh-ing) into a server dedicated for computing_.
For this, we use a `slurm` command `srun`. 

Take a look at the demo below.
BTW this movie is a magic: you can use your mouse to copy and paste typed commands from the movie for yourself to try at SDF :)

[![asciicast](https://asciinema.org/a/xjm8siU4p0hPLH005ktDoMhI9.svg)](https://asciinema.org/a/xjm8siU4p0hPLH005ktDoMhI9)

Let's decode this command:
```
srun --partition=neutrino --mem-per-cpu=20G --pty /bin/bash
```
* `--partition` specifies a group account through which you use computing tokens to request resource
* `--mem-per-cpu` specifies the amount of RAM memory per CPU you request (20G means 20 giga-byte) 
* `--pty` specifies what command to execute: here we execute `bash` interactive session
The number of CPU allocated is, by-default, 1.

Then you also saw another command to request a job with a GPU:
```
srun --partition=neutrino  --gpus geforce_rtx_2080_ti:1 --mem-per-cpu=20G --pty /bin/bash
```
* `--gpus geforce_rtx_2080_ti:1` requests `1` of NVIDIA RTX 2080 Ti GPUs for your job.

You mimght wonder:
* How much memory should I rquest?
* Does my job run indefinitely? What happens if I `exit`?
* etc. etc. ...
Look at the FAQ + [message kazu](mailto:kterao@slac.stanford.edu) or slack him if you have a question.

There are more option flags to better control the use of computing resource.
You learn more below where we try a batch job submission.

### Batch job submission

Once you finish developing your code, you may have a script that runs data processing by a single line of a command. 
You may not want to wait for this process to finish running in an interactive session.
This is when you want to use a __batch job submission__.

Here, let's practice an extremely simple batch job submission.
We will submit a batch job to execute `example.sh` script which contents is shown below.

```
#!/bin/bash
#SBATCH --partition=neutrino
#
#SBATCH --job-name=test
#SBATCH --output=job-%j.out
#SBATCH --error=job-%j.err
#
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=5g
#
#SBATCH --time=10:00
#                                                                                                           
#SBATCH --gpus=geforce_rtx_2080_ti:1                             

uname -a
nvidia-smi
```

The script executes two commands: `uname -a` then `nvidia-smi`.
Respectively, these commands print out the name of a server where the process is running and the status of GPUs.

I submit this script by a simple command:
```
sbatch example.sh
```

[![asciicast](https://asciinema.org/a/532704.svg)](https://asciinema.org/a/532704)

Yep, it's that simple! 

Let's walk through the contents of `example.sh`.
* `#!/bin/bash` tells the terminal excution to use `/bin/bash` (i.e. "bash") interpreter to execute the script.
* `--job-name` specifies the name of this job (i.e. handy if you want to find your job in `squeue`)
* `--output` specifies a text file to contain the standard output from the script
* `--error` specifies a text file to contain the standard error from the script
* `--ntasks` specifies the number of task to run. You can set this to the number main process to run.
* `--cpus-per-task` specifies the number of hyperthreads (i.e. "virtual cores") to be allocated per task. Note our machines have 2 hyperthreads per physical CPU core. This should match with the number of parallel threads/processes run by your main script.
* `--mem-per-cpu` specifies the amount of RAM memory per hyperthread. `5g` means 5 GByte. This job will have 20 GByte memory total with 4 hyperthreads.
* `--time` specifies the maximum amount of time allocated for this job. When this time is reached, if the job is still running, the job will be terminated.
* `--gpus` specify the type and count of GPUs to be allocated.

Some of these flags already showed up in the example of an interactive session with `srun`. 
I hope this explains the gists of how to submit a slurm batch job.

### Interactive access through a web-browser (Jupyter)

This is probably the most popular way of accessing the computing resource (hence it comes at the end :D).

Click the thumbnail below and watch a [YouTube tutorial movie](https://youtu.be/NhigtAK2BGM)!

[![Youtube Movie](https://img.youtube.com/vi/NhigtAK2BGM/0.jpg)](https://www.youtube.com/watch?v=NhigtAK2BGM)



