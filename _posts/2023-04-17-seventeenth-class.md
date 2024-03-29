---
layout: post
title: 第十七课
date: 2023-04-17
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# OSPF基础配置

##一、OSPF功能介绍：

OSPF（Open Shortest Path First）为 IETF OSPF 工作组开发的一种基于链路状态的内部网关路由协议。OSPF 是专为 IP 开发的路由协议，直接运行在 IP 层上面，协议号为 89，采用组播方式进行 OSPF 包交换，组播地址为 224.0.0.5 （全部 OSPF 设备）和 224.0.0.6（指定设备）。当 OSPF 路由域规模较大时，一般采用分层结构，即将 OSPF 路由域分割成几个区域（AREA），区域之间通过一个骨干区域互联，每个非骨干区域都需要直接与骨干区域连接。

## 二、OSPF应用场景：

OSPF路由协议是目前主流的IGP协议，被绝大部分客户所认可并实际采用，广泛应用于各个行业，像教育，金融，医疗，政府，运营商，企业等，不论组网模型是复杂还是简单，设备数量多少，路由条目的多少，OSPF都能很好的满足各类需求，他的丰富的路由策略控制功能，分层设计也是一大优势，所以在网络部署IGP协议的时候，可优先考虑OSPF组网。

## 三、OSPF实验配置：

1、拓扑图

[![ppO8T3T.jpg](https://s1.ax1x.com/2023/04/11/ppO8T3T.jpg)](https://imgse.com/i/ppO8T3T)

2、实验目的：全网路由器运行ospf协议，使全网路由可达

3、配置思路：

- 搭建好拓扑图环境，标出规划好的IP地址
- 修改网络设备默认名称、配置好IP地址
- 配置OSPF路由，使各网段之间实现互访

4、配置过程：

步骤一：修改网络设备默认名称、配置好IP地址

- 配置各PC信息
- 配置路由器AR1默认名称及接口IP

```shell
sys //进入系统视图模式
Enter system view, return user view with Ctrl+Z.
[Huawei]sysname AR1 //给设备修改名称
[AR1]int g0/0/0 //进入接口模式
[AR1-GigabitEthernet0/0/0]ip add 192.168.1.2 24
[AR1-GigabitEthernet0/0/0]int g0/0/1
[AR1-GigabitEthernet0/0/1]ip add 192.168.12.1 24
```

- 配置路由器AR2默认名称及接口IP
```shell
sys
Enter system view, return user view with Ctrl+Z.
[Huawei]sysname AR2
[AR2]int g0/0/0
[AR2-GigabitEthernet0/0/0]i add 192.168.12.2 24
[AR2-GigabitEthernet0/0/0]int g0/0/1
[AR2-GigabitEthernet0/0/1]ip add 192.168.23.1 24
[AR2-GigabitEthernet0/0/1]quit
```

- 配置路由器AR3默认名称及接口IP
```shell
sys
Enter system view, return user view with Ctrl+Z.
[Huawei]sysname AR3
[AR3]int g0/0/0
[AR3-GigabitEthernet0/0/0]ip add 192.168.23.2 24
[AR3-GigabitEthernet0/0/0]int g0/0/1
[AR3-GigabitEthernet0/0/1]ip add 192.168.2.2 24
[AR3-GigabitEthernet0/0/1]quit
```

步骤二、配置RIP路由，使各网段之间通过该链路实现互访

- 配置路由器AR1的OSPF路由
```shell
[AR1]ospf router-id 1.1.1.1 //启用OSPF，并配router id 为1.1.1.1
[AR1-ospf-1]area 0 //区域为0
[AR1-ospf-1-area-0.0.0.0]network 192.168.1.0 0.0.0.255 //发布直连网段与通配符
[AR1-ospf-1-area-0.0.0.0]network 192.168.12.0 0.0.0.255
```

> 通配符0.0.0表示这一部分要与192.168.1完全一致，最后为255表示可在1-255内取值，也即192.168.1.0/24这一网段*

- 配置路由器AR2的OSPF路由
```shell
[AR2]ospf router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]network 192.168.12.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]area 1
[AR2-ospf-1-area-0.0.0.1]network 192.168.23.0 0.0.0.255
```

[![ppO87gU.png](https://s1.ax1x.com/2023/04/11/ppO87gU.png)](https://imgse.com/i/ppO87gU)

出现该提示信息说明邻居建立成功

- 配置路由器AR3的OSPF路由
```shell
[AR3]ospf router-id 3.3.3.3
[AR3-ospf-1]area 1
[AR3-ospf-1-area-0.0.0.1]network 192.168.23.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.1]network 192.168.2.0 0.0.0.255
```

[![ppO8HvF.jpg](https://s1.ax1x.com/2023/04/11/ppO8HvF.jpg)](https://imgse.com/i/ppO8HvF)

5、配置验证：

查看各路由器路由表，输入命令dis ip routing-table

路由器AR1：[AR1]dis ip routing-table

[![ppO8Ob9.jpg](https://s1.ax1x.com/2023/04/11/ppO8Ob9.jpg)](https://imgse.com/i/ppO8Ob9)

路由器AR2：[AR2]dis ip routing-table

[![ppO8x4x.jpg](https://s1.ax1x.com/2023/04/11/ppO8x4x.jpg)](https://imgse.com/i/ppO8x4x)

路由器AR3：[AR3]dis ip routing-table

[![ppOGSC6.jpg](https://s1.ax1x.com/2023/04/11/ppOGSC6.jpg)](https://imgse.com/i/ppOGSC6)

测试两台主机连通性：

[![ppOGp8K.jpg](https://s1.ax1x.com/2023/04/11/ppOGp8K.jpg)](https://imgse.com/i/ppOGp8K)

## 总结与扩展：

1、适用范围：应用于规模适中的网络中，最多可支持几百台NE。例如，中小型企业网络。

2、收敛速度：收敛速度快，小于1s。

3、扩展性：通过划分区域扩展网路支撑能力。

4、无自环：由于OSPF根据收集到的链路状态用最短路径树算法计算路由，从算法本身保证了不会生成自环路由。

5、区域划分：允许自治系统的网络被划分成区域来管理，区域间传送的路由信息被进一步抽象，从而减少了占用的网络带宽。

6、组播发送：在某些类型的链路上以组播地址发送协议报文，减少对其他设备的干扰。
