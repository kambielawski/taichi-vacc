# !! Under construction !!

## 1. Building a custom Docker container
First we need to build a custom Docker container that contains an OS version where Taichi will run. 

We start with an NVIDIA Grid Cloud (NGC) container for our base image ([python-cuda120](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-workbench/containers/python-cuda120)). This contains the necessary OS version and software (e.g. Python, CUDA) to run Taichi on DeepGreen. 

Then we install X11 (the UNIX GUI system) because Taichi needs this to run even though we almost always run things headlessly on the VACC. 

The full Dockerfile:

```
# Use NVIDIA CUDA 12.3.1 runtime image with Ubuntu 22.04 as the base
FROM nvcr.io/nvidia/cuda:12.3.1-runtime-ubuntu22.04

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Update apt repositories and install Python 3.10 and X11 libraries
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3.10 python3-pip python3.10-venv python3.10-dev \
    libx11-6 \
    && rm -rf /var/lib/apt/lists/*

# Update alternatives to use Python 3.10 as the default python3
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1 \
    && update-alternatives --install /usr/bin/python python /usr/bin/python3.10 1 \
    && update-alternatives --set python /usr/bin/python3.10 \
    && update-alternatives --set python3 /usr/bin/python3.10

# Upgrade pip to the latest version
RUN python3.10 -m pip install --upgrade pip

RUN pip install taichi
```

Yep, that's it. Just run `docker build -t <DockerHub username>/<name of container>`. For me, it's `kamtb28/taichi-vacc-x11`.

#### 1.1. Upload to Docker Hub 