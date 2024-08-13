---
layout: post
title: 关于 proton 和 podman 的一些使用方式
date: 2024-08-13
Author: Turcking
categories: 
tags: [proton, podman, unix, container]
comments: false
toc: false
---

> 这个文章我至少应该在两年甚至一年半（0.6 坤年）前就可以写了，我真能拖。
> 
> 但也因此开始用了容器之类的隔离环境，可喜可贺可喜可贺。

# 不使用 steam 直接运行 proton

之前想着 steam 不挂点东西启动太慢了，所以想着直接运行 proton。结果根本不会用，所以上网研究了很久。

我看网上有启动脚本，各种各样的根本看不懂，最后看到了一篇文章：[用 steam 的 proton 執行 windows exe](https://gholk.github.io/linux-proton-wine-outside-steam.html "用 steam 的 proton 執行 windows exe") 以及一个项目：[proton-standalone-script](https://github.com/7oxicshadow/proton-standalone-script "proton-standalone-script")，并且结合实践分析得来结果：只需要设置 `STEAM_COMPAT_DATA_PATH` 和 `STEAM_COMPAT_CLIENT_INSTALL_PATH` 就可以启动 proton。

我比较习惯临时用的东西放到 `/tmp` 中，所以一般将这两个环境变量都设为 `/tmp`。
我还很喜欢用 archlinuxcn 的 proton，即使没 archlinuxcn，aur 也有 proton 可以用，它们的 proton 放在 `/usr/share/steam/compatibilitytools.d/proton/` 中。

这是我运行程序时使用的语句，`<command>` 是要运行的指令或程序，`[<arg1> [<arg2> ...]]` 是参数：

```shell
STEAM_COMPAT_DATA_PATH=/tmp \
STEAM_COMPAT_CLIENT_INSTALL_PATH=/tmp \
/usr/share/steam/compatibilitytools.d/proton/proton run \
<command> [<arg1> [<arg2> ...]]
```

如果由 steam 启动，这两个环境变量可能被初始化为 `STEAM_COMPAT_DATA_PATH=~/.local/share/Steam/steamapps/compatdata/1112345678999 STEAM_COMPAT_CLIENT_INSTALL_PATH=~/.local/share/Steam`。
写在这里是用来作参考，毕竟我根本不知道这两个参数干啥用的。
但是我知道，proton 会在 `STEAM_COMPAT_DATA_PATH` 放入 Windows 运行环境。如果游戏把存档之类放在家目录或者其它地方，那么下一次启动可以继续使用这个文件夹，即游戏的配置和存档就不会丢失。

# 使用 podman 运行图形程序

我会研究这个的契机应该是我的电脑从 Ubuntu 换到 Arch Linux 后 Linux QQ NT 变得特别容易闪退。
前期我还能发几句话，回一两张图片，到后来刚登陆就闪退，甚至连主界面都看不见。
所以我想，既然 Ubuntu 里不容易闪退，容器可以创建环境，那为什么不在 Ubuntu 容器里使用 QQ 呢？

这里就拿我最初的起源：QQ 作演示：

```shell
podman run -it --rm --net=host \
	-e DISPLAY \
	-e XAUTHORITY \
	-v $HOME:/root:rw \
	-v /etc/machine-id:/etc/machine-id \
	-v $XAUTHORITY:$XAUTHORITY \
	-v /tmp/.X11-unix:/tmp/.X11-unix:rw \
	docker.nju.edu.cn/longjunyu/linuxqq:latest qq
```

~~悼念所有关停的 docker hub 镜像。~~

要使用图形，主要是 DISPLAY 和 XAUTHORITY 两个环境变量以及 XAUTHORITY 指向的文件和 `/tmp/.X11-unix` 中的连接需要映射到容器中。
这里 DISPLAY 必须和外部一致，因为程序依赖它以在 `/tmp/.X11-unix` 中寻找连接。
容器内的 XAUTHORITY 环境变量需要指向外部 XAUTHORITY 指向的文件映射到容器内对应的那个文件，XAUTHORITY 的文件没有规定路径与名称，程序需要通过该环境变量来找到这个文件。
machine-id 我觉得没什么必要，但我当时还是加进去了~~，可能当时的我是有什么深意吧~~。

目前还不能使用输入法，我拿 QQ 研究过，但是最后没能成功。目前推断是 QQ 连接 dbus 延迟要求太短了，以后有机会我再研究研究。

还有我发现这个 docker.io/longjunyu/linuxqq 的信息都不见了，那正好介绍一下这个镜像（虽然已经一年多没有更新但还能用）：

- 这是一个在容器里使用 Linux QQ 的镜像，它有两个端口 5800(http) 和 5900(vnc)。

- 你可以使用 `podman run -p 5800:5800 -p 5900:5900 docker.io/longjunyu/linuxqq` 启动它，然后你可以使用浏览器访问 `http://localhost:5800/` 或者使用 vnc 客户端访问 `localhost:5900`。

- 对于 VNC，它使用 Xvnc TigerVNC 作为服务端。对于 http，它使用 noVNC 作为服务端。

- 通过浏览器或 VNC 客户端连接后，你会发现背景是黑的，只有一个 QQ 窗口，毕竟这个镜像只用来用 QQ。

- 你可以在里面使用 QQ。

- 如果你关闭了里面的 QQ，容器会直接关闭。
