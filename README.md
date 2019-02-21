# NYU HPC

A quick reference to access NYU's High Performance Computing Prince Cluster.

The official wiki is [here](https://wikis.nyu.edu/display/NYUHPC/Clusters+-+Prince), this is an unofficial document created as a quick-start guide for first-time users with a focus in Python and PyTorch.

## Get an account

You need to be affiliated to NYU and have a sponsor. 

To get an account approved, follow [this steps.](https://wikis.nyu.edu/display/NYUHPC/Requesting+an+HPC+account+with+IIQ)

## Log in

Once you have been approved, you can access HPC from:

 1. Within the NYU network (in campus):

```bash
ssh NYUNetID@prince.hpc.nyu.edu
```

Remember to replace NYUNetID for your own NetID.  

Once logged in, the root should be:
`/home/NYUNetID`, so running `pwd` should print:

```bash
[NYUNetID@log-0 ~]$ pwd
/home/NYUNetID
```

2. From an off-campus location:

First, Login to your VPN and then login to the bastion host, :

```bash
ssh NYUNetID@gw.hpc.nyu.edu
```

Then login to the cluster:

```bash
ssh prince.hpc.nyu.edu
```
## Using Windows.  

I use the [MobaXterm](https://mobaxterm.mobatek.net/) ssh client with the following settings for the Prince Cluster:

```
Remote host: prince.hpc.nyu.edu
Username: NYUNetID
Port: 22
```
This makes it one click to open a terminal to Prince. 

## File Systems

You can get acces to three filesystems: `/home`, `/scratch`, and `/archive`.

Scratch is a file system mounted on Prince that is connected to the compute nodes where we can upload files faster. Notice that the content gets flushed every 60 days with no backup!

```bash
[NYUNetID@log-0 ~]$ cd /scratch/NYUNetID
[NYUNetID@log-0 ~]$ pwd
/scratch/NYUNetID
```

`/home` and `/scratch` are separate filesystems in separate places. 
Depending on how often you use your files you might want to choose the appropiate file system. I use /home for the files I won't touch often. 

## Loading Modules

Slurm allows you to load and manage multiple versions and configurations of software packages.

To see available package environments:
```bash
module avail
```

To load a model:
```bash
module load [package name]
```

For example if you want to use Tensorflow-gpu:
```bash
module load cudnn/8.0v6.0
module load cuda/8.0.44
module load tensorflow/python3.6/1.3.0
```

To check what is currently loaded:
```bash
module list
```

To remove all packages:
```bash
module purge
```

To get helpful information about the package:
```bash
module show torch/gnu/20170504
```
Will print something like 
```bash
--------------------------------------------------------------------------------------------------------------------------------------------------
   /share/apps/modulefiles/torch/gnu/20170504.lua:
--------------------------------------------------------------------------------------------------------------------------------------------------
whatis("Torch: a scientific computing framework with wide support for machine learning algorithms that puts GPUs first")
whatis("Name: torch version: 20170504 compilers: gnu")
load("cmake/intel/3.7.1")
load("cuda/8.0.44")
load("cudnn/8.0v5.1")
load("magma/intel/2.2.0")
...
```
`load(...)` are the dependencies that are also loaded when you load a package.


## Interactive Mode: Request CPU

You can submit batch jobs in prince to schedule jobs. This requires to write custom bash scripts. Batch jobs are great for longer jobs, and you can also run in interactive mode, which is great for short jobs and troubleshooting.

To run in interactive mode:

```bash 
[NYUNetID@log-0 ~]$ srun --pty /bin/bash
```

This will run the default mode: a single CPU core and 2GB memory for 1 hour.

To request more CPU's:

```bash
[NYUNetID@log-0 ~]$ srun -n4 -t2:00:00 --mem=4000 --pty /bin/bash
[NYUNetID@c26-16 ~]$ 
```
That will request 4 compute nodes for 2 hours with 4 Gb of memory.


To exit a request:
```
[NYUNetID@c26-16 ~]$ exit
[NYUNetID@log-0 ~]$
```

## Interactive Mode: Request GPU

```bash
[NYUNetID@log-0 ~]$ srun --gres=gpu:1 --pty /bin/bash
[NYUNetID@gpu-25 ~]$ nvidia-smi
Mon Oct 23 17:49:19 2017
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 367.48                 Driver Version: 367.48                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           On   | 0000:12:00.0     Off |                    0 |
| N/A   37C    P8    29W / 149W |      0MiB / 11439MiB |      0%   E. Process |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```


## Submit a job

You can write a script that will be executed when the resources you requested became available.

A simple CPU demo:

```bash
## 1) Job settings

#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --time=5:00:00
#SBATCH --mem=2GB
#SBATCH --job-name=CPUDemo
#SBATCH --mail-type=END
#SBATCH --mail-user=itp@nyu.edu
#SBATCH --output=slurm_%j.out
  
## 2) Everything from here on is going to run:

cd /scratch/NYUNetID/demos
python demo.py
```

Request GPU:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:4
#SBATCH --time=10:00:00
#SBATCH --mem=3GB
#SBATCH --job-name=GPUDemo
#SBATCH --mail-type=END
#SBATCH --mail-user=itp@nyu.edu
#SBATCH --output=slurm_%j.out

cd /scratch/NYUNetID/trainSomething
source activate ML
python train.py
```

Submit your job with:

```bash
sbatch myscript.s
```

Monitor the job:

```bash
squeue -u $USER
```

More info [here](https://wikis.nyu.edu/display/NYUHPC/Submitting+jobs+with+sbatch)


## Transfer Files

I transfer files using MobaXTerm. If you need to setup a tunnel look [here](cvalenzuela/hpc) 
