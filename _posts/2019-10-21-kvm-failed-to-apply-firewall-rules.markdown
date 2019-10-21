---
layout: post
title: "(转)libvirtd:内部错误：Failed to apply firewall rule"
date: 2019-10-21
tag: libvirt libvirtd failed
---
- [libvirtd 服务运行不正常](#orge09e24d)
- [解决办法](#orgef646fe)
- [原文连接](#org9be0b45)


<a id="orge09e24d"></a>

# libvirtd 服务运行不正常

```shell
systemctl status libvirtd 
```

```shell
      错误：内部错误：Failed to apply firewall rules /sbin/iptables -w –table nat...
```


<a id="orgef646fe"></a>

# 解决办法

-   分析：

该错误的出现一般是内核中的NAT模块不支持MASQUERADE相关的操作，需要重新配置内核编译选项，然后编译安装内核。

-   涉及到的内核配置如下：

```shell
CONFIG_NF_NAT_MASQUERADE_IPV4=m
CONFIG_IP_NF_TARGET_MASQUERADE=m
CONFIG_NF_NAT_MASQUERADE_IPV6=m
```

重新编译安装内核后，该问题便可以解决。

```shell
systemctl status libvirtd 
```

-   启动正常

```shell
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/libvirtd.service.d
           └─00gentoo.conf
   Active: active (running) since Mon 2019-10-21 09:02:32 CST; 8min ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 8930 (libvirtd)
   Memory: 93.2M
   CGroup: /system.slice/libvirtd.service
           ├─5178 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           ├─5179 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           └─8930 /usr/sbin/libvirtd --timeout 120
```


<a id="org9be0b45"></a>

# 原文连接

<https://blog.csdn.net/wangwei222/article/details/80731382>
