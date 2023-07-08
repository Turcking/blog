---
layout: post
title: matplotlib 笔记
date: 2023-07-08
Author: 
categories: 
tags: [python, matplotlib]
comments: false
toc: true
---

> Matplotlib 是 Python 中最流行的绘图库之一，可以用于创建各种类型的静态和动态图表。
> 能将数据进行可视化，更直观的呈现，使数据更加客观、更具说服力。

# 概念

matplotlib 庞大的代码库可能吓到很多人，但只要掌握简单的概念和要点，就能用好 matplotlib。

## 一般概念

绘图需要执行从高级的“画出这些数据的等高线”命令到低级的“这个像素显示红色”命令。绘图软件包的目的是让您轻松地画出图表，并提供所有必要的控制——也就是说，在多数情况下使用高级的命令，但在需要时依旧能够使用低级命令。
所以 matplotlib 中所有内容都按层次结构组织。顶部是 matplotlib “状态机环境”，它由 matplotlib.pyplot 模块提供。在此级别上，可以用简单的函数将线条，图像，文本等绘制到图中的轴上。

下一级是面向对象接口的第一级，此级上 pyplot 只用来创建图形，用户显示并跟踪图形和表。在此级上，用户可以创建图形，并在图形上创建多个表，这些表将用来绘图。

对于更多的控制，例如在 GUI 程序中嵌入 matplotlib 图表，pyplot 级别可能被完全丢弃，留下纯粹的面向对象方法。

## 图形结构

