---
layout: post
title: "File Sharing Service -- Samba Configuration and Use"
date: "2021-10-23 17: 43: 00"
tag: samba linux server
---
- [Samba](#org043c452)
  - [安装配置](#org3e2ba04)
  - [添加用户](#orgc47ab51)
  - [配置共享文件夹](#org2546f0f)
  - [使用](#org6211fdf)


<a id="org043c452"></a>

# Samba

[Samba](https://www.samba.org/) 是适用于 Linux 和 Unix 的标准 Windows 互操作性程序套件。 自 1992 年以来，Samba 一直为所有使用 SMB/CIFS 协议的客户端提供安全、稳定和快速的文件和打印服务，例如所有版本的 DOS 和 Windows、OS/2、Linux 等。 一般用于 Linux 和 Windows 系统之间的资源共享。


<a id="org3e2ba04"></a>

## 安装配置

如果使用 Ubuntu 系统，可用如下命令安装：

```shell
sudo apt-get install samba  # 下载
systemctl start smbd        # 启动服务
systemctl enable smbd       # 设置为开机自动启动
```


<a id="orgc47ab51"></a>

## 添加用户

一般我会添加两个用户，一个有读写权限，一个仅有只读权限。可以用如下命令添加用户。需要注意的是 Samba 的用户必须是Linux的用户， 所以确保添加的用户已经存在于系统中。

```shell
smbpasswd -a $USER  # 添加用户

# pdbedit — manage the SAM database (Database of Samba Users)
pdbedit -L  	 # 查看smb中加入的用户
pdbedit -a          # 添加用户
pdbedit -x          # 删除用户
pdbedit -Lv         # 列出用户详细的信息列表
```


<a id="org2546f0f"></a>

## 配置共享文件夹

默认的配置文件在 /etc/samba/smb.conf.

```shell
cat /etc/samba/smb.conf | grep -v '#' | grep -v ';' | grep -v -e '^$'
```

可以看到如下内容（有省略）,作为了解即可：

```conf
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba, Ubuntu)
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

配置自己的共享文件夹

```conf
[Share]                           # 指定的共享名称
   comment = larry guest share    # 描述信息
   path =                         # 共享文件夹的位置
   browseable =                   # 指定共享是否在 “网上邻居” 可见
   writable = yes                 # 指定是否可以写入操作
   directory mask = 0775
   create mask = 0775
   valid users = user0,user1      # 设定两个可以登录的用户，但只给了一个用户有写的权限
   write list = user0
   browseable = yes
   available = yes
```


<a id="org6211fdf"></a>

## 使用

-   如果是 Linux, 可以用如下命令连接使用：

```shell
smbclient -L //127.0.0.1 -U $USER 
smbclient //127.0.0.1/Share -U $USER
```

-   也可以挂载后使用:

需要先安装 CIFS(Common Internet File System) 支持， 通用 Internet 文件系统 (CIFS) 是一种网络文件系统协议，用于在网络上的机器之间提供对文件和打印机的共享访问。 CIFS 客户端应用程序可以读取、写入、编辑甚至删除远程服务器上的文件。 CIFS 客户端可以与任何设置为接收 CIFS 客户端请求的服务器通信。

```shell
sudo apt-get install cifs-utils
sudo mount -t cifs //127.0.0.1/Share /smb -o user= ,password=
```

如果不想在命令行里直接输入用户名和密码, 可以建立一个文件存储用户名和密码，并给这个文件设置管理员可读写权限：

```shell
touch smbpass
echo "username= " >> smbpass
echo "password= " >> smbpass
mount.cifs -o credentials=/root/smbpass //127.0.0.1/Share /smb
```

如果是 Windows 系统, 可以在 此电脑（右键） -> 映射网络驱动器，按提示添加共享目录，比如 //ip/Share, 再通过验证就可以进行访问。
