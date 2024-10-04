---
title: how to install gpu version of genesis on group server
date: 2024-10-03 17:10:55
categories:
    - MD
tags:
    - MD
---

## Install cuda 11 environment with miniconda

Because the server only have `cuda-12` and `cuda-12` will make the genesis get wrong results. Therefore, we need use miniconda to install `cuda-11` enviroment. We use below command to make a environment include `cuda-11.8`.

<!--more-->

```bash
conda create -n cuda_11 -c nvidia cudatoolkit=11 cuda-nvcc=11 cuda-cudart-dev=11 cuda-nvtx=11
```

You can activate the environment by `conda activate cuda_11`.

Then, we source the intel compiler and add the cuda library to the `LD_LIBRARY_PATH`:

```bash
source /opt/intel/oneapi/setvars.sh
export LD_LIBRARY_PATH=/your/miniconda/envs/cuda_11/lib:$LD_LIBRARY_PATH
```

## Compile genesis

Then we can compile `genesis` use below command
```bash
./configure --enable-gpu --enable-mixed --program-suffix='-intel-mixed-cuda11-conda'
make -j 8
make install
```

After that, the genesis can preform correct calculation on cell.




