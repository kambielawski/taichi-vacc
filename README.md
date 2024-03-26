# taichi-vacc
Basic template and instructions for running Taichi programs on the VACC 

# Links
- [Docker Hub](https://hub.docker.com/)
- [Taichi](https://docs.taichi-lang.org/)
- [VACC knowledge base](https://www.uvm.edu/vacc/kb/knowledge-base/)
- [NVIDIA Grid Cloud Containers](https://catalog.ngc.nvidia.com/containers)


# Running Taichi programs on the VACC

## 0. Why? The error
Why can't we just run a Taichi program on the VACC like any other Python program?

The VACC runs RedHat Enterprise Linux 7.9 which contains a version of a C library, glibc, which is incompatible with the latest several versions of Taichi.

This is the error you will get if you try to run Taichi programs directly on a VACC node: 
```
[Taichi] version 1.7.0, llvm 16.0.0git, commit 2fd24490, linux, python 3.11.5
You have installed a restricted version of taichi, certain features (e.g. Vulkan & GGUI) will not work.
!! Taichi requires glibc >= 2.27 to run, please try upgrading your OS to a recent one (e.g. Ubuntu 18.04 or later) if possible.
```


## 1. Using a .sif file on the VACC
A `.sif` file is a Singularity Image File, which essentially specifies a Docker image, but it can run on the VACC (it's generated from a Docker image). 

Here, you can just copy the .sif file from my (ktbielaw) user directory into your user directory: 
```
cp /users/k/t/ktbielaw/taichi-vacc-x11_latest.sif ~
```

There is another Markdown file in this directory if you want to know the details of building a custom Singularity container (as of 3/26/2024 that guide is still under construction)

(I asked the VACC people if this pre-made Taichi container can be stored in a common place that is not my user directory, waiting to hear back)

## 2. Running a Taichi program
From the VACC Knowledgebase: "Docker is not allowed on the VACC due to the security risks of running it in a multitenant environment. However, Singularity can interface with Docker repositories and run containers in most cases."

Therefore we use Singularity. 

To execute the singularity container and run whatever taichi program: 

```
module load singularity

singularity exec --nv ~/taichi-vacc-x11_latest.sif python3 ~/projects/difftaichi/run_taichi.py
```

The `--nv` specifies that the container is running on a machine with NVIDIA hardware and allows the container to interface with it. It's very important.

## sbatch example

```
#!/bin/bash

#SBATCH --partition=dggpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --gpus=1
#SBATCH --mem=8G
#SBATCH --job-name=ciliadiff_exp_%j
#SBATCH --output=./vacc_out_files/%x_%j.out
#SBATCH --time=03:00:00

set -x

module load singularity

singularity exec --nv ~/taichi-vacc-x11_latest.sif python3 ~/projects/cilia-diff/run_trial.py --file $1
```

This executes my `run_trial.py` file with a command line argument. I submit this to the VACC with:

```
sbatch run_trial_bind.sh experiment_file.exp
```

## Tip: SSHing into the container for debugging

If you want to debug your code inside the container (in the execution environment) before actually submitting jobs, you can do that. 

First you make a sandbox from a singularity container (this basically creates a directory on the host machine which has all of the files from the container)
```
singularity build --sandbox taichi-vacc-x11_latest ~/taichi-vacc-x11_latest.sif
```

Then you can get a shell inside of a Singularity container 
```
singularity shell --bind /gpfs1/home:/users --writable ./taichi-vacc-x11_latest
```

Now you are SSHed into the container. (Only do this if you're already SSHed onto a DG node, I don't think this works on a user node. 3 SSH shells deep.)


