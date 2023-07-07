---
title: recipe of Makefile
date: 2023-06-17 12:34:59
mathjax: true
categories:
    - 计算机
tags:
    - 计算机系统基础
---

`make` programe is used to manage huge program. The file named `Makefile` tells `make` what to do. The main content of `Makefile` is "rules" and "rules" have the following shape:
<!--more-->
```make
target  ...: prerequisites ...
    recipe
    ...
    ...
```
