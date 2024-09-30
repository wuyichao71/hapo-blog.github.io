---
title: 在opensuse上外网远程连接
date: 2023-08-07 14:38:51
catagories:
    - 计算机
tags:
    - ssh
---

## 设置路由器
需要把光猫设置成桥接模式并且需要知道宽带拨号的帐号密码, 这部分可以咨询宽带师傅. 不需要关闭DHCP服务, 否则会导致需要调节光猫的时候需要设置内网IP, 为了减少麻烦就不关闭了. 之后通过网线从光猫LAN口接出, 接入路由器WAN口. 为了使得外网可以通过外网IP访问内部电脑, 可以设置先设置静态IP地址绑定, 再设置端口映射或者DMZ主机. 如果使用端口映射, 那么暂时只开放了一个服务, 如果设置DMZ主机, 那么服务就都开放了. 
<!--more-->
通过设置DDNS可以与花生壳的域名绑定, 但是花生壳的免费域名只有1年的使用时间, 因此还是需要去获得公网IP比较稳妥.

## 获得公网IP
如果使用的是华为路由器, 那么就可以使用华为智慧生活APP获得公网IP. 

## 安装花生壳
opensuse可以安装花生壳的rpm包, 可以去[官网](https://hsk.oray.com/download)下载安装包. 之后需要做一些设置才能安装. 首先需要创建文件夹`/lib/systemd/system`, 其次需要安装`netstat`. 在opensuse上需要使用以下命令:
```bash
sudo zypper in net-tools-deprecated
```
这样才会在有`netstat`.

之后就可以使用

```bash
sudo rpm -i phddns_<version>.rpm
```

来安装了.

卸载的命令是
```bash
sudo rpm -e phddns
```

> 这里有个小坑, 用花生壳做内网穿透的话, 如果ip和刚刚做DDNS的ip不一致, 那么会把前面的DDNS设置冲掉. 因为是坑所以就不介绍如何设置内网穿透了.

## 安装向日葵

## 安装teamviewer