---
title: opensuse15.4安装packman
date: 2023-02-06 20:42:33
tags:
    - linux
categories:
    - 计算机
---
最近因为opensuse15.2坏掉了所以升级成了15.4，而安装packman源解决编码器问题一直是个复杂的过程，而最近则发现这个过程已经有人写好了包，所以在这里记录下：
```bash
sudo zypper install opi
opi codecs
```
以上两行代码就解决了编码器问题。
