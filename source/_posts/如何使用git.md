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
我们可以使用`git log`来查看已有的存档，并决定需要回到哪个过去，每一份存档都有一个hash code，例如`8e4fac44d3b567591bc3768fc94d53575726b866`，我们需要通过hash code来告诉`git`我们希望读取哪一个档。使用一下命令进行独档:
```bash
git reset --hard 8e4f
```

其中`8e4f`是上文hash code的前缀，我们不需要输入整个hash code。这时候我们的代码已经回到过去了。  
但事实上，使用`git reset`的hard模式之前，我们需要再三确认选择的存档是不是我们的真正目标。如果我们读入了一个较早的存档，那么比这个存档新的所有记录都将被删除！这意味着你不能随便回到"将来"了。就和别的软件中的撤销操作一样。

### 第三视点
当然还是有办法来避免上文提到的副作用的，这就是`git`的分支功能，使用命令
```bash
git branch
```
查看所有分支。其中`master`是主分支，使用`git init`初始化之后会自动建立主分支。  
读档的时候使用以下命令
```bash
git checkout 8e4f
```
而不是`git reset`。这时你将处于一个虚拟的分支中，你可以  
1. 查看`8e4f`存档的内容
2. 使用以下命令切换到其它分支
```bash
git checkout 分支名
```
3. 对代码的内容进行修改，但你不能使用`git commit`进行存档，你需要使用
```bash
git check -B 分支名
```
把修改保存到一个新的分支中，如果分支已存在，其内容将会被覆盖。  
不同的分支之间不会相互干扰， 这也给项目的分布式开发带来了便利，有了分支功能，我们就可以像但视点那样在一个世界的不同时间(一个分支的多个存档)，或者是多个平行时间(多个分支)之间来回穿梭。



