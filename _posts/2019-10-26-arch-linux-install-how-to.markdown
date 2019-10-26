---
layout: post
title: "ArchLinux install notes(only commands, not friendly for novice)"
date: 2019-10-26
tag: ArchLinux install note
---
- [Install Arch Linux](#orgf2d25d6)
  - [Partition](#org0a4890c)
  - [Mount](#org1c6a296)
  - [Installation](#org5dba5ea)
  - [Fstab](#org86c893b)
  - [Chroot](#org574f557)
  - [Time zone](#org758afba)
  - [Localization](#orga20d15e)
  - [Initramfs](#orgd86f42d)
  - [Root password](#orgaca9a9d)
  - [Add tsinghua source](#org702dfde)
  - [Grub](#org8bdb35e)
  - [Install Xorg](#org2607d08)
  - [Create new user](#org58d4590)
  - [Install software](#orge8aabb2)
    - [解决安装archlinux后没有ifconfig命令](#orgdbc08f1)
  - [Start service](#orgf84b949)


<a id="orgf2d25d6"></a>

# Install Arch Linux

[官方安装链接](https://wiki.archlinux.org/index.php/Installation_guide#Post-installation)


<a id="org0a4890c"></a>

## Partition

```shell
fdisk -l 
fdisk /dev/sda 

mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2 
mkswap /dev/sda3
swapon /dev/sda3
```


<a id="org1c6a296"></a>

## Mount

```sh
mount /dev/sda1 /mnt 
mkdir /mnt/boot 
mount /dev/sda2 /mnt/boot
```


<a id="org5dba5ea"></a>

## Installation

```shell
pacstrap /mnt base linux linux-firmware
```


<a id="org86c893b"></a>

## Fstab

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```


<a id="org574f557"></a>

## Chroot

```shell
arch-chroot /mnt
```


<a id="org758afba"></a>

## Time zone

```shell
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock –systohc
```


<a id="orga20d15e"></a>

## Localization

```shell
nano /etc/locale.gen # at least uncomment en_US.UTF-8 UTF-8 Or zh_CN.UtF-8 UTF-8 
locale-gen
echo LANG=en_US.UTF-8 >> /etc/locale.conf
echo ‘yourhostname’ >> /etc/hostname
```


<a id="orgd86f42d"></a>

## Initramfs

```shell
mkinitcpio -P
```


<a id="orgaca9a9d"></a>

## Root password

```shell
passwd 
```


<a id="org702dfde"></a>

## Add tsinghua source

-   编辑 /etc/pacman.d/mirrorlist， 在文件的最顶端添加：

Server = <https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch>

-   更新软件包缓存：

```shell
pacman -Syy
```


<a id="org8bdb35e"></a>

## Grub

```shell
pacman -S grub
grub-install --target=i386-pc /dev/sda 
grub-mkconfig -o /boot/grub/grub.cfg
```


<a id="org2607d08"></a>

## Install Xorg

```shell
pacman -S xorg-server 
```


<a id="org58d4590"></a>

## Create new user

```shell
useradd -m -g wheel -s /bin/bash username 
# pacman -S zsh 
# useradd -m -g wheel -s /bin/zsh username 
passwd username 
```


<a id="orge8aabb2"></a>

## Install software

```shell
pacman -S sddm plasma plasma-nm 
pacman -S dialog ntfs-3g ark gimp kate 
pacman -S firefox chromium 
pacman -S gimp gwenview vlc okular sudo 
```


<a id="orgdbc08f1"></a>

### 解决安装archlinux后没有ifconfig命令

```shell
pacman -S net-tools dnsutils inetutils iproute2
```


<a id="orgf84b949"></a>

## Start service

```shell
systemctl enable sddm 
systemctl enable NetworkManager
```
