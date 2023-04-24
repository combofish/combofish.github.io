---
layout: post
title: "Sync Time on Ubuntu 20.04"
date: "2022-11-17 08: 19: 00"
tag: time ntp ubuntu
---
- [Sync Time on Ubuntu](#orgb03695a)
- [使能时间同步](#org160768e)
- [同步时间](#org06c70dc)


<a id="orgb03695a"></a>

# Sync Time on Ubuntu


<a id="org160768e"></a>

## 使能时间同步

```shell
timedatectl set-ntp on
```

-   如果上述操作没有起作用，则需要配置 ntp (Network Time Protocol).


<a id="org06c70dc"></a>

## 同步时间

```shell
timedatectl set-ntp on
sudo apt install ntp
reboot  # 重启之后才能生效
# -----
date
```
## 博主的应用场景

- 日常使用的电脑自然不会遇到这样的问题，一般安装ubuntu，时间同步服务自动就开启了，但是在某些硬件平台上，比如 NVIDIA NX 板卡， 安装英伟达官网提供的ubuntu系统后，默认时间比较老，导致更新源的时候一直报错，找了好久才发现是时间不匹配的原因，于是一顿操作，终于可以正常更新了~~
