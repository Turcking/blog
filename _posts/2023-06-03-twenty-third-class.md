---
layout: post
title: 第二十三课
date: 2023-06-03
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# 华为交换机ACL的配置规则及实例

ACL(访问控制列表)的应用原则：

- 标准ACL，尽量用在靠近目的点
- 扩展ACL，尽量用在靠近源的地方（可以保护带宽和其他资源）
- 方向：在应用时，一定要注意方向

ACL分类
```shell
基本ACL	#范围为2000～2999	可使用IPv4报文的源IP地址、分片标记和时间段信息来定义规则
 
高级ACL	#范围为3000～3999	既可使用IPv4报文的源IP地址，也可使用目的地址、IP优先级、ToS、DSCP、IP协议类型、ICMP类型、TCP源端口/目的端口、UDP源端口/目的端口号等来定义规则
 
二层ACL	#范围为4000～4999	可根据报文的以太网帧头信息来定义规则，如根据源MAC（Media Access Control）地址、目的MAC地址、以太帧协议类型等
 
自定义ACL	范围为5000～5999	#可根据偏移位置和偏移量从报文中提取出一段内容进行匹配
```

场景一：（机房交换机不允许非管理网络ssh登录）

```shell
#创建基于命名的基本acl
acl name ssh-kongzhi 2001
rule 5 permit source 192.168.1.0 0.0.0.255
rule 10 deny
 
#在vty接口应用acl 2001
user-interface vty 0 4
acl 2001 inbound
authentication-mode aaa
protocol inbound all
```

场景二：（如下图，主机一不能ping通http服务器，但是可以访问）

[![pCP9Lg1.png](https://s1.ax1x.com/2023/06/05/pCP9Lg1.png)](https://imgse.com/i/pCP9Lg1)

```shell
#创建高级acl
acl number 3001
rule 5 permit tcp source 192.168.21.11 0 destination 192.168.21.100 0 destination-port eq www
rule 10 deny icmp source 192.168.21.11 0 destination 192.168.21.100 0
 
#在接口的上应用acl
interface GigabitEthernet0/0/2
traffic-filter inbound acl 300
```

场景三：（自反acl的应用，类是于防火墙）

[![pCP9ju6.png](https://s1.ax1x.com/2023/06/05/pCP9ju6.png)](https://imgse.com/i/pCP9ju6)

#目标(1)：在icmp协议上，实现外部网络中，只有smokeping主机能ping通内网，其余外部主机不能ping内网地址，并且要求内网可以ping外网
	
#目标(2)：不让外部网络通过tcp协议连接内部，但是内部可以用tcp连接外部（实际意义不大，但是很形象，凡是设计tcp的应用协议都受控）

#分析(1)：先允许smokeping的主机ping内网，ping包分（去包：echo，回包：echo-raly），在公网出口的入方向拒绝所有ping的去包类型echo，作用与所有主机

#分析(2)：tcp协议，需要建立三次握手，只有第一次中不带ack标志，其余都带有ack，（这表示发起tcp连接的一方第一个包不带ack，根据这个在公网出口入方向进行设置）

```shell
#创建acl
acl name kongzhi 3001  
 rule 5 permit icmp source 6.6.6.6 0 
 rule 10 deny icmp icmp-type echo
 rule 15 permit  tcp tcp-flag ack
 rule 20 deny tcp
 
#应用到公网出口上
interface GigabitEthernet0/0/1
 ip address 4.4.4.4 255.255.255.0 
 traffic-filter inbound acl name kongzhi
 ```
 
 场景四：（基于时间的acl应用）
 
 [![pCPCSED.png](https://s1.ax1x.com/2023/06/05/pCPCSED.png)](https://imgse.com/i/pCPCSED)
 
#目标：教师每天6点到23点，可以上网；学生周一到周五8点半到22点可以上网，周六周日两天任何时刻都上网

#分析：要实现基于时间的，就需要进行划分流量，然后阻断，所以，时间可以看成,教师(jiaoshi)室每天早上0点到6点半不能，以及晚上23点到23点59不能上网;学生(xuesheng)周一到周五的早上0点到8点半，以及晚上22点到23点59不能上网
 
```shell
#第一：配置时间段（由于不支持"23:0 to 6:30"，所以写成下面这种形式）
 time-range jiaoshi-deny 00:00 to 6:30 daily
 time-range jiaoshi-deny 23:00 to 23:59 daily
 time-range xuesheng-deny 00:00 to 8:30 working-day
 time-range xuesheng-deny 22:00 to 0:0 working-day
```

```shell
#第二：创建配置高级acl
acl number 3001
rule deny ip source 192.168.21.0 0.0.0.255 time-range jiaoshi-deny
acl number 3002
rule deny ip source 192.168.22.0 0.0.0.255 time-range xuesheng-deny
```

```shell
#第三：配置流分类（对匹配ACL 3001和3002的报文进行分类）
traffic classifier d_jiaoshi
if-match acl 3001
traffic classifier d_xuesheng
if-match acl 3002
```

```shell
#第四：配置流行为（动作为拒绝报文通过）
traffic behavior d_jiaoshi
 deny
traffic behavior d_xuesheng
 deny
```

```shell
#第五：配置流策略（将流分类d_jiaoshi与流行为d_jiaoshi关联，组成流策略，这里为了方便直接将后面应用到一个接口上，将上面两个对应关系进行了合计）
traffic policy all_deny
 classifier d_jiaoshi behavior d_jiaoshi
 classifier d_xuesheng behavior d_xuesheng
```

```shell
#第六：在接口上应用流策略
interface gigabitethernet 0/0/2
ip address 114.114.114.1 255.255.255.0
traffic-policy all_deny outbound
```
