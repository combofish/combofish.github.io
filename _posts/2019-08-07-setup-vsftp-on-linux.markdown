---
layout: post
title: "Linux上搭建FTP文件服务器(Vsftp)"
date: 2019-08-07
tag: Linux FTP vsftp
---

- [文件服务](#org916d50c)
    - [ftp 文件传输协议](#orgb3f7f7a)
    - [samba](#org2c40dd5)
    - [nfs](#orgbbac3d5)
- [vsftp](#org6594a9f)
  - [FTP 协议](#org1caebe9)
  - [配置文件位置](#org9ba4723)
- [配置(config)](#orge0a50e6)
  - [建立虚拟用户](#org4526223)
  - [PAM配置](#org81d1cc0)
    - [添加虚拟用户的名称和密码](#org56eec60)
    - [生成.db 文件](#org1ad7723)
    - [添加 pcm 认证](#org637e6ec)
  - [vsftpd.conf](#org209826c)
    - [匿名用户配置](#orgd67cfc8)
    - [本地用户配置](#org39d6bf9)
    - [用户登陆配置](#orgde9658d)
    - [虚拟用户](#orgbee5e34)
- [启动服务](#org594456c)
- [ftp](#orgd9ed8ce)
  - [常用命令](#orga2f05cd)
- [遇见的错误](#orgc62cbd1)
- [selinux](#org3df23a1)
- [参考链接](#orgf5015f1)


<a id="org916d50c"></a>

# 文件服务

-   ftp, samba, nfs 介绍


<a id="orgb3f7f7a"></a>

### ftp 文件传输协议

-   是用于在网络上进行文件传输的一套标准协议， 使用 TCP 传输


<a id="org2c40dd5"></a>

### samba

-   Linux/Unix 系统中实现了 Windows 文件共享所使用的 CIFS 协议，也叫做 SMB（Simple Message Block）协议。
-   这使得 Windows/Linux/Unix 间可以自由的进行文件共享。


<a id="orgbbac3d5"></a>

### nfs

-   NFS 全称是 Network FileSystem，NFS 和其他文件系统一样，是在 Linux 内核中实现的，因此 NFS 很难做到与 Windows 兼容。
-   NFS 共享出的文件系统会被客户端识别为一个文件系统，客户端可以直接挂载并使用。

-   NFS 的实现使用了 RPC（Remote Procedure Call） 的机制，远程过程调用使得客户端可以调用服务端的函数。
-   由于有 VFS 的存在，客户端可以像使用其他普通文件系统一样使用 NFS 文件系统，
-   由操作系统内核将 NFS 文件系统的调用请求通过 TCP/IP 发送至服务端的 NFS 服务，执行相关的操作，之后服务端再讲操作结果返回客户端。

-   NFS（Network File System）即网络文件系统，是FreeBSD支持的文件系统中的一种，它允许网络中的计算机之间共享资源。
-   在NFS的应用中，本地NFS的客户端应用可以透明地读写位于远端NFS服务器上的文件，就像访问本地文件一样。


<a id="org6594a9f"></a>

# vsftp

-   VSFTP是一个基于GPL发布的类Unix系统上使用的FTP服务器软件，它的全称是Very Secure FTP 从此名称可以看出来，编制者的初衷是代码的安全。


<a id="org1caebe9"></a>

## FTP 协议

-   FTP 协议中使用了类似于 HTTP 协议的响应码：
    -   1xx：提示信息
    -   2xx：成功类状态码
    -   3xx：提示需进一步提供补充类信息的状态码
    -   4xx：客户端错误
    -   5xx：服务端错误

下面介绍 vsftpd 这款服务端软件的使用，FTP 在用户认证时，会使用系统中的用户进行身份认证， 同时 FTP 支持虚拟用户，虚拟用户最终也需要映射为系统中的某个用户，匿名用户会被映射为系统中的 ftp 用户。


<a id="org9ba4723"></a>

## 配置文件位置

-   vsftpd 使用了 pam 模块进行用户身份认证，配置文件为 /etc/pam.d/vsftpd
-   vsftpd 的主配置文件为/etc/vsftpd/vsftpd.conf
-   用户访问 FTP 时，默认访问的位置是用户自己的家目录。ftp 用户的家目录为 /var/ftp/，因此使用匿名用户将默认访问 /var/ftp 目录。


<a id="orge0a50e6"></a>

# 配置(config)

-   配置文件目录/etc/vsftpd/


<a id="org4526223"></a>

## 建立虚拟用户

-   这ftpuser只个虚拟帐户的宿主，本身是不用登录的

```shell
$useradd ftpuser -s /sbin/nologin

```


<a id="org81d1cc0"></a>

## PAM配置

-   为了使服务器能够使用数据库文件，对客户端进行身份验证，需要调用系统的PAM模块.
-   PAM(Plugable Authentication Module)为可插拔认证模块，不必重新安装应用系统，通过修改指定的配置文件，调整对该程序的认证方式。
-   PAM模块配置文件路径为/etc/pam.d/目录，此目录下保存着大量与认证有关的配置文件，并以服务名称命名。


<a id="org56eec60"></a>

### 添加虚拟用户的名称和密码

-   文件位置 /etc/vsftpd/config/vusers
-   文件内容
    
    ```shell
    virtual-user-name
    passwd
    ```


<a id="org1ad7723"></a>

### 生成.db 文件

```shell
db_load -T -t hash -f ./vusers ./vusers.db
```

1.  限制权限

    ```shell
    chmod  600 /etc/vsftpd/config/vusers.db
    ```


<a id="org637e6ec"></a>

### 添加 pcm 认证

-   添加文件 /etc/pam.d/vsftpd
-   编辑内容

```shell
auth       required     pam_userdb.so   db=/etc/vsftpd/config/vusers
account    required     pam_userdb.so   db=/etc/vsftpd/config/vusers
session    required     pam_loginuid.so

```


<a id="org209826c"></a>

## vsftpd.conf


<a id="orgd67cfc8"></a>

### 匿名用户配置

-   为了安全，禁用匿名用户
-   将第一项设置为NO, 其他的项注释掉

```shell
anonymous_enable=NO # 是否允许匿名用户
anon_upload_enable=YES # 匿名用户是否可以上传
anon_mkdir_write_enable=YES # 匿名用户是否可以创建文件夹
anon_ohter_write_enable=YES # 匿名用户的其他权限，如删除文件夹的权限

```


<a id="org39d6bf9"></a>

### 本地用户配置

```shell
local_enable=YES # 是否启用系统用户
write_enable=YES # 是否允许系统上传文件
local_umask=022 # 上传的文件的默认 umask 值

```

-   当系统用户登录 FTP 后，默认位于其家目录中，但是也可以访问系统的其他目录，这样通常是不安全的。
-   如下设置可以将系统用户禁锢于其家目录中：
    
    ```shell
    chroot_local_user=YES # 是否禁锢系统用户与其家目录中
    
    ```

-   也可以指定用户将其禁锢于家目录中：

```shell
chroot_list_enable=YES # 启用 chroot list 文件
chroot_list_file=/etc/vsftpd/chroot_list # chroot list 文件

```

-   如果 chroot_local_user 为 NO，那么 chroot_list 文件中的用户将被禁锢至家目录中，
-   如果chroot_local_user 为 YES，那么表示仅 chroot_list 文件中的用户不会被禁锢至家目录中。

-   注意：/etc/vsftp/chroot_list 默认是不存在的


<a id="orgde9658d"></a>

### 用户登陆配置

```shell
userlist_enable=YES # 启用userlist文件
userlist_deny=YES # YES 表示 userlist 为用户黑名单，NO 表示 userlist 为白名单
userlist_file=/etc/vsftpd/user_list # userlist 文件位置
pam_service_name=vsftpd  # 配置vsftpd使用的PAM模块为vsftpd  
```

1.  参考user<sub>list</sub>

    -   可在次基础上添加不允许登陆的用户
        
        ```shell
        $ cat user_list
        # vsftpd userlist
        # If userlist_deny=NO, only allow users in this file
        # If userlist_deny=YES (default), never allow users in this file, and
        # do not even prompt for a password.
        # Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
        # for users that are denied.
        root
        bin
        daemon
        adm
        lp
        sync
        shutdown
        halt
        mail
        news
        uucp
        operator
        games
        nobody
        
        ```


<a id="orgbee5e34"></a>

### 虚拟用户

-   注意：主配置文件是虚拟账号共享的配置，所以主配置文件的设置对于虚拟账号是生效的
-   如test1用户对于的配置文件名为test1,设置虚拟帐号的主目录为 *etc/vsftpd/config*

1.  开启虚拟用户

    ```shell
    guest_enable=YES
    guest_username=ftpuser
    virtual_use_local_privs=YES
    user_config_dir=/etc/vsftpd/config # 自建配置，该文件夹内主要是各虚拟用户的同名配置文件
    
    ```

2.  虚拟用户的配置文件内容

    ```shell
    $ cat virtual-user-name 
    local_root=/home/ftpuser/virtual-user-name/ # 虚拟用户的个人目录路径
    max_clients=10
    max_per_ip=5
    local_max_rate=1048576
    ```

3.  为虚拟用户添加目录

    ```shell
    mkdir /home/ftpuser/virtual-user-name
    touch /home/ftpuser/virtual-user-name/test  # 随便添加一个文件
    chown ftpuser:ftpuser -R /home/ftpuser/virtual-user-name/
    
    ```


<a id="org594456c"></a>

# 启动服务

-   推荐在启动后，看以下状态，因为没有正常启动时，也没有任何提示

```shell
systemctl start vsftpd   # 启动服务
systemctl enable vsftpd  # 设置为开机启动
systemctl status vsftpd  # 查看运行状态
```


<a id="orgd9ed8ce"></a>

# ftp


<a id="orga2f05cd"></a>

## 常用命令

-   open xxx.xxx.xxx.xxx 打开
-   lcd 切换本地目录
-   get 下载文件
-   put 上传文件
-   bye 中断与服务器的连接。
-   !dir 查看本地文件夹中的文件及目录
-   !ls 查看本地文件夹中的文件及目录
-   pwd 查看当前目录
-   ! 退出当前的窗口，返回Linux 终端，当我们退出终端的时候，又会返回到FTP上。
-   delete 删除文件
-   mdelete a\* 批量删除文件

-   [参考链接](https://www.cnblogs.com/mingforyou/p/4103022.html)


<a id="orgc62cbd1"></a>

# 遇见的错误

-   500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
-   解决办法，在配置文件添加

```shell
allow_writeable_chroot=YES
```

-   [参考链接](https://blog.csdn.net/bluishglc/article/details/42399439)
-   [](#orgf5015f1) <https://www.cnblogs.com/wi100sh/p/4542819.html>


<a id="org3df23a1"></a>

# selinux

PS: 有些错误是由于 selinux 设置引起的，我的电脑没有设置 selinux ，所以下条没有试验。[参考链接](https://blog.csdn.net/csucxcc/article/details/1770598)

```shell
setsebool -P ftpd_disable_trans 1 
service vsftpd restart 
```


<a id="orgf5015f1"></a>

# 参考链接

<https://www.cnblogs.com/wxl-dede/p/5042398.html>