![图形结构](https://matplotlib.org/_images/anatomy.png "图形结构")

### Figure 图形

图形会跟踪所有从其打开的表，画和画布（不用关系画布，虽然一切都是由它绘制，但您并不需要操作它）。
一个图形可以有任意数量个表，但至少拥有一个表来让这个图形有用。

创建图形的简单方法是使用 pyplot:

```python
fig = plt.figure()  # 一个不包含任何表的图形
fig.subtitle("No axes on this figure")  # 添加一个标签来让我们知道这是什么
fig, ax = plt.subplots()  # 一个包含了一个表的图形
fig, axs = plt.subplots(2, 2)  # fig 是一个包含了 2x2 个表的图形，axs 是包含这些表的列表
# 一个左边一个表右边两个表的图形
fig, axs = plt.subplot_mosaic([["left", "right-top"],
	["left", "right_bottom"]])
```

### Axes 表

这就是你真正绘图的区域。一个图形可以包含多个表，一个表只能被包含在一个图形中。
一个表一般包含两个轴（3D 情况下是三个），每个表有一个标题（参见 [set_title()](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_title.html "set_title()")），一个 x 标签（参见 [set_xlabel](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_xlabel.html "set_xlabel()")）和一个 y 标签（参见 [set_ylabel()](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_ylabel.html "set_ylabel()")）。

Axes 类及成员函数主要使用 OOP 接口，并拥有大多数在其上定义的绘图方法，例如 ax.plot():

```python
fig, ax = plt.subplots()  # 创建包含一个表的图形
ax.plot([1, 2, 3, 4], [1, 4, 2, 3])  # 绘制一条线
```

![ax.plot() 演示图](https://matplotlib.org/stable/_images/sphx_glr_quick_start_001.png "ax.plot() 演示")

### Axis 轴

这些是数字线状对象（[原文](https://matplotlib.org.cn/tutorials/introductory/usage.html "原文链接")是 number-line-like object 不知道咋翻译，但是 Rizline!），它们规定了图形的范围和刻度标签（标记刻度的字符串）。
刻度位置由 Locator 对象确定，而刻度标签由 Formatter 对象确定。正确的 Locator 与 Formatter 组合可以非常精细地控制刻度位置和标签。

### Artist 画

基本上，在图形中所有可见内容都是一个画（甚至 Figure, Axes 和 Axis）。
当图形被渲染时，所有画都被绘制到画布上。大多数画都被绑在一根轴上，这样的画不能被多个轴共享，或者从一个轴移动到另一个轴。

# 使用风格

matplotlib 基本上有两种使用方法：

- 显式创建图形和表，并调用它们的方法（ “面向对象风格（OO-style）”）。

```python
x = np.linspace(0, 2, 100)  # 示例数据

# 注意，尽管在面向对象风格中，我们也用 `.pyplot.figure` 进行图形的创建。
fig, ax = plt.subplots(figsize=(5, 2.7), layout='constrained')
ax.plot(x, x, label='linear')  # 在表上绘制一些数据
ax.plot(x, x**2, label='quadratic')  # 在表上绘制更多数据
ax.plot(x, x**3, label='cubic')  # ... 以及更多 ...
ax.set_xlabel('x label')  # 为表添加 x 标签
ax.set_ylabel('y label')  # 为表添加 y 标签
ax.set_title("Simple Plot")  # 为表添加标题
ax.legend()  # 添加一个图例
```

- 依赖 pyplot 隐式地创建和管理图形和表，以及使用 pyplot 函数进行绘图。

```python
x = np.linspace(0, 2, 100)  # 一些数据

plt.figure(figsize=(5, 2.7), layout='constrained')
plt.plot(x, x, label='linear')  # 在（隐式的）表上绘制一些数据
plt.plot(x, x**2, label='quadratic')  # 以及更多
plt.plot(x, x**3, label='cubic')
plt.xlabel('x label')
plt.ylabel('y label')
plt.title("Simple Plot")
plt.legend()
```

- 嵌入在 GUI 应用程序中的 Matplotlib，完全放弃了 pyplot，[这](https://matplotlib.org/stable/gallery/user_interfaces/embedding_in_tk_sgskip.html "matplotlib 嵌入 tkinter 示例")是在 tkinter 中嵌入 matplotlib 的官方示例，请参阅[官方文档](https://matplotlib.org/stable/gallery/user_interfaces/index.html "Embedding Matplotlib in graphical user interfaces")了解更多信息：

```python
import tkinter

from matplotlib.backends.backend_tkagg import (
    FigureCanvasTkAgg, NavigationToolbar2Tk)
# Implement the default Matplotlib key bindings.
from matplotlib.backend_bases import key_press_handler
from matplotlib.figure import Figure

import numpy as np


root = tkinter.Tk()
root.wm_title("Embedding in Tk")

fig = Figure(figsize=(5, 4), dpi=100)
t = np.arange(0, 3, .01)
ax = fig.add_subplot()
line, = ax.plot(t, 2 * np.sin(2 * np.pi * t))
ax.set_xlabel("time [s]")
ax.set_ylabel("f(t)")

canvas = FigureCanvasTkAgg(fig, master=root)  # A tk.DrawingArea.
canvas.draw()

# pack_toolbar=False will make it easier to use a layout manager later on.
toolbar = NavigationToolbar2Tk(canvas, root, pack_toolbar=False)
toolbar.update()

canvas.mpl_connect(
    "key_press_event", lambda event: print(f"you pressed {event.key}"))
canvas.mpl_connect("key_press_event", key_press_handler)

button_quit = tkinter.Button(master=root, text="Quit", command=root.destroy)


def update_frequency(new_val):
    # retrieve frequency
    f = float(new_val)

    # update data
    y = 2 * np.sin(2 * np.pi * f * t)
    line.set_data(t, y)

    # required to update canvas and attached toolbar!
    canvas.draw()


slider_update = tkinter.Scale(root, from_=1, to=5, orient=tkinter.HORIZONTAL,
                              command=update_frequency, label="Frequency [Hz]")

# Packing order is important. Widgets are processed sequentially and if there
# is no space left, because the window is too small, they are not displayed.
# The canvas is rather flexible in its size, so we pack it last which makes
# sure the UI controls are displayed as long as possible.
button_quit.pack(side=tkinter.BOTTOM)
slider_update.pack(side=tkinter.BOTTOM)
toolbar.pack(side=tkinter.BOTTOM, fill=tkinter.X)
canvas.get_tk_widget().pack(side=tkinter.TOP, fill=tkinter.BOTH, expand=True)

tkinter.mainloop()
```

> 注：为了更清晰地体现图形的结构，这篇笔记会使用 OO-style。

# 导入 matplotlib

一般在导入时会因为名字太长而为它起别名：

```python
import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
```

也有人会直接 `from matplotlib.pyplot import *`，但我没有这个习惯，并且考虑到这会导致分不清哪些是自己定义的，哪些是模块里的，所以这篇整理我会使用 `import matplotlib.pyplot`。

# 基本用法

## Hello world!

这是一个类似于 Hello world 的程序。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	matplotlib.pyplot.figure(figsize=(12, 4))
	matplotlib.pyplot.plot([0, 0], [8, 0])
	matplotlib.pyplot.plot([0, 3], [4, 4])
	matplotlib.pyplot.plot([3, 3], [8, 0])
	matplotlib.pyplot.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	matplotlib.pyplot.plot([9, 9], [8, 0])
	matplotlib.pyplot.plot([13, 13], [8, 0])
	matplotlib.pyplot.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	matplotlib.pyplot.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	matplotlib.pyplot.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	matplotlib.pyplot.plot([32.5, 32.5], [4, 0])
	matplotlib.pyplot.plot([32.5, 34.5], [3, 4])
	matplotlib.pyplot.plot([37, 37], [8, 0])
	matplotlib.pyplot.plot([43, 40, 40, 43], [4, 4, 0, 0])
	matplotlib.pyplot.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

## matplotlib.pyplot.figure()

可以把图形想象成一个大窗口，一个窗口就是一个图形。当然，你看到的这个窗口里不只有一个图形，还有一个 toolbar（工具栏）。

### 创建新图形

使用 `matplotlib.pyplot.figure()` 创建一个新图形，让 world 显示在另外一个图形中。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure0 = matplotlib.pyplot.figure(figsize=(12, 4))
	figure0_axes = figure0.subplots()
	figure0_axes.plot([0, 0], [8, 0])
	figure0_axes.plot([0, 3], [4, 4])
	figure0_axes.plot([3, 3], [8, 0])
	figure0_axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	figure0_axes.plot([9, 9], [8, 0])
	figure0_axes.plot([13, 13], [8, 0])
	figure0_axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])

	figure1 = matplotlib.pyplot.figure()
	figure1_axes = figure1.subplots()
	figure1_axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	figure1_axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	figure1_axes.plot([32.5, 32.5], [4, 0])
	figure1_axes.plot([32.5, 34.5], [3, 4])
	figure1_axes.plot([37, 37], [8, 0])
	figure1_axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	figure1_axes.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

### 调整图形大小

在调用 `matplotlib.pyplot.figure()` 时加入参数 figsize，它的类型是一个两元素元祖（列表经过测试似乎也可以，但文档里写的是元祖），第一个元素是长度的英尺数，第二个是高度的英尺数。

让我们把 Hello 和 world 的图形大小调整好：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure0 = matplotlib.pyplot.figure(figsize=(8, 4))
	figure0_axes = figure0.subplots()
	figure0_axes.plot([0, 0], [8, 0])
	figure0_axes.plot([0, 3], [4, 4])
	figure0_axes.plot([3, 3], [8, 0])
	figure0_axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	figure0_axes.plot([9, 9], [8, 0])
	figure0_axes.plot([13, 13], [8, 0])
	figure0_axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])

	figure1 = matplotlib.pyplot.figure(figsize(8, 4))
	figure1_axes = figure1.subplots()
	figure1_axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	figure1_axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	figure1_axes.plot([32.5, 32.5], [4, 0])
	figure1_axes.plot([32.5, 34.5], [3, 4])
	figure1_axes.plot([37, 37], [8, 0])
	figure1_axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	figure1_axes.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

