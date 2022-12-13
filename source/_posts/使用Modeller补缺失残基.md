---
title: 使用Modeller补缺失残基
date: 2022-12-08 03:19:31
categories:
    - 分子模拟
tags:
    - pdb预处理
---
写这个是因为每次用的时候发现都忘记了，甚至不知道网页在哪，每次都要在一堆链接中反复点击，宛如蒙特卡洛模拟，有时候甚至还找不到想要的网站。因此我现在就把[使用modeller补全缺失残基](https://salilab.org/modeller/wiki/Missing_residues)的网页放在这里。  

我们以[4GNX](https://files.rcsb.org/download/4GNX.pdb)为例进行补全。首先4GNX在PDB中是个二聚体结构，我们只需要其中的一半的信息，所以我们删除了X，Y，Z和L链，改文件命名为`4gnx_half.pdb`。之后我们需要得到pdb中的序列信息。但是modeller只会得到存在的残基的序列。对于中间缺失的残基，虽然pdb文件的`REMARK 465`中记录了缺失的残基序号和类型，pdb文件中的`SEQRES`也记录了生物分子的序列信息，但是modeller并不会帮你从pdb中提取出来在序列上补充上，因为modeller认为这部分信息是不可靠的。并且有的经过处理的pdb甚至会丢失这些信息。  

我们可以使用一下代码提取pdb文件中的序列信息：
```python
from modeller import *
# Get the sequence of the 4gnx PDB file, and write to an alignment file
code = '4gnx'

e = Environ()
m = Model(e, file=code)
aln = Alignment(e)
aln.append_model(m, align_codes=code)
aln.write(file=code+'.seq')
```
<!--more-->
用以上代码生成的序列文件`4gnx_half.seq`如下：
```

>P1;4gnx_half
structureX:4gnx_half:   1 :A:+688 :K:MOL_ID  1; MOLECULE  PUTATIVE UNCHARACTERIZED PROTEIN; CHAIN  A, X; ENGINEERED  YES; MOL_ID  2; MOLECULE  PUTATIVE UNCHARACTERIZED PROTEIN; CHAIN  B, Y; ENGINEERED  YES; MOL_ID  3; MOLECULE  PUTATIVE UNCHARACTERIZED PROTEIN; CHAIN  C, Z; ENGINEERED  YES; MOL_ID  4; MOLECULE  DNA (25-MER); CHAIN  K, L; ENGINEERED  YES:MOL_ID  1; ORGANISM_SCIENTIFIC  USTILAGO MAYDIS; ORGANISM_COMMON  SMUT FUNGUS; ORGANISM_TAXID  237631; STRAIN  521 / FGSC 9021; GENE  UM04165.1; EXPRESSION_SYSTEM  SPODOPTERA FRUGIPERDA; EXPRESSION_SYSTEM_TAXID  7108; EXPRESSION_SYSTEM_CELL_LINE  HI5; MOL_ID  2; ORGANISM_SCIENTIFIC  USTILAGO MAYDIS; ORGANISM_COMMON  SMUT FUNGUS; ORGANISM_TAXID  237631; STRAIN  521 / FGSC 9021; GENE  UM02579.1; EXPRESSION_SYSTEM  SPODOPTERA FRUGIPERDA; EXPRESSION_SYSTEM_TAXID  7108; EXPRESSION_SYSTEM_CELL_LINE  HI5; MOL_ID  3; ORGANISM_SCIENTIFIC  USTILAGO MAYDIS; ORGANISM_COMMON  SMUT FUNGUS; ORGANISM_TAXID  237631; STRAIN  521 / FGSC 9021; GENE  UM05156.1; EXPRESSION_SYSTEM  SPODOPTERA FRUGIPERDA; EXPRESSION_SYSTEM_TAXID  7108; EXPRESSION_SYSTEM_CELL_LINE  HI5; MOL_ID  4; SYNTHETIC  YES; ORGANISM_SCIENTIFIC  SYNTHETIC; ORGANISM_TAXID  32630: 2.80: 0.22
MEKPTPLINSSMLGQYVGQTVRIVGKVHKVTGNTLLMQTSDLGNVEIAMTPDSDVSSSTFVEVTGKVSDAGSSFQ
ANQIREFTTVDVDLTLVENVVQISAAFPNLFSD/NTLRPVTIRQILNAEQPHPDAEFILDGAELGQLTFVAVVRN
ISRNATNVAYSVEDGTGQIEVRQWLDASEIRNNVYVRVLGTLKSFQNRRSISSGHMRPVIDYNEVMFHRLEAVHA
HLQVTR/IYPIEGLSPYQNRWTIKARVTSKSDIRHWSNQRGEGKLFSVNLLDDSGEIKATGFNDAVDRFYPLLQE
NHVYLISKARVNIAKKQFSNLQNEYEITFENSTEIEECTDATDVPEVKYEFVRINELESVEANQQCDVIGILDSY
GELSEIVSKASQRPVQKRELTLVDQGNRSVKLTLWGKTAETFPTNAGVDEKPVLAFKGVKVGDFGGRSLSMFSSS
TMLINPDITESHVLRGWYDNDGAHAQFQPYTNGGGAGANMAERRTIVQVKDENLGMSEKPDYFNVRATVVYIKQE
NLYYTACASEGCNKKVNLDHENNWRCEKCDRSYATPEYRYILSTNVADATGQMWLSGFNEDATQLIGMSAGELHK
LREESESEFSAALHRAANRMYMFNCRAKMDTFNDTARVRYTISRAAPVDFAKAGMELVDAIRAYM/ttttttttt
tttttttttttttttt*
```
从`REMARK 465`和`SEQRES`中我们可以知道缺失的残基是哪些，进而填入以上生成的序列中。这里我们要有一份用`-`填补缺失残基的模板序列和一份完整序列，这两个序列可以写在`alignment.aln`文件中：
```
>P1;4gnx_half
structureX:4gnx_half:   1 :A:+688 :K:MOL_ID  1; MOLECULE  PUTATIVE UNCHARACTERIZED PROTEIN; CHAIN  A, X; ENGINEERED  YES; MOL_ID  2; MOLECULE  PUTATIVE UNCHARACTERIZED PROTEIN; CHAIN  B, Y; ENGINEERED  YES; MOL_ID  3; MOLECULE  PUTATIVE UNCHARACTERIZED PROTEIN; CHAIN  C, Z; ENGINEERED  YES; MOL_ID  4; MOLECULE  DNA (25-MER); CHAIN  K, L; ENGINEERED  YES:MOL_ID  1; ORGANISM_SCIENTIFIC  USTILAGO MAYDIS; ORGANISM_COMMON  SMUT FUNGUS; ORGANISM_TAXID  237631; STRAIN  521 / FGSC 9021; GENE  UM04165.1; EXPRESSION_SYSTEM  SPODOPTERA FRUGIPERDA; EXPRESSION_SYSTEM_TAXID  7108; EXPRESSION_SYSTEM_CELL_LINE  HI5; MOL_ID  2; ORGANISM_SCIENTIFIC  USTILAGO MAYDIS; ORGANISM_COMMON  SMUT FUNGUS; ORGANISM_TAXID  237631; STRAIN  521 / FGSC 9021; GENE  UM02579.1; EXPRESSION_SYSTEM  SPODOPTERA FRUGIPERDA; EXPRESSION_SYSTEM_TAXID  7108; EXPRESSION_SYSTEM_CELL_LINE  HI5; MOL_ID  3; ORGANISM_SCIENTIFIC  USTILAGO MAYDIS; ORGANISM_COMMON  SMUT FUNGUS; ORGANISM_TAXID  237631; STRAIN  521 / FGSC 9021; GENE  UM05156.1; EXPRESSION_SYSTEM  SPODOPTERA FRUGIPERDA; EXPRESSION_SYSTEM_TAXID  7108; EXPRESSION_SYSTEM_CELL_LINE  HI5; MOL_ID  4; SYNTHETIC  YES; ORGANISM_SCIENTIFIC  SYNTHETIC; ORGANISM_TAXID  32630: 2.80: 0.22
MEKPTPLINSSMLGQYVGQTVRIVGKVHKVTGNTLLMQTSDLGNVEIAMTPDSDVSSSTFVEVTGKVSDAGSSFQ
ANQIREFTTV----DVDLTLVENVVQISAAFPNLFSD/NTLRPVTIRQILNAEQPHPDAEFILDGAELGQLTFVA
VVRNISRNATNVAYSVEDGTGQIEVRQWLD--------ASEIRNNVYVRVLGTLKSFQNRRSISSGHMRPVIDYN
EVMFHRLEAVHAHLQVTR/IYPIEGLSPYQNRWTIKARVTSKSDIRHWSNQRGEGKLFSVNLLDDSGEIKATGFN
DAVDRFYPLLQENHVYLISKARVNIAKKQFSNLQNEYEITFENSTEIEECTDATDVPEVKYEFVRINELESVEAN
QQCDVIGILDSYGELSEIVSKASQRPVQKRELTLVDQGNRSVKLTLWGKTAETFPTNAGVDEKPVLAFKGVKVGD
FGGRSLSMFSSSTMLINPDITESHVLRGWYDNDGAHAQFQPYTN---------GGGAGANMAERRTIVQVKDENL
GMSEKPDYFNVRATVVYIKQENLYYTACASEGCNKKVNLDHENNWRCEKCDRSYATPEYRYILSTNVADATGQMW
LSGFNEDATQLIGMSAGELHKLREESESEFSAALHRAANRMYMFNCRAKMDTFNDTARVRYTISRAAPVDFAKAG
MELVDAIRAYM/ttttttttttttttttttttttttt*
>P1;4gnx_half_fill
sequence:::::::::
MEKPTPLINSSMLGQYVGQTVRIVGKVHKVTGNTLLMQTSDLGNVEIAMTPDSDVSSSTFVEVTGKVSDAGSSFQ
ANQIREFTTVDCGHDVDLTLVENVVQISAAFPNLFSD/NTLRPVTIRQILNAEQPHPDAEFILDGAELGQLTFVA
VVRNISRNATNVAYSVEDGTGQIEVRQWLDSSSDDSSKASEIRNNVYVRVLGTLKSFQNRRSISSGHMRPVIDYN
EVMFHRLEAVHAHLQVTR/IYPIEGLSPYQNRWTIKARVTSKSDIRHWSNQRGEGKLFSVNLLDDSGEIKATGFN
DAVDRFYPLLQENHVYLISKARVNIAKKQFSNLQNEYEITFENSTEIEECTDATDVPEVKYEFVRINELESVEAN
QQCDVIGILDSYGELSEIVSKASQRPVQKRELTLVDQGNRSVKLTLWGKTAETFPTNAGVDEKPVLAFKGVKVGD
FGGRSLSMFSSSTMLINPDITESHVLRGWYDNDGAHAQFQPYTNGGVGGGAMGGGGAGANMAERRTIVQVKDENL
GMSEKPDYFNVRATVVYIKQENLYYTACASEGCNKKVNLDHENNWRCEKCDRSYATPEYRYILSTNVADATGQMW
LSGFNEDATQLIGMSAGELHKLREESESEFSAALHRAANRMYMFNCRAKMDTFNDTARVRYTISRAAPVDFAKAG
MELVDAIRAYM/ttttttttttttttttttttttttt*
```
现在我们可以用Modeller中的['LoopModel' class](https://salilab.org/modeller/10.0/manual/node33.html)生成所有的残基，并对loop区域进行优化，代码如下：
```python
from modeller import *
from modeller.automodel import *    # Load the AutoModel class

log.verbose()
env = Environ()

# directories for input atom files
env.io.atom_files_directory = ['.', '../atom_files']

a = LoopModel(env, alnfile = 'alignment.ali',
              knowns = '4gnx_half', sequence = '4gnx_half_fill')
a.starting_model= 1
a.ending_model  = 1

a.loop.starting_model = 1
a.loop.ending_model   = 2
a.loop.md_level       = refine.fast

a.make()
```
使用该代码会生成一个使用model生成的结构(`a.starting_model = 1`和`a.ending_model = 1`)和两个使用loopmodel生成的结构(`a.loop.starting_model = 1`和`a.loop.ending_model = 2`)。如果我们要生成更多的结构，那么我们可以把`a.ending_model`和`a.loop.ending_model`设定为更大的值。  

如果你不需要对loop进行优化，那么你可以选择`AutoModel`代替`LoopModel`，同时移除与loop相关的三个参数。  

使用`LoopModel`和`AutoModel`补残基时，默认所有的原子都可以移动，如果你想让不缺失的残基不被移动的话，你可以设置`select_atoms`方法。在Modeller中，残基序号是从1开始并且按顺序加一的，因此在写`residue_range`有可能需要重新编号。同时`residue_range`是包括最后一个列出的残基的。


```python
from modeller import *
from modeller.automodel import *    # Load the AutoModel class

log.verbose()
env = Environ()

# directories for input atom files
env.io.atom_files_directory = ['.', '../atom_files']

class MyModel(AutoModel):
    def select_atoms(self):
        return Selection(self.residue_range('86:A', '89:A'),
                         self.residue_range('180:B', '187:B'),
                         self.residue_range('493:C', '501:C'))

a = MyModel(env, alnfile = 'alignment.ali',
            knowns = '4gnx_half', sequence = '4gnx_half_fill')
a.starting_model= 1
a.ending_model  = 1

a.make()
```
如果使用的是`LoopModel`，那么在使用以上的方式进行约束时，两个边界上的残基还是会被移动，因此还需要添加``select_loop_atoms`进行限制。  
```python
from modeller import *
from modeller.automodel import *    # Load the AutoModel class

log.verbose()
env = Environ()

# directories for input atom files
env.io.atom_files_directory = ['.', '../atom_files']

class MyModel(LoopModel):
    def select_atoms(self):
        return Selection(self.residue_range('86:A', '89:A'),
                         self.residue_range('180:B', '187:B'),
                         self.residue_range('493:C', '501:C'))
    def select_loop_atoms(self):
        return Selection(self.residue_range('86:A', '89:A'),
                         self.residue_range('180:B', '187:B'),
                         self.residue_range('493:C', '501:C'))

a = MyModel(env, alnfile = 'alignment.ali',
            knowns = '4gnx_half', sequence = '4gnx_half_fill')
a.starting_model= 1
a.ending_model  = 1

a.make()
```
