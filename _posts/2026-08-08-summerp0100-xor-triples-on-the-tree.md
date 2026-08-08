---
layout: post
title: SummerP0100 树上异或三元组 题解
date: 2026-08-08
Author: Turcking
categories: 
tags: [acm]
comments: false
toc: false
---

先看[题目](https://ac.njxzu.cn/p/SummerP0100 "SummerP0100 树上异或三元组 题目链接")，说一棵 $n$ 个带权节点的无根树，求由三个点构成的无序三元组 $\left \\{u, v, w\right \\}$ 的数量，满足三元组中两两点之间构成的路径上所有点权的异或和为零。

注意到对于三元组，只有两种情况：一种是三个点中没有一个点在另外两个点构成的路径上，此时该三元组中两两点之间构成的路径上所有点权的异或和为两两点构成的三条路径中唯一一个公共点的点权，且该公共点不在三元组中；还有一种是其中某个点在另外两个点构成的路径上，此时该点既为三元组中两两点之间构成的三条路径中唯一一个公共点，其两两点之间构成的路径上所有点权的异或和既为该点点权。

由于要求三元组中两两点之间构成的路径上所有点权的异或和为零，不难发现就是求公共点点权为零的三元组的数量。而且不难注意到三元组只有一个公共点，所以不存在多个点权为零的点同时作为一个三元组的公共点。也就是计算每个点权为零的点作为公共点的三元组的数量，只需要对于每个点权为零的点，分别计算其作为公共点的三元组数量，最后加起来就是结果。

对于第一种情况，假设公共点作为整棵树的根，其三元组的数量就是所有选三棵子树后将它们的大小相乘的和，其中 $b$ 是所有子树的大小构成的数组：

$$
\sum_{1 \le i \lt j \lt k \le \left | b \right |} b_i * b_j * b_k
$$

将 $k$ 和 $j$ 提出来：

$$
\sum_{k = 3}^{\left | b \right |} b_k \sum_{j = 2}^{k - 1} b_j \sum_{i = 1}^{j - 1} b_i
$$

对于第二种情况，依旧假设公共点作为整棵树的根，其三元组的数量就是所有选两棵子树后将它们的大小相乘的和：

$$
\sum_{1 \le i \lt j \le \left | b \right |} b_i * b_j
$$

将 $j$ 提出来：

$$
\sum_{j = 2}^{\left | b \right |} b_j * \sum_{i = 1}^{j - 1} b_i
$$

不难发现第二种情况在求第一种情况时就可以顺便求出来了，时间复杂度 $\mathcal O \left ( \left \| b \right \| \right )$。

现在问题只剩如何快速求出所有子树的大小，而这个问题也很好解决。先假设有一个其它节点 $u$ 作为根，求出所有树 $i$ 的大小，记为 $\mathit{size}_i$，然后对于实际需要作为根的节点 $v$，其除了包含 $u$ 的子树 $j$ 的大小为 $\mathit{size}_j$，包含 $u$ 的子树的大小为 $\mathit{size}_u - \mathit{size}_v$。

```py
import sys
sys.setrecursionlimit(4000000)

mod = 998244353

T = int(input())
for i in range(T):
	n = int(input())
	a = list(map(int, input().split()))
	graph = [set() for i in range(n)]
	for i in range(1, n):
		u, v = map(int, input().split())
		graph[u - 1].add(v - 1)
		graph[v - 1].add(u - 1)

	parent = [None] * n
	size = [None] * n

	def dfs(node, node_parent):
		parent[node] = node_parent
		size[node] = 1
		for child in graph[node]:
			if child != node_parent:
				dfs(child, node)
				size[node] += size[child]

	dfs(0, -1)  # 假设以零为根节点

	result = 0
	for i in range(n):
		if a[i] == 0:
			# 为方便计算，假设 b 的第一个子树包含实际作为根节点的零，直接预计算第一次计算
			# 即使此时作为根的节点是零，实际也不过是多往 b 数组中增添一个零而已，并且不难发现无论加多少个零都不影响结果
			pre_sum = [size[0] - size[i], 0, 0]
			for child in graph[i]:
				if child != parent[i]:
					for j in range(2, -1, -1):
						pre_sum[j] = (pre_sum[j] + (pre_sum[j - 1] if j else 1) * size[child] % mod) % mod
			result = (result + pre_sum[1] + pre_sum[2]) % mod
	print(result)
```
