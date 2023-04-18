---
title: 安装amber22的坑
date: 2023-04-18 20:49:10
categories:
    - 分子模拟
tags:
    - 软件安装
---
amber22已经把Amber部分开源了, 再加上最近需要使用amber, 所以将amber的安装研究了下, 经过两天的debug, 终于可以在集群上安装起来.

## 最基本的安装过程

amber22使用cmake进行构建程序, 这样的好处是不会破坏源程序文件夹, 并且amber22已经不需要再分开编译serial, mpi, cuda版本, 只要开启了相应的选项就能都编译出来. 在`/path/to/amber22_src/build`中有`run_make`和`configure_make.py`两个文件. 其中, `run_make`写了基本的编译命令, 你需要对自己需要的选项进行修改从编译需要的版本, 而`configure_make.py`则是一个`python`脚本, 可以通过命令行设置对应的选项. 两个文件可以任意选一个进行configure. 当编译出现问题时, 可以用`clean_build`清理掉生成的文件.

最基本的安装命令是

```bash
./run_make
make install
source /path/to/amber/amber.sh
```

如果amber源文件夹为`/path/to/amber22_src`, 则默认安装好的文件夹在`/path/to/amber`.
<!--more-->

## 配置python

amber22的一些程序需要python环境, 有很多方法可以python环境.

### 1

amber22默认会下载miniconda, 这样可以搭建出amber22适配的环境. 安装完成后会生成一个`amber.python`软连接, 这样可以和系统默认的python区分开. 但是在hpcc集群上没有办法连接外网, 此时可以如下操作: (1) 在`amber22_src/build`下新建文件夹`CMakeFiles/miniconda/download/`, 下载好`Miniconda3-latest-Linux-x86_64.sh`放入该文件夹中, 之后cmake会认为该文件已下载好, 从而跳过下载过程. (2) 修改`amber22_src/cmake`文件夹下的`UseMiniconda.cmake`文件, 修改91行

```cmake
91  set(INSTALLER_URL "http://repo.continuum.io/miniconda/${MINICONDA_INSTALLER_FILENAME}")
```

例如修改成如下

```cmake
91  set(INSTALLER_URL "http://mirrors.nju.edu.cn/anaconda/miniconda/${MINICONDA_INSTALLER_FILENAME}")
```

之后会下载一些miniconda需要的python库. 因为hpcc集群上无法连接外网, 因此我们可以配置成内网的镜像. 可以修改家目录下的`.condarc`文件

```json
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirror.nju.edu.cn/anaconda/pkgs/main
  - https://mirror.nju.edu.cn/anaconda/pkgs/r
  - https://mirror.nju.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirror.nju.edu.cn/anaconda/cloud
  msys2: https://mirror.nju.edu.cn/anaconda/cloud
  bioconda: https://mirror.nju.edu.cn/anaconda/cloud
  menpo: https://mirror.nju.edu.cn/anaconda/cloud
  pytorch: https://mirror.nju.edu.cn/anaconda/cloud
  simpleitk: https://mirror.nju.edu.cn/anaconda/cloud
```

以及配置pip

```bash
pip config set global.index-url https://mirror.nju.edu.cn/pypi/web/simple/
```

此时会在家目录下生成如下配置文件`～/.config/pip/pip.conf`, 其中内容为

```json
[global]
index-url = https://mirror.nju.edu.cn/pypi/web/simple/

```

这样后使用miniconda的python环境就可以顺利安装了.

### 2

如果已经在本地装过anaconda, 那么我们就可以用anaconda生成一个安装amber22的本地环境, 这样可以和已经安装过的python库同时在一个环境下使用. 使用本地python环境需要设置如下的选项`-DDOWNLOAD_MINICONDA=FALSE`(不下载miniconda)和`-DUSE_CONDA_LIBS=TRUE`(使用conda的python库). 可以用以下命令生成一个新的环境.

```bash
conda create -n amber python=3.10
conda activate amber
```

amber22的安装需要一些python库

```conda
conda install numpy
conda install scipy
conda install matplotlib
conda install setuptools
conda install thinker
```
