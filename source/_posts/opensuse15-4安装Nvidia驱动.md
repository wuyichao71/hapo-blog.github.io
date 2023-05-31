---
title: opensuse15.4安装Nvidia驱动
date: 2023-02-07 13:37:36
tags:
    - linux
categories:
    - 计算机
---
更新为opensuse15.4后显卡驱动被卸载了，每次安装显卡驱动都异常复杂，因此记录下来以便以后查阅。  

# 添加Nvidia软件源
首先需要添加Nvidia的软件源:
```bash
sudo zypper addrepo --refresh 'https://download.nvidia.com/opensuse/leap/$releasever' NVIDIA
```

# 获得硬件信息
用以下命令可以获得硬件信息:
```bash
sudo lspci |grep VGA
sudo lscpu |grep Arch # 中文要改成"架构"
```
或者使用以下命令查看:
```bash
sudo hwinfo --gfxcard | grep Model
sudo hwinfo --arch
```
又或者使用`inxi`命令:
```bash
inxi -G
inxi -Ga
```

# 安装
现在查看下所需要的显卡驱动，显卡驱动的名字有如下含义:  
1. G03 = driver v340 = legacy driver for GT8xxx/9xxx devices  
2. G04 = driver v390 = legacy driver for GTX4xx/5xx Fermi devices  
3. G05 = current driver for current devices  
4. G06 = covers all cards GT700 and up  

可以用一下命令查看显卡驱动信息:
```bash
sudo zypper se x11-video-nvidiaG0*
```
或者:
```bash
sudo zypper se -s x11-video-nvidiaG0*
```
如果要或者OpenGL加速效果，可以用一下命令查看额外的包的信息:
```bash
zypper se nvidia-glG0*
```











