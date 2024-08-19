---
layout: post
title: 一些 qemu 参数
date: 2024-08-15
Author: Turcking
categories: 
tags: [qemu, unix]
comments: false
toc: false
---

我发现网上一搜 qemu 就一堆 libvirt 或者不是乱七八糟就是完全看不懂的文章~~（我写得也很乱七八糟）~~，libvirt 虽然对于很多人来讲应该用起来很方便，但是我觉得它这种结构不适合放到一个个人 PC 里面。

现在我就是把 qemu 的启动命令写到文件里面，要运行虚拟机就运行这个文件。把磁盘镜像和这个脚本建一个目录放一起，想改什么设置、启动顺序之类的直接改脚本就行了。

这就是一个实例：

```shell
qemu-system-x86_64 -machine q35,hmat=on,vmport=on \
	-cpu host,vmx -enable-kvm -smp 4 -m 4G \
	-boot c \
	-vga virtio -display gtk,gl=on \
	-hda archlinux.qcow2 \
	-cdrom ~/isos/archlinux-2022.11.01-x86_64.iso
#	-net nic -net user,hostfwd=tcp::8080-:8080 \
#	-hdb archlinux2.qcow2 \
```

所有参数就像配置文件一样列出来，修改起来非常方便。为了造福自己，在这里把我研究总结的要点以及常用参数分享一下。

- `-machine` 虚拟机类型以及属性

	主要配置虚拟机类型，比如 i440fx 而不是 q35。你可以通过 `-machine help` 列出可用虚拟机。其次就是虚拟机属性，比如可以打开 ACPI 的 HMAT。

	按理来讲这类参数我不会在这介绍，但应该是之前遇到过什么问题（哪天想到或者遇到就拿出来说说），导致我现在所有虚拟机都设置为 `q35,hmat=on,vmport=on`，用到现在没什么问题，那应该是足够满足大多数环境了。

- `-cpu` CPU 类型

	可以修改虚拟的 CPU 型号，通过 `-cpu help` 查询可用型号及参数。

	我使用的是 `host`，拥有所有宿主机特性，为了使用 kvm 我还加上了 `vmx` 功能。

- `-enable-kvm`

	不用多说，需要使用 kvm 就加上它。

- `smp` CPU 数量，核心数

	一般来讲不会放多个 CPU，我就直接写想要的核心数了：`-smp 4` 就是一颗 4 核的 CPU。

- `-m` 内存

	这个也简单，`-m 4G` 就是 4G 内存，默认 128MiB。

- `-boot` 启动项

	a 与 b 是软盘，c 是硬盘，d 是光驱，n-p 网络启动。默认是硬盘，可以加上 `menu=on` 在开机时按 ESC 显示菜单。

- `-vga` 显卡

	一看名字还以为是 vga 的什么参数，实际上是显卡。它能选 cirrus, std, vmware, qxl, tcx, cg3, virtio 和 none。

	之前看人家用 qemu 虚拟机分辨率能跟着窗口变，看了参数好像就是这个参数的事，但是又看不懂，后来就选了个 virtio 用。

- `-display` 显示类型

	就是使用什么作为前端，可以使用 spice-app, vnc 等，或者不要显示，配置为 none，这里我使用 gtk，并加上 `gl=on` 使用 OpenGL 显示 。

- `-hda` 硬盘

	可以从 `-hda` 到 `-hdd`，更多的可以看看 `-drive`。硬盘镜像，直接写要用的硬盘精心的路径即可。

- `-cdrom` 光驱

	顾名思义，与上面硬盘一样直接写光盘镜像的路径即可。也可以使用 `/dev/cdrom` 使用物理机的光驱，不过我没试过。

- `-net` 网络

	这个就厉害了，我没搞懂，但是我知道这样写： `-net nic -net user,hostfwd=tcp::8080-:8080` 可以将主机 8080 端口映射到外部 8080，并且哪个是虚拟机哪个是宿主机我也不知道。
