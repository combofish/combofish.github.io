---
layout: post
title: "OpenMediavault Raspberry Pi setup"
date: 2020-01-26
tag: NAS raspberrypi openmediavault 
---

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. OMV</a>
<ul>
<li><a href="#sec-1-1">1.1. OpenMediaVault</a></li>
<li><a href="#sec-1-2">1.2. 帮助文档</a></li>
<li><a href="#sec-1-3">1.3. 准备</a></li>
<li><a href="#sec-1-4">1.4. install</a></li>
<li><a href="#sec-1-5">1.5. 启动</a></li>
</ul>
</li>
</ul>
</div>
</div>

# OMV<a id="sec-1" name="sec-1"></a>

-家中闲置树梅派，正好又有闲置的1T的机械硬盘，做一个网络存储来备份闲置的文件。-

## OpenMediaVault<a id="sec-1-1" name="sec-1-1"></a>

OpenMediaVault是一个基于Debian的专用Linux发行版，用于构建网络连接存储（NAS）系统。 它提供了一个易于使用的基于Web的界面，
多语言支持，卷管理，监控和插件系统，以通过LDAP，Bittorrent和iSCSI功能进行扩展。 本文介绍OpenMediaVault的安装和配置。

-   OpenMediaVault 包含以下重要特性：
    -   Debian GNU/Linux 系统(i386 or x64)
    -   基于Web方式的系统管理
    -   通过Debian软件包简单的系统升级
    -   插件系统

## 帮助文档<a id="sec-1-2" name="sec-1-2"></a>

**注意：没有直接可用于树梅派的镜像，要先安装树梅派的raspbian系统，再通过omv的脚本安装。**
[可在这里查看](https://github.com/OpenMediaVault-Plugin-Developers/docs) <https://github.com/OpenMediaVault-Plugin-Developers/docs>

## 准备<a id="sec-1-3" name="sec-1-3"></a>

-   raspberry pi 官方镜像 Raspbian
-   烧写软件 Etcher
-   raspberry pi 3 硬件版
-   移动硬盘

## install<a id="sec-1-4" name="sec-1-4"></a>

    git clone https://github.com/OpenMediaVault-Plugin-Developers/installScript
    cd installScript && sudo bash ./install

## 启动<a id="sec-1-5" name="sec-1-5"></a>

从路由器查看树梅派的IP地址，在浏览器登陆。网页界面的管理员默认的账户和密码分别是 admin 和 openmediavault，当你登入后要马上修改。
进行后续硬盘，各种服务以及共享的设置。
