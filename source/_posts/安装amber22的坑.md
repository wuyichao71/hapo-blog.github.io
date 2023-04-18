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

amber22使用cmake进行构建程序, 这样的好处是不会破坏源程序文件夹, 并且amber22已经不需要再分开编译serial, mpi, cuda版本, 只要开启了相应的选项就能都编译出来. 在`/path/to/amber22_src/build`中有`run_make`和`configure_make.py`两个文件. 其中, `run_make`写了基本的编译命令, 你需要对自己需要的选项进行修改从编译需要的版本, 而`configure_make.py`则是一个`python`脚本, 可以通过命令行设置对应的选项. 两个文件可以任意选一个进行configure.