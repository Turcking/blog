---
layout: post
title: SummerP0044 幸运区间 题解
date: 2026-07-16
Author: Turcking
categories: 
tags: [acm]
comments: false
toc: false
---

先看[题目](https://ac.njxzu.cn/p/SummerP0044 "SummerP0044 幸运区间 题目链接")，说给定由数字构成的长度为 $n$ 的字符串 $s$，求 $s$ 中构成的十进制整数为 $7$ 的倍数的子串的数量。

拆分问题，最终答案就是以每个位置结尾的符合要求的子串的数量之和。而以每个位置结尾的子串的十进制整数，都是上个位置结尾的子串的十进制整数乘上十加上这个位置的数，还有一个这个位置的数。

对于一个数 $a$，设 $b = a \% 7$，如果 $\left ( a * 10 + c \right ) \% 7 = 0$，则 $\left ( b * 10 + c \right ) \% 7 = 0$，也就是说，对于某个位置结尾的符合要求的子串（$a * 10 + c$）的个数，我们不需要把它上个位置结尾的子串的数（$a$）全部遍历一遍，只需要算出它们中模 $7$ 的结果（$b$）带上这一位后模 $7$ 结果（$\left ( b * 10 + c \right ) \% 7$）为零的个数，所以需要算将上个位置的数模 $7$ 后的结果（$b$）的个数。为了方便后面位置的计算，我们还需要把模 $7$ 后其余结果的个数也存起来。

```py
T = int(input())
for i in range(T):
	n = int(input())
	s = input()
	result = 0
	left = [0] * 7  # 上个位置模 7 结果为下标的个数
	for i in s:
		new_left = [0] * 7  # 这个位置模 7 结果为下标的个数
		new_left[int(i) % 7] += 1  # 以这个位置开头的子串
		for j in range(7):
			new_left[(j * 10 + int(i)) % 7] += left[j]  # 对于所有上个位置模 7 结果为 j 的子串，这个位置模 7 的结果为 (j * 10 + i) % 7
		left = new_left
		result += new_left[0]  # 将各个位置模 7 为零的子串的个数加起来，就是最终答案
	print(result)
```

但是这个代码以 python3.12 交上去还是 t 了，有没有什么地方可以优化呢？

当然有了，$7$ 这个数很神奇，对于每一个数字 $b$ 和结果 $c$，都有唯一的 $a$ 始得 $\left ( a * 10 + b \right ) \% 7 = c$，其中 $a, c \in \left [0, 7 \right )$。

我用了以下代码生成这张表，第一维是 $b$，第二维是 $c$，值是 $a$：

```py
c_to_a = []
for i in range(10):
	c = [0] * 7
	for j in range(7):
		c[(j * 10 + i) % 7] = j
	c_to_a.append(c)

for i in c_to_a:
	print(i)
```

然后代码：

```py
c_to_a = [
	[0, 5, 3, 1, 6, 4, 2],
	[2, 0, 5, 3, 1, 6, 4],
	[4, 2, 0, 5, 3, 1, 6],
	[6, 4, 2, 0, 5, 3, 1],
	[1, 6, 4, 2, 0, 5, 3],
	[3, 1, 6, 4, 2, 0, 5],
	[5, 3, 1, 6, 4, 2, 0],
	[0, 5, 3, 1, 6, 4, 2],
	[2, 0, 5, 3, 1, 6, 4],
	[4, 2, 0, 5, 3, 1, 6]
]

T = int(input())
for i in range(T):
	n = int(input())
	s = input()
	result = 0
	left = [0] * 7
	for i in s:
		int_i = ord(i) - 48
		left = [left[c_to_a[int_i][i]] for i in range(7)]  # 这个位置的 c 就是上个位置的 a
		left[int_i % 7] += 1  # 以这个位置开头的子串
		result += left[0]
	print(result)
```

很快啊，峰值时间只有不到 $800$ 毫秒。
