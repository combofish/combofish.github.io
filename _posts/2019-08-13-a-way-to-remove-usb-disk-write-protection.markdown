---
layout: post
title: "一种解决U盘写保护的可行办法(dd命令解决)"
date: 2019-08-13
tag: dd Linux USB disk 
---

- [舍友拿来14年8GU盘（有写保护）](#org47cc291)
  - [现象](#orga6d09a6)
    - [挂载](#org2d66334)
    - [格式化](#org88c814f)
    - [连接 win 10](#orge48abcd)
  - [成因](#orgead8939)
    - [被用来作系统盘](#org2376459)
  - [采取的措施](#org7832744)
    - [fdisk](#orgdbf950a)
    - [Win 10PE](#org03b5e9c)
    - [hdparm](#orge571f9b)
    - [dd](#org9aa3c95)


<a id="org47cc291"></a>

# 舍友拿来14年8GU盘（有写保护）


<a id="orga6d09a6"></a>

## 现象


<a id="org2d66334"></a>

### 挂载

Linux 上挂载时提示，设备有写保护，只能以只读方式挂载


<a id="org88c814f"></a>

### 格式化

fdisk 格式化能把正常流程走下来，好神奇，但一点用都没有


<a id="orge48abcd"></a>

### 连接 win 10

磁盘管理中有，文件管理中无，格式化不管用


<a id="orgead8939"></a>

## 成因


<a id="org2376459"></a>

### 被用来作系统盘

1.  烧录工具 Etcher

2.  系统 Zorin linux

    咱也不知道这是什么，咱也不敢问。


<a id="org7832744"></a>

## 采取的措施


<a id="orgdbf950a"></a>

### fdisk     :fail:

-   fdisk /dev/sdd, 删掉原有分区, 重新分区, 并格式化 mkfs.ntfs /dev/sdd1


<a id="org03b5e9c"></a>

### Win 10PE     :fail:

1.  DiskGenius软件,分区

    分完之后神奇的能在PE 系统下使用，弹出，在插上也可以，到普通 Win 10 又不行了


<a id="orge571f9b"></a>

### hdparm     :fail:

-   hdparm -r0 /dev/sdd
-   再挂载，不起作用


<a id="org9aa3c95"></a>

### dd     :success:

-   dd if=/dev/zero of=/dev/sdd
-   我没等到写完，直接中断，关机了，
-   连到 Win 10, 提示格式化
-   格式化后， 能正常使用

![xiaoguotu](/images/a-way-to-remove-usb-write-protection_diagram.png)