### 图形的编号和标题

为了修改这个标题，在调用 `matplotlib.pyplot.figure()` 时填入 num 参数，它可以接收一个数字（似乎小数会被取整，但不是四舍五入）或字符串。数字会设置图形的编号，而字符串会设置图形和窗口的标题。
既然如此，给显示 Hello 的窗口命名为 Hello，显示 world 的命名为 world 吧：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure0 = matplotlib.pyplot.figure(figsize=(8, 4), num="Hello")
	figure0_axes = figure0.subplots()
	figure0_axes.plot([0, 0], [8, 0])
	figure0_axes.plot([0, 3], [4, 4])
	figure0_axes.plot([3, 3], [8, 0])
	figure0_axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	figure0_axes.plot([9, 9], [8, 0])
	figure0_axes.plot([13, 13], [8, 0])
	figure0_axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])

	figure1 = matplotlib.pyplot.figure(figsize=(8, 4), num="world")
	figure1_axes = figure1.subplots()
	figure1_axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	figure1_axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	figure1_axes.plot([32.5, 32.5], [4, 0])
	figure1_axes.plot([32.5, 34.5], [3, 4])
	figure1_axes.plot([37, 37], [8, 0])
	figure1_axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	figure1_axes.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.figure.html "matplotlib.pyplot.figure 文档")

## matplotlib.figure.Figure.subplots()

这个方法可以帮助你在图形中创建表，如果只有一个表，它会直接返回这个表的对象，如果创建了多个，它会返回一个包含这些表对象的列表。

### 创建多个表

在 `Figure.subplots()` 中传入 nrows（行数）和 ncols（列数）参数就能创建多个表了。

```python
#! /usr/bin/env python3

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes0, axes1 = figure.subplots(1, 2)

	axes0.plot([0, 0], [8, 0])
	axes0.plot([0, 3], [4, 4])
	axes0.plot([3, 3], [8, 0])
	axes0.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes0.plot([9, 9], [8, 0])
	axes0.plot([13, 13], [8, 0])
	axes0.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])

	axes1.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes1.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes1.plot([32.5, 32.5], [4, 0])
	axes1.plot([32.5, 34.5], [3, 4])
	axes1.plot([37, 37], [8, 0])
	axes1.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes1.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

更多请参阅[官方文档](https://matplotlib.org/stable/api/figure_api.html#matplotlib.figure.Figure.subplots "Figure.subplots()")

## matplotlib.axes.Axes.plot()

这是函数可以画线。

### 画线

`Axes.plot()` 可以绘制一条不间断的折线，所以一般也使用这个函数来做折线统计图。它传入的第一个参数是包含要绘制的点的所有 x 坐标的列表，以此类推，第二个参数是包含它们 y 坐标的列表。
如果不传入 x 的坐标而只传 y 的，`Axes.plot()` 会从 x = 0 开始绘制这些点。

画一个逗号：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])

	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0])  # 这是逗号

	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

### 更粗，更大

在调用 `Axes.plot()` 时传入 linewidth 或者 lw 参数可以给画的线加粗，它需要一个浮点类型的变量（官方文档上只写了 float，但是按理来说整形也可以）。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])

	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)  # 这是逗号

	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])

	matplotlib.pyplot.show()

```

### 我的形状

在调用 `Axes.plot()` 时传入 linestyle 或 ls 就能改变线型，这是官方文档提供的可用线型：

|值|描述|
|---|---|
|"-" 或者 "solid"|实线|
|"--" 或者 "dashed"|虚线（段）|
|"-." 或者 "dashdot"|点画线|
|":" 或者 "dotted"|虚线（点）|
|"none", "None", " " 或者 ""|什么都不画|

当然也可以自己定义线型，~~但是我不会，~~还是建议看[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.lines.Line2D.html#matplotlib.lines.Line2D.set_linestyle "linestyle")

画一个叹号：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])

	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22)  # 这是一个叹号

	matplotlib.pyplot.show()

```

### 变色

虽然不知道不同环境默认颜色会不会不同，但是我这给叹号的颜色是褐色的，在这样一个五彩斑斓的画中这么大一坨褐色不太好看。
color 或者 c 参数可以指定绘制的颜色，可以传这些缩写的值：

|字符|颜色|
|---|---|
|'b'|blue|
|'g'|green|
|'r'|red|
|'c'|cyan|
|'m'|magenta|
|'y'|yellow|
|'k'|black|
|'w'|white|

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])

	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	matplotlib.pyplot.show()

```

更多内容参阅[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.plot.html#matplotlib.axes.Axes.plot "Axes.plot()")

## matplotlib.axes.Axes.legend()

你可以给你的表添加图例。

### 画线时就添加标签

在调用 `matplotlib.axes.Axes.plot()` 时可以传入 label 参数，之后调用 `matplotlib.axes.Axes.legend()` 就能显示各线的图例了。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8, label="comma")  # 这是逗号
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen", label="exclamation mark")  # 这是叹号

	axes.legend()

	matplotlib.pyplot.show()

```

也可以获取画出的线的句柄传给 `Axes.legend()`，它就只会绘制传去的线的图例。
可以把句柄想象成指针，但 python 中实际上传过去的就是对象。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8, label="comma")  # 这是逗号
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	line, = axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen", label="exclamation mark")  # 这是叹号

	axes.legend(handles = [line])  # 只画叹号的图例

	matplotlib.pyplot.show()

```

### 画线后添加标签

在画完线后，可以在调用 `Axes.legend()` 时传入 label 参数，它会按序把标签对应到线上。
此时在画线时传入的 label 参数就不再有用了。
这种方法 label 参数名可以不用写。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8, label="comma")  # 这是逗号
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen", label="exclamation mark")  # 这是叹号

	axes.legend(["l", "-", "l", "e", "l", "l", "o", ","])  # 把 Hello 的各线给标出来了，逗号的标签被覆盖，叹号没有显示图例

	matplotlib.pyplot.show()

```

也可以在传入 label 参数时同时传入 handles 参数，它会把 label 里的标签按序对应上 handles 里的句柄，此时画线时传入的 label 参数也没用了。
可以把 handles 参数写在 label 参数之前，这样就不用写参数名了。

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	lines = []
	lines.extend(axes.plot([0, 0], [8, 0]))
	lines.extend(axes.plot([0, 3], [4, 4]))
	lines.extend(axes.plot([3, 3], [8, 0]))
	lines.extend(axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0]))
	lines.extend(axes.plot([9, 9], [8, 0]))
	lines.extend(axes.plot([13, 13], [8, 0]))
	lines.extend(axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4]))
	lines.extend(axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8, label="comma"))  # 这是逗号
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen", label="exclamation mark")  # 这是叹号

	axes.legend(lines, ["l", "-", "l", "e", "l", "l", "o", ","])  # 把 Hello 的各线给标出来了，逗号的标签被覆盖，叹号没有显示图例

	matplotlib.pyplot.show()

```

[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.legend.html "matplotlib.axes.Axes.legend")

## matplotlib.axes.Axes.set_xlim()/.set_ylim()

设置 x 与 y 轴视图限制。

### 设置 x 轴的视图限制

使用 `Axes.set_xlim()` 可以设置 x 轴的视图限制。第一个参数为左侧的坐标，第二个为右侧的坐标，也可以传入一个包含左侧坐标和右侧坐标的元祖。
只设置一侧的坐标也是可行的。左侧的坐标设置时不用写参数名 left，因为它是这个方法的第一个参数。右侧的参数名是 right，这是第二个参数，如果只设置右侧的，需要写上参数名或第一个参数写 None。

这是一个例子，它将只显示 Hello：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlim(-1, 20)  # 只显示 x∈(-1,20) 的内容

	matplotlib.pyplot.show()

```

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_xlim.html "matplotlib.axes.Axes.set_xlim()")

### 设置 y 轴的视图限制

`Axes.set_ylim()` 与 x 一样，只不过第一个参数是 bottom，第二个是 top。

现在，你只能看到 Hello, world! 的脚底了：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_ylim(-7.5, 0.5)

	matplotlib.pyplot.show()

```

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_ylim.html "matplotlib.axes.Axes.set_ylim()")

## matplotlib.axes.Axes.set_xlabel()/.set_ylabel()/.set_label()

它们可以设置轴标签与标题。

### 设置 x 轴标签

`Axes.set_xlabel()` 可以传一个字符串作为轴标签：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlabel("This is x's label!")

	matplotlib.pyplot.show()

```

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_xlabel.html "matplotlib.axes.Axes.set_xlabel()")

### 设置 y 轴标签

`Axes.set_ylabel()` 与上一个相同：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlabel("This is x's label!")
	axes.set_ylabel("This is y's label!")

	matplotlib.pyplot.show()

```

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_ylabel.html "matplotlib.axes.Axes.set_ylabel()")

## 设置表标题

`Axes.set_title()` 同样：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlabel("This is x's label!")
	axes.set_ylabel("This is y's label!")
	axes.set_title("This is an axes!")

	matplotlib.pyplot.show()

```

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_title.html "matplotlib.axes.Axes.set_title()")

## matplotlib.axes.Axes.set_xticks()/.set_yticks()

### 设置 x 刻度位置和可选标签

使用 `Axes.set_xticks` 可以设置显示的刻度与这些刻度的标签。它的第一个参数是要显示刻度的列表，第二个参数是这些刻度的标签。
当需要设置标签时，第二个参数的长度需要与第一个参数的长度相同

注意：如有必要，它将改变“轴”的视图限制，以便所有给定标签都可见：

```python
#! /usr/bin/env python3

import matplotlib.pyplot

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlim(-1, 20)
	axes.set_xticks([ i-0.5 for i in range(2, 48, 4)], "Hello,world!")

	matplotlib.pyplot.show()

```

这个演示中，表不会因为设置了 x 的视图限制而只显示 Hello，因为在下面的 `Axes.set_xticks()` 中包含了 `!` 所在的刻度，导致右侧视图限制拓到了 `!` 上。
如果还要只显示 Hello，就需要在设置了刻度标签后再设置视图限制。

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_xticks.html "Axes.set_xticks()")

### 设置 x 刻度位置和可选标签

`Axes.set_yticks()` 与 `Axes.set_xticks()` 相同，只是坐标轴从 x 变到了 y，这就不再演示了。

[文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.set_yticks.html "Axes.set_yticks()")

## matplotlib.axis.Axis.set_major_formatter()

设置一个轴的刻度标签的格式化程序。

### 以刻度进行格式化

`Axis.set_major_formatter()` 接收一个函数，这个函数会被传入两个参数：x 和 pos，参数 x 是该刻度在所在坐标轴上标识的值，是一个浮点数。参数 pos 是该刻度在所在坐标轴的所有显示的刻度中的从 1 开始的索引值，是一个整数。
对于参数 x 的使用，一个直观的例子，让 x 轴显示时间：

```python
#! /usr/bin/env python3

