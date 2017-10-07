---
title: libguestfs 使用经验小结
date: 2017-09-14 21:49:34
category: technology
tags:
  - libguestfs
  - opensource
  - P2V
  - V2V
---

## 概述
libguestfs是一组访问或修改虚拟机镜像文件的工具。本文将重点介绍下V2V与P2V两个工具。

## 介绍
* V2V
> Virt-v2v converts guests from a foreign hypervisor to run on KVM. It can read Linux and Windows guests running on VMware, Xen, Hyper-V and some other hypervisors, and convert them to KVM managed by libvirt, OpenStack, oVirt, Red Hat Virtualisation (RHV) or several other targets.
* P2V
> Virt-p2v converts a physical machine to run virtualized on KVM, managed by libvirt, OpenStack, oVirt, Red Hat Virtualisation (RHV), or one of the other targets supported by virt-v2v.

## 用法
* V2V
V2V提供了很多不同的输入输出模式，用户通过`-i`与`-o`参数指定了输入输出模式后，V2V即可以开始正常的进行转换，使用起来十分方便。[这里](http://libguestfs.org/virt-v2v.1.html)详细介绍了virt-v2v命令的用法。
```
                          ┌────────────┐  ┌─────────> -o null
 -i disk ────────────┐    │            │ ─┘┌───────> -o local
 -i ova  ──────────┐ └──> │ virt-v2v   │ ──┘┌───────> -o qemu
                   └────> │ conversion │ ───┘┌────────────┐
 VMware─>┌────────────┐   │ server     │ ────> -o libvirt │─> KVM
 Xen ───>│ -i libvirt ──> │            │     │  (default) │
 ... ───>│  (default) │   │            │ ──┐ └────────────┘
         └────────────┘   │            │ ─┐└──────> -o glance
 -i libvirtxml ─────────> │            │ ┐└─────────> -o rhv
 -i vmx ────────────────> │            │ └──────────> -o vdsm
                          └────────────┘
```
* P2V
对于P2V来说，它的功能是将物理机转换为运行在KVM上的虚拟机。正常来说，我们不会直接运行virt-p2v命令，而是用一个包含virt-p2v程序的启动盘来启动物理机。[这里](http://libguestfs.org/virt-p2v-make-kickstart.1.html)详细介绍了如何制作这个ISO。
```
                           ┌────────────┐  ┌─────────> -o null
                           │            │ ─┘┌───────> -o local
                           │ virt-v2v   │ ──┘┌───────> -o qemu
                           │ conversion │ ───┘┌────────────┐
                           │ server     │ ────> -o libvirt │─> KVM
┌──────────┐               │            │     │  (default) │
│ Physical │               │            │ ──┐ └────────────┘
│ machine  │---(P2V ISO)─> │            │ ─┐└──────> -o glance
└──────────┘               │            │ ┐└─────────> -o rhv
                           │            │ └──────────> -o vdsm
                           └────────────┘
```

## 安装
* In Fedora or Red Hat Enterprise Linux:<br/>
`sudo yum install libguestfs-tools`
* On Debian/Ubuntu:<br/>
`sudo apt-get install libguestfs-tools`

## 源码获取
libguestfs是一个开源的软件，代码托管在GitHub上。https://github.com/libguestfs/libguestfs

## 源码分析
#### 目录结构（为了更加清晰，这里只列举部分目录）
```
libguestfs
    |
    |-----p2v
    |      |----main.c
    |      |----gui.c
    |      |----conversion.c
    |      |----...
    |
    |-----v2v
    |      |----cmdline.ml
    |      |----v2v.ml
    |      |----convert_linux.ml
    |      |----convert_windows.ml
    |      |----input_XXX.ml
    |      |----output_XXX.ml
    |      |----types.ml
    |      |----...
    |
    |-----...
```
#### 下面是对这些文件功能的解读
1. **main.c**<br/>
P2V客户端(基于GTK)启动文件，包含了一些初始化的内容。
2. **gui.c**<br/>
P2V客户端页面渲染，已经初始赋值。
3. **conversion.c**<br/>
将用户在UI界面上选择的信息与物理机基本属性放置于配置文件，并连接V2Vserver进行转换。
4. **cmdline.ml**<br/>
解析传给virt-v2v命令的参数。
5. **v2v.ml**<br/>
V2V的主方法。主要做了以下几件事：
  - 解析输入内容
  - 生成空白disk image
  - 将source OS的内容进行转换并写入之前创建好的空白disk image
  - 调用目标hypervisor的接口来创建虚拟机
6. **convert_linux.ml**<br/>
转换Linux系统相关的代码，包括修改fstab文件等。
7. **convert_windows.ml**<br/>
转换Windows系统相关的代码，包括对注册表的修改，virtio驱动的拷贝等。
8. **input_XXX.ml**<br/>
所有以*input_*开头的文件，每个分别对应一个`-i`选项，负责解析输入源。
9. **output_XXX.ml**<br/>
所有以*output_*开头的文件，每个分别对应一个`-o`选项，负责调用目标hypervisor的接口，来进行创建disk image，创建虚拟机等操作。

## 常见问题（持续更新）
1. 有的时候，我们会发现对于Windows的系统的转换，在转换成功的虚拟机中，我们会发现**virtio**驱动并没有成功安装。这是因为libguestfs中的virtio驱动签名是RedHat，而对于Windows来说，RedHat的签名是不受信的，所以无法自动安装。遇到这种情况，一般有两种解决方案：
  - 在转换好的虚拟机中，打开设备管理器，选中驱动没有正常安装的设备(如：网卡)，然后手动选择驱动安装即可。（注：在转换的时候，P2V工具已经将相应的virtio驱动拷贝到了虚拟机中）
  - 在转换之前，先将RedHat证书加入Windows受信列表中，P2V即可在转换后的虚拟机首次启动后自动完成virtio驱动的安装。

*****
转载请注明：[Franklin的博客](https://franklinzhang1992.github.io/)
