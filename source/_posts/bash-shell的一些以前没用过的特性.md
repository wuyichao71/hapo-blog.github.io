---
title: bash shell的一些以前没用过的特性
date: 2023-01-09 16:03:43
categories:
    - bash
tags:
    - bash
---

# 转化编码格式
有时候在linux打开文件会出现乱码，这时候可以使用`iconv`转化编码格式:
```shell
iconv -f GB2312 -t utf-8 -o output.txt input.txt
```
以上命令可以将GB2312编码格式的`input.txt`文件转化为utf-8编码格式的`output.txt`文件。

<!--more-->
# 数组
用以下命令可以设定bash数组:
```bash
#!/bin/bash
b='b'
array=('a' b 1)
```
用以下命令可以取出列表中的元素:
```bash
c=${a[0]}
```
可以用以下命令取数组长度:
```bash
length=${#a[@]}
```
