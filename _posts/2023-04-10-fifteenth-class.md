---
layout: post
title: 第十五课
date: 2023-04-10
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# 1. OSPF简介

## 定义

开放式最短路径优先OSPF（Open Shortest Path First）是IETF组织开发的一个基于链路状态的内部网关协议（Interior Gateway Protocol）。

- OSPF把自治系统AS（Autonomous System）划分成逻辑意义上的一个或多个区域；
- OSPF通过链路状态通告LSA（Link State Advertisement）的形式发布路由；
- OSPF依靠在OSPF区域内各设备间交互OSPF报文来达到路由信息的统一；
- OSPF报文封装在IP报文内，可以采用单播或组播的形式发送。

目前针对IPv4协议使用的是OSPF Version 2（RFC2328）；针对IPv6协议使用OSPF Version 3（RFC2740）。如无特殊说明，本文中所指的OSPF均为OSPF Version 2。

## 优点

在OSPF出现前，RIP是网络上使用最广泛的IGP协议。但随着网络的快速成长和扩展，RIP的某些限制可能会导致其在大型网络中不再适用，OSPF则能够解决RIP所面临的诸多问题。

## RIP

- 基于距离矢量算法，以跳数作为度量方式，忽略带宽的影响。
- RIP的跳数限制为15个，限制了RIP的网络规模。
- 按照路由通告进行路由更新和选择，路由器不了解整个网络拓扑，容易产生路由环路。
- 收敛速度慢，路由更新会经历一段抑制和垃圾收集期，容易导致路由器之间的路由不一致。
- 不能处理可变长子网掩码（VLSM）。

## OSPF

- 基于链路状态，以链路开销作为度量方式，并把带宽作为参考值，度量方式更科学。
- 没有跳数限制，适用的网络规模更大。
- 每台路由器都能够掌握全网拓扑，通过最短路径优先算法SPF（Shortest Path First）计算路由，不会产生路由环路。
- 收敛速度快，因为路由更新是及时的，并且能够快速传递到整个网络。
- 能够处理VLSM，灵活进行IP地址分配。

此外，OSPF还有以下优点：

- OSPF可以采用组播形式收发报文，这样可以减少对未运行OSPF的路由器的影响。
- OSPF支持无类型域间选路（CIDR）。
- OSPF支持对等价路由进行负载分担。
- OSPF支持报文验证。

# 2. OSPF的特点

在OSPF网络中，每台路由器根据自己周围的网络拓扑结构生成链路状态通告LSA（Link State Advertisement），并通过更新报文将LSA发送给网络中的其它路由器。

RIP交互的是路由。与RIP不同，OSPF交互的是链路状态信息。也就是说，RIP中，路由器的选路依赖于邻居路由器的路由信息，但不管邻居路由器传达的信息是否正确；而OSPF中，路由器的选路是一种“自主行为”，LSA只是一种选路的参考信息。

每台路由器都通过链路状态数据库LSDB(Link State DataBase)掌握全网的拓扑结构。如图1所示，每台路由器都会收集其它路由器发来的LSA，所有的LSA放在一起便组成了链路状态数据库LSDB。LSA是对路由器周围网络拓扑结构的描述，LSDB则是对整个自治系统的网络拓扑结构的描述。路由器将LSDB转换成一张带权的有向图，这张图便是对整个网络拓扑结构的真实反映。在网络拓扑稳定的情况下，各个路由器得到的有向图是完全相同的。

