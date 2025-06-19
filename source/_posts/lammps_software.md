---
title: lammps源代码一些函数分析
date: 2025-06-19 14:12:13
categories:
  - MD
tags:
  - C
  - C++
---

# lammps 源代码一些函数分析

## 各个模块的结构

运行命令为`./lmp_mpi -in in.eq -log log.phe2`

### 解析命令行

函数调用为:

```
#0  LAMMPS_NS::LAMMPS::LAMMPS (this=0x55555616a4b0, narg=5, arg=0x7fffffffd638, communicator=0x55555607f7a0 <ompi_mpi_comm_world>) at /home/hapo/Documents/software/lammps-29Aug2024/src/lammps.cpp:544
#1  0x00005555555e02df in main (argc=5, argv=0x7fffffffd638) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:77
```

在`LAMMPS_NS::LAMMPS::LAMMPS`中解析了命令行并打开`in.eq`文件和`log.phe2`文件 (没有读取)

`LAMMPS_NS::LAMMPS::LAMMPS`中的`pfirst`和`plast`用来记录命令行中的添加包命令 (`-pk`)的起始和结束。

从源代码(lammps.cpp:458)来看, lammps 最多只能添加两个 suffix, 无法用 hybrid 添加三个及以上。

`universe->me`用来指出这个进程是 rank 几, 0 表示主进程

### 读取 input 文件 (in.eq)

函数调用为:

```
#0  LAMMPS_NS::Input::file (this=0x55555611f910) at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:198
#1  0x00005555555e0313 in main (argc=3, argv=0x7fffffffd668) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

### 创建进程 grid

函数调用为:

```
#0  0x000055555591e071 in LAMMPS_NS::Comm::set_proc_grid (this=0x5555560cf1c0, outflag=1)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/comm.cpp:600
#1  0x000055555571f3fe in LAMMPS_NS::ReadData::command (this=0x5555561e0270, narg=1, arg=0x5555561e01b0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:684
#2  0x00005555555e6339 in LAMMPS_NS::Input::execute_command (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#3  0x00005555555e2a59 in LAMMPS_NS::Input::file (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#4  0x00005555555e02f3 in main (argc=5, argv=0x7fffffffd638) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

这是用来设置把 rank 映射到 grid 上的

### 计算 xyz 分成多少个 grid，能有多少个因子(factor)

```
#0  LAMMPS_NS::ProcMap::factor (this=0x55555616a5e0, n=1, factors=0x5555561dfc40)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/procmap.cpp:731
#1  0x0000555555ce88e2 in LAMMPS_NS::ProcMap::onelevel_grid (this=0x55555616a5e0, nprocs=1, user_procgrid=0x5555560cf2e4,
    procgrid=0x5555560cf2d8, otherflag=0, other_style=1443715056, other_procgrid=0x5555560cf3e4, other_coregrid=0x5555560cf3f0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/procmap.cpp:56
#2  0x000055555591dc4d in LAMMPS_NS::Comm::set_proc_grid (this=0x5555560cf1c0, outflag=1)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/comm.cpp:566
#3  0x000055555571f3fe in LAMMPS_NS::ReadData::command (this=0x5555561e0230, narg=1, arg=0x5555561e0170)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:684
#4  0x00005555555e6339 in LAMMPS_NS::Input::execute_command (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#5  0x00005555555e2a59 in LAMMPS_NS::Input::file (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#6  0x00005555555e02f3 in main (argc=5, argv=0x7fffffffd638) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

factor 函数计算有多少个分解方式，返回的是有多少种组合，各种组合储存在 factors 中

### 选择最好的分配方式

```
#0  LAMMPS_NS::ProcMap::best_factors (this=0x55555616a5e0, npossible=1, factors=0x5555561dfc40, best=0x5555560cf2d8, sx=1,
    sy=1, sz=1) at /home/hapo/Documents/software/lammps-29Aug2024/src/procmap.cpp:847
#1  0x0000555555ce8a27 in LAMMPS_NS::ProcMap::onelevel_grid (this=0x55555616a5e0, nprocs=1, user_procgrid=0x5555560cf2e4,
    procgrid=0x5555560cf2d8, otherflag=0, other_style=1443715056, other_procgrid=0x5555560cf3e4, other_coregrid=0x5555560cf3f0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/procmap.cpp:73
