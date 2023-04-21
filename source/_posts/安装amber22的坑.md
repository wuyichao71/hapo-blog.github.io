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

有的python库可能已经自带. 经过以上的配置, amber22就能使用anaconda下现成的python环境, 不需要额外安装anaconda.

## 其余的坑

### Boost

amber需要使用zlib和bzip2编译过的boost库. 集群上一般没有安装, 或者安装后也boost并没有使用zlib和bzip2编译, 因此amber22一般会自己编译. 如果你确定系统中的版本可用, 那么可以设置`-DFORCE_EXTERNAL_LIBS='boost'`. amber22要编译自己的boost库需要zlib和bzip2, 其中zlib的缺失会在`run_make`的过程中检查到, 而bzip2的缺失则会在编译的过程中才会报错. 如果这两个库在集群上缺失, 那么可以使用anaconda安装库.  

```bash
conda install zlib #zlib 在前面的配置环境的时候已经安装了.
conda install bzip2
```

### libSM

在集群上, libSM库存在问题, 这会导致xaLeap编译出问题, 这时候我们可以使用anaconda安装该库文件.

```bash
conda install -c conda-forge xorg-libsm
```

并且设置如下两个变量： `-DX11_SM_INCLUDE_PATH=/path/to/anaconda/env/amber/include`和`-DX11_SM_LIB=/path/to/anaconda/env/amber/lib/libSM.so`

并且设置`LD_LIBRARY_PATH`用于链接`libuuid.so`

```bash
export LD_LIBRARY_PATH=/path/to/anaconda3/env/amber/lib:$LD_LIBRARY_PATH
```

> 这里实际上是系统的libsm库和uuid库的匹配有问题, 使用conda安装了libsm库后会下载uuid库, 而设置了`-DX11_SM_INCLUDE`、`-DX11_SM_LIB`和`LD_LIBRARY_PATH`后会使用anaconda下的libsm库并且会优先查找`/path/toanaconda3/env/amber/lib`下的uuid库. 通过使用`objdump -d /usr/lib64/libuuid.so.1`发现其中的函数名为`uuid_generate@@UUID_1.0`而不是`uuid_generate@UUID_1.0`.
>
> <font color='salmon'>更简单的方法是设置`-DCMAKE_PREFIX_PATH=/path/to/anaconda/env/amber/`来让cmake自动查找libsm库和uuid库, 这样就不用设置`-DX11_SM_LIB`和`-DX11_SM_INCLUDE_PATH`了, 也不需要设置系统的`LD_LIBRARY_PATH`.</font>
>
>> 实际上经过测试, 只设置了`-DX11_SM_INCLUDE`和`-DX11_SM_LIB`依旧会使用`/usr/lib64/libSM.so.1`.
>>
>> 只设置`LD_LIBRARY_PATH`也无法通过编译

### 一些额外的选项

1. `-DTRUST_SYSTEM_LIBS`: 相信系统的库文件, 开启后会将某些库会使用系统中自带的(例如boost), 开启命令`-DTRUST_SYSTEM_LIBS=TRUE`
2. `-DDISABLE_TOOLS`: 关闭一些工具的编译, 例如`-DDISABLE_TOOLS=cpptraj`
3. `-DFORCE_DISABLE_LIBS`: 关闭某些库文件, 使用`;`分隔开(注意用引号`'`引起来, 以防和bash冲突), 例如`-DFORCE_DISABLE_LIBS=boost`
4. `-DFORCE_INTERNAL_LIBS`: 强制某些库文件使用内部编译, 例如`-DFORCE_INTERNAL_LIBS=zlib`
5. `-DFORCE_EXTERNAL_LIBS`: 强制某些库文件使用外部编译, 例如`-DFORCE_INTERNAL_LIBS=zlib`

### CUDA与INTEL编译器版本问题

`run_cmake`会检查编译器的版本和CUDA版本, 如果版本不适配则会配置不通过. 但是当你使用intel编译器时, 它依旧是按照gnu的编译器版本在比较, 因此intel几乎无法编译cuda版本. 为了解决这个问题我们可以修改`/path/to/amber22_src/cmake/CudaConfig.cmake`文件的112行

```cmake
111             CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 12
112             AND CUDA_VERSION VERSION_GREATER 11.6
```

将cuda的上限版本(11.6)调低即可, 我一般调至11.0.
