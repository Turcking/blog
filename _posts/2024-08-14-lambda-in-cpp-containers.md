---
layout: post
title: 在 C++ 的容器中使用 lambda 表达式
date: 2024-08-14
Author: Turcking
categories: 
tags: [cpp, acm]
comments: false
toc: false
---

在网上搜容器的使用，比如 set 自定义排序、去重时，很少看见有用 lambda 的。
就算有介绍，也是使用 `auto` 定义成变量之后再使用，看着就感觉没有把 lambda 的方便发挥到极致~~，虽然最终感觉不考虑维护的情况下或许更快~~。

后来看到有视频介绍打蓝桥尽量不要使用新标准的功能，说是要求解答完全符合 ANSI C++ 标准。经我查证，在第十五届蓝桥杯大赛（个人赛）竞赛大纲中是支持 C++ 11 标准的~~（那不还是老吗）~~。
之前只在 C 语言考试才想过不能用新标准之类的问题，到了大学了觉得一切事物都很新鲜，但现在觉得不都一个样吗。

但秉持着这是我的博客，又不是什么算法竞赛教程，所以这个东西我肯定是要写的。

这里吐槽一下，蓝桥往期的竞赛大纲放哪的，找半天没找到。

# 首先简单介绍下 lambda

简单来说，lambda 就是一个可以作为函数调用的表达式，它可以很方便地写在需要一个函数的地方。

比如这段代码：

```cpp
#include <bits/stdc++.h>

int main(){
	// 定义变量
	std::vector<pair<string, int>> this_is_a_vector;
	// 插入数据
	this_is_a_vector.push_back({"world", 7});
	this_is_a_vector.push_back({"Hello", 3});
	// 排序
	sort(this_is_a_vector.begin(), this_is_a_vector.end(), [](auto &a, auto &b){
		return a.second < b.second;
	});
	// 输出
	for(auto i: this_is_a_vector){
		std::cout << i.first << ' ';
	}
	std::cout << std::endl;
}
```

它会将 `this_is_a_vector` 按元素的第二个元素排序，可以看到在原本放入一个函数的地方写了一个 lambda 表达式。

`[]` 是 lambda 的引导，所有 lambda 表达式都应该有这个部分。里面可以放捕获的声明，比如 `[this_is_a_vector]` 就代表我要在这个表达式中使用 `this_is_a_vector`。
后面 `(auto &a, auto &b)` 与 `{}` 就是参数的声明与主体部分，与普通函数一样。

使用 lambda 大大简化了代码，并且拥有更好的可读性，以及更方便的书写。如果不用 lambda，你将会需要在外部额外定义一个函数。

这里就简单地写一下，看看[微软](https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp)和 [cppreference](https://zh.cppreference.com/w/cpp/language/lambda) 的教程是更好的。

# 在容器内使用 lambda

要在 stl 中的对象中使用 lambda 就与上面 `sort()` 不一样了，比如 [set](https://zh.cppreference.com/w/cpp/container/set)。
它的模板 `template<class Key, class Compare = std::less<Key>, class Allocator = std::allocator<Key>>` 中 `class Compare` 应该放构造函数中要放的比较函数的类型，但是 lambda 是什么类型呢？

1. 函数指针

	不需要捕获的 lambda 表达式可以隐式转换为对应类型的函数指针：

	```cpp
	set<string, bool (*)(string a, string b)> this_is_a_set([](string a, string b){
		return a[1] < b[1];  // 这里使用字符串的第二个字符作比较
	});

	this_is_a_set.insert("world");
	this_is_a_set.insert("Hello");

	for(auto i: this_is_a_set){
		cout << i << ' ';
	}
	cout << endl;
	```

	最后结果为 `Hello world`。

2. `std::function`

	在 cppreference 关于 `std::function` 的[介绍](https://zh.cppreference.com/w/cpp/utility/functional/function)中写到，它的实例能够存储 lambda 表达式。
	其用法需要在它的泛型中说明表达式的类型：返回值和参数类型，参数类型带上或没有参数名是允许的。

	```cpp
	char flag_word = 'm';  // 假设是函数中的一个变量
	std::set<std::string, std::function<bool (std::string, std::string)>> this_is_a_set([&flag_word](std::string a, std::string b){  // 引用捕获外部 flag_word
		return std::abs(flag_word - a[1]) < std::abs(flag_word - b[1]);  // 这里使用字符串第二个字符与 flag_word 的绝对值作比较
	});

	this_is_a_set.insert("Hello");
	this_is_a_set.insert("world");

	for(auto i: this_is_a_set){
		std::cout << i << ' ';
	}
	std::cout << std::endl;
	```

	结果为 `world Hello`。

可以看出，虽然使用 `std::function` 可以使用外部函数的普通变量，但是写法会比使用函数指针多几个词。
并且虽然使用隐式转换的函数指针不能使用外部函数的普通变量，但是静态变量不需要捕获就可以使用。在不需要递归和并发的时候，静态变量可以提供更快的编写速度。