#2  0x000055555591dc4d in LAMMPS_NS::Comm::set_proc_grid (this=0x5555560cf1c0, outflag=1)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/comm.cpp:566
#3  0x000055555571f3fe in LAMMPS_NS::ReadData::command (this=0x5555561e0230, narg=1, arg=0x5555561e0170)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:684
#4  0x00005555555e6339 in LAMMPS_NS::Input::execute_command (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#5  0x00005555555e2a59 in LAMMPS_NS::Input::file (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#6  0x00005555555e02f3 in main (argc=5, argv=0x7fffffffd638) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

best_factors 用于给出最好的组合方式。分割方式储存在 procgrid 中。

### 用 cart 方式创建 proc 和 grid 的映射

```
#0  LAMMPS_NS::ProcMap::cart_map (this=0x55555616a5e0, reorder=0, procgrid=0x5555560cf2d8, myloc=0x5555560cf2f0,
    procneigh=0x5555560cf2fc, grid2proc=0x5555561df2c0) at /home/hapo/Documents/software/lammps-29Aug2024/src/procmap.cpp:358
#1  0x000055555591e071 in LAMMPS_NS::Comm::set_proc_grid (this=0x5555560cf1c0, outflag=1)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/comm.cpp:600
#2  0x000055555571f3fe in LAMMPS_NS::ReadData::command (this=0x5555561e0230, narg=1, arg=0x5555561e0170)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:684
#3  0x00005555555e6339 in LAMMPS_NS::Input::execute_command (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#4  0x00005555555e2a59 in LAMMPS_NS::Input::file (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#5  0x00005555555e02f3 in main (argc=5, argv=0x7fffffffd638) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

grid 到 proc 的映射储存在`grid2proc`中，每个 proc 的相邻的 proc 储存在 procneigh 中。

`comm->xsplit`和`comm->ysplit`和`comm->zsplit`用于按照核数分割盒子，是分割的边。

`domain->sublo`和`domain->subhi`用于记录分解到该 rank 下的盒子范围

`atom->tag`记录的是`atom ID`

这些数据读取方式是 (以`tag`为例): 其是属于`Atom`类的, 然后会在`peratom_create`中用`add_peratom`把地址和`id`这个词绑定。之后会根据`fields_data_atom`这个变量中设置的字符串个数来算出`nfield`来决定之后要为`mdata_atom.pdata`这个变量开多少空间。之后会用`AtomVec::grow`为每一个 field 开空间。

### parse_keyword 用来分析各个块的标志

`parse_keyword`用来分析`Atoms`, `Bonds`, `Angles`, `Dihedrals`这些标记

```
#0  LAMMPS_NS::ReadData::parse_keyword (this=0x5555561e0230, first=0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:2461
#1  0x0000555555722ff2 in LAMMPS_NS::ReadData::command (this=0x5555561e0230, narg=1, arg=0x5555561e0170)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:1016
#2  0x00005555555e6339 in LAMMPS_NS::Input::execute_command (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#3  0x00005555555e2a59 in LAMMPS_NS::Input::file (this=0x55555611f930)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#4  0x00005555555e02f3 in main (argc=5, argv=0x7fffffffd638) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

### 每个 rank 如何读取原子

原子属于哪个 rank 是通过 sub box 来确定的 (sublo 和 subhi)。每个 rank 的原子个数记录在`atom->nlocal`中。

### 读取`DATA.FILE`文件

lammps 总共会对`DATA.FILE`读取两遍, 第一遍确定每个原子最多会有多少个相互作用 (比如每个原子有多少个 bond), 从而分配空间, 第二遍读取文件数据。

### 构造 neighbor list 的数据结构 (create)

函数调用为:

```
#0  LAMMPS_NS::Neighbor::Neighbor (this=0x5555561d0c10, lmp=0x55555616a4d0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/neighbor.cpp:125
#1  0x000055555560bc95 in LAMMPS_NS::LAMMPS::create (this=0x55555616a4d0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/lammps.cpp:858
#2  0x0000555555609f3a in LAMMPS_NS::LAMMPS::LAMMPS (this=0x55555616a4d0, narg=3, arg=0x7fffffffd668,
    communicator=0x55555607f7a0 <ompi_mpi_comm_world>) at /home/hapo/Documents/software/lammps-29Aug2024/src/lammps.cpp:744
#3  0x00005555555e02ff in main (argc=3, argv=0x7fffffffd668) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:77
```

