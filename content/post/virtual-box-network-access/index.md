---
title: "Virtual Box Network Access"
date: 2023-08-15T09:33:07+08:00
draft: false
description: "Virtual Box Network Access"
categories:

tags:
  - virtual-box
---
> 新建的虚拟机默认是NAT网络，主机无法访问，如果需要在主机访问虚拟机中的web服务，需要新增一个网卡



新建的虚拟机默认是NAT网络，主机无法访问，如果需要在主机访问虚拟机中的web服务

1. 虚拟机新增一个网卡选择Host-Only模式
2. 在虚拟机中安装net-tool工具（否则无法使用ifconfig命令）
3. 默认新增加的网卡是down状态，启用他
```bash
sudo ifconfig eth0s8 up #查看新增加的网卡名称(ip addr)，我这里是eth0s8
#再次查看网络状态，是up状态，但是网络地址是ipv6格式的

```
4. 通过virtual box“全局设定”-“网络管理工具”查看里面启用的ip（ipv4）去访问


[Reference https://www.cnblogs.com/Reyzal/p/7743747.html](https://www.cnblogs.com/Reyzal/p/7743747.html)