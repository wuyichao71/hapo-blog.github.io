---
title: 解决U盘刻盘空间变小问题
date: 2023-02-06 20:54:38
tags:
    - linux
categories:
    - 计算机
---
U盘刻盘ISO后空间会变得和ISO问题大小一样，最初是在折腾树莓派的时候发现这个问题。最近在刻opensuse15.4的盘的时候也发生了这个问题，因此在这里记录下解决这个问题的代码：
```bash
sudo parted /dev/sdc # 使用parted来调整磁盘/dev/sdc
print # 打印查看当前的
resizepart 2 -1 # 将第二个分区充满剩下的空间
quit # 退出
sudo resize2fs /dev/sdc2 # 使用resize2fs来调整sdc2分区大小
```
亲测在树莓派的SD卡以及U盘上都可用。