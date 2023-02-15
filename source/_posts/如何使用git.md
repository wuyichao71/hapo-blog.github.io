---
title: 如何使用git
date: 2023-02-15 15:26:02
tags:
    - git
catagories:
    - 计算机
---
git是一个版本控制软件，以前使用的方法过程粗浅，因此我希望记录下git的一些命令，这些命令主要是从jyy的[ics的PA](https://nju-projectn.github.io/ics-pa-gitbook/ics2022/git.html)上抄来的。  

安装好git后我们需要先进行一些配置工作。在终端中输入一下命令:
```bash
git config --global user.name "hapo"
git config --global user.email "hapo@mail.com"
git config --global core.editor vim
git config --global color.ui true
```
这些配置会储存在家目录下的`.gitconfig`中，经过了配置之后，我们就可以开始使用git了。

## 本地管理  
### 初始化  
我们可以用`git clone`拉取远程的框架代码，或者在本地进行初始化新的项目:
```bash
git init
```

### 查看存档信息
使用
```bash
git log
```
查看目前为止所有的存档。
使用
```bash
git status
```
可以得知，与当前存档相比，哪些文件发生变化。 
<!--more--> 

### 存档
当我们代码写到一定程度的时候，就应该进行"存档"。  
首先我们需要使用`git status`查看是否有新的文件或者已经修改的文件未被跟踪，若有，则使用`git add`将文件加入跟踪列表，例如
```bash
git add file.c
```
会将`file.c`加入跟踪列表中，如果需要一次添加所有未被跟踪的文件，可以使用
```bash
git add -A
```
这个我爱用，但是这是不对的，因为可能会跟踪一些不必要的文件，例如编译产生的 .o 文件, 和最后产生的可执行文件。事实上，我们只需要跟踪代码源文件即可。为了让`git`在添加跟踪文件之前作筛选，我们可以编辑`.gitignore`文件(你可以使用`ls -a`命令看到它，<font color="orange">但是我没看到</font>)，在里面给出了需要被`git`忽略的文件和文件类型。  
把新文件加入跟踪列表后，使用`git status`再次确认。确认无误后就可以存档了，使用
```bash
git commit
```
提交工程当前的状态。执行这条命令后，将会弹出文本编辑器，我们需要在第一行中添加本次存档的注释，例如"fix bug for xxx"。我们应该尽可能添加详细的注释，将来我们需要根据这些注释来区别不同的存档。编写好注释之后，保存并退出文本编辑器，存档成功。我们可以使用`git log`查看存档记录，你应该能看到刚才编辑的注释。<font color='orange'>但是现在我偏爱`git commit -m "COMMIT"`，虽然我知道上面说的是对的。</font>  

### 读档




