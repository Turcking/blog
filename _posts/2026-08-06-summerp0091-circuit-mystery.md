---
layout: post
title: SummerP0091 电路谜团 题解
date: 2026-08-06
Author: Turcking
categories: 
tags: [acm]
comments: false
toc: false
---

~~✍✍✍做题一定要看懂题。~~

先看[题目](https://ac.njxzu.cn/p/SummerP0091 "SummerP0091 电路谜团 题目链接")，这是一个交互题，说原先有一棵有 $n$ 个节点的无根树，在经过操作后，有些边从无向边变成了有向边，可以询问从点 $u$ 到点 $v$ 需要经过多少个点（包括起点和终点），如果到不了终点则返回 $-1$，求操作后这棵树的邻接表。

换句话说，对于树上的一条连接点 $u$ 和点 $v$ 的无向边，经过操作后，变成了一条从 $u$ 连向 $v$ 的边。

由于我们可以询问 $2 * 10^4$ 次，而 $n$ 只有不超过 $100$，所以对于每个节点，我们只需要把其余节点都询问一遍，看看到那个节点的路径是否只有起点和终点，也就是起点是否直接连接终点。

```py
n = int(input())
results = [list() for i in range(n)]
for i in range(n):
	for j in range(n):
		if i != j and int(input("? %d %d\n" % (i + 1, j + 1))) == 2:  # 对于 i != j 的节点，如果从 i 出发直接连接到 j，则 i 到 j 只经过两个节点
			results[i].append(str(j + 1))  # j 是正序遍历的，之后输出邻接表时不需要排序了
print("!")
for i in results:
	print(' '.join(i) if i else "null")
```