[![ppO1SC8.jpg](https://s1.ax1x.com/2023/04/11/ppO1SC8.jpg)](https://imgse.com/i/ppO1SC8)

< center>图1 通过LSDB掌握全网的拓扑结构< /center>

路由器根据最短路径优先(Shortest Path First)算法计算到达目的网络的路径，而不是根据路由通告来获取路由信息。如图2所示，每台路由器根据有向图，使用SPF算法计算出一棵以自己为根的最短路径树，这棵树给出了到自治系统中各节点的路由。相对于RIP，这种机制极大地提升了路由器的自主选路能力，使得路由器不再依靠路由通告进行选路。

[![ppO1p8S.jpg](https://s1.ax1x.com/2023/04/11/ppO1p8S.jpg)](https://imgse.com/i/ppO1p8S)

<center>图2 根据SPF计算到达目的网络的路径</center>

总之，LSDB保证路由器能够时刻掌握全网的拓扑机构，SPF算法保证路由器能够迅速计算出到达目的网络的最短路径。

# 3. OSPF运行机制

OSPF的运行机制包括以下5个步骤：

- 交互Hello报文
- 泛洪LSA
- 组建LSDB
- SPF算法计算
- 维护和更新路由表

## 交互Hello报文

##通过交互Hello报文形成邻居关系

如图3所示，路由器运行OSPF协议后，会从所有启动OSPF协议的接口上发送Hello报文。如果两台路由器共享一条公共数据链路，并且能够成功协商各自Hello报文中所指定的某些参数，就能形成邻居关系。

[![ppO1CvQ.jpg](https://s1.ax1x.com/2023/04/11/ppO1CvQ.jpg)](https://imgse.com/i/ppO1CvQ)

## 泛洪LSA

## 通过泛洪LSA通告链路状态信息

形成邻居关系的路由器之间进一步交互LSA形成邻接关系，如图4所示。每台路由器根据自己周围的网络拓扑结构生成LSA，LSA描述了路由器所有的链路、接口、邻居及链路状态等信息，路由器通过交互这些链路信息来了解整个网络的拓扑信息。由于链路的多样性，OSPF协议定义了多种LSA类型。详见OSPF LSA类型。

[![ppO1iuj.jpg](https://s1.ax1x.com/2023/04/11/ppO1iuj.jpg)](https://imgse.com/i/ppO1iuj)

<center>图4 通过泛洪LSA通告链路状态信息</center>

## 组建LSDB

## 通过组建LSDB形成带权有向图

通过LSA的泛洪，路由器会把收到的LSA汇总记录在LSDB中。最终，所有路由器都会形成同样的LSDB，如图5所示。LSA是对路由器周围网络拓扑结构的描述，而LSDB则是对整个自治系统的网络拓扑结构的描述，LSDB是LSA的汇总。

[![ppOJYeH.jpg](https://s1.ax1x.com/2023/04/11/ppOJYeH.jpg)](https://imgse.com/i/ppOJYeH)

<center>图5 通过组建LSDB形成带权有向图</center>

## SPF算法计算

## 通过SPF算法计算并形成路由

如图6所示，当LSDB同步完成之后，每一台路由器都将以其自身为根，使用SPF算法来计算一个无环路的拓扑图来描述它所知道的到达每一个目的地的最短路径（最小的路径代价）。这个拓扑图就是最短路径树，有了这棵树，路由器就能知道到达自治系统中各个节点的最优路径。

[![ppOJBSf.jpg](https://s1.ax1x.com/2023/04/11/ppOJBSf.jpg)](https://imgse.com/i/ppOJBSf)

<center>图6 通过SPF算法计算并形成路由</center>

## 维护和更新路由表

根据SPF算法得出最短路径树后，每台路由器将计算得出的最短路径加载到OSPF路由表形成指导数据转发的路由表项，并且实时更新，如图7所示。同时，邻居之间交互Hello报文进行保活，维持邻居关系或邻接关系，并且周期性地重传LSA。

[![ppOJBSf.jpg](https://s1.ax1x.com/2023/04/11/ppOJBSf.jpg)](https://imgse.com/i/ppOJBSf)

<center>图7 维护和更新路由表</center>

# 4. OSPF报文类型

OSPF的报文类型可以分为以下五种：

- Hello报文
- DD报文
- LSR报文
- LSU报文
- LSAck报文

## Hello报文

- 邻居发现：使能了OSPF功能的接口会周期性地发送Hello报文，与网络中其他收到Hello报文的路由器协商报文中的指定参数，决定是否建立邻居关系。
- 建立双向通信：如果路由器发现收到的Hello报文的邻居列表中有自己Router ID，则认为已经和对端建立了双向通信，邻居关系建立。
- 指定DR和BDR：Hello报文包含DR优先级和Router ID等信息，每台路由器将自己选出的DR和BDR写入Hello报文的DR和BDR字段中，然后进行DR和BDR的选举，详细的选举原则和过程请参见DR和BDR选举。
- 保活：在建立邻居关系后，使能OSPF功能的接口仍周期性地发送Hello报文维护邻居关系，如果在一定的时间间隔内没有收到邻居发来的Hello报文，则中断邻居关系。

## DD报文

两台路由器在邻接关系初始化时，DD报文（Database Description packet）用来协商主从关系，此时报文中不包含LSA的Header。在两台路由器交换DD报文的过程中，一台为Master，另一台为Slave。由Master规定起始序列号，每发送一个DD报文序列号加1，Slave方使用Master的序列号作为确认。

邻接关系建立之后，路由器使用DD报文描述本端路由器的LSDB，进行数据库同步。DD报文里包括LSDB中每一条LSA的Header（LSA的Header可以唯一标识一条LSA），即所有LSA的摘要信息。LSA Header只占一条LSA的整个数据量的一小部分，这样可以减少路由器之间的协议报文流量。对端路由器根据LSA Header就可以判断出是否已有这条LSA。

## LSR报文

两台路由器互相交换过DD报文之后，需要发送LSR报文（Link State Request packet）向对方请求更新LSA。LSR报文里包括所需要的LSA的摘要信息。

## LSU报文

LSU报文（Link State Update packet）用来向对端路由器发送其所需要的LSA或者泛洪本端更新的LSA，其报文内容是多条完整的LSA的集合。为了实现泛洪的可靠性传输，需要LSAck报文对其进行确认，对没有收到确认报文的LSA进行重传，重传的LSA是直接发送到邻居的。

## LSAck报文

LSAck报文（Link State Acknowledgment packet）用来对接收到的LSU报文进行确认，内容是需要确认的LSA的Header。一个LSAck报文可对多个LSA进行确认。

# 5. OSPF支持的网络类型

OSPF根据链路层协议类型，将网络分为如下四种类型：

- MA类型
- NBMA类型
- P2MP类型
- P2P类型

## MA类型

广播类型（Broadcast）

当链路层协议是Ethernet或FDDI（Fiber Distributed Digital Interface）时，缺省情况下，OSPF认为网络类型是Broadcast。

在该类型的网络中：

- 通常以组播形式发送Hello报文、LSU报文和LSAck报文。其中，224.0.0.5的组播地址为OSPF设备的预留IP组播地址；224.0.0.6的组播地址为OSPF DR/BDR的预留IP组播地址。
- 以单播形式发送DD报文和LSR报文。

## NBMA类型

NBMA类型（Non-Broadcast Multi-Access）

当链路层协议是帧中继或X.25时，缺省情况下，OSPF认为网络类型是NBMA。

在该类型的网络中，以单播形式发送协议报文（Hello报文、DD报文、LSR报文、LSU报文、LSAck报文）。

## P2MP类型

点到多点P2MP类型（Point-to-Multipoint）

没有一种链路层协议会被缺省的认为是P2MP类型。点到多点必须是由其他的网络类型强制更改的。常用做法是将非全连通的NBMA改为点到多点的网络。

在该类型的网络中：

- 以组播形式（224.0.0.5）发送Hello报文。
- 以单播形式发送其他协议报文（DD报文、LSR报文、LSU报文、LSAck报文）。

## P2P类型

点到点P2P类型（Point-to-Point）

当链路层协议是PPP、HDLC或LAPB时，缺省情况下，OSPF认为网络类型是P2P。

在该类型的网络中，以组播形式（224.0.0.5）发送协议报文（Hello报文、DD报文、LSR报文、LSU报文、LSAck报文）。

# 6. DR和BDR选举

## Router ID

在DR和BDR选举的过程中，如果两台路由器的DR优先级相等，需要进一步比较两台路由器的Router ID，Router ID大的路由器将被选为DR或BDR。

Router ID是用于在自治系统中唯一标识一台运行OSPF的路由器的32位整数。每个运行OSPF的路由器都有一个Router ID。Router ID的格式和IP地址的格式是一样的。

OSPF的Router ID的选取有两种方式：手动配置和设备自动选取。在实际网络部署中，考虑到协议的稳定，推荐手工配置Loopback接口的IP地址做为Router ID。

如果没有手动配置OSPF的Router ID，设备会选取全局Router ID作为OSPF的RouterID，如果二者都没有配置，设备会按照一定的规则自动选取。具体的选取规则，请参见路由的Router ID。

以下3种情况会进行Router ID的重新选取：

- 通过本命令重新配置OSPF的Router ID
- 重新配置全局Router ID，并且重新启动OSPF进程
- 原来被选举为全局Router ID的IP地址被删除并且重新启动OSPF进程

## 选举的原因

在广播网络和NBMA网络中，任意两台路由器之间都要传递路由信息。如图8所示，网络中有n台路由器，则需要建立n*(n-1)/2个邻接关系。这使得任何一台路由器的路由变化都会导致多次传递，浪费了带宽资源。

为解决这一问题，OSPF定义了DR。通过选举产生DR后，所有其他设备都只将信息发送给DR，由DR将网络链路状态LSA广播出去。

为了防止DR发生故障，重新选举DR时会造成业务中断，除了DR之外，还会选举一个备份指定路由器BDR。这样除DR和BDR之外的路由器（称为DR Other）之间将不再建立邻接关系，也不再交换任何路由信息，这样就减少了广播网和NBMA网络上各路由器之间邻接关系的数量。

[![ppOJOt1.jpg](https://s1.ax1x.com/2023/04/11/ppOJOt1.jpg)](https://imgse.com/i/ppOJOt1)

<center>图8 DR和BDR选举</center>

## 选举的原则

在广播网络和NBMA网络中，为了稳定地进行DR和BDR选举，OSPF规定了一系列的选举规则：选举制、终身制、继承制。

## 选举制

选举制是指DR和BDR不是人为指定的，而是由本网段中所有的路由器共同选举出来的。如图9所示，路由器接口的DR优先级决定了该接口在选举DR、BDR时所具有的资格，本网段内DR优先级大于0的路由器都可作为“候选人”。选举中使用的“选票”就是Hello报文，每台路由器将自己选出的DR写入Hello报文中，发给网段上的其他路由器。当处于同一网段的两台路由器同时宣布自己是DR时，DR优先级高者胜出。如果优先级相等，则Router ID大者胜出。如果一台路由器的优先级为0，则它不会被选举为DR或BDR。

[![ppOYC0H.jpg](https://s1.ax1x.com/2023/04/11/ppOYC0H.jpg)](https://imgse.com/i/ppOYC0H)

<center>图9 DR和BDR选举的原则—选举制</center>

## 终身制

终身制也叫非抢占制。每一台新加入的路由器并不急于参加选举，而是先考察一下本网段中是否已存在DR。如图10所示，如果目前网段中已经存在DR，即使本路由器的DR优先级比现有的DR还高，也不会再声称自己是DR，而是承认现有的DR。因为网段中的每台路由器都只和DR、BDR建立邻接关系，如果DR频繁更换，则会引起本网段内的所有路由器重新与新的DR、BDR建立邻接关系。这样会导致短时间内网段中有大量的OSPF协议报文在传输，降低网络的可用带宽。终身制有利于增加网络的稳定性、提高网络的可用带宽。实际上，在一个广播网络或NBMA网络上，最先启动的两台具有DR选举资格的路由器将成为DR和BDR。

[![ppOYBNR.jpg](https://s1.ax1x.com/2023/04/11/ppOYBNR.jpg)](https://imgse.com/i/ppOYBNR)

<center>图10 DR和BDR选举的原则—终身制</center>

## 继承制

继承制是指如果DR发生故障了，那么下一个当选为DR的一定是BDR，其他的路由器只能去竞选BDR的位置。这个原则可以保证DR的稳定，避免频繁地进行选举，并且DR是有备份的（BDR），一旦DR失效，可以立刻由BDR来承担DR的角色。由于DR和BDR的数据库是完全同步的，这样当DR故障后，BDR立即成为DR，履行DR的职责，而且邻接关系已经建立，所以从角色切换到承载业务的时间会很短。同时，在BDR成为新的DR之后，还会选举出一个新的BDR，虽然这个过程所需的时间比较长，但已经不会影响路由的计算了。

[![ppOY0E9.jpg](https://s1.ax1x.com/2023/04/11/ppOY0E9.jpg)](https://imgse.com/i/ppOY0E9)

<center>图11 DR和BDR选举的原则—继承制</center>

## 选举过程

广播链路或者NBMA链路上DR和BDR的选举过程如下：
1.接口UP后，发送Hello报文，同时进入到Waiting状态。在Waiting状态下会有一个WaitingTimer，该计时器的长度与DeadTimer是一样的。默认值为40秒，用户不可自行调整。OSPF接口状态的详细描述，请参见OSPF接口状态机。

2.在WaitingTimer触发前，发送的Hello报文是没有DR和BDR字段的。在Waiting阶段，如果收到Hello报文中有DR和BDR，那么直接承认网络中的DR和BDR，而不会触发选举。直接离开Waiting状态，开始邻居同步。

3.假设网络中已经存在一个DR和一个BDR，这时新加入网络中的路由器，不论它的Router ID或者DR优先级有多大，都会承认现网中已有的DR和BDR。

4.当DR因为故障Down掉之后，BDR会继承DR的位置，剩下的优先级大于0的路由器会竞争成为新的BDR。

5.只有当不同Router ID，或者配置不同DR优先级的路由器同时起来，在同一时刻进行DR选举才会应用DR选举规则产生DR。该规则是：优先选择DR优先级最高的作为DR，次高的作为BDR。DR优先级为0的路由器只能成为DR Other；如果优先级相同，则优先选择Router ID较大的路由器成为DR，次大的成为BDR，其余路由器成为DR Other。

## 选举过程验证

五台路由器组成一个广播网络，R5作为纯二层设备，R1～R4作为路由设备。R1～R4都规划在OSPF的Area0区域内，各路由器的IP地址及Router ID如图12所示。

[![ppOYaB4.jpg](https://s1.ax1x.com/2023/04/11/ppOYaB4.jpg)](https://imgse.com/i/ppOYaB4)

<center>图12 DR和BDR选举组网图</center>

## 网络中可以正常选举出DR和BDR时

假设R1～R4各接口的配置已经完成，这里仅给出OSPF相关的配置。

R1的配置
```shell
#
ospf 1 Router ID 10.1.1.1 
  area 0.0.0.0 
  network 192.168.1.0 0.0.0.255 
#
```
R2的配置
```shell
#
ospf 1 Router ID 10.2.2.2
  area 0.0.0.0 
  network 192.168.1.0 0.0.0.255 
#
```
R3的配置
```shell
#
ospf 1 Router ID 10.3.3.3 
  area 0.0.0.0 
  network 192.168.1.0 0.0.0.255 
#
```
R4的配置
```shell
#
ospf 1 Router ID 10.4.4.4
  area 0.0.0.0 
  network 192.168.1.0 0.0.0.255 
#
```

> 配置完成后，待网络稳定后查看当前网络中DR和BDR的选举情况。

## 在R1上查看OSPF的邻居信息。

```shell
<R1> display ospf peer

         OSPF Process 1 with Router ID 10.1.1.1
                 Neighbors

 Area 0.0.0.0 interface 192.168.1.1(GigabitEthernet1/0/1)'s neighbors
 Router ID: 10.2.2.2         Address: 192.168.1.2
   State: Full  Mode:Nbr is Master  Priority: 1
   DR: 192.168.1.1  BDR: 192.168.1.2   MTU: 0
   Dead timer due in 38  sec
   Retrans timer interval: 5
   Neighbor is up for 00:22:16
   Authentication Sequence: [ 0 ]

 Router ID: 10.3.3.3         Address: 192.168.1.3
   State: Full  Mode:Nbr is Master   Priority: 1
   DR: 192.168.1.1  BDR: 192.168.1.2   MTU: 0
   Dead timer due in 35  sec
   Retrans timer interval: 5
   Neighbor is up for 00:21:30
   Authentication Sequence: [ 0 ]

 Router ID: 10.4.4.4         Address: 192.168.1.4
   State: Full  Mode:Nbr is Master   Priority: 1
   DR: 192.168.1.1  BDR: 192.168.1.2   MTU: 0
   Dead timer due in 33  sec
   Retrans timer interval: 5
   Neighbor is up for 00:20:24
   Authentication Sequence: [ 0 ]
```

可以看出，该网络已经完成了DR和BDR的选举，R1是DR，R2是BDR，R3和R4是DR Other。这里R1是DR，R2是BDR跟系统的启动顺序是直接相关的。本例中按照R1、R2、R3、R4的顺序依次启动设备，所以R1和R2首先完成了初始化，自然成为了DR和BDR。

在R1、R2、R3和R4上查看OSPF邻居的概要信息。

```shell
<R1> display ospf 1 peer brief

         OSPF Process 1 with Router ID 10.1.1.1
                   Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id         Interface                  Neighbor id      State
 0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         Full
 0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         Full
 0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         Full
 ----------------------------------------------------------------------------
 Total Peer(s):     3

<R2> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.2.2.2
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         Full
0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         Full
0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         Full
----------------------------------------------------------------------------
Total Peer(s):     3

<R3> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.3.3.3
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         Full
0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         Full
0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3

<R4> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.4.4.4
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1        10.1.1.1         Full
0.0.0.0         GigabitEthernet1/0/1        10.2.2.2         Full
0.0.0.0         GigabitEthernet1/0/1        10.3.3.3         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3
```

可以看出，R1、R2和其他三台路由器的邻居关系都是Full，而R3和R4之间的邻居关系是2-Way状态。这表示DR、BDR与邻居间建立的是邻接关系，而DR Other之间建立的只是邻居关系。OSPF邻居状态的详细描述，请参见OSPF邻居状态机。

## 网络中无法选举出BDR时

如果在R2、R3、R4的接口GE1/0/1上执行ospf dr-priority命令将接口的DR优先级配置为0，那么这个时候这三台路由器将失去DR和BDR的选举资格，只能作为DR Other，网络中仅存在一台具备DR和BDR选举资格的路由器，就是R1。

在R1上查看OSPF邻居信息。

可以看到，此时DR是R1，BDR显示为None，即网络中不存在BDR。

```shell
<R1> display ospf peer

         OSPF Process 1 with Router ID 10.1.1.1
                 Neighbors

 Area 0.0.0.0 interface 192.168.1.1(GigabitEthernet1/0/1)'s neighbors
 Router ID: 10.2.2.2         Address: 192.168.1.2
   State: Full  Mode:Nbr is Master  Priority: 0
   DR: 192.168.1.1  BDR: None   MTU: 0
   Dead timer due in 38  sec
   Retrans timer interval: 5
   Neighbor is up for 00:04:31
   Authentication Sequence: [ 0 ]

 Router ID: 10.3.3.3         Address: 192.168.1.3
   State: Full  Mode:Nbr is Master   Priority: 0
   DR: 192.168.1.1  BDR: None   MTU: 0
   Dead timer due in 35  sec
   Retrans timer interval: 5
   Neighbor is up for 00:03:45
   Authentication Sequence: [ 0 ]

 Router ID: 10.4.4.4         Address: 192.168.1.4
   State: Full  Mode:Nbr is Master   Priority: 0
   DR: 192.168.1.1  BDR: None   MTU: 0
   Dead timer due in 33  sec
   Retrans timer interval: 5
   Neighbor is up for 00:03:36
   Authentication Sequence: [ 0 ]
```

在R1、R2、R3和R4上查看OSPF邻居的概要信息。

可以看出，R2、R3、R4分别和R1建立了邻接关系（状态为FULL），而R2、R3、R4之间的邻居状态只停留在2-Way的状态。

```shell
<R1> display ospf 1 peer brief

         OSPF Process 1 with Router ID 10.1.1.1
                   Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id         Interface                  Neighbor id      State
 0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         Full
 0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         Full
 0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         Full
 ----------------------------------------------------------------------------
 Total Peer(s):     3

<R2> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.2.2.2
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         Full
0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3

<R3> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.3.3.3
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         Full
0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3

<R4> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.4.4.4
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         Full
0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3
``` 

由此可知，如果在一个广播网络或NBMA网络上只有一台路由器具有选举资格，那么这台路由器将成为DR，并且在这个网络上没有BDR，其他所有的路由器都将只和DR建立邻接关系。

## 网络中无法选举出DR和BDR时

在以上配置的基础上，如果在R1的接口GE1/0/1上执行ospf dr-priority命令将接口的DR优先级配置为0，则R1也失去DR、BDR的选举资格。此时该网络中将没有任何路由器具备DR和BDR的选举资格。

在R1上查看OSPF邻居信息。

```shell
<R1> display ospf peer

         OSPF Process 1 with Router ID 10.1.1.1
                 Neighbors

 Area 0.0.0.0 interface 192.168.1.1(GigabitEthernet1/0/1)'s neighbors
 Router ID: 10.2.2.2         Address: 192.168.1.2
   State: Full  Mode:Nbr is Master  Priority: 0
   DR: None  BDR: None   MTU: 0
   Dead timer due in 38  sec
   Retrans timer interval: 5
   Neighbor is up for 00:00:00
   Authentication Sequence: [ 0 ]

 Router ID: 10.3.3.3         Address: 192.168.1.3
   State: Full  Mode:Nbr is Master   Priority: 0
   DR: None  BDR: None   MTU: 0
   Dead timer due in 35  sec
   Retrans timer interval: 5
   Neighbor is up for 00:00:00
   Authentication Sequence: [ 0 ]

 Router ID: 10.4.4.4         Address: 192.168.1.4
   State: Full  Mode:Nbr is Master   Priority: 0
   DR: None  BDR: None   MTU: 0
   Dead timer due in 33  sec
   Retrans timer interval: 5
   Neighbor is up for 00:00:00
   Authentication Sequence: [ 0 ]
```

在R1、R2、R3和R4上查看OSPF邻居的概要信息。

可以看出，此时所有的邻居状态都只停留在2-Way的状态，网络不能建立邻接关系，各个路由器之间不能完成路由信息的交互。

```shell
<R1> display ospf 1 peer brief

         OSPF Process 1 with Router ID 10.1.1.1
                   Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id         Interface                  Neighbor id      State
 0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         2-Way
 0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         2-Way
 0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         2-Way
 ----------------------------------------------------------------------------
 Total Peer(s):     3

<R2> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.2.2.2
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3

<R3> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.3.3.3
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.4.4.4         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3

<R4> display ospf 1 peer brief

        OSPF Process 1 with Router ID 10.4.4.4
                  Peer Statistic Information
----------------------------------------------------------------------------
Area Id         Interface                  Neighbor id      State
0.0.0.0         GigabitEthernet1/0/1       10.1.1.1         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.2.2.2         2-Way
0.0.0.0         GigabitEthernet1/0/1       10.3.3.3         2-Way
----------------------------------------------------------------------------
Total Peer(s):     3
```

由此可知，如果在一个广播网络或NBMA网络上不存在具备DR和BDR选举资格的路由器，那么这个网络上将没有DR或者BDR，而且也不会建立任何邻接关系。这种情况下，网络上所有路由器的邻居状态都将停留在2-Way状态。

# 7. OSPF状态机

## 接口状态机

OSPF设备从接口获取链路信息后，与相邻设备建立邻接关系，交互这些信息。在建立邻接关系之前，邻居设备间需要明确角色分工才能正常建立连接。OSPF接口信息的State字段（可通过display ospf interface命令查看）表明了OSPF设备在对应链路中的作用。

## OSPF接口共有以下七种状态：
- Down：接口的初始状态。表明此时接口不可用，不能用于收发流量。
- Loopback：设备到网络的接口处于环回状态。环回接口不能用于正常的数据传输，但可以通过Router-LSA进行通告。因此，进行连通性测试时能够发现到达这个接口的路径。
- Waiting：设备正在判定网络上的DR和BDR。在设备参与DR和BDR选举前，接口上会启动Waiting定时器。在这个定时器超时前，设备发送的Hello报文不包含DR和BDR信息，设备不能被选举为DR或BDR。这样可以避免不必要地改变链路中已存在的DR和BDR。仅NMBA网络、广播网络有此状态。
- P-2-P：接口连接到物理点对点网络或者是虚拟链路，这个时候设备会与链路连接的另一端设备建立邻接关系。仅P2P、P2MP网络有此状态。
- DROther：设备没有被选为DR或BDR，但连接到广播网络或NBMA网络上的其他设备被选举为DR。它会与DR和BDR建立邻接关系。
- BDR：设备是相连的网络中的BDR，并将在当前的DR失效时成为DR。该设备与接入该网络的所有其他设备建立邻接关系。DR：设备是相连的网络中的DR。该设备与接入该网络的所有其他设备建立邻接关系。

OSPF接口根据不同的情况（即输入事件）在各状态中进行灵活转换，这样就形成了一个高效运作的接口状态机，如图13所示。

[![ppOtp80.jpg](https://s1.ax1x.com/2023/04/11/ppOtp80.jpg)](https://imgse.com/i/ppOtp80)

<center>图13 OSPF接口状态机</center>

## 邻居状态机

在OSPF网络中，相邻设备间通过不同的邻居状态切换，最终可以形成完全的邻接关系，完成LSA信息的交互。

OSPF邻居信息的State字段（可通过display ospf peer命令查看）表明了OSPF设备的邻居状态。

## OSPF邻居共有以下八种状态：

- Down：邻居会话的初始阶段。

表明没有在邻居失效时间间隔内收到来自邻居设备的Hello报文。

除了NBMA网络OSPF路由器会每隔PollInterval时间对外轮询发送Hello报文，包括向处于Down状态的邻居路由器（即失效的邻居路由器）发送之外，其他网络是不会向失效的邻居路由器发送Hello报文的。

- Attempt：这种状态适用于NBMA网络，邻居路由器是手工配置的。

邻居关系处于本状态时，路由器会每隔HelloInterval时间向自己手工配置的邻居发送Hello报文，尝试建立邻居关系。

- Init：本状态表示已经收到了邻居的Hello报文，但是对端并没有收到本端发送的Hello报文，收到的Hello报文的邻居列表并没有包含本端的Router ID，双向通信仍然没有建立。
- 2-Way：互为邻居。本状态表示双方互相收到了对端发送的Hello报文，报文中的邻居列表也包含本端的Router ID，邻居关系建立。如果不形成邻接关系则邻居状态机就停留在此状态，否则进入ExStart状态。

DR和BDR只有在邻居状态处于2-Way及之后的状态才会被选举出来。

- ExStart：协商主从关系。建立主从关系主要是为了保证在后续的DD报文交换中能够有序的发送。邻居间从此时才开始正式建立邻接关系。
- Exchange：交换DD报文。本端设备将本地的LSDB用DD报文来描述，并发给邻居设备。
- Loading：正在同步LSDB。两端设备发送LSR报文向邻居请求对方的LSA，同步LSDB。
- Full：建立邻接。两端设备的LSDB已同步，本端设备和邻居设备建立了完全的邻接关系。

OSPF邻居状态的切换如图14所示。

[![ppOUkN9.jpg](https://s1.ax1x.com/2023/04/11/ppOUkN9.jpg)](https://imgse.com/i/ppOUkN9)

<center>图14 OSPF邻居状态机</center>

# 8. OSPF邻接关系的建立

## 邻居关系

使能OSPF功能的接口会周期性地发送Hello报文，与网络中其他收到Hello报文的路由器协商报文中的指定参数，包括区域号、验证模式、发送Hello报文的时间间隔、路由器失效时间等参数。

如果协商一致，则在返回的Hello报文的邻居列表中添加发送该Hello报文的设备的Router ID，双方建立双向通信，邻居关系建立。

邻居关系建立后，如果在路由器失效时间内没有收到邻居发来的Hello报文，则中断邻居关系。

## 邻接关系

OSPF邻接关系位于邻居关系之上，两端需要进一步交换DD报文、交互LSA信息时才建立邻接关系。

并非所有邻居都会建立邻接关系，是否建立邻接关系主要取决网络类型和DR/BDR。

在P2P链路和P2MP链路上，每一台设备都需要交换LSA信息，因此只存在邻接关系。

在广播链路和NBMA链路上，因为DR Other之间不需要交换LSA信息，所以建立的是邻居关系。

而DR与BDR之间，DR、BDR与DR Other之间需要交互LSA信息，所以建立的是邻接关系。

如图1所示，两台DR Other各有三个邻居，但是分别只有两个邻接。

## 邻接关系建立过程

不同类型的网络，OSPF邻接关系建立过程不同。

## 广播网络

在广播网络中，DR、BDR和网段内的每一台路由器都形成邻接关系，但DR other之间只形成邻居关系。

邻接关系建立的过程如图15所示。

[![ppOUUu8.jpg](https://s1.ax1x.com/2023/04/11/ppOUUu8.jpg)](https://imgse.com/i/ppOUUu8)

<center>图15 广播网络中邻接关系的建立过程</center>

## 建立邻居关系

- RouterA连接到广播类型网络的接口上使能了OSPF协议，并发送了一个Hello报文（使用组播地址224.0.0.5）。

此时，RouterA认为自己是DR设备（DR=1.1.1.1），但不确定邻居是哪台设备（Neighbors Seen=0）。

- RouterB收到RouterA发送的Hello报文后，发送一个Hello报文回应给RouterA，并且在报文中的Neighbors Seen字段中填入RouterA的Router ID（Neighbors Seen=1.1.1.1），表示已收到RouterA的Hello报文，并且宣告DR设备是RouterB（DR=2.2.2.2），然后RouterB的邻居状态机置为Init。

- RouterA收到RouterB回应的Hello报文后，将邻居状态机置为2-Way状态，下一步双方开始发送各自的链路状态数据库。

在广播网络中，两个接口状态是DR Other的设备之间将停留在此步骤。

## 主从关系协商、DD报文交换

- RouterA首先发送一个DD报文，宣称自己是Master（即将DD报文中的MS字段置为1），并规定序列号Seq=X。I=1表示这是第一个DD报文，报文中并不包含LSA的摘要，只是为了协商主从关系。M=1说明这不是最后一个报文。

- 为了提高发送的效率，RouterA和RouterB首先了解对端数据库中哪些LSA是需要更新的。

如果某一条LSA在LSDB中已经存在，就不再需要请求更新了。

为了达到这个目的，RouterA和RouterB先发送DD报文，DD报文中包含了对LSDB中LSA的摘要描述（每一条摘要可以唯一标识一条LSA）。

为了保证报文在传输过程中的可靠性，在DD报文的发送过程中需要确定双方的主从关系，作为Master的一方定义一个序列号Seq，每发送一个新的DD报文将Seq加1，作为Slave的一方，每次发送DD报文时使用接收到的上一个Master的DD报文中的Seq。

- RouterB在收到RouterA的DD报文后，将邻居状态机改为ExStart，并且回应一个DD报文（该报文中同样不包含LSA的摘要信息）。

由于RouterB的Router ID较大，所以在报文中RouterB认为自己是Master，并且重新规定了序列号Seq=Y。

- RouterA收到报文后，同意了RouterB为Master，并将邻居状态机改为Exchange。

RouterA使用RouterB的序列号Seq=Y来发送新的DD报文，该报文开始正式传送LSA的摘要。

在报文中RouterA将MS字段置为0，说明自己是Slave。

- RouterB收到报文后，将邻居状态机改为Exchange，并发送新的DD报文来描述自己的LSA摘要，此时RouterB将报文的序列号改为Seq=Y+1。

上述过程持续进行，RouterA通过重复RouterB的序列号来确认已收到RouterB的报文。

RouterB通过将序列号Seq加1来确认已收到RouterA的报文。当RouterB发送最后一个DD报文时，在报文中写上M=0。

## LSDB同步（LSA请求、LSA传输、LSA应答）

- RouterA收到最后一个DD报文后，发现RouterB的数据库中有许多LSA是自己没有的，将邻居状态机改为Loading状态。

此时RouterB也收到了RouterA的最后一个DD报文，但RouterA的LSA，RouterB都已经有了，不需要再请求，所以直接将RouterA的邻居状态机改为Full状态。

- RouterA发送LSR报文向RouterB请求更新LSA。

RouterB用LSU报文来回应RouterA的请求。RouterA收到后，发送LSAck报文确认。

## NBMA网络

在NBMA网络中，所有路由器只与DR和BDR之间形成邻接关系。邻接关系建立的过程如图16所示。

[![ppOUsCn.jpg](https://s1.ax1x.com/2023/04/11/ppOUsCn.jpg)](https://imgse.com/i/ppOUsCn)

<center>图16 NBMA网络中邻接关系的建立过程</center>

建立邻居关系

- RouterB向RouterA的一个状态为Down的接口发送Hello报文后，RouterB的邻居状态机置为Attempt。

此时，RouterB认为自己是DR设备（DR=2.2.2.2），但不确定邻居是哪台设备（Neighbors Seen=0）。

- RouterA收到Hello报文后将邻居状态机置为Init，然后再回复一个Hello报文。

此时，RouterA同意RouterB是DR设备（DR=2.2.2.2），并且在Neighbors Seen字段中填入邻居设备的Router ID（Neighbors Seen=2.2.2.2）。

在NBMA网络中，两个接口状态是DR Other的设备之间将停留在此步骤。

主从关系协商、DD报文交换过程与广播网络中邻接关系建立过程相同。

LSDB同步（LSA请求、LSA传输、LSA应答）过程与广播网络中邻接关系建立过程相同。

## 点到点网络和点到多点网络

在点到点/点到多点网络中，邻接关系的建立过程和广播网络一样，唯一不同的是不需要选举DR和BDR，DD报文在点到点网络中是由组播发送的，在点到多点网络中是由单播发送的。

# 9. OSPF区域

## 简介

随着网络规模日益扩大，当一个大型网络中的路由器都运行OSPF路由协议时，会出现以下问题：

- 网络拓扑发生变化概率增大，LSA泛洪严重，降低网络带宽利用率。
- 路由器数量增多，LSDB庞大，占用大量存储空间，并使得运行SPF算法的复杂度增加。
- 每台路由器需要维护的路由表越来越大。

OSPF协议通过将自治系统划分成不同的区域，将LSA泛洪限制在一个区域内，提高网络的利用率和路由的收敛速率；

每个区域内的路由器数量减少，维护的LSDB规模降低，SPF计算也仅限于区域内的LSA；每台路由器需要维护的路由表也越来越小。此外，多区域提高了网络的扩展性，有利于组建大规模的网络。

区域是从逻辑上将路由器划分为不同的组，每个组用区域号（Area ID）来标识。区域的边界是路由器，而不是链路。一个网段（链路）只能属于一个区域，或者说每个运行OSPF的接口必须指明属于哪一个区域。

> 在了解OSPF区域之前，需要先了解与区域相关的两个概念：路由器类型和路由类型。

## 路由器类型

OSPF协议中常用到的路由器类型如图17所示。

[![ppOU6g0.jpg](https://s1.ax1x.com/2023/04/11/ppOU6g0.jpg)](https://imgse.com/i/ppOU6g0)

<center>图17 路由器类型</center>

## 路由类型

AS区域内和区域间路由描述的是AS内部的网络结构。

AS外部路由则描述了应该如何选择到AS以外目的地址的路由。

OSPF将引入的AS外部路由分为Type1和Type2两类。

表中按优先级从高到低顺序列出了路由类型。

## 区域类型

OSPF的区域类型包括普通区域、Stub区域、NSSA区域。

## 普通区域

缺省情况下，OSPF区域被定义为普通区域。普通区域包括：

- 标准区域：最通用的区域，它传输区域内路由，区域间路由和外部路由。
- 骨干区域：连接所有其他OSPF区域的中央区域，用Area 0表示。

骨干区域负责区域之间的路由，非骨干区域之间的路由信息必须通过骨干区域来转发。

说明：

> 骨干区域自身必须保持连通。
> 所有非骨干区域必须与骨干区域保持连通。

## Stub区域

Stub区域是一些特定的区域，Stub区域的ABR不传播它们接收到的自治系统外部路由，因此这些区域中路由器的路由表规模以及路由信息传递的数量都会大大减少。

Stub区域是一种可选的配置属性，但并不是每个区域都符合配置的条件。

一般情况下，Stub区域位于自治系统的边界，是只有一个ABR的非骨干区域，为保证到自治系统外的路由依旧可达，Stub区域的ABR将生成一条缺省路由，并发布给Stub区域中的其他非ABR路由器。

Totally Stub区域允许ABR发布Type3缺省路由，不允许发布自治系统外部路由和区域间的路由，只允许发布区域内路由。

说明：

> 骨干区域不能配置成Stub区域。
> 如果要将一个区域配置成Stub区域，则该区域中的所有路由器都要配置Stub区域属性。
> Stub区域内不能存在ASBR，因此自治系统外部的路由不能在本区域内传播。
> 虚连接不能穿过Stub区域。

## NSSA（Not-So-Stubby Area）区域

NSSA是Stub区域的一个变形，它和Stub区域有许多相似的地方。

NSSA区域不允许存在Type5 LSA。

NSSA区域允许引入自治系统外部路由，携带这些外部路由信息的Type7 LSA由NSSA的ASBR产生，仅在本NSSA内传播。

当Type7 LSA到达NSSA的ABR时，由ABR将Type7 LSA转换成Type5 LSA，泛洪到整个OSPF域中。

Totally NSSA区域不允许发布自治系统外部路由和区域间的路由，只允许发布区域内路由。

说明：

> 骨干区域不能配置成NSSA区域。
> 如果要将一个区域配置成NSSA区域，则该区域中的所有路由器都要配置NSSA区域属性。
> NSSA区域的ABR会发布Type7 LSA缺省路由传播到本区域内。
> 所有域间路由都必须通过ABR才能发布。
> 虚连接不能穿过NSSA区域。

OSPF网络划分区域以后，一个区域内参与SPF算法的只有区域内的LSA，其他的区域的LSA不参与本区域的SPF算法。如图18所示，Area 1中的链路质量不好一直处于闪断中，所以Area 1的SPF算法会频繁运算。但是这种影响仅局限在Area 1内，其他区域不会因此而重新进行SPF运算，网络的震荡被限制在一个更小的范围内，提高了网络的稳定性。

[![ppOUWbF.jpg](https://s1.ax1x.com/2023/04/11/ppOUWbF.jpg)](https://imgse.com/i/ppOUWbF)

<center>图18 划分区域后链路震荡的影响范围减小</center>

Stub区域和Totally Stub区域

如图19所示，OSPF划分了Area 0和Area 2，并且Area 0内的ASBR引入了外部路由。通常情况下，为了保证网络的路由可达性，可能把网络的各个角落的路由全都发布进了OSPF。

此时，虽然各路由设备都能够到达网络的各个角落了，但如果网络越来越大，设备越来越多，那么每台设备的路由表项就会越来越大，而维护一个大规模的路由表项是需要消耗很多CPU及内存资源的。特别是对于一些边缘区域，设备性能可能比较低，维护大规模的路由表项会对设备性能带来巨大压力。

[![ppOdsXV.jpg](https://s1.ax1x.com/2023/04/11/ppOdsXV.jpg)](https://imgse.com/i/ppOdsXV)

<center>图19 Stub区域和Totally Stub区域</center>

从网络优化的角度考虑，通常在保证网络可达性的同时应尽量减小路由表项的规模，减少网络中LSA报文的泛洪。Area 2如果作为一个常规区域，那么可能存在Type1、Type2、Type3、Type4、Type5共计5中类型的LSA。对于Area 2中的路由器，无论想到达区域外的哪个网络，都必须首先到达到ABR路由器，也就是说这个时候Area 2中的其他路由器并不需要了解外部网络的细节。

这种情况下，就产生了OSPF的Stub区域。对于Area 2中的路由器来说，其实区域间的明细路由也没必要都了解，仅保留一个出口让Area 2中的路由器的数据包能够出去就足够了，这就产生了OSPF的Totally Stub区域。Totally Stub区域中，既不允许自治系统外部的路由在区域内传播，也不允许区域间路由在区域内传播，这样就进一步减少了区域内LSA的数量。

NSSA区域和Totally NSSA区域

如图20所示，假设Area 2原来作为一个Stub区域运行，但是有个外部网络需要通过Area 2接入到这个OSPF网络，也就是需要将自治系统外部路由引入并传播到整个OSPF自治系统中。此时可以在RouterA上将外部路由注入到OSPF自治系统，但是这样RouterA将成为ASBR，因此，Area 2也就不是Stub区域了。针对这种场景，OSPF定义了NSSA区域。

[![ppOdc0U.jpg](https://s1.ax1x.com/2023/04/11/ppOdc0U.jpg)](https://imgse.com/i/ppOdc0U)

<center>图20 NSSA区域和Totally NSSA区域</center>

相比于Stub区域，NSSA区域能够将自治系统外部路由引入并传播到整个OSPF自治系统中，同时又不会学习来自OSPF网络其它区域的路由。

在NSSA区域中，为保证到自治系统外的路由可达，NSSA区域的ABR将生成一条缺省路由，并发布给NSSA区域中的其他路由器。

在NSSA区域中，可能同时存在多个ABR，为了防止路由环路产生，边界路由器之间不计算对方发布的缺省路由。

一个区域内所有路由器上配置的区域类型必须保持一致。

OSPF在Hello报文中使用N-bit来标识路由器支持的区域类型，区域类型选择不一致的路由器不能建立OSPF邻居关系。

虽然协议有要求，但有些厂商实现时违背了这一原则，在OSPF DD报文中也置位了N-bit。

为了和这些厂商互通，交换机的实现方式是可以通过命令设置N-bit来兼容。

与Totally Stub区域类似，为了进一步减少NSSA区域中LSA的数量，OSPF还定义了Totally NSSA区域。

## OSPF区域间环路及防环方法

OSPF在区域内部运行的是SPF算法，这个算法能够保证区域内部的路由不会成环。

然而划分区域后，区域之间的路由传递实际上是一种类似距离矢量算法的方式，这种方式容易产生环路。

为了避免区域间的环路，OSPF规定直接在两个非骨干区域之间发布路由信息是不允许的，只允许在一个区域内部或者在骨干区域和非骨干区域之间发布路由信息。

因此，每个ABR都必须连接到骨干区域。

假设OSPF允许非骨干区域之间直接传递路由，则可能会导致区域间环路。

如图21所示，骨干区连接到其他网络的路由信息会传递至Area 1。

[![ppOdg7F.jpg](https://s1.ax1x.com/2023/04/11/ppOdg7F.jpg)](https://imgse.com/i/ppOdg7F)

<center>图21 OSPF区域间环路</center>

假设非骨干区之间允许直接传递路由信息，那么这条路由信息最终又被传递回去，形成区域间的路由环路。

为了防止这种区域间环路，OSPF禁止Area 1和Area 3，以及Area 2和Area 3之间直接进行路由交互，而必须通过骨干区域进行路由交互。

这样就能防止区域间环路的产生。

## OSPF缺省路由

缺省路由是指目的地址和掩码都是0的路由。

当设备无精确匹配的路由时，就可以通过缺省路由进行报文转发。

由于OSPF路由的分级管理，Type3缺省路由的优先级高于Type5或Type7路由。

OSPF缺省路由通常应用于下面两种情况：

- 由区域边界路由器（ABR）发布Type3缺省Summary LSA，用来指导区域内设备进行区域之间报文的转发。
- 由自治系统边界路由器（ASBR）发布Type5外部缺省ASE LSA，或者Type7外部缺省NSSA LSA，用来指导自治系统（AS）内设备进行自治系统外报文的转发。

OSPF缺省路由的发布原则如下：

- OSPF路由器只有具有对区域外的出口时，才能够发布缺省路由LSA。
- 如果OSPF路由器已经发布了缺省路由LSA，那么不再学习其它路由器发布的相同类型的缺省路由LSA。

即路由计算时不再计算其它路由器发布的相同类型的缺省路由LSA，但数据库中存有对应LSA。- 外部缺省路由的发布如果要依赖于其它路由，那么被依赖的路由不能是本OSPF路由域内的路由，即不是本进程OSPF学习到的路由。因为外部缺省路由的作用是用于指导报文的域外转发，而本OSPF路由域的路由的下一跳都指向了域内，不能满足指导报文域外转发的要求。

不同区域缺省路由发布原则:

## 普通区域

缺省情况下，普通OSPF区域内的OSPF路由器是不会产生缺省路由的，即使它有缺省路由。

当网络中缺省路由通过其他路由进程产生时，路由器必须将缺省路由通告到整个OSPF自治系统中。

实现方法是在ASBR上手动通过命令进行配置，产生缺省路由。

配置完成后，路由器会产生一个缺省ASE LSA（Type5 LSA），并且通告到整个OSPF自治系统中。

## Stub区域

Stub区域不允许自治系统外部的路由（Type5 LSA）在区域内传播。

区域内的路由器必须通过ABR学到自治系统外部的路由。

实现方法是ABR会自动产生一条缺省的Summary LSA（Type3 LSA）通告到整个Stub区域内。

这样，到达自治系统的外部路由就可以通过ABR到达。

## Totally Stub区域

Totally Stub区域既不允许自治系统外部的路由（Type5 LSA）在区域内传播，也不允许区域间路由（Type3 LSA）在区域内传播。

区域内的路由器必须通过ABR学到自治系统外部和其他区域的路由。

实现方法是配置Totally Stub区域后，ABR会自动产生一条缺省的Summary LSA（Type3 LSA）通告到整个Stub区域内。

这样，到达自治系统外部的路由和其他区域间的路由都可以通过ABR到达。

## NSSA区域

NSSA区域允许引入通过本区域的ASBR到达的少量外部路由，但不允许其他区域的外部路由ASE LSA（Type5 LSA）在区域内传播。即到达自治系统外部的路由只能通过本区域的ASBR到达。

只配置了NSSA区域是不会自动产生缺省路由的。

此时，有两种选择：

- 如果希望到达自治系统外部的路由通过该区域的ASBR到达，而其它外部路由通过其它区域出去。此时，ABR会产生一条Type7 LSA的缺省路由，通告到整个NSSA区域内。这样，除了某少部分路由通过NSSA的ASBR到达，其它路由都可以通过NSSA的ABR到达其它区域的ASBR出去。
- 如果希望所有的外部路由只通过本区域NSSA的ASBR到达。则必须在ASBR上手动通过命令进行配置，使ASBR产生一条缺省的NSSA LSA（Type7 LSA），通告到整个NSSA区域内。这样，所有的外部路由就只能通过本区域NSSA的ASBR到达。

上面两种情况的区别是：

- 在ABR上无论路由表中是否存在缺省路由0.0.0.0，都会产生Type7 LSA的缺省路由。
- 在ASBR上只有当路由表中存在缺省路由0.0.0.0时，才会产生Type7 LSA的缺省路由。

因为缺省路由只是在本NSSA区域内泛洪，并没有泛洪到整个OSPF域中，所以本NSSA区域内的路由器在找不到路由之后可以从该NSSA的ASBR出去，但不能实现其他OSPF域的路由从这个出口出去。Type7 LSA缺省路由不会在ABR上转换成Type5 LSA缺省路由泛洪到整个OSPF域。

## Totally NSSA区域

Totally NSSA区域既不允许其他区域的外部路由ASE LSA（Type5 LSA）在区域内传播，也不允许区域间路由（Type3 LSA）在区域内传播。

区域内的路由器必须通过ABR学到其他区域的路由。

实现方法是配置Totally NSSA区域后，ABR会自动产生一条缺省的Type3 LSA通告到整个NSSA区域内。

这样，其他区域的外部路由和区域间路由都可以通过ABR在区域内传播。

# 10. OSPF LSA类型

## 简介

OSPF网络中划分了不同的区域，每个区域都维护自己独立的LSDB，同时路由器也被定义成不同的类型。封装了路由描述信息的LSA根据路由器的类型也可以分门别类。

图22是一个被划分区域的OSPF网络。R4上配置了静态路由，在R4上将静态路由引入到OSPF进程中。

[![ppOdWtJ.jpg](https://s1.ax1x.com/2023/04/11/ppOdWtJ.jpg)](https://imgse.com/i/ppOdWtJ)

<center>图22 划分区域的OSPF网络</center>

下面结合图22所示的网络介绍各类LSA。

## Router-LSA

Router-LSA是一种最基本的LSA，即Type1 LSA。

OSPF网络里的每一台路由设备都会发布Type1 LSA。

这种类型的LSA用于描述设备的链路状态和开销，在路由器所属的区域内传播。

以R2为例，如图23所示，R2在Area 0、Area 1会分别发布Router-LSA。

[![ppOdfh9.jpg](https://s1.ax1x.com/2023/04/11/ppOdfh9.jpg)](https://imgse.com/i/ppOdfh9)

<center>图23 Type1 Router-LSA</center>

以R2在接口GE1/0/1上泛洪的一条Router-LSA为例，该LSA中包含的信息如图24所示。

[![ppOd4pR.jpg](https://s1.ax1x.com/2023/04/11/ppOd4pR.jpg)](https://imgse.com/i/ppOd4pR)

<center>图24 Router-LSA信息</center>

LSA报文包括LSA头部和LSA信息字段。所有类型的LSA报文，其LSA头部包含的字段都是一样的，唯一不同的是Link State ID字段含义。在LSA头部中，主要关注以下三个字段：

- Link-State Advertisement Type：LSA类型。
- Link State ID：链路状态ID。在Router-LSA中代表始发该LSA的设备的Router ID，这里即是R2自己的Router ID。
- Advertising Router：通告路由器。

Router-LSA的信息字段有三个，用于将自己连接的所有链路的状况以及开销告诉该LSA泛洪区域的其他路由器。

图3所示的LSA描述的信息为：链路类型（Type）为一个传送网络（Transit），DR接口的IP地址（ID）为192.168.23.2，和网络相连的通告路由器接口的IP地址是192.168.23.1（Data），到达该网络的开销（Metric）是1。收到该LSA报文的路由器根据这些链路状态的描生成拓扑。

其中，Link Type有四种类型，并且ID和Data的值会根据Link Type而有不同：

- P2P（点对点）：此时ID表示邻居路由设备的Router ID，Data表示和网络相连的通告路由器接口的IP地址。
- Transit（传送网络）：此时ID表示DR接口的IP地址，Data表示和网络相连的通告路由器接口的IP地址。
- Stub（末梢网络）：此时ID表示IP网络或子网地址，Data表示网络的IP地址或子网掩码。
- Virtual Link（虚链路）：此时ID表示邻居路由设备的Router ID，Data表示通告路由器接口的MIB-II ifIndex值。

## Network-LSA

Network-LSA，也就是Type2 LSA，由DR（Designated Router）产生，描述本网段的链路状态，在所属的区域内传播。

如图25所示，R3向R2发送一条Network-LSA，列出了所有与DR形成完全邻接关系的路由器的Router ID。

[![ppOd511.jpg](https://s1.ax1x.com/2023/04/11/ppOd511.jpg)](https://imgse.com/i/ppOd511)

<center>图25 Type2 Network-LSA</center>

该Network-LSA中包含的信息如图26所示。

[![ppOdoX6.jpg](https://s1.ax1x.com/2023/04/11/ppOdoX6.jpg)](https://imgse.com/i/ppOdoX6)

<center>图26 Network-LSA信息</center>

在Network-LSA中，Link State ID字段的含义是DR接口上的IP地址。

通过Router-LSA和Network-LSA在区域内洪泛，区域内每个路由器可以完成LSDB同步，这就解决了区域内部的通信问题。

## Network-summary-LSA

Network-summary-LSA，也叫Type3 LSA，由ABR发布，用来描述区域间的路由信息。

ABR将Network-summary-LSA发布到一个区域，通告该区域到其他区域的目的地址。

实际上，ABR是将区域内部的Type1和Type2的信息收集起来并汇总之后扩散出去，这就是Summary的含义。

如图27所示，R2作为ABR，将Area 0和Area 1中的路由信息分别发布对方区域。

[![ppOdoX6.jpg](https://s1.ax1x.com/2023/04/11/ppOdoX6.jpg)](https://imgse.com/i/ppOdoX6)

<center>图27 Type3 Network-summary-LSA</center>

如图28所示，是R2在接口GE1/0/1上发布的一条Network-summary-LSA。

[![ppOdx1I.jpg](https://s1.ax1x.com/2023/04/11/ppOdx1I.jpg)](https://imgse.com/i/ppOdx1I)

<center>图28 Network-summary-LSA信息</center>

在Network-summary-LSA中，Link State ID字段代表该LSA所描述网络的网络地址。从LSA的信息中可以看出，该LSA由R2发布（10.2.2.2），可以到达192.168.12.0，掩码为255.255.255.0的网络，代价为1。R2将Area 1中的网络地址在Area 0中发布，从而让Area 0中的路由器知道去该网络的路径，实现区域间的通信。

如果—台ABR在与它本身相连的区域内有多条路由可以到达目的地，那么它将只会始发单一的一条网络汇总LSA到骨干区域，而且这条网络汇总LSA是上述多条路由中代价最低的。

Network-summary-LSA不会通告给Totally Stub和Totally NSSA区域。

## ASBR-Summary-LSA

ASBR-summary-LSA，也叫Type4 LSA，由ABR发布，描述到ASBR的路由信息，并通告给除ASBR所在区域的其他相关区域。

如图29所示，R3作为ABR通告ASBR-summary-LSA到Area 0中。

[![ppOwSjP.jpg](https://s1.ax1x.com/2023/04/11/ppOwSjP.jpg)](https://imgse.com/i/ppOwSjP)

<center>图29 Type4 ASBR-summary-LSA</center>

ASBR-summary-LSA信息如图30所示。其中，Link State ID表示该LSA所描述的ASBR的Router ID（10.4.4.4），即R4，发布该LSA的路由设备是R3（10.3.3.3），R3到达R4的代价是1。

[![ppOwFAg.jpg](https://s1.ax1x.com/2023/04/11/ppOwFAg.jpg)](https://imgse.com/i/ppOwFAg)

<center>图30 ASBR-summary-LSA信息</center>

## AS-external-LSA

AS-external-LSA，也叫Type5 LSA，由ASBR产生，描述到AS外部的路由，通告到除Stub区域和NSSA区域以外所有的区域。

如图31所示，R4作为ASBR发布了一条OSPF AS到外部目的网络的路由信息。

[![ppOwAhj.jpg](https://s1.ax1x.com/2023/04/11/ppOwAhj.jpg)](https://imgse.com/i/ppOwAhj)

<center>图31 Type5 AS-external-LSA</center>

AS-external-LSA中包含的信息如图32所示。其中，Link State ID代表外部网络目的IP地址，转发地址是指到达该外部网络的数据包应该被转发到的地址。此处的转发地址为0.0.0.0表示数据包将被转发到始发ASBR上。

[![ppOwmj0.jpg](https://s1.ax1x.com/2023/04/11/ppOwmj0.jpg)](https://imgse.com/i/ppOwmj0)

<center>图32 AS-external-LSA信息</center>

## NSSA LSA

除了上述几种LSA之外，还有一种比较特殊的LSA，NSSA LSA，也叫Type7 LSA。NSSA LSA由ASBR产生，描述到AS外部的路由，仅在NSSA区域内传播。NSSA区域的ABR收到NSSA LSA时，会有选择地将其转化为Type5 LSA，以便将外部路由信息通告到OSPF网络的其它区域。

如果图22中的Area 2为NSSA区域，R4的接口GE1/0/2会始发一条NSSA LSA，如图33所示。

[![ppOwsCd.jpg](https://s1.ax1x.com/2023/04/11/ppOwsCd.jpg)](https://imgse.com/i/ppOwsCd)

<center>图33 NSSA LSA</center>

NSSA LSA所有的字段与AS-external-LSA字段均相同，但这两种LSA泛洪的区域不同。AS-external-LSA是在整个AS泛洪，而NSSA LSA仅在NSSA区域中泛洪。

NSSA区域允许引入外部路由，但描述外部路由信息的NSSA LSA只能在本区域泛洪。为了使外部路由能被引入到除NSSA区域以外的其他区域，NSSA LSA在ABR（R3）上会转换成AS-external-LSA，并且泛洪到骨干区直至整个自治系统中。

- P-bit（Propagate bit）用于告知转化路由器该条Type7 LSA是否需要转化。
- 缺省情况下，转化路由器是NSSA区域中Router ID最大的ABR。
- 只有P-bit置位并且FA（Forwarding Address）不为0的NSSA LSA才能转化为AS-external-LSA。FA用来表示发送的某个目的地址的报文将被转发到FA所指定的地址。
- 区域边界路由器产生的NSSA LSA缺省路由不会置位P-bit。

## Opaque LSA

Opaque LSA包括Type9 LSA，Type10 LSA和Type11 LSA，用于OSPF的扩展通用机制。

- Type9 LSA仅在接口所在网段范围内传播。用于支持GR的Grace LSA就是Type9 LSA的一种。
- Type10 LSA在区域内传播。用于支持TE的LSA就是Type10 LSA的一种。
- Type11 LSA在自治系统内传播，目前还没有实际应用的例子。

LSA在各区域中传播的支持情况如表所示。

## 表LSA在各区域中传播的支持情况

[![ppOwcvt.jpg](https://s1.ax1x.com/2023/04/11/ppOwcvt.jpg)](https://imgse.com/i/ppOwcvt)

---END---