在`LAMMPS_NS::LAMMPS::create`中构造了一大堆结构, 它们之间的相互交互一定会让我很头疼。

### 运行能量最小化

函数调用为:

```
#0  LAMMPS_NS::Minimize::command (this=0x5555561dfc80, narg=4, arg=0x5555561e01b0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/minimize.cpp:34
#1  0x00005555555e6359 in LAMMPS_NS::Input::execute_command (this=0x55555611f910)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#2  0x00005555555e2a79 in LAMMPS_NS::Input::file (this=0x55555611f910)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#3  0x00005555555e0313 in main (argc=3, argv=0x7fffffffd668) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

neighbor list 的 pair 部分的数据结构初始化

函数调用为:

```
#0  LAMMPS_NS::Neighbor::init_pair (this=0x5555561d0c10) at /home/hapo/Documents/software/lammps-29Aug2024/src/neighbor.cpp:803
#1  0x000055555569a595 in LAMMPS_NS::Neighbor::init (this=0x5555561d0c10)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/neighbor.cpp:672
#2  0x000055555560ce0c in LAMMPS_NS::LAMMPS::init (this=0x55555616a4d0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/lammps.cpp:984
#3  0x000055555567b05f in LAMMPS_NS::Minimize::command (this=0x5555561dfc80, narg=4, arg=0x5555561e01b0)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/minimize.cpp:57
#4  0x00005555555e6359 in LAMMPS_NS::Input::execute_command (this=0x55555611f910)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#5  0x00005555555e2a79 in LAMMPS_NS::Input::file (this=0x55555611f910)
    at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#6  0x00005555555e0313 in main (argc=3, argv=0x7fffffffd668) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

## Appendix

在从`DATA.FILE`中读取坐标时，原子会被 remap 到盒子中。这在手册中提到:

["If the system is periodic (in a dimension), then atom coordinates can be outside the bounds (in that dimension); they will be remapped (in a periodic sense) back inside the box. "](https://docs.lammps.org/read_data.html)

函数调用为:

```
#0  LAMMPS_NS::Domain::remap (this=0x5555561d10e0, x=0x7fffffff8ba0, image=@0x7fffffff8bbc: 537395712) at /home/hapo/Documents/software/lammps-29Aug2024/src/domain.cpp:1565
#1  0x00005555558b784a in LAMMPS_NS::Atom::data_atoms (this=0x5555561d1680, n=5, buf=0x7ffff7e65054 "2       1       2        0.0000   22.6470    3.4060   -0.4600 # XYR", id_offset=0, mol_offset=0, type_offset=0, shiftflag=0, shift=0x5555561e0518, labelflag=0,
    ilabel=0x5555561de760, triclinic_general=0) at /home/hapo/Documents/software/lammps-29Aug2024/src/atom.cpp:1227
#2  0x000055555572c93d in LAMMPS_NS::ReadData::atoms (this=0x5555561e0230) at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:1529
#3  0x000055555571f6e6 in LAMMPS_NS::ReadData::command (this=0x5555561e0230, narg=1, arg=0x5555561e0170) at /home/hapo/Documents/software/lammps-29Aug2024/src/read_data.cpp:708
#4  0x00005555555e6359 in LAMMPS_NS::Input::execute_command (this=0x55555611f910) at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:868
#5  0x00005555555e2a79 in LAMMPS_NS::Input::file (this=0x55555611f910) at /home/hapo/Documents/software/lammps-29Aug2024/src/input.cpp:313
#6  0x00005555555e0313 in main (argc=3, argv=0x7fffffffd668) at /home/hapo/Documents/software/lammps-29Aug2024/src/main.cpp:78
```

`universe->uni2orig`记录的是在 universe 到 origin 的映射, 比如在 universe 的 I proc 在 origin 中是`uni2orig[I]` (具体我也不知道什么意思, 貌似和命令行参数`-mpicolor`有关)
