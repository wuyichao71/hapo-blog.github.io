---
title: 如何使用git
date: 2023-02-15 15:26:02
tags:
    - git
catagories:
    - 计算机
---
git是一个版本控制软件，以前使用的方法过程粗浅，因此我希望记录下git的一些命令，这些命令主要是从jyy的ics的PA上抄来的。  

安装好git后我们需要先进行一些配置工作。在终端中输入一下命令:
```bash
git config --global user.name "hapo"
git config --global user.email "hapo@mail.com"
git config --global core.editor vim
git config --global color.ui true
```
这些配置会储存在家目录下的`.gitconfig`中，经过了配置之后，我们就可以开始使用git了