import matplotlib.pyplot, time

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	start_time = time.time()  # 获取初始时间
	axes.xaxis.set_major_formatter(  # 设置 x 轴的刻度标签的格式化函数
		lambda x, pos:(
			"%d:%d"%(time.gmtime(start_time + x)[4:6])  # 刻度代表的时间的分和秒
		)
	)

	matplotlib.pyplot.show()

```

### 以索引值进行格式化

对于参数 pos 的使用，没找到什么好例子，就直接输出吧：

```python
#! /usr/bin/env python3

import matplotlib.pyplot, time

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.xaxis.set_major_formatter(  # 设置 x 轴的刻度标签的格式化函数
		lambda x, pos:(
			"%d"%(pos,) if pos != None else "None"  # 刻度的索引值
		)
	)

	matplotlib.pyplot.show()

```

可以发现，右下角显示的坐标也受这个函数的影响，并且在格式化类似右下角显示的坐标时传入的参数 pos 为 None。

[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axis.Axis.set_major_formatter.html "matplotlib.axis.Axis.set_major_formatter()")

## matplotlib.axes.Axes.grid()

配置网格线。

### 显示网格线

一般来说，在不显示网格线时，直接调用此方法会显示网格线，但是这样调用的意思是切换网格线的显示状态。也就是说，当网格线显示时，直接调用该方法将不会显示网格线。
可以在调用时传入 visible 参数，该参数默认值是 None，在调用该方法时，如果 visible 参数值是 None 并且没有给除 which 和 axis 参数的其它参数时，它会切换网格线的显示状态。
可以给 visible 参数传入一个布尔值来指定网格线的显示状态：

```python
#! /usr/bin/env python3

import matplotlib.pyplot, time

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	start_time = time.time()
	axes.xaxis.set_major_formatter(
		lambda x, pos:(
			"%d:%d"%(time.gmtime(start_time + x)[4:6])
		)
	)

	axes.grid(visible=True)  # 显示网格线

	matplotlib.pyplot.show()

```

### 显示单个轴的网格线

可以给 axis 参数传入 "x", "y" 或 "both" 来指定操作的是哪个坐标轴。
只显示 x 轴的网格线：

```python
#! /usr/bin/env python3

