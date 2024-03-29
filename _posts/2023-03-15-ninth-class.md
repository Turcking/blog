---
layout: post
title: 第九课
date: 2023-03-15
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

不同网段的用户想要进行三层通信，必须借助于路由表项，而VLANIF接口只能生成直连路由，实现不同网段间通过同一台设备互通，对于不同网段间跨设备的互通，必须手动配置静态路由或配置通过路由协议生成的动态路由。

# 组网需求
> 如下图所示，将用户1和用户2划为在不同的VLAN下，其中用户1属于VLAN 10，用户2属于VLAN 20，通过配置路由数据实现用户1与用户2通过VLAN 30互通（中央交换机、车站交换机是三层交换机）。

![拓补图](https://camo.github.com/a3a99772c178ef1992eb643cf3c6ac1da7cf606ac572b98b73e7663a173362ed/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f32303230303732393137303132363239332e706e67 "Topo")

# 配置思路

1. 创建vlan
在中央交换机SW1上配置VLANIF10，作为用户1的网关，在车站交换机SW2上配置VLANIF20，作为用户2的网关。

2. 配置vlanif
在中央交换机SW1、车站交换机SW2上分别配置VLANIF30，作为互传网段。

3. 配置route
在中央交换机SW1上配置从SW1达到VLANIF20网段的静态路由，在车站交换机SW2上配置从SW2到VLANIF10网段的静态路由，实现跨网段通信。

# 操作步骤

## 1.SW1-vlan

### 创建VLAN

```shell
<HUAWEI>system-view
[HUAWEI]sysname SW1  //修改设备的名称为SW1
[SW1]vlan batch 10 30  //批量创建VLAN 10 30
```

### 将接口加入相应VLAN

```shell
[SW1]interface gigabitethernet 0/0/1  //进入GE端口0/0/1
[SW1-GigabitEthernet0/0/1]port link-type access  //将与用户1相连接口的接口类型设置为access
[SW1-GigabitEthernet0/0/1]port default vlan 10  //将用户1划分到VLAN 10
[SW1-GigabitEthernet1/0/1]quit
[SW1]interface gigabitethernet 0/0/2  //进入GE端口0/0/2
[SW1-GigabitEthernet0/0/2]port link-type trunk  //将与车站交换机SW2相连接口的接口类型设置为trunk
[SW1-GigabitEthernet0/0/2]port trunk allow-pass vlan 30  //透传和车站交换机SW2互连的VLAN 30
[SW1-GigabitEthernet0/0/2]quit
```

## 2.SW2-vlan

### 创建VLAN

```shell
<HUAWEI> system-view
[HUAWEI]sysname SW2  //修改设备的名称为SW2
[SW2]vlan batch 20 30  //批量创建VLAN 20 30
```

### 将接口加入相应VLAN

```shell
[SW2]interface gigabitethernet 0/0/1  //进入GE端口0/0/1
[SW2-GigabitEthernet0/0/1]port link-type access  //将与用户2相连接口的接口类型设置为access
[SW2-GigabitEthernet0/0/1]port default vlan 20  //将用户2划分到VLAN 20
[SW2-GigabitEthernet0/0/1]quit
[SW2]interface gigabitethernet 0/0/2  //进入GE端口0/0/2
[SW2-GigabitEthernet0/0/2]port link-type trunk  //将与中央交换机SW1相连接口的接口类型设置为trunk
[SW2-GigabitEthernet1/0/2]port trunk allow-pass vlan 30 //透传和中央交换机SW1互连的VLAN 30
[SW2-GigabitEthernet1/0/2]quit
```

## 3.SW1-route

### 创建VLANIF10并配置对应的IP地址，作为用户1的网关

```shell
[SW1]interface vlanif 10  //创建VLANIF10接口
[SW1-Vlanif10]ip address 192.168.10.254 24  //配置IP地址，此IP地址是用户1的网关地址
[SW1-Vlanif10]quit
```

### 创建VLANIF30和对应的IP地址

```shell
[SW1]interface vlanif 30  //创建VLANIF30接口
[SW1-Vlanif30]ip address 10.10.1.254 24  //配置互连的IP地址，此IP地址不能和用户1、用户2的IP地址冲突
[SW1-Vlanif30]quit
```

### 配置静态路由，使用户1可以访问用户2

```shell
[SW1]ip route-static 192.168.20.0 255.255.255.0 10.10.1.253 //目的IP是192.168.20.0/24网段的报文，转发下一跳是车站交换机SW2 VLANIF30的IP地址10.10.1.253
```

## 4.SW2-route

### 创建VLANIF20并配置对应的IP地址，作为用户2的网关

```shell
[SW2]interface vlanif 20  //创建VLANIF20接口
[SW2-Vlanif20]ip address 192.168.20.254 24  //配置IP地址，此IP地址是用户2的网关地址
[SW2-Vlanif20]quit
```

### 创建VLANIF30和对应的IP地址

```shell
[SW2]interface vlanif 30  //创建VLANIF30接口
[SW2-Vlanif30]ip address 10.10.1.253 24  //配置互连的IP地址
[SW2-Vlanif30]quit
```

### 配置静态路由，使用户2可以访问用户1

```shell
[SW2]ip route-static 192.168.10.0 255.255.255.0 10.10.1.254  //目的IP是192.168.10.0/24网段的报文，转发下一跳是中央交换机SW1 VLANIF 30的IP地址10.10.1.254
```

# 5.检查配置结果

在VLAN 10中的用户1上配置IP地址为192.168.10.1/24，缺省网关为VLANIF10接口的IP地址192.168.10.254。

在VLAN 20中的用户2上配置IP地址为192.168.20.1/24，缺省网关为VLANIF20接口的IP地址192.168.20.254。

配置完成后，VLAN 10内的用户1与VLAN 20内的用户2能够相互访问。

[原文链接](https://blog.csdn.net/kerrysu2015/article/details/107669947)
