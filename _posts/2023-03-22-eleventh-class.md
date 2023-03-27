---
layout: post
title: 第十一课
date: 2023-03-22
Author: 
categories: 
tags: [eNSP]
comments: false
toc: false
---

路由表，在网络中，也被称为路由择域信息库（RIB）。

路由表是一个存储在路由器或者我们计算机中的电子表格或类数据库。路由表存储着指向特定网络地址的路径（在有些情况下，还记录有路径的路由度量值）。

路由表中含有网络周边的拓扑信息。路由表建立的主要目标是为了实现路由协议和静态路由选择。

# 给路游配置静态路游

使用命令 `ip route-static <Destination> <Mask> <NextHop>`。

```shell
[Huawei]ip route-static 0.0.0.0 0 192.168.1.1
[Huawei]
```