import matplotlib.pyplot, time

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(12, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")
	start_time = time.time()
	axes.xaxis.set_major_formatter(
		lambda x, pos:(
			"%d:%d"%(time.gmtime(start_time + x)[4:6])
		)
	)

	axes.grid(visible=False, axis="y")  # 关闭 y 轴的
	axes.grid(visible=True, axis="x")  # 显示 x 轴的

	matplotlib.pyplot.show()

```

[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.grid.html "matplotlib.axes.Axes.grid()")

## matplotlib.lines.Line2D.set()

画出去的线，是可以修改的。

### 修改数据

可以在调用 `Line2D.set()` 时传入 data 来修改一条线表示的数据，比如把叹号改成问号：

```python
#! /usr/bin/env python3

import matplotlib.pyplot, time

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(14, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	line, = axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")  # 这是叹号

	line.set(data=([44, 47, 47, 45.5, 45.5], [7.5, 7.5, 4, 4, 0]), linewidth=29)  # 改成问号

	axes.set_xlim(right=50)  # 显示不全，拓宽一点
	matplotlib.pyplot.show()

```

调用 `Line2D.set()` 时传入 linewidth 也可以修改线宽。

使用 `Line2D.set_data()` 修改数据会更方便：

```python
#! /usr/bin/env python3

import matplotlib.pyplot, time

if __name__ == "__main__":

	figure = matplotlib.pyplot.figure(figsize=(14, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	line, = axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")  # 这是叹号

	line.set_data([44, 47, 47, 45.5, 45.5], [7.5, 7.5, 4, 4, 0])  # 更方便的方法
	line.set(linewidth=29)  # 此时问号不能正常显示，要改一下线宽

	axes.set_xlim(right=50)
	matplotlib.pyplot.show()

```

### 修改其它属性

还有很多很方便的方法可以用来修改线的属性，但都可以使用 `Line2D.set()` 方法进行修改，这里列出一点：

|方便的方法|set() 的参数名|属性内容|
|---|---|---|
|set_c()/set_color()|c/color|颜色|
|set_data()/set_xdata()/set_ydata()|data/xdata/ydata|数据|
|set_ls()/set_linestyle()|ls/linestyle|线型|
|set_lw()/set_linewidth()|lw/linewidth|线宽|

[官方文档](https://matplotlib.org/stable/api/_as_gen/matplotlib.lines.Line2D.html#matplotlib.lines.Line2D.set "matplotlib.lines.Line2D.set()")

## 显示中文

matplotlib 默认没法显示中文，因为默认的英文字体无法显示汉字。
在 \*nix 中，可以使用 `fc-list` 查看支持的字体，用 `fc-list :lang=zh` 查看支持中文的字体。
有两种方式可以修改 matplotlib 的字体：

- 通过 matplotlib.rc。

```python
#! /usr/bin/env python3

import matplotlib.pyplot, matplotlib

if __name__ == "__main__":

	matplotlib.rc("font", family="unifont")  # 设置字体

	figure = matplotlib.pyplot.figure(figsize=(14, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlabel("Hello world")

	matplotlib.pyplot.show()
```

- 通过 matplotlib 的 font_manager。

```python
#! /usr/bin/env python3

import matplotlib.pyplot, matplotlib.font_manager

if __name__ == "__main__":

	font = matplotlib.font_manager.FontProperties(family="Unifont")  # 创建一个字体属性（我觉得应该理解成上下文？）

	figure = matplotlib.pyplot.figure(figsize=(14, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	axes.set_xlabel("Hello world", fontproperties=font)  # 使用字体属性

	matplotlib.pyplot.show()
```

> 注：虽然好像很多都推荐使用第二种办法，但经过研究，第二种办法并不是所有显示文字的地方都能用，或者要查文档才能知道要怎么用。
> 第一种办法的优点是全局设置，所有显示文字的地方在不单独设置时都会用它。但是缺点是不能通过设置 `fname` 来指定字体文件，而第二种方法可以。
>
> 在我的环境中，会出现安装的字体找不到的问题，这是因为它会在家目录的 .matplotlib 文件夹中创建一个文件，这个文件中存放了字体的索引。
> 这个索引并不会增加字体项，但是会修改有问题的字体项。
> 有两种办法解决这个问题：
> - 删除或移走这个文件：
> 使用这个办法，如果在原来的文件里设置过什么，就需要重新设置了，它会重新创建一个新的文件，包含了默认设置和字体索引。
> - 增加需要的字体项：
> 这个文件的结构一看就懂，所以就不说怎么增加了，需要注意的一点是字体文件可以不写，它会自己从系统中安装的字体里寻找，并在运行时帮你写上去。


# 嵌入 GUI

## Tkinter

这是一段演示：

```python
#! /usr/bin/env python3

import tkinter, matplotlib.backends.backend_tkagg, matplotlib.figure

if __name__ == "__main__":

	root = tkinter.Tk()
	root.title("This is a test window.")

	figure = matplotlib.figure.Figure(figsize=(14, 4))
	axes = figure.subplots()
	axes.plot([0, 0], [8, 0])
	axes.plot([0, 3], [4, 4])
	axes.plot([3, 3], [8, 0])
	axes.plot([4, 7, 7, 4, 4, 7], [2, 2, 4, 4, 0, 0])
	axes.plot([9, 9], [8, 0])
	axes.plot([13, 13], [8, 0])
	axes.plot([19, 16, 16, 19, 19], [4, 4, 0, 0, 4])
	axes.plot([21.5, 21, 21, 21.5, 21.5, 21], [0.5, 0.5, 1, 1, 0.5, 0], linewidth=8)
	axes.plot([24, 25, 25.5, 26, 27], [4, 0, 4, 0, 4])
	axes.plot([31, 28, 28, 31, 31], [4, 4, 0, 0, 4])
	axes.plot([32.5, 32.5], [4, 0])
	axes.plot([32.5, 34.5], [3, 4])
	axes.plot([37, 37], [8, 0])
	axes.plot([43, 40, 40, 43], [4, 4, 0, 0])
	axes.plot([43, 43], [8, 0])
	axes.plot([45, 45], [8, 0], linestyle="dashdot", linewidth=22, color="lightgreen")

	canvas = matplotlib.backends.backend_tkagg.FigureCanvasTkAgg(figure, master=root)
	canvas.draw()
	canvas.get_tk_widget().pack(fill="both", expand=True)
	root.mainloop()

```

### 创建窗口

使用前，得先有个窗口才能在上面使用 matplotlib。
演示中，使用了 `root = tkinter.Tk()` 创建了一个窗口。

### 创建图形

演示中，首先使用 `matplotlib.figure.Figure()` 创建了一个图形 `figure`。
有了这个图形，就可以和先前使用 `matplotlib.pyplot` 一样绘制了，在此不再多述。

### 创建画布

演示中，在图形上画完数据后，使用了 `matplotlib.backends.backend_tkagg.FigureCanvasTkAgg(figure, master=root)` 创建了一个画布 `canvas`。
可以说这个画布是建立在 matplotlib 与 tkinter 的桥梁。

### 渲染画布

在图形中绘制完后，需要渲染画布才能让绘制的内容显示。
你可以把它想象成缓存区，你绘制的内容会存入缓存区中，只有渲染画布后，缓存区的内容才会被刷到画布上。
演示中使用了 `FigureCanvasTkAgg.draw()` 进行渲染画布。

### 布局画布

在创建了 `master=root` 的画布后，还需要为画布布局（调整位置）才能让它在 Tkinter 中显示。
在使用 pack() 之前，需要使用 `FigureCanvasTkAgg.get_tk_widget()` 获取给 Tkinter 用的小组件。
帮助文档是这样解释的：

>get_tk_widget(self)
>    Return the Tk widget used to implement FigureCanvasTkAgg.
>    返回用于实现 FigureCanvasTkAgg 的 Tk 小部件。
>
>    Although the initial implementation uses a Tk canvas,  this routine
>    is intended to hide that fact.
>    尽管最初的实现使用了 Tk 画布，但这个例程旨在隐藏这一事实。

### 进入循环

最后肯定是要进入 Tkinter 的循环了。但在进入循环前，还可以对窗口进行一些调整，在此不再多述。

### 另一个演示

```python
import tkinter

from matplotlib.backends.backend_tkagg import (
    FigureCanvasTkAgg, NavigationToolbar2Tk)
# Implement the default Matplotlib key bindings.
from matplotlib.backend_bases import key_press_handler
from matplotlib.figure import Figure

import numpy as np


root = tkinter.Tk()
root.wm_title("Embedding in Tk")

fig = Figure(figsize=(5, 4), dpi=100)
t = np.arange(0, 3, .01)
ax = fig.add_subplot()
line, = ax.plot(t, 2 * np.sin(2 * np.pi * t))
ax.set_xlabel("time [s]")
ax.set_ylabel("f(t)")

canvas = FigureCanvasTkAgg(fig, master=root)  # A tk.DrawingArea.
canvas.draw()

# pack_toolbar=False will make it easier to use a layout manager later on.
toolbar = NavigationToolbar2Tk(canvas, root, pack_toolbar=False)
toolbar.update()

canvas.mpl_connect(
    "key_press_event", lambda event: print(f"you pressed {event.key}"))
canvas.mpl_connect("key_press_event", key_press_handler)

button_quit = tkinter.Button(master=root, text="Quit", command=root.destroy)


def update_frequency(new_val):
    # retrieve frequency
    f = float(new_val)

    # update data
    y = 2 * np.sin(2 * np.pi * f * t)
    line.set_data(t, y)

    # required to update canvas and attached toolbar!
    canvas.draw()


slider_update = tkinter.Scale(root, from_=1, to=5, orient=tkinter.HORIZONTAL,
                              command=update_frequency, label="Frequency [Hz]")

# Packing order is important. Widgets are processed sequentially and if there
# is no space left, because the window is too small, they are not displayed.
# The canvas is rather flexible in its size, so we pack it last which makes
# sure the UI controls are displayed as long as possible.
button_quit.pack(side=tkinter.BOTTOM)
slider_update.pack(side=tkinter.BOTTOM)
toolbar.pack(side=tkinter.BOTTOM, fill=tkinter.X)
canvas.get_tk_widget().pack(side=tkinter.TOP, fill=tkinter.BOTH, expand=True)

tkinter.mainloop()
```

这是[官网](https://matplotlib.org/stable/gallery/user_interfaces/embedding_in_tk_sgskip.html "嵌入 Tk")的演示，它还创建了 toolbar，监听了键盘事件，使用 `tkinter.Scale()` 来控制 matplotlib 的数据等。

