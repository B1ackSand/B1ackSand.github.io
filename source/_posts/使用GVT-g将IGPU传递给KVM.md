---
title: 使用GVT-g将IGPU传递给KVM
date: 2021-10-08 03:42:11
tags: [KVM, Linux]
categories: Linux
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/10/08/使用GVT-g将IGPU传递给KVM/
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者
---

# WIP

# 简介

## 动机

在使用Linux系统时候，由于生态问题，偶尔也要使用到Windows系统。当需要使用显卡加速进行一些工作时（例如简单的P图，galgame等），通常情况下使用Win + Linux双系统是最简单的解决方案。当然，重新启动PC再从grub选择Windows Boot Manager是一件很麻烦的事情。所以，如果我们可以在虚拟机中获得显卡加速，那岂不是一件很美好的事情？

Intel对这种情况有一个解决方案—— GVT-g：

> Intel GVT-g is a full GPU virtualization solution with mediated pass-through (VFIO mediated device framework based), starting from 5th generation Intel Core(TM) processors with Intel Graphics processors. GVT-g supports both Xen and KVM (a.k.a XenGT & a.k.a KVMGT). A virtual GPU instance is maintained for each VM, with part of performance critical resources directly assigned. The capability of running native graphics driver inside a VM, without hypervisor intervention in performance critical paths, achieves a good balance among performance, feature, and sharing capability.
>
> 谷歌机翻：
>
> 英特尔 GVT-g 是具有中介传递（基于 VFIO 中介设备框架）的完整 GPU 虚拟化解决方案，从带有英特尔图形处理器的第 5 代英特尔酷睿(TM) 处理器开始。 GVT-g 支持 Xen 和 KVM（又名 XenGT & 又名 KVMGT）。 每个VM维护一个虚拟GPU实例，直接分配部分性能关键资源。 在 VM 内运行原生图形驱动程序的能力，无需管理程序干预性能关键路径，实现了性能、特性和共享能力之间的良好平衡。
>
> ——From [GVTg_Setup_Guide](https://github.com/intel/gvt-linux/wiki/GVTg_Setup_Guide#1-introduction)

本文将演示如何设置一个 Linux 上的 KVM 虚拟机，让其成为带有 Intel HD Graphics 的 Windows 环境。我使用的系统为Arch，所以本文也基于Arch进行编写，该文章应当也适用于其他发行版。

本文章稍有特殊，因为我的电脑是笔记本输出主屏+DP外接显示屏输出副屏，没有显卡直连功能，但文章并没有禁用独显或者禁用Linux上的显卡输出，所以单屏幕的PC也可进行参考。

## 硬件和软件配置

**硬件：**

笔记本型号：Lenovo Legion Y7000 2018
中央处理器：Intel Core i5-8300H 2.3 GHz
内存：Kingston DDR4-2,666 8.0 GB + Samsung DDR4-2,666 8.0 GB
核显：Intel HD Graphics 630
独显：NVIDIA GeForce GTX 1050 Ti 4.0 GB
SSD：Kioxia RC10 500.0 GB
机械磁盘：Seagate 2.0 TB SATA3 (5,400RPM)
主板：Lenovo LNVNB161216

![硬件](https://z3.ax1x.com/2021/10/12/5mndRe.jpg)



软件：

系统：Arch Linux x86_64
内核：5.14.8-zen1-1-zen
OpenGL：Mesa 21.2.3，NVIDIA 470.74
DE：Plasma 5.22.5
VM：KWin
声音服务器：Pipewire 0.3.38
虚拟化相关：QEMU emulator version 6.1.0，libvirt version: 7.8.0， virt-manager 3.2.0

![image-20211012133204114](https://i.loli.net/2021/10/12/Eis94VzcYv3JU2Q.png)

## 主要步骤

+ 设置主机以支持GVT-g
+ 创建一个虚拟 ICPU
+ 安装一个包含VirtIO驱动的Windows 10 虚拟机
+ 更改虚拟机xml文件将虚拟ICPU传入虚拟机
+ 通过Looking-Glass和Scream优化使用



# 主机设置

## 准备工作

[下载 Windows 10 镜像文件](https://www.microsoft.com/software-download/windows10ISO)

[下载VirtIO驱动程序文件](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.208-1/virtio-win.iso)



确认主机是否安装了qemu, virt-manager, virsh以及相关依赖：

在Arch上：`sudo pacman -S qemu libvirt edk2-ovmf virt-manager dnsmasq ebtables iptables bridge-utils`

确认qemu, Linux内核以及virt-manager的版本信息：

+ Libvirt 要求最低版本为 4.6.0

  ```bash
  $ libvirtd -v
  libvirtd (libvirt) 7.8.0
  ```

+ qemu 要求最低版本 4.0.0

  ```bash
  $ qemu-system-x86_64 --version
  QEMU emulator version 6.1.0
  Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
  ```

+ Linux 内核要求最低为 4.16

  ```bash
  $ neofetch
  Kernel: 5.14.11-zen1-1-zen
  ```

  

建议将当前用户加入`kvm`和`libvirt`组，启动virt-manager将无需密码：

`sudo usermod $(id -un) -a -G kvm,libvirt`



## 配置引导加载程序参数

以grub为例，使用`sudo vim /etc/default/grub`，编辑其中的`GRUB_CMDLINE_LINUX_DEFAULT`行，在括号中加入`intel_iommu=on iommu=pt i915.enable_gvt=1 i915.enable_guc=0` 

编辑完成保存后，我们将通过重新生成 grub.cfg 并重新启动计算机以更新内核参数。

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

如果不是使用grub启动，请参阅你当前发行版的文档，了解如何修改内核参数。



## 在启动时加载GVT-g模块

创建一个`/etc/modules-load.d/kvm-gvt-g.conf`文件，在其中添加：

```
kvmgt
vfio-iommu-type1
vfio-mdev
```



