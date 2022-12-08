---
title: 使用Modeller补缺失残基
date: 2022-12-08 03:19:31
categories:
    - 分子模拟
tags:
    - pdb预处理
---
写这个是因为每次用的时候发现都忘记了，甚至不知道网页在哪，每次都要在一堆链接中反复点击，宛如蒙特卡洛模拟，有时候甚至还找不到想要的网站。因此我现在就把[使用modeller补全缺失残基](https://salilab.org/modeller/wiki/Missing_residues)的网页放在这里。  

我们以[4GNX](https://files.rcsb.org/download/4GNX.pdb)为例进行补全。首先我们需要得到pdb中的序列信息，modeller可以使用如下的方式得到序列信息。但是modeller只会得到存在的残基的序列。对于中间缺失的残基，虽然pdb文件的`REMARK 465`中记录了缺失的残基序号和类型，pdb文件中的`SEQRES`也记录了生物分子的序列信息，但是modeller并不会帮你从pdb中提取出来在序列上补充上，因为modeller认为这部分信息是不可靠的。并且有的经过处理的pdb甚至会丢失这些信息。  

我们可以使用一下代码提取pdb文件中的序列信息：
```python
from modeller import *
# Get the sequence of the 1qg8 PDB file, and write to an alignment file
code = '4gnx'

e = Environ()
m = Model(e, file=code)
aln = Alignment(e)
aln.append_model(m, align_codes=code)
aln.write(file=code+'.seq')
```
