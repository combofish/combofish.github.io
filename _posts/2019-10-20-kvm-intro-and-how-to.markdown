---
layout: post
title: "KVM虚拟化的介绍与使用"
date: 2019-10-20
tag: KVM qemu libvirt virt-manager
---
- [KVM 介绍](#orged841fc)
  - [KVM 架构](#orgba190b5)
  - [KVM 工具集合](#org33db86b)
- [kvm 使用](#org9eb55f5)
  - [查看 CPU 是否支持](#org9ae6394)
  - [查看内核模块是否打开(默认开启)](#org3b97e92)
  - [开启 libvirtd 服务](#orgc6fa72f)
    - [安装中遇到的错误](#org2b04805)
  - [virt-manager](#org90229f2)
    - [虚拟化管理应用程序（包括命令行工具）](#orgb38bd74)
    - [支持工具](#orge48d51f)
    - [参考链接](#org6053ebc)
  - [virt-convert](#orgf201098)
  - [qemu-img](#org5a1690e)
    - [create](#orga26fbae)
    - [convert](#orgad17a85)
    - [check](#org7cfebea)
    - [resize](#org12220f9)
    - [snapshot](#orgdcccfa5)
- [参考链接](#org2cb5b06)


<a id="orged841fc"></a>

# KVM 介绍

-   KVM 全称是 基于内核的虚拟机（Kernel-based Virtual Machine），它是Linux 的一个内核模块，该内核模块使得 Linux 变成了一个 Hypervisor


<a id="orgba190b5"></a>

## KVM 架构

KVM 是基于虚拟化扩展（Intel VT 或者 AMD-V）的 X86 硬件的开源的 Linux 原生的全虚拟化解决方案。KVM 中， 虚拟机被实现为常规的 Linux 进程，由标准 Linux 调度程序进行调度； 虚机的每个虚拟 CPU 被实现为一个常规的 Linux 线程。这使得 KMV 能够使用 Linux 内核的已有功能。 但是，KVM 本身不执行任何硬件模拟，需要用户空间程序通过 /dev/kvm 接口设置一个客户机虚拟服务器的地址空间， 向它提供模拟 I/O，并将它的视频显示映射回宿主的显示屏。目前这个应用程序是 QEMU。

-   Guest：客户机系统，包括CPU（vCPU）、内存、驱动（Console、网卡、I/O 设备驱动等），被 KVM 置于一种受限制的 CPU 模式下运行。
-   KVM：运行在内核空间，提供 CPU 和内存的虚级化，以及客户机的 I/O 拦截。Guest 的 I/O 被 KVM 拦截后，交给 QEMU 处理。
-   QEMU：修改过的被 KVM 虚机使用的 QEMU 代码，运行在用户空间，提供硬件 I/O 虚拟化，通过 IOCTL /dev/kvm 设备和 KVM 交互。


<a id="org33db86b"></a>

## KVM 工具集合

-   libvirt：操作和管理KVM虚机的虚拟化 API，使用 C 语言编写，可以由 Python,Ruby, Perl, PHP, Java 等语言调用。可以操作包括 KVM，vmware，XEN，Hyper-v, LXC 等在内的多种 Hypervisor。
-   Virsh：基于 libvirt 的 命令行工具 （CLI）
-   Virt-Manager：基于 libvirt 的 GUI 工具
-   virt-v2v：虚机格式迁移工具
-   virt-\* 工具：包括 Virt-install （创建KVM虚机的命令行工具）， Virt-viewer （连接到虚机屏幕的工具），Virt-clone（虚机克隆工具），virt-top 等
-   sVirt：安全工具


<a id="org9eb55f5"></a>

# kvm 使用


<a id="org9ae6394"></a>

## 查看 CPU 是否支持

grep &#x2013;color -E "vmx|svm" /proc/cpuinfo


<a id="org3b97e92"></a>

## 查看内核模块是否打开(默认开启)

```shell
lsmod | grep kvm 
```


<a id="orgc6fa72f"></a>

## 开启 libvirtd 服务

```shell
systemctl enable libvirtd
systemctl status libvirtd
```

-   启动失败

```shell
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/libvirtd.service.d
           └─00gentoo.conf
   Active: active (running) since Fri 2019-10-18 22:36:22 CST; 1min 26s ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 178703 (libvirtd)
   Memory: 7.0M
      CPU: 741ms
   CGroup: /system.slice/libvirtd.service
           └─178703 /usr/sbin/libvirtd --timeout 120

10月 18 22:36:22 localhost systemd[1]: Starting Virtualization daemon...
10月 18 22:36:22 localhost systemd[1]: Started Virtualization daemon.
10月 18 22:36:22 localhost libvirtd[178703]: libvirt version: 5.6.0
10月 18 22:36:22 localhost libvirtd[178703]: hostname: localhost
10月 18 22:36:22 localhost libvirtd[178703]: direct firewall backend requested, but /sbin/ebtables is not available: No such file or directory
10月 18 22:36:22 localhost libvirtd[178703]: 内部错误：Failed to initialize a valid firewall backend
```

-   解决办法： 安装 ebtables

```shell
emerge -pvt net-firewall/ebtables
```

-   成功启动
    
    ```shell
    systemctl status libvirtd
    ```


<a id="org2b04805"></a>

### 安装中遇到的错误

-   XML::Parser&#x2026; configure: error: XML::Parser perl module is required for intltool
-   解决办法：
    
    ```shell
    cpan > install XML::Parser
    ```

1.  参考链接

    <https://www.linuxquestions.org/questions/linux-from-scratch-13/configure-error-xml-parser-perl-module-is-required-for-intltool-4175578941/>


<a id="org90229f2"></a>

## virt-manager

-   注：在一台机器上的virt-manager可以通过add connection管理其它机器上的虚拟机


<a id="orgb38bd74"></a>

### 虚拟化管理应用程序（包括命令行工具）

| 应用程序     | 描述                |
|------------ |------------------- |
| virt-manager | 虚拟机桌面管理工具  |
| virt-install | 虚拟机配给工具      |
| virt-clone   | 虚拟机映像克隆工具  |
| virt-image   | 从一个 XML 描述符构造虚拟机 |
| virt-viewer  | 虚拟机图形控制台    |
| virsh        | virsh Guest 域的交互式终端 |

virt-manager 使用 libvirt 虚拟化库来管理可用的虚拟机管理程序。 libvirt 公开了一个应用程序编程接口 (API)， 该接口与大量开源虚拟机管理程序相集成，以实现控制和监视。libvirt 提供了一个名为 libvirtd 的守护程序，帮助实施控制和监视


<a id="orge48d51f"></a>

### 支持工具

尽管 virt-manager 是 libvirt 虚拟化 API 的一个首要用户，但有一个越来越庞大的工具生态系统在使用此接口进行虚拟化管理。

-   virt-manager 包提供了一个便捷的 GUI，用来在多个虚拟机管理程序和主机上创建和管理虚拟机。如果更喜欢命令行，那么会由许多工具为您提供只有命令行才能提供的能力和控制力。
-   virt-install 工具提供了配给新虚拟机的能力。virt-manager 提供了少量针对虚拟机创建的配置选项，而 virt-install 提供了丰富的配置选项，包括安装方法、存储配置、网络配置、图形配置、虚拟化选项，以及一个庞大的虚拟化设备选项列表。
-   virt-image 工具类似于 virt-install 工具，但支持您在 XML 中定义虚拟机创建过程的细节。该 XML 描述符文件指定了虚拟机的一般元数据、域属性（CPU、内存等），以及存储配置。
-   virt-clone 工具提供了一种克隆现有的虚拟机映像的方式。提到克隆，我指的是复制现有的虚拟机，该虚拟机具有更新的参数，可确保新虚拟机是唯一的，从而避免发生冲突（比如 MAC 地址冲突）。
-   virt-viewer 工具为一个使用 VNC 协议的给定虚拟机提供了一个图形控制台。virt-viewer 可附加到在本地主机或远程主机上运行的虚拟机。
-   最后，管理 Guest 域的最强大的工具是虚拟化 shell，或者称为 virsh。virsh 可用于列出、启动和停止虚拟机，以及创建虚拟机。简言之，您可使用 virsh 跨虚拟机管理程序执行全面地管理公开在其他工具中未提供的虚拟化特性。


<a id="org6053ebc"></a>

### 参考链接

<https://www.ibm.com/developerworks/cn/cloud/library/cl-managingvms/>

KVM之二：KVM工具简介一：virt-manager，virt-viewer，virt-convert,qemu-img <https://www.jianshu.com/p/b894ca1abd51>


<a id="orgf201098"></a>

## virt-convert

-   virt-convert 可以OVF或VMX文件转换为KVM的支持格式。默认转换是”raw”。这个工具主要是实现v2v,将现用的VM打包并导入新的KVM环境。

```shell
# 命令用法： virt-convert INPUT.vmx|INPUT.ovf|INPUT-DIR|INPUT.zip [OPTIONS]
# 选项：
-i / --input-format [ 输入格式（.vmx, .ovf, .zip） ]
-D / --disk-format [ 输出格式（.raw, .qcow2） ]

--destination [ 输出目录 ]
# 举例：

virt-convert fedora18.ova
virt-convert centos6.vmx --disk-format qcow2
```


<a id="org5a1690e"></a>

## qemu-img

qemu-img command [command options]

-   qemu-img是QEMU的磁盘管理工具，在KVM环境中该工具必不可少。与virt-convert不同，qemu-img是使用在磁盘类型的转换。


<a id="orga26fbae"></a>

### create

用于创建一个格式为fmt大小为size文件名为filename的镜像文件。 根据文件格式fmt的不同，还可以添加一个或多个选项（options）来附加对该文件的各种功能设置， 可以使用“-o ?”来查询某种格式文件支持那些选项，在“-o”选项中各个选项用逗号来分隔。

```shell
qemu-img create [-f fmt] [-o options] filename [size]
-f  [raw|qcow2] :创建的格式
-o backing_file=/path/[file_name],size=[file_size] :可选择后端镜像
# size [K,M,G,T,P] :镜像大小
```


<a id="orgad17a85"></a>

### convert

```shell
qemu-img convert [-c] [-f fmt] [-O output_fmt] [-o options] filename [filename2 […]] output_filename
#支持的格式完全能满足你的需求：vvfat vpc vmdk vdi sheepdog rbd raw host_cdrom host_floppy host_device file qed qcow2 qcow parallels nbd dmg tftp ftps ftp https http cow cloop bochs blkverify blkdebug

-c：采用压缩，只有qcow和qcow2才支持
-f：源镜像的格式，它会自动检测，所以省略之
-O 目标镜像的格式
-o 其他选项
filename：源文件
out_filename:转化后的文件
```


<a id="org7cfebea"></a>

### check

对磁盘镜像文件进行一致性检查，查找镜像文件中的错误，目前仅支持对“qcow2”、“qed”、“vdi”格式文件的检查。

```shell
qemu-img check [-f fmt] filename
```


<a id="org12220f9"></a>

### resize

改变镜像文件的大小，使其不同于创建之时的大小。“+”和“-”分别表示增加和减少镜像文件的大小， 而size也是支持K、M、G、T等单位的使用。缩小镜像的大小之前，需要在客户机中保证里面的文件系统有空余空间， 否则会数据丢失，另外，qcow2格式文件不支持缩小镜像的操作。在增加了镜像文件大小后， 也需启动客户机到里面去应用“fdisk”、“parted”等分区工具进行相应的操作才能真正让客户机使用到增加后的镜像空间。 不过使用resize命令时需要小心（最好做好备份），如果失败的话，可能会导致镜像文件无法正常使用而造成数据丢失。 注意：只有raw格式的镜像才可以改变大小。

```shell
qemu-img resize filename [+ | -]size
```


<a id="orgdcccfa5"></a>

### snapshot

```shell
Snapshot [-l | -a snapshot | -c snapshot | -d snapshot] filename
-l: 选项是查询并列出镜像文件中的所有快照，
-a snapshot: 是让镜像文件使用某个快照，
-c snapshot: 是创建一个快照，
-d: 是删除一个快照。
```


<a id="org2cb5b06"></a>

# 参考链接

<https://virt-manager.org/> <https://www.cnblogs.com/sammyliu/p/4543110.html> <https://www.tecmint.com/install-and-configure-kvm-in-linux/> <https://baike.baidu.com/item/KVM%E8%99%9A%E6%8B%9F%E6%9C%BA/11016451> <https://www.ibm.com/developerworks/cn/linux/l-linux-kvm/index.html> <https://blog.csdn.net/lqx0405/article/details/50033017>

-   Centos7.4安装kvm虚拟机（使用virt-manager管理）

<https://www.centos.bz/2018/02/centos7-4%e5%ae%89%e8%a3%85kvm%e8%99%9a%e6%8b%9f%e6%9c%ba%ef%bc%88%e4%bd%bf%e7%94%a8virt-manager%e7%ae%a1%e7%90%86%ef%bc%89/>
