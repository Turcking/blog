---
layout: post
title: 第十九课
date: 2023-04-24
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# 给 RIP 配置多个网段

在一个 RIP 进程中配置多个 `network <X.X.X.X>`。

```shell
[Huawei-rip-1]network 192.168.1.0
[Huawei-rip-1]network 192.168.2.0
[Huawei-rip-1]network 192.168.3.0
[Huawei-rip-1]
```
