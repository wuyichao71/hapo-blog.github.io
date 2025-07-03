---
title: dcd文件格式
date: 2025-06-19 12:08:13
categories:
  - MD
tags:
  - C
  - Fortran
---

# dcd 文件格式

dcd 有各种格式，比如 [X-PLOR DCD 格式](https://www.ks.uiuc.edu/Research/vmd/plugins/molfile/dcdplugin.html)和 CHARMM 格式，暂时我们主要处理 CHARMM 格式。

由于最初的 dcd 格式可能是用 fortran 写的，并且是使用以下的方式打开文件

```fortran
open(unit=10, file="trajectory.dcd", form="unformatted")
```

<!--more-->

所以在写入数据的时候会在数据的两头添加一个 record marker，这个 record marker 记录的是写入的数据有多长。比如如下的程序：

```fortran
! test.f90
program main
  implicit none
  real :: a(3) = [1.0, 2.0, 3.0]
  open(unit=10, file="data.bin", form="unformatted", access="sequential")
  write(10) a
  close(10)
end program main
```

我们用以下的命令对它进行编译运行并查看文件大小的话

```bash
gfotran test.f90 && ./a.out
wc -c data.bin
```

会发现其大小是 20 bytes

```raw
20 data.bin
```

这是因为其在两端都添加了 record marker

如果我们用`xxd`命令查看其中的数据的话

```bash
xxd data.bin
```

其输出如下

```raw
00000000: 0c00 0000 0000 803f 0000 0040 0000 4040  .......?...@..@@
00000010: 0c00 0000
```

对应的值如下

| 1-4 bytes           | 5-8 bytes             | 9-12 bytes            | 13-16 bytes           | 17-20 bytes        |
| ------------------- | --------------------- | --------------------- | --------------------- | ------------------ |
| 0c00 0000 (int(12)) | 0000 803f(float(1.0)) | 0000 0040(float(2.0)) | 0000 4040(float(3.0)) | 0c00 0000(int(12)) |

而这个 record marker 的长度是由编译器决定的，有的编译器是 4 bytes (32 bits)，而有的编译器是 8 bytes (64 bits)。

所以用 C 语言读取 dcd 格式的时候，需要考虑这些问题，编写起来非常繁琐。暂时我们只考虑 CHARMM 格式，并且 record marker 长度为 4 bytes 的情况。我们以 GENESIS 输出的 dcd 格式为例进行说明。
