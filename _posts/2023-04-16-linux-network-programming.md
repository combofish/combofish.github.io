---
layout: post
title: "[学习笔记] Linux 网络编程"
date: "2023-04-16 08: 21: 22"
tag: linux network programming tcp
---
<!-- TOC -->
* [Linux 网络编程](#linux-网络编程)
  * [网络结构模式](#网络结构模式)
  * [MAC 地址](#mac-地址)
  * [IP 地址](#ip-地址)
    * [特殊的地址](#特殊的地址)
  * [子网掩码 (subnet mask)](#子网掩码-subnet-mask)
  * [端口 (port)](#端口-port)
    * [端口的类型](#端口的类型)
  * [网络模型](#网络模型)
    * [TCP/IP 四层模型](#tcpip-四层模型)
  * [协议](#协议)
    * [常见协议](#常见协议)
    * [UDP 协议](#udp-协议)
    * [TCP 协议](#tcp-协议)
    * [IP 协议](#ip-协议)
    * [以太网帧协议](#以太网帧协议)
    * [ARP 协议](#arp-协议)
  * [网络通信的过程](#网络通信的过程)
    * [封装](#封装)
    * [分用](#分用)
  * [Socket 介绍](#socket-介绍)
  * [字节序](#字节序)
  * [字节序转换函数](#字节序转换函数)
  * [Socket 地址](#socket-地址)
    * [专用 socket 地址](#专用-socket-地址)
  * [IP 地址转换函数](#ip-地址转换函数)
  * [TCP 通信流程](#tcp-通信流程)
    * [TCP 通信的流程](#tcp-通信的流程)
    * [TCP Server](#tcp-server)
    * [TCP Client](#tcp-client)
  * [TCP 三次握手](#tcp-三次握手)
  * [滑动窗口](#滑动窗口)
  * [TCP 四次挥手](#tcp-四次挥手)
  * [多进程实现并发服务器](#多进程实现并发服务器)
  * [多线程实现并发服务器](#多线程实现并发服务器)
  * [TCP状态转换](#tcp状态转换)
  * [半关闭](#半关闭)
  * [端口复用](#端口复用)
  * [IO多路复用(IO多路转接)](#io多路复用io多路转接)
    * [阻塞](#阻塞)
    * [非阻塞，忙轮询](#非阻塞忙轮询)
  * [select_API介绍](#selectapi介绍)
  * [poll](#poll)
  * [epoll](#epoll)
  * [EPOLL的工作模式](#epoll的工作模式)
  * [UDP通信](#udp通信)
  * [广播](#广播)
  * [组播(多播)](#组播多播)
  * [套接字通信](#套接字通信)
<!-- TOC -->

# Linux 网络编程

## 网络结构模式

+ C/S 结构： 客户机/服务端

服务器负责数据的管理，客户机负责完成与用户的交互任务。

+ B/S 结构

## MAC 地址

MAC 地址： (Media Access Control Address)， 48 位，6个字节，前三个由 IEEE 分配。

网卡的主要功能：1.数据的封装与解封装、2.链路管理、3.数据编码与译码

在 OSI 模型中，第三层网络层负责 IP 地址，第二层数据链路层则负责 MAC 地址。
MAC 地址用于在网络中唯一标识一个网卡。

## IP 地址

IP 协议是为计算机网络相互连接进行通信而设计的协议。

IP 地址 （Internet Protocol Address）， 是指互联网协议地址。是一个 32 位的二进制数。四个字节。通常用点分十进制表示。
其中每位可以表示 0~255 之间的十进制整数。

可分为 A,B,C等类地址。A类每个网络最大支持的主机数为 256^3 - 2; 以此类推。B 类：65534，C 类：254 台。

### 特殊的地址

0.0.0.0 对应当前主机， 255.255.255.255 当前子网的广播地址，IP 地址中不能以十进制 127 开头，改地址中 127.0.0.1 ~
127.255.255.255 用于回路测试，
如 127.0.0.1 代表本机 IP 地址。

## 子网掩码 (subnet mask)

## 端口 (port)

一个 IP 地址的端口可以有 65536 个， 从 0 ~ 65535。

### 端口的类型

1. 周知端口 (Well Known Ports), 范围 0-1023。

2. 注册端口 (Registered Ports) 范围 1024-49151。

3. 动态端口、私有端口。

## 网络模型

+ OSI (Open System Interconnection) 七层参考模型，即开放式系统互联。

应用层 -> 表示层 -> 会话层 -> 传输层 -> 网络层 -> 数据链路层 -> 物理层

+ 物理层：主要定义物理设备标准，如网线的接口类型，光纤的接口类型，各种传输介质的传输速率等。
+ 数据链路层：建立逻辑连接、进行硬件地址寻址、差错校验等功能。
+ 网络层：进行逻辑地址寻址，在位于不同地址位置的网络中的两个主机系统之间提供和路径选择。
+ 传输层：定义了一些传输数据的协议和端口号
+ 会话层：通过传输层（端口号： 传输端口与接收端口）建立数据传输的通路。
+ 表示层：数据的表示、安全、压缩。
+ 应用层：网络服务与最终用户的一个接口。

### TCP/IP 四层模型

应用层 -> 传输层(ICMP、IP) -> 网络层(TCP、UDP) -> 数据链路层 （3，1，1，2）

应用层在用户空间，其他三层在内核空间。

## 协议

协议，网络协议的简称，网络协议是通信计算机双方必须共同遵守的一组约定。它的三个要素是： 语法、语义、时序。它最终体现在网络上传输的数据包的格式。

### 常见协议

应用层常见协议： FTP (File Transfer Protocol 文本传输协议)、HTTP (Hyper Text Transfer Protocol 超文本传输协议)、
NFS (Network File Sysetm 网络文件系统)。

传输层常见协议有：TCP (Transmission Control Protocal 传输控制协议)、UDP (User Datagram Protocol 用户数据报协议)。

网络层常见的协议有：IP 协议 (Internet Protocol 因特网互联协议)、(ICMP Internet Control Message Protocal
因特网控制报文协议)、
IGMP (Internet Group Management Protocol 因特网组管理协议)。

网络接口层常见的协议有 ARP (Address Resolution Protocol 地址解析协议)、 RARP (Reverse Address Resolution Protocol)
反向地址解析协议。

### UDP 协议

+ 16 bit, 源端口号：发送方端口号
+ 16 bit, 目的端口号：接收方端口号
+ 16 bit, 长度：UDP 用户数据报的长度，最小值是8（仅有首部）
+ 16 bit, 校验和： 检测 UDP 用户数据报在传输中是否有错，有错就丢弃。

### TCP 协议

+ 16 bit，源端口号
+ 16 bit, 目的端口号
+ 32 bit, 序列号： 本报文段的数据的第一个字节的序号
+ 32 bit, 确认序号： 期望收到对方下一个报文段的第一个数据字节的序号
+ 4 bit, 首部长度（数据偏移），TCP 报文段的数据起始处距离 TCP 报文段的起始处有多远，即首部长度。单位 32bit,即以 4 字节为计算单位。
+ 6 bit, 保留为今后使用，目前应设置为 0。
+ 标志位： URG,ACK,PSH,RST,SYN,FIN.
+ 16 bit, 窗口大小
+ 16 bit, 校验和
+ 16 bit, 紧急指针
+ 选项，最多 40 字节。

### IP 协议

+ 4 bit, 版本号
+ 4 位头部长度： 单位是 32 位（4字节）
+ 8 bit 服务类型 TOS
+ 16 bit, 总长度（字节数）,指首部加上数据的总长度，单位为字节。
+ 16 bit，标识
+ 3 bit 标志
+ 13 bit, 片偏移
+ 8 bit, 生存时间 TTL
+ 8 bit, 协议， 指出此数据报携带的数据时使用何种协议，以便目的主机的 IP 层知道应将数据部分上交给哪个处理过, ICMP(1), ICMP(
  2),TCP(6),UDP(17),IPv6(41).
+ 16 bit, 头部校验和
+ 32 bit 源端IP地址
+ 32 bit 目的端 IP 地址
+ 选项 40 bit

### 以太网帧协议

+ 6 字节, 目的物理地址 （ff:ff:ff:ff:ff:ff 表示给所有的主机发送）
+ 6 字节, 源物理地址
+ 2 字节， 类型 0x800 -> IP， （0x806 表示ARP）、（0x835 表示 RARP）
+ 46~1500 字节，数据
+ 4 字节， CRC

### ARP 协议

通过 IP 地址 找到物理（MAC）地址。

ARP 请求/ 应答报文

+ 2 字节，硬件类型： 1 表示MAC地址
+ 2 字节，协议类型 0x80 表示IP地址
+ 1 字节，硬件地址长度
+ 1 字节，协议地址长度
+ 2 字节 操作： 1 表示ARP请求，2 表示ARP应答， 3 表示RARP请求，4 表示RARP 应答
+ 6 字节 发送端以太网地址
+ 4 字节 发送端IP地址
+ 6 字节 目的端以太网地址
+ 4 字节 目的端IP地址

## 网络通信的过程

### 封装

每层协议都将在上层数据的基础上加上自己的头部信息（有时还包括尾部信息），以实现该层的功能，这个过程就是封装。

### 分用

当帧到达目的主机时，将沿着协议栈自底向上依次传递。分用是依靠头部信息中的类型字段实现的。

## Socket 介绍

Socket 套接字，对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。
一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位上讲，套接字上联应用进程，下联网络协议栈，
是应用程序通过网络协议进行通信的接口，是应用程序与网络协议根进行交互的接口。

Socket 是由IP地址和端口结合的，提供向应用层进程传送数据包的机制。

在Linux 环境下，用于表示进程间网络通信的特殊文件类型。本质为内核借助缓冲区形成的伪文件。既然是文件，可以使用文件描述符引用套接字。
与管道类似的，Linux 系统将其封装成文件的目的是为了统一接口，使得读写套接字和读写文件的操作一直。
区别是管道主要应用与本地进程间通信，而套接字多应用与网络进程间数据的传递。

+ 读缓冲区
+ 写缓冲区

套接字通信分两部分：

+ 服务器端，被动接受连接，一般不会主动发起连接
+ 客户端：主动像服务器发起连接

Socket 是一套通信的接口。

## 字节序

字节序，字节的顺序，就是大于一个字节类型的数据在内存中存放顺序（一个字节的数据当然就无需谈荀淑的问题）。

字节序分位大端字节序 （Big Endian）, 和 小端字节序 （Little Endian）。
大端字节序是指一个整数的最高位字节 23~31 bit 存储在内存的低地址处，低位字节 0-7 bit 存储在内存的高地址处；
小端字节序则是指整数的高位字节存储在内存的高地址处，而低位字节则存储在内存的低地址处。

大部分计算机使用小端字节序。

## 字节序转换函数

网络字节顺序是 TCP/IP 中规定好的一种数据表示格式，它与具体的CPU类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释，
网络字节顺序采用大端排序方式。

+ h: host 主机，主机字节序
+ to
+ n: network 网络字节序
+ s: short, unsigned short
+ l: long, unsigned int

```c
#include <arpa/inet.h>
uint16_t htons(uint16_t hostshort);
uint16_t htons(uint16_t netshort);

uint32_t htons(uint32_t hostlang);
uint32_t htons(uint32_t netlong);
```

网络通信时，需要将主机字节序转换成网络字节序（大端），
另外一端获取到数据以后根据情况将网络字节序转换成主机字节序。

```c
#include <stdio.h>
#include <arpa/inet.h>

int main() {

    // htons 转换端口
    unsigned short a = 0x0102;
    printf("a: %x\n", a);

    unsigned short b = htons(a);

    printf("b: %x\n", b);
    printf("===============================\n");

    // htonl 转换 IP
    char buf[4] = {192, 168, 1, 100};
    int num = *(int *) buf;

    int ans = htonl(num);

    unsigned char *p = (unsigned char *) &ans;

    printf("%d,%d,%d,%d\n", *p, *(p + 1), *(p + 2), *(p + 3));
    printf("===============================\n");

    // ntohs
    unsigned char buf1[4] = {1, 1, 168, 192};
    num = *(int *) buf1;
    ans = ntohl(num);

    p = (char *) &ans;
    printf("%d %d %d %d\n", *p, *(p + 1), *(p + 2), *(p + 3));
    printf("===============================\n");

    // ntohs
    a = 0x0108;
    printf("a: %x\n", a);
    b = ntohs(a);
    printf("b: %x\n", b);

    return 0;
}
```

## Socket 地址

Socket 地址启示是一个结构体，封装端口号和IP 等信息。

通用 socket 地址： socket 网络编程接口中表示 socket 地址的是结构体 sockaddr, 定义如下：

```c
#include <bits/socket.h>
struct socketaddr{
    sa_family_t sa_family;
    char sa_data[14];    
};

typedef unsigned short int sa_family_t;
```

### 专用 socket 地址

很多网络编程函数诞生早于 IPv4 协议，那时候都使用的是 struct sockaddr 结构体，为了向前兼容，
现在 sockaddr 退化成了 (void*) 的作用，传递一个地址给函数，之余这个函数是 sockaddr_in 还是
socketaddr_in6 由地址族确定，然后函数内部再强制类型转化为所需的地址类型。

## IP 地址转换函数

字符串 IP -> 整数

下面三个函数可用于点分十进制字符串表示的 IPv4 地址和用网络字节序整数表示的 IPv4 之间的转换：

```c
#include <arpa/inet.h>
in_addr_t inet_addr(const char *cp);
int inet_aton(const char *cp, struct in_addr *inp);
char *inet_ntoa(struct in_addr in);
```

新函数： 同时适用 IPv4 和 IPv6 地址。

+ p: 点分十进制的 IP 字符串，
+ n: 表示 network 字节序的整数

+ af: 地址族： AF_INET, AFF_INET6
+ src: 需要转换的点分十进制的IP字符串
+ dst: 转换后的结果保存在这个里面
+ size: 第三个参数的大小（数组的大小）

```c
#include <arpa/inet.h>
int inet_pton(int af, const char*src, void *dst);
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size); 
// 返回值：返回转换后的数据的地址（字符串），和 dst 是一样的。
```

```c
#include <stdio.h>
#include <arpa/inet.h>

int main() {

    // 创建一个字符串，点分十进制的IP地址字符串
    char buf[] = "192.168.1.4";

    unsigned int num = 0;

    // 将点分十进制的IP 字符串转换成网络字节序的整数
    inet_pton(AF_INET, buf, &num);

    printf("%s\n", buf);

    unsigned char *p = (unsigned char *) &num;
    printf("%d %d %d %d\n", *p, *(p + 1), *(p + 2), *(p + 3));

    // 将网络字节序的IP整数转换成点分十进制的 IP 字符串
    char ip[16] = "";
    const char* str = inet_ntop(AF_INET, &num,ip,16 );
    printf("str: %s\n", str);
    printf("=ip: %s\n", ip);
    printf("ip == str %d\n", ip == str);

    return 0;
}
```

## TCP 通信流程

TCP 和 UDP 传输层的协议。

+ UDP：用户数据报协议，面向无连接， 可以单播，多播，广播，面向数据报，不可靠。
+ TCP：传输控制协议，面向连接的，可靠的、基于字节流，仅支持单播传输。

+ 首部开销： UDP：8个字节，TCP：最少20个字节
+ 适用场景： UDP：实时应用（视频会议、直播），TCP： 可靠性的应用（文件传输）

### TCP 通信的流程

![](/images/tiny-web-server/chapter-4/TCP-communication-process.png)

服务器端（被动接受连接的角色）

+ 1、创建一个用于监听的套接字，监听有客户端的连接，套接字其实就是一个文件描述符。
+ 2、将监听的文件描述符和本地的IP和端口绑定（IP和端口就是服务器的地址信息），客户端连接服务器时使用IP和端口。
+ 3、设置监听，监听的fd开始工作。
+ 4、阻塞等待，当有客户端发起连接，解除阻塞，接受客户端连接，会得到一个和客户端通信的套接字。
+ 5、通信
+ 6、通信结束，断开连接

客户端（主动发起连接）

- 1、创建一个用于通信的套接字（fd）
- 2、连接服务器，需要指定连接的服务器的IP和端口
- 3、连接成功了，客户端和服务器通信
- 4、通信结束，断开连接

```c
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h> // 包含了这个头文件，上面两个就可以省略

/**
NAME
       socket - create an endpoint for communication

SYNOPSIS
       #include <sys/types.h>          
       #include <sys/socket.h>

       int socket(int domain, int type, int protocol);
       功能 创建一个套接字
       参数 domain 协议族: AF_INET: IPv4, AF_INET6: IPv6, AF_UNIX,AF_LOCAL: 本地套接字通信
       type: 通信过程中使用的协议类型: SOCK_STREAM: 流式协议, SOCK_DGRAM: 报式协议
       protocol: 具体的一个协议, 一般写0, 流式协议默认使用TCP, 报式协议默认使用 UDP.
       
RETURN VALUE
       On success, a file descriptor for the new socket is returned.  On error, -1 is returned, and errno is set appropriately.

NAME
       bind - bind a name to a socket

SYNOPSIS
       #include <sys/types.h>        
       #include <sys/socket.h>

       int bind(int sockfd, const struct sockaddr *addr,
                socklen_t addrlen);
       功能: 绑定, 将 fd 和本地的 IP 和端口进行绑定
       sockfd: 通过 socket 函数得到的文件描述符
       addr: 需要绑定的 socket 地址, 这个地址封装了 IP 和 端口号的信息
       addrlen: 第二个参数结构体占用的大小
       
NAME
       listen - listen for connections on a socket

SYNOPSIS
       #include <sys/types.h>        
       #include <sys/socket.h>

       int listen(int sockfd, int backlog);
       功能: 监听这个 Socket 上的连接
       - sockfd: 通过 socket() 函数得到的文件描述符
       - backlog: 未连接的和已连接的个数最大值, cat /proc/sys/net/core/somaxconn, 不能超过这个值, 一般设置为 5.
       
NAME
       accept, accept4 - accept a connection on a socket

SYNOPSIS
       #include <sys/types.h>       
       #include <sys/socket.h>

       int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
       功能: 接收客户端连接,默认是一个阻塞的函数,阻塞等待客户端的连接
       sockfd: 用于监听的文件描述符
       addr: 传出参数, 记录了连接成功后客户端的地址信息 (ip, addr)
       addrlen: 指定第二个参数的对应的大小
       返回值: 成功: 用于通信的文件描述符, -1 失败
       
NAME
       connect - initiate a connection on a socket

SYNOPSIS
       #include <sys/types.h>         
       #include <sys/socket.h>

       int connect(int sockfd, const struct sockaddr *addr,
                   socklen_t addrlen);
       功能: 客户端连接服务器
       参数: sockfd: 用于通信的文件描述符
       addr: 客户端要连接的服务器的地址信息
       addrlen: 第二个参数的内存大小
       
       ssize_t write(int fd, const void *buf, size_t count);
       ssize_t read(int fd, void *buf, size_t count);
*/
```

### TCP Server

```c
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {
    // 创建用于监听的套接字
    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        exit(-1);
    }
    // 绑定
    struct sockaddr_in so_addr;
    so_addr.sin_family = AF_INET;
    // inet_pton(AF_INET, "127.0.0.1", &so_addr.sin_addr.s_addr);
    so_addr.sin_addr.s_addr = INADDR_ANY;
    so_addr.sin_port = htons(9999);
    int ret = bind(lfd, (struct sockaddr *) &so_addr, sizeof(so_addr));
    if (-1 == ret) {
        perror("bind");
        exit(-1);
    }


    ret = listen(lfd, 8);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }

    // accept
    struct sockaddr_in client_addr;
    socklen_t len = sizeof(client_addr);
    int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);
    if (cfd == -1) {
        perror("error");
        exit(-1);
    }

    // output
    char client_ip[16];
    inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));

    unsigned short client_port = ntohs(client_addr.sin_port);
    printf("client ip is: %s, port is %d.\n", client_ip, client_port);


    while(1) {
        // get client's data
        char recv_buf[1024] = {0};
        int rlen = read(cfd, recv_buf, sizeof(recv_buf));
        if (rlen == -1) {
            perror("read");
            exit(-1);
        } else if (rlen > 0) {
            printf("recv client data: %s\n", recv_buf);
        } else if (rlen == 0) {
            printf("client closed...\n");
        }

        // send data to client.
        char *data = "hello, i am server.";
        write(cfd, data, strlen(data));

    }
    close(cfd);
    close(lfd);

    return 0;
}
```

### TCP Client

```c
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {

    int fd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    server_addr.sin_port = htons(9999);
    int ret = connect(fd, (struct sockaddr *) &server_addr, sizeof(server_addr));

    if (ret == -1) {
        perror("connect");
        exit(-1);
    }

    while (1) {
        // send data to server
        char *data = "hello, i am client.";
        write(fd, data, strlen(data));

        sleep(1);

        // recv data from server
        char recv_buf[1024] = {0};
        int len = read(fd, recv_buf, sizeof(recv_buf));
        if (len == -1) {
            perror("read");
            exit(-1);
        } else if (len > 0) {
            printf("recv server data: %s\n", recv_buf);
        } else if (len == 0) {
            printf("server closed...\n");
            break;
        }

    }
    close(fd);

    return 0;
}
```

## TCP 三次握手

TCP 是一种面向连接的单播协议，在发送数据之前，通信双方必须在彼此间建立一条连接。所谓的连接，其实是客户端和服务器的内存里保存的一份关于对方的信息，
如 IP 地址，端口号等。

TCP 可以看成是一种字节流，它会处理 IP 层或以下层的丢包、重复以及错误问题。在连接的建立过程中，双方需要交换一些连接的参数。这些参数可以放在
TCP 头部。

TCP 提供了一种可靠、面向连接、字节流、传输层的服务，采用三次握手建立一个连接。采用四次挥手来关闭一个连接。

![](/images/tiny-web-server/chapter-4/tcp-head.png)

三次握手的目的是保证双方互相之间建立了连接。

- ACK、SYN、FIN， 16 位的窗口序号

三次握手发生在客户端连接的时候，当调用 connect() , 底层会通过 TCP 协议进行三次握手。

- --> SYN = 1
- <-- ACK = 1，SYN = 1
- --> ACK = 1

双方互相确认对方可以收和发数据。

TCP 三次握手时序图

![](/images/tiny-web-server/chapter-4/tcp-three.png)

怎么确认发送的数据是完整的？怎么确认收到的数据的顺序？

- --> SYN=1, cseq=1000
- <-- ACK=1, sack=1001, SYN=1, sseq = 2000
- --> ACK=1, cack=2001
- --> 1001(100) cack=2001
- <-- sack=1101
- --> 1101(200) cack=2001
- <-- sack=1301

第一次握手：

1. 客户端将 SYN 标志位置设置为 1
2. 生成一个随机的 32 位序号，这个序号后面是可以携带的数据（数据的大小）

第二次握手：

1. 服务器端接收客户端的连接 ACK=1
2. 服务器会回发一个确认序号 ack = 客户端的序号 + 数据长度 + SYN/FIN（按一个字节计算）
3. 服务器会向客户端发起连接请求： SYN=1
4. 服务器会产生一个随机序号 seq = K

第三次握手：

1. 客户端应答服务器的连接请求 ACK = 1
2. 客户端收到了服务器端的数据： ack = 服务器端的序号 + 数据长度 + SYN/FIN（按一个字节算）

## 滑动窗口

滑动窗口 sliding window 是一种流量控制技术。早期的网络通信中，通信双方不会考虑网络的拥挤情况，直接发送数据。
由于大家不知道网络阻塞状况，同时发送数据，导致中间节点阻塞掉包，谁也发不了数据，所以就有了滑动窗口机制来解决此问题。
滑动窗口协议是用来改善吞吐量的一种技术，即容许发送放在接收任何应答之前传送附加的包。
接收方告诉发送方在某一时刻能送多少包（称窗口尺寸）。

TCP 窗口中采用滑动窗口来进行传输控制，滑动串口的大小意味着接收方还有多大的缓冲区可以用于接收数据。
发送方可以通过滑动窗口的大小来确定应该发送多少字节的数据。当滑动窗口是0时，发送方一般不能再发送数据报。

> 滑动窗口是TCP 中实现诸如 ACK 确认，流量控制，拥塞控制的承载结构。

窗口理解为缓冲区的大小。滑动窗口的大小会随着发送数据和接收数据而变换。通信双方都有发送缓冲区和接收缓冲区。

![](/images/tiny-web-server/chapter-4/slide-window.png)

+ 发送方缓冲区
    - 白色格子： 空闲的空间
    - 灰色格子： 数据已经发送出去了，但是海没有被接收
    - 紫色格子： 还没有发送出去的数据

+ 接收方的缓冲区
    - 白色格子： 空闲的空间
    - 紫色格子： 已经接收到的数据

![](/images/tiny-web-server/chapter-4/send-slide-window.png)

+ mss： Maximum segment size 一条数据的最大的数据量
+ win： 滑动窗口

FIN，和 SYN 标志位发送后，回复的 ACK 要加 1。

+ 客户端向服务器发起连接，客户端的滑动窗口是 4096, 一次发送的最大数据量是 1460
+ 服务器接收连接情况，告诉客户端服务器窗口大小是 6144, 一次发送的最大数据量是 1024
+ 第三次握手
+ 4-9 客户端连续给服务器发送了 6k 的数据，每次发送 1k
+ 10 服务器告诉客户端，发送的 6k 数据已经接收到，存储在缓冲区中，缓冲区已经处理了 2k，窗口大小是 2k
+ 11 服务器告诉客户端，发送的 6k 数据已经接收到，存储在缓冲区中，缓冲区已经处理了 4k，窗口大小是 4k
+ 12 客户端给服务器发送了 1k 的数据
+ 13 客户端主动请求和服务器断开连接，并且给服务器发送了1k的数据
+ 14 服务器回复 ACK 8194 a):同意断开连接请求，b): 告诉客户端已经接收到刚才的2k的数据，滑动窗口是2k。
+ 15-16 通知客户端滑动窗口的大小
+ 17 第三次挥手，服务器端给客户端发送 FIN，请求断开连接
+ 18 第四次挥手，客户端同意了断开服务器的连接。

## TCP 四次挥手

![](/images/tiny-web-server/chapter-4/tcp-4.png)

四次挥手是发生在断开连接的时候，在程序中当调用了 close(), 会使用 TCP 协议进行四次挥手。
客户端和服务器端都可以主动发起断开连接，谁先调用 close() 就是谁发起。
因为在 TCP 连接的时候，采用三次握手建立的连接是双向的，在断开的时候也需要双向断开。

+ --> FIN = 1;
+ <-- ACK = 1;
+ <-- FIN = 1;
+ --> ACK = 1;

## 多进程实现并发服务器

要实现 TCP 通信服务器处理并发的任务，使用多线程或者多进程来解决。

思路：

1. 一个父进程，多个子进程。
2. 父进程负责等待并接受客户端的连接
3. 子进程：完成通信，接收一个客户端，就创建一个子进程用于通信。

多进程服务器端程序，断开连接后要回收资源。

```c
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>
#include <wait.h>
#include <errno.h>

void recyle_child(int arg) {
    while (1) {
        int ret = waitpid(-1, NULL, WNOHANG);
        if (ret == -1) { // 所有子进程都回收了
            break;
        } else if (ret == 0) { // 还有子进程活着
            break;
        } else if (ret > 0) { // 被回收了
            printf("子进程 %d 被回收了\n", ret);
        }
    }
}

int main() {
    struct sigaction act;
    act.sa_flags = 0;
    sigemptyset(&act.sa_mask);
    act.sa_handler = recyle_child;
    // 注册信号捕捉
    sigaction(SIGCHLD, &act, NULL);


    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    server_addr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(lfd, (struct sockaddr *) &server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("bind");
        exit(0);
    }

    // 监听
    ret = listen(lfd, 128);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 不断循环，等待客户端连接
    while (1) {
        // 接受连接
        struct sockaddr_in client_addr;
        int s_len = sizeof(client_addr);
        int cfd = accept(lfd, (struct sockaddr *) &client_addr, &s_len);
        if (cfd == -1) {
            if (errno == EINTR) {
                continue;
            }
            perror("accept");
            exit(-1);
        }

        // 每一个连接进来，创建一个子进程跟客户端通信。
        pid_t pid = fork();
        if (pid == 0) {
            // 子进程
            // 获取子进程的信息
            char client_ip[16];
            inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
            unsigned short client_port = ntohs(client_addr.sin_port);

            printf("client ip is: %s, port is %d\n", client_ip, client_port);

            // 接收客户端发来的数据
            char recvBuf[1024] = {0};
            while (1) {
                int len = read(cfd, &recvBuf, sizeof(recvBuf));
                if (len == -1) {
                    perror("read");
                    exit(-1);
                } else if (len > 0) {
                    printf("recv client data: %s\n", recvBuf);
                } else if(len == 0){
                    printf("client closed...\n");
                    break;
                }
                write(cfd, recvBuf, strlen(recvBuf) + 1);
            }

            close(cfd);
            exit(0); // 退出当前子进程
        };
    }

    close(lfd);
    return 0;
}
```

客户端程序

```c
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {

    int fd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    server_addr.sin_port = htons(9999);
    int ret = connect(fd, (struct sockaddr *) &server_addr, sizeof(server_addr));

    if (ret == -1) {
        perror("connect");
        exit(-1);
    }

    char recv_buf[1024] = {0};
    int i = 0;
    while (1) {
        // send data to server
        // char *data = "hello, i am client.";
        sprintf(recv_buf, "data: %d", i++);
        write(fd, recv_buf, strlen(recv_buf) + 1);

        // recv data from server
        int len = read(fd, recv_buf, sizeof(recv_buf));
        if (len == -1) {
            perror("read");
            exit(-1);
        } else if (len > 0) {
            printf("recv server data: %s\n", recv_buf);
        } else if (len == 0) {
            printf("server closed...\n");
            break;
        }

//        sleep(1);
        usleep(1000);
    }
    close(fd);

    return 0;
}
```

## 多线程实现并发服务器

```c
//
// Created by larry on 23-4-12.
//

#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <errno.h>
#include <pthread.h>
#include <stdlib.h>
#include <string.h>

struct sockInfo {
    int fd;
    pthread_t tid; // 线程号
    struct sockaddr_in addr;
};


struct sockInfo sock_infos[128];

void *working(void *arg) {
    // 子线程和客户端通信 cfd, 客户端的信息

    struct sockInfo *p_info = (struct sockInfo *) arg;
    struct sockaddr_in client_addr = p_info->addr;
    int cfd = p_info->fd;

    // 获取子进程的信息
    char client_ip[16];
    inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, client_ip, sizeof(client_ip));
    unsigned short client_port = ntohs(client_addr.sin_port);

    printf("client ip is: %s, port is %d\n", client_ip, client_port);

    // 接收客户端发来的数据
    char recvBuf[1024] = {0};
    while (1) {
        int len = read(cfd, &recvBuf, sizeof(recvBuf));
        if (len == -1) {
            perror("read");
            exit(-1);
        } else if (len > 0) {
            printf("recv client data: %s\n", recvBuf);
        } else if (len == 0) {
            printf("client closed...\n");
            break;
        }
        write(cfd, recvBuf, strlen(recvBuf) + 1);
    }

    close(cfd);


    return NULL;
}

int main() {

    int lfd = socket(AF_INET, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        exit(-1);
    }
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    server_addr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(lfd, (struct sockaddr *) &server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("bind");
        exit(0);
    }
    // 监听
    ret = listen(lfd, 128);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }

    // 初始化 数据
    int max_infos = sizeof(sock_infos) / sizeof(sock_infos[0]);
    for (int i = 0; i < max_infos; i++) {
        bzero(&sock_infos[i], sizeof(sock_infos[i]));
        sock_infos[i].fd = -1;
        sock_infos[i].tid = -1;
    }

    // 循环等待客户端连接，一旦一个客户端连接进来，就创建一个子线程进行通信
    while (1) {
        // 接受连接
        struct sockaddr_in client_addr;
        int s_len = sizeof(client_addr);
        int cfd = accept(lfd, (struct sockaddr *) &client_addr, &s_len);
        if (cfd == -1) {
            if (errno == EINTR) {
                continue;
            }
            perror("accept");
            exit(-1);
        }

        struct sockInfo *p_info;
        for (int i = 0; i < max_infos; i++) {
            // 从数组中找到一个可用的 sockInfo 元素
            if (sock_infos[i].fd == -1) {
                p_info = &sock_infos[i];
                break;
            }
            if (i == max_infos - 1) {
                sleep(1);
                i--;
            }
        }

        p_info->fd = cfd;
        memcpy(&p_info->addr, &client_addr, s_len);


        // pthread_t tid;
        pthread_create(&p_info->tid, NULL, working, p_info);

        pthread_detach(p_info->tid);
    }
    close(lfd);

    return 0;
}
```

## TCP状态转换

![](/images/tiny-web-server/chapter-4/tcp-status-turn-1.png)
![](/images/tiny-web-server/chapter-4/tcp-status-turn-2.png)

进入 TIME_WAIT 状态后要等待 2MSL （Maximum Segment Lifetime)

主动断开连接的一方，最后进出入一个 TIME_WAIT 状态，这个状态会持续 2msl。
msl： 官方建议：2分钟，实际是30秒。

> 当 TCP 连接主动关闭方接收到被动关闭方发送的 FIN 和最终的 ACK 后，连接的主动关闭方必须处于
> TIME_WAIT 状态并持续 2 MSL 时间。
> 这样就能够让 TCP 连接的主动关闭方在它发送的 ACK 丢失的情况下重新发送最终的 ACK。
> 主动关闭方重新发送的最终 ACK 并不是因为被动关闭方重传了 ACK （它们并不消耗序列号，被动关闭方也不会重传），
> 而是因为被动关闭方重传了它的 FIN。事实上，被动关闭方总是重传 FIN 知道它收到一个最终的 ACK。

半关闭

> 当 TCP 连续中 A 向 B 发送 FIN 请求关闭，另一端 B 回应 ACK 之后 （A 端进入 FIN_WAIT_2 状态），
> 并没有立即发送 FIN 给 A，A 方处于半连接状态（半开关），
> 此时 A 可以接收 B 发送的数据，但是 A 已经不能再向 B 发送数据。

灵魂拷问：

1. 第一次挥手丢失，会发生什么？
   答：主动断开方重传FIN请求报文
2. 第二次挥手丢失，会发生什么？
   答：由于ACK报文不会重传，所以主动断开方会重传FIN报文
3. 第三次挥手丢失，会发生什么？
   答：被动断开方会重传FIN+ACK报文
4. 第四次挥手丢失，会发生什么？
   答：同第三次挥手

## 半关闭

从程序的角度，可以使用 API 来控制实现半关闭连接的状态：

```c
#include <sys/socket.h>
int shutdown(int sockfd, int how);
// sockfd: 需要关闭的 socket 的描述符
// how: 允许为 shutdown 操作选择以下集中方式
```

+ SHUT_RD(0): 关闭 sockfd 上的读功能，此选项将不允许 sockfd 进行读操作。该套接字不再接收数据，任何当前在套接字接收缓冲区的数据将被无声的丢弃掉。
+ SHUT_WR(1): 关闭 sockfd 的写功能，此选项将不允许 sockfd 进行写操作。进程不能在对此套接字发出写操作。
+ SHUT_RDWR(2): 关闭 sockfd 的读写功能。相当于调用 shutdown 两次，首先是以 SHUT_RD， 然后是 SHUT_WR。

使用 close 中止一个连接，但它只是减少描述符的引用计数，并不直接关闭连接，只有当描述符的引用计数为 0 的时候才关闭连接。
shutdown 不考虑描述符的引用计数，
直接关闭描述符。也可以选择中止一个方向的连接，只中止读或只中止写。

注意：

1. 如果多个进程共享一个套接字， close 每被调用一次，计数减1,直到计数为0时，也就是所用进程都调用了 close，套接字将被释放。
2. 在多进程中如果一个进程调用了 shutdown(sfd, SHUT_RDWR) 后，其它的进程将无法进行通信，但如果一个进程 close(sfd)
   将不会影响到其他进程。

## 端口复用

常看网络相关信息的命令

```shell
netstat
# 参数
# -a 所有的 socket
# -p 显示正在使用 socket 的程序的名称
# -n 直接使用 IP 地址，而不通过域名服务器
# -l listening
# -t tcp
```

端口复用最常见的用途是：

+ 防止服务器重启时之前绑定的端口还未释放
+ 程序突然退出而系统还没有释放端口

```c
#include <sys/types.h>
#include <sys/socket.h>
// 设置套接字的属性（不仅仅能设置端口复用）
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

// 使用
setsockopt();
bind();

//
int optval = 1;
setsockopt(lfd, SOL_SOCKET, SO_REUSEPORT, &optval, sizeof(optval));
```

参数：

- sockfd: 要操作的文件描述符
- level: 级别 SOL_SOCKET (端口复用的级别)
- optname: 选项的名称 SO_REUSEADDR, SO_REUSEPORT
- optval: 端口复用的值（整型） 1： 可以复用， 0 不可以复用
- optlen：optval 参数的大小

端口复用，设置时机是在服务器绑定端口之前。

## IO多路复用(IO多路转接)

I/O 多路复用使得程序能够同时监听多个文件描述符，能够提高程序的性能，Linux 下实现IO多路复用的系统调用主要有
select，poll，epoll。

### 阻塞

监听阻塞等待：

+ 好处：不占用 CPU 宝贵的时间片
+ 缺点：同一时刻只能处理一个操作，效率低下

多线程或多进程解决多个客户端访问

+ 缺点： 线程或者进程会消耗资源，线程或进程调度会消耗 CPU 资源

BIO 模型： 每一个线程或者进程对应一个 Client

```c
int lfd = socket();
bind(lfd,...);
listen(lfd,...);
int cfd = accept(lfd,...); // 阻塞
read(cfd,...); // 阻塞
recv(cfd,...);
```

### 非阻塞，忙轮询

+ 优点：提高论程序的执行效率
+ 缺点：需要占用更多的 CPU 和系统资源

NIO 模型： 当有很多客户端连接的时候，需要多次循环遍历，每个循环内 O(n) 的系统调用

```c
int lfd = socket();
bind(lfd,...);
listen(lfd,...);
int cfd = accept(lfd,...); // 非阻塞
read(cfd,...); // 非阻塞
recv(cfd,...);
```

IO 多路转接技术

第一种： select / poll

select 代收员比较懒，它只会告诉你有几个快递到了，但是哪个快递，需要挨个遍历一遍。

第二种： epoll

epoll 代收快递员很勤快，不仅会告诉有几个快递到了，还会告诉是那个快递公司的。

## select_API介绍

1. 首先构造一个关于文件描述符的列表，将要监听的文件描述符添加到该列表中。
2. 调用一个系统函数，监听该列表中的文件描述符直到这些描述符中的一个或者多个进行了 IO 操作时，该函数才返回。
    - 该函数是阻塞的
    - 函数对文件描述符的检测的操作是由内核完成的。
3. 在返回时，它会告诉进程有多少（哪些）描述符要进行 IO 操作。

select

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/selcet.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
// nfds: 委托内核检测的最大文件描述符加1
// readfds： 要检测的文件描述符的读的集合，委托内核检测哪些文件描述符读的属性。
//           - 一般检测读操作
//           - 对应的是对方发送过来的数据，因为读是被动的接收数据，检测的就是读缓冲区
//           - 是一个传入传出参数
// writefd：要检测的文件描述符的写的集合，委托内核检测哪些文件描述符的写的属性
//            - 委托内核检测写缓冲区是不是还可以写数据（不满的就可以写）
// exceptfds：检测发生异常的文件描述符的集合
// timeout: 设置的超时时间
//              - null： 永久阻塞，直到检测到了文件描述符有变化
//              - = 0: 不阻塞
//              - > 0: 阻塞对应时间
// 返回值 -1： 调用失败, > 0(n) : 检测的集合中有 n 个文件描述符发生了变化

// 减参数文件描述符fd对应的标志位设置为0
void FD_CLR(int fd, fd_set *set);

// 判断fd 对应的标志位是 0 还是 1, fd 对应的标志位的值，返回 0 或 1。
int FD_ISSET(int fd, fd_set *set);

// 将参数文件描述符 fd 对应的标志位设置为 1
void FD_SET(int fd, fd_set *set);

// fd_set 一共有 1024 位，全部设置为 0。
void FD_ZERO(fd_set *set);

// sizeof(fd_set) = 128; 1024

fd_set read;
FD_SET(3, &reads);
FD_SET(4, &reads);
FD_SET(100, &reads);
FD_SET(101, &reads);

select(101+1, &reads, NULL, NULL, NULL);
```

示例程序
```c
//
// Created by larry on 23-4-15.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <sys/select.h>

int main() {

    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in s_addr;
    s_addr.sin_port = htons(9999);
    s_addr.sin_family = AF_INET;
    s_addr.sin_addr.s_addr = INADDR_ANY;

    // bind
    bind(lfd, (struct sockaddr *) &s_addr, sizeof(s_addr));

    // listen;
    listen(lfd, 8);

    // create fd_sets
    fd_set rd_set, tmp;
    FD_ZERO(&rd_set);
    FD_SET(lfd, &rd_set);
    int max_fd = lfd;

    while (1) {
        tmp = rd_set;

        // 调用 select 系统函数，让系统内核帮检测哪些文件描述符有数据
        int ret = select(max_fd + 1, &tmp, NULL, NULL, NULL);
        if (ret == -1) {
            perror("select");
            exit(-1);
        } else if (ret == 0) {
            continue;
        } else if (ret > 0) { // 说明检测到了有文件描述符对应的缓冲区的数据发生论改变
            if (FD_ISSET(lfd, &tmp)) {  // 表示有新的客户端连接进来
                struct sockaddr_in client_addr;
                int len = sizeof(client_addr);
                int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);

                FD_SET(cfd, &rd_set); // 将新的文件描述符加入到集合中
                max_fd = max_fd > cfd ? max_fd : cfd; // 更新最大的文件描述符
            }

            for (int i = lfd + 1; i <= max_fd; i++) {
                if (FD_ISSET(i, &tmp)) {
                    // 说明客户端发来了数据
                    char buf[1024] = {0};
                    int len = read(i, &buf, sizeof(buf));
                    if (len == -1) {
                        perror("read");
                        exit(-1);
                    } else if (len == 0) {
                        printf("client closed...\n");
                        close(i);
                        FD_CLR(i, &rd_set);
                    } else if (len > 0) {
                        printf("read buf = %s\n", buf);
                        write(i, buf, strlen(buf) + 1);
                    }

                }
            }
        }
    }
    close(lfd);
    return 0;
}
```

select 的缺点
1. 每次调用 select， 都需要把 fd 集合从用户态拷贝到内核态，这个开销在 fd 很多的时候会很大
2. 同时每次调用 select 都需要在内核遍历传递进来的所有 fd， 这个开销在 fd 很多的时也很大
3. select 支持的文件描述符数量太小论，默认是 1024,
4. fds 集合不能重用，每次都需要重置。

## poll

```c
#include <poll.h>
struct pollfd{
    int fd;        // 委托内核检测的文件描述符
    short evects;  // 委托内核检测文件描述符的什么事件
    short revents; // 文件描述符实际发生的事件
};

int poll(struct pollfd *fds, nfds_t nfds, int timeout);
// 参数： fds，是一个 struct pollfd 结构体数组，这是一个需要检测的文件描述符符的集合
//          - nfds, 这个是第一个参数数组中最后一个有效元素的下标加一
//          - timeout： 阻塞时长，0 不阻塞， -1 阻塞，当检测到需要检测的文件描述符有变化，解除阻塞，> 0 阻塞时长
// 返回值： -1 失败， 0（n）成功，表示检测到集合中有n个文件描述符发生变化
```

示例程序
```c
//
// Created by larry on 23-4-15.
//

#include <stdio.h>
#include <arpa/inet.h>

#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <sys/select.h>
#include <poll.h>

int main() {

    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in s_addr;
    s_addr.sin_port = htons(9999);
    s_addr.sin_family = AF_INET;
    s_addr.sin_addr.s_addr = INADDR_ANY;

    // bind
    int ans = bind(lfd, (struct sockaddr *) &s_addr, sizeof(s_addr));
    if(ans == -1){
        perror("bind");
        exit(-1);
    }

    // listen;
    listen(lfd, 8);

    // create poll
    struct pollfd fds[1024];
    for (int i = 0; i < 1024; i++) {
        fds[i].fd = -1;
        fds[i].events = POLLIN;
    }
    fds[0].fd = lfd;
    int nfds = 0;

    while (1) {
        // 调用 poll 系统函数，让系统内核帮检测哪些文件描述符有数据
        int ret = poll(fds, nfds + 1, -1);

        if (ret == -1) {
            perror("poll");
            exit(-1);
        } else if (ret == 0) {
            continue;
        } else if (ret > 0) { // 说明检测到了有文件描述符对应的缓冲区的数据发生论改变
            if (fds[0].revents & POLLIN) {
                struct sockaddr_in client_addr;
                int len = sizeof(client_addr);
                int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);

                for (int i = 1; i < 1024; i++) {
                    if (fds[i].fd == -1) {
                        fds[i].fd = cfd;
                        fds[i].events = POLLIN;
                        break;
                    }
                }
                nfds = nfds > cfd ? nfds : cfd;
            }

            for (int i = 1; i <= nfds; i++) {
                if (fds[i].revents & POLLIN) {    // 说明客户端发来了数据
                    char buf[1024] = {0};
                    int len = read(fds[i].fd, &buf, sizeof(buf));
                    if (len == -1) {
                        perror("read");
                        exit(-1);
                    } else if (len == 0) {
                        printf("client closed...\n");
                        close(fds[i].fd);
                        fds[i].fd = -1;
                    } else if (len > 0) {
                        printf("read buf = %s\n", buf);
                        write(fds[i].fd, buf, strlen(buf) + 1);
                    }

                }
            }

        }
    }

    close(lfd);

    return 0;

}
```



## epoll

```c
int lfd = socket();
bind(lfd, );
listen(lfd, );
int cfd = accept(lfd, ...);
int epfd = epoll_create(2000); // eventpoll (rbr, rdlist)

struct eventpoll{
    ...
    struct rb_root rbr; // 红黑树
    struct list_head rdlist; // 双链表
    ...
};
struct epoll_event ev;
ev.events = EPOLLIN;
ev.data.fd = lfd;

epoll_ctl(epfd, EPOLL_CTL_ADD, lfd, &ev);

int ret = epoll_wait(epfd,...);
for(...){ read();};
```

```c
#include <sys/epoll.h>

// 创建一个新的 epoll 实例，在内核中创建论一个数据，这个数据中有两个比较重要的数据，
// 一个是需要检测的文件描述符的信息（红黑树），
// 还有一个就绪列表，存放检测到数据发送改变的文件描述符信息（双向链表）。
// 参数： size 目前没有意义了，随便写一个数。必须大于零。
// 返回值： -1 失败， >0： 文件描述符，操作 epoll 实例。
int epoll_create(int size);
typedef union epoll_data{
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

struct epoll_event{
    uint32_t events;
    epoll_data_t data;
};
// 常见 epoll 检测事件
// EPOLLIN, EPOLLOUT, EPOLLERR

// 用来对 epoll 实例进行管理，添加文件描述符信息，删除信息，修改信息。
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// 参数： epfd： epollfd 实例对应的文件描述符
// op 要进行什么操作： EPOLL_CTL_ADD， EPOLL_ctl_MOD 修改, EPOLL_CTL_DEL
// event 检测文件描述符什么事情

// 检测函数
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
// 参数： epfd epoll 实例对应的文件描述符
// events： 传出参数， 保存了发生论变化的文件描述符的信息
// maxevents： 第二个参数结构体数组的大小
// timeout： 阻塞时间， 0 不阻塞， -1 阻塞， 直到检测到 fd 有数据发生变化，解除阻塞， >0 阻塞的时长，毫秒
// 返回值： 成功返回发送变化的文件描述符的个数 > 0, 失败 -1
```

示例程序
```c
//
// Created by larry on 23-4-16.
//
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/epoll.h>
#include <arpa/inet.h>
#include <stdlib.h>

int main() {

    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in s_addr;
    s_addr.sin_port = htons(9999);
    s_addr.sin_family = AF_INET;
    s_addr.sin_addr.s_addr = INADDR_ANY;

    // bind
    int ans = bind(lfd, (struct sockaddr *) &s_addr, sizeof(s_addr));
    if (ans == -1) {
        perror("bind");
        exit(-1);
    }

    // listen;
    listen(lfd, 8);

    // create epoll
    int efd = epoll_create(1);

    // 将监听号的文件描述符相关的检测信息添加到 epoll
    struct epoll_event epev;
    epev.events = EPOLLIN | EPOLLOUT; // EPOLLIN | EPOLLOUT
    epev.data.fd = lfd;
    epoll_ctl(efd, EPOLL_CTL_ADD, lfd, &epev);

    struct epoll_event epevs[1024];

    while (1) {
        int ret = epoll_wait(efd, epevs, 1024, -1);
        if (ret == -1) {
            perror("epoll_wait");
            exit(-1);
        }

        printf("ret = %d\n", ret);

        for (int i = 0; i < ret; ++i) {
            if(epevs[i].events & EPOLLOUT){
                continue;
            }
            int cur_fd = epevs[i].data.fd;
            if (cur_fd == lfd) { // 监听的文件描述符有数据到达，有客户端连接
                struct sockaddr_in client_addr;
                int len = sizeof(client_addr);
                int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);

                epev.events = EPOLLIN;
                epev.data.fd = cfd;
                epoll_ctl(efd, EPOLL_CTL_ADD, cfd, &epev);
            } else { // 有数据到达，需要通信 // 说明客户端发来了数据
                char buf[1024] = {0};
                int len = read(cur_fd, &buf, sizeof(buf));
                if (len == -1) {
                    perror("read");
                    exit(-1);
                } else if (len == 0) {
                    printf("client closed...\n");
                    close(cur_fd);
                    epoll_ctl(efd, EPOLL_CTL_DEL, cur_fd, NULL);
                } else if (len > 0) {
                    printf("read buf = %s\n", buf);
                    write(cur_fd, buf, strlen(buf) + 1);
                }
            }
        }
    }

    close(lfd);
    close(efd);
    return 0;
}
```

## EPOLL的工作模式

+ LT 模式 （水平触发）
> LT (level-triggered) 缺省的工作方式，并且同时支持 block 和 no-block socket。
> 在这种做法中，内核会告诉你一个文件描述符是否就绪论，然后你就可以对这个就绪的fd进行IO操作，如果你不作任何操作，
> 内核还是会继续通知你。

1. 假设委托内核检测读事件 --> 检测 fd 的读缓冲区
2. 读缓冲区有数据 --> epoll 检测到了会给用户通知

+ ET 模式 （边沿触发）
> ET （edge-triggered） 是告诉工作方式，只支持 no-block socket。 在这个模式下，当描述符从未就绪变为就绪时，内核通过 epoll 告诉你。
> 然后它会假设你知道文件描述符已经就绪，并且不会再为哪个文件描述符发送更多的就绪通知，知道你做了某些操作导致哪些文件描述符不再为就绪状态了。
> 但是请注意，如果一直不对这个fd作IO操作（从而导致它再次变成未就绪），内核不会发送更多的通知（only once）。
> 
> ET 模式在很大程度上减少论 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。epoll 工作在 ET 模式的时候，必须使用非阻塞套接口，
> 以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

1. 假设委托内核检测读事件 --> 检测 fd 的读缓冲区
2. 读缓冲区有数据 --> epoll 检测到了会给用户通知，只通知一次

+ EPOLL LT 模式示例代码

```c
//
// Created by larry on 23-4-16.
//
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/epoll.h>
#include <arpa/inet.h>
#include <stdlib.h>

int main() {

    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in s_addr;
    s_addr.sin_port = htons(9999);
    s_addr.sin_family = AF_INET;
    s_addr.sin_addr.s_addr = INADDR_ANY;

    // bind
    int ans = bind(lfd, (struct sockaddr *) &s_addr, sizeof(s_addr));
    if (ans == -1) {
        perror("bind");
        exit(-1);
    }

    // listen;
    listen(lfd, 8);

    // create epoll
    int efd = epoll_create(1);

    // 将监听号的文件描述符相关的检测信息添加到 epoll
    struct epoll_event epev;
    epev.events = EPOLLIN | EPOLLOUT; // EPOLLIN | EPOLLOUT
    epev.data.fd = lfd;
    epoll_ctl(efd, EPOLL_CTL_ADD, lfd, &epev);

    struct epoll_event epevs[1024];

    while (1) {
        int ret = epoll_wait(efd, epevs, 1024, -1);
        if (ret == -1) {
            perror("epoll_wait");
            exit(-1);
        }

        printf("ret = %d\n", ret);

        for (int i = 0; i < ret; ++i) {
            if(epevs[i].events & EPOLLOUT){
                continue;
            }
            int cur_fd = epevs[i].data.fd;
            if (cur_fd == lfd) { // 监听的文件描述符有数据到达，有客户端连接
                struct sockaddr_in client_addr;
                int len = sizeof(client_addr);
                int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);

                epev.events = EPOLLIN;
                epev.data.fd = cfd;
                epoll_ctl(efd, EPOLL_CTL_ADD, cfd, &epev);
            } else { // 有数据到达，需要通信 // 说明客户端发来了数据
                char buf[5] = {0};
                int len = read(cur_fd, &buf, sizeof(buf));
                if (len == -1) {
                    perror("read");
                    exit(-1);
                } else if (len == 0) {
                    printf("client closed...\n");
                    close(cur_fd);
                    epoll_ctl(efd, EPOLL_CTL_DEL, cur_fd, NULL);
                } else if (len > 0) {
                    printf("read buf = %s\n", buf);
                    write(cur_fd, buf, strlen(buf) + 1);
                }
            }
        }
    }

    close(lfd);
    close(efd);
    return 0;
}
```

+ EPOLL ET 模式示例代码

```c
//
// Created by larry on 23-4-16.
//
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/epoll.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>

int main() {

    int lfd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in s_addr;
    s_addr.sin_port = htons(9999);
    s_addr.sin_family = AF_INET;
    s_addr.sin_addr.s_addr = INADDR_ANY;

    // bind
    int ans = bind(lfd, (struct sockaddr *) &s_addr, sizeof(s_addr));
    if (ans == -1) {
        perror("bind");
        exit(-1);
    }

    // listen;
    listen(lfd, 8);

    // create epoll
    int efd = epoll_create(1);

    // 将监听号的文件描述符相关的检测信息添加到 epoll
    struct epoll_event epev;
    epev.events = EPOLLIN | EPOLLOUT; // EPOLLIN | EPOLLOUT
    epev.data.fd = lfd;
    epoll_ctl(efd, EPOLL_CTL_ADD, lfd, &epev);

    struct epoll_event epevs[1024];

    while (1) {
        int ret = epoll_wait(efd, epevs, 1024, -1);
        if (ret == -1) {
            perror("epoll_wait");
            exit(-1);
        }

        printf("ret = %d\n", ret);

        for (int i = 0; i < ret; ++i) {
            int cur_fd = epevs[i].data.fd;
            if (cur_fd == lfd) { // 监听的文件描述符有数据到达，有客户端连接
                struct sockaddr_in client_addr;
                int len = sizeof(client_addr);
                int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);

                // 设置 cfd 非阻塞属性
                int flag = fcntl(cfd, F_GETFL);
                flag |= O_NONBLOCK;
                fcntl(cfd, F_SETFL, flag);

                epev.events = EPOLLIN | EPOLLET; // 设置边沿触发
                epev.data.fd = cfd;
                epoll_ctl(efd, EPOLL_CTL_ADD, cfd, &epev);
            } else { // 有数据到达，需要通信 // 说明客户端发来了数据
                if (epevs[i].events & EPOLLOUT) {
                    continue;
                }
                char send_buf[1024];
                char recv_buf[5] = {0};
                int len = 0;
                int send_n = 0;
                while (1) {
                    memset(recv_buf, 0, sizeof(recv_buf));
                    int len = read(cur_fd, &recv_buf, sizeof(recv_buf) - 1);
                    if (len > 0) {
                        printf("recv client data: %s, len = %d\n", recv_buf, len);
                        memcpy(&send_buf[send_n], recv_buf, len);
                        send_n += len;
                    } else if (len == -1) {
                        if (errno == EAGAIN) {
                            printf("recv over...\n");
                            write(cur_fd, &send_buf, send_n);
                            break;
                        }
                        perror("read");
                        exit(-1);
                    } else if (len == 0) {
                        printf("client close fd: %d\n", cur_fd);
                        epoll_ctl(efd, EPOLL_CTL_DEL, cur_fd, NULL);
                        close(cur_fd);
                        continue;
                    }
                }
            }
        }
    }

    close(lfd);
    close(efd);
    return 0;
}
```

+ 所用 client 的代码

```c
//
// Created by larry on 23-4-8.
//

// TCP client

// #include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);
    server_addr.sin_port = htons(9999);
    int ret = connect(fd, (struct sockaddr *) &server_addr, sizeof(server_addr));

    if (ret == -1) {
        perror("connect");
        exit(-1);
    }

    char recv_buf[1024] = {0};
    int i = 0;
    while (1) {
        fgets(recv_buf, sizeof(recv_buf), stdin);
//        sprintf(recv_buf, "data: %d", i++);
        write(fd, recv_buf, strlen(recv_buf) + 1); // send data to server

        // recv data from server
        int len = read(fd, recv_buf, sizeof(recv_buf));
        if (len == -1) {
            perror("read");
            exit(-1);
        } else if (len > 0) {
            printf("recv server data: %s\n", recv_buf);
        } else if (len == 0) {
            printf("server closed...\n");
            break;
        }

//        sleep(1);
//        usleep(1000);
    }
    close(fd);

    return 0;
}
```

## UDP通信

![](/images/tiny-web-server/chapter-4/udp-communication.png)

```c
#include <sys/types.h>
#include <sys/socket.h>
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
// 参数： sockfd 通信的fd
// buf： 要发送的数据
// len： 要发送数据的长度
// flags： 0
// dest_addr： 通信的另外一端的地址信息
// addrlen: 地址的内存大小

ssize_t recvform(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
// 参数 sockfd： 通信的fd
// buf： 接收数据的数组
// len： 数组的大小
// flags： 0
// src_addr： 用来保存另外一端的地址信息，不需要可以指定 null
// addrlen： 地址的内存大小
```

+ UDP 通信客户端代码

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1) {
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);

    int num = 0;
    while (1) {
        char send_buf[128];
        sprintf(send_buf, "hello, i am client %d\n", num++);
        sendto(fd, send_buf, strlen(send_buf) + 1, 0, (struct sockaddr*)&server_addr, sizeof(server_addr));

        int ret  = recvfrom(fd, send_buf, sizeof(send_buf), 0, NULL, NULL);
        printf("server say: %s\n", send_buf);
        sleep(1);
    }
    close(fd);
    return 0;
}
```

+ UDP 通信服务器端代码

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1) {
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    addr.sin_addr.s_addr = INADDR_ANY;
    int ret = bind(fd, &addr, sizeof(addr));
    if (ret == -1) {
        perror("bind");
        exit(-1);
    }

    while (1) {
        char buf[128] = {0};
        char ip_buf[16];
        struct sockaddr_in client_addr;
        int len = sizeof(client_addr);
        int num = recvfrom(fd, buf, sizeof(buf), 0, (struct sockaddr *) &client_addr, &len);

        printf("client ip: %s, port: %d\n",
               inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, ip_buf, sizeof(ip_buf)),
               ntohs(client_addr.sin_port));

        printf("client say: %s\n", buf);

        sendto(fd, buf, strlen(buf) + 1, 0, (struct sockaddr *) &client_addr, sizeof(client_addr));
    }
    close(fd);
    return 0;
}
```

## 广播

> 向子网中多台计算机发送消息，并且子网中所有的计算机都可以接收到发送方发送的消息，每个广播消息都包含一个特殊的IP地址，
> 这个IP中子网内主机标志部分的二进制全部为1。

+ 只能在局域网中使用。
+ 客户端需要绑定服务器广播使用端端口，才可以接收到广播消息。

```c
// 设置广播属性的函数
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
// 参数： sockfd： 文件描述符
// level: SOL_SOCKET
// optname: SO_BROADCAST
// optval: int 类型的值， 1 表示允许广播
// optlen： optval 的大小
```

+ 广播 客户端示例代码

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1) {
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    server_addr.sin_addr.s_addr = INADDR_ANY;
    // inet_pton(AF_INET, " ", &server_addr.sin_addr.s_addr);

    int ret = bind(fd, (struct sockaddr*)&server_addr, sizeof (server_addr));
    if(ret == -1){
        perror("read");
        exit(-1);
    }

    int num = 0;
    while (1) {
        char buf[128];
        int ans = recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("server say: %s\n", buf);
    }
    close(fd);
    return 0;
}
```

+ 广播 服务器端示例代码

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 设置广播属性
    int op = 1;
    setsockopt(fd, SOL_SOCKET, SO_BROADCAST, &op, sizeof(op));

    // 创建广播的地址
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    inet_pton(AF_INET, "xx,xx,xx.255",& addr.sin_addr.s_addr);

    int num = 0;
    while (1) {
        char send_buf[128] = {0};
        sprintf(send_buf, "hello, client....%d\n", num++);

        sendto(fd, send_buf, strlen(send_buf) + 1, 0, (struct sockaddr *) &addr, sizeof(addr));
        printf("广播的数据： %s\n", send_buf);
        sleep(1);
    }
    close(fd);
    return 0;
}
```

##  组播(多播)

> 单播地址标识单个 IP 接口，广播地址标识某个子网的所有 IP 接口，多播地址标识一组 IP 接口。单播和广播是寻址方案的两个极端（要么单个要么全部），
> 多播则意在两者之间提供一种折中方案。多播数据报只应该由对他感兴趣的接口接收，也就是说由相应多播会话应用系统的主机上 的接口接收。
> 另外，广播一般局限于局域网内使用，而多播则既可以用于局域网，也可以跨广域网使用。

1. 组播既可以用于局域网，也可以用于广域网
2. 客户端需要加入多播组，才能接收到多播的数据

组播地址：
> IP 多播通信必须依赖于 IP 多播地址， 在 IPv4 中它的范围从 224.0.0.0 ～ 239.255.255.255, 并被划分为
> 局部连接多播地址、预留多播地址和管理权限多播地址三类。

设置组播

```c
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
// 服务器设置多播的信息， 外出接口
// level : IPPROTO_IP
// optname: IP_MULTICAST_IF
// optval : struct in_addr

// 客户端加入到多播组：
// level: IPPROTO_IP
// optname: IP_ADD_MEMBERSHIP
// optval:  struct ip_mreq

struct ip_mreq{
    struct in_addr imr_multiaddr; // 组播组的 IP 地址
    struct in_addr imr_address; // 本地某一网络设备接口的 IP 地址
};

typedef uint32_t in_addr_t;
struct in_addr{
    in_addr_t s_addr;
};
```

+ 多播服务器端代码

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1) {
        perror("socket");
        exit(-1);
    }

    // 设置多播属性，设置外出接口
    struct in_addr imr_multiaddr;
    inet_pton(AF_INET, "239.0.0.10", &imr_multiaddr.s_addr);
    setsockopt(fd, IPPROTO_IP, IP_MULTICAST_IF, &imr_multiaddr, sizeof(imr_multiaddr));

    // 创建广播的地址
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9999);
    inet_pton(AF_INET, "239.0.0.10",& addr.sin_addr.s_addr);

    int num = 0;
    while (1) {
        char send_buf[128] = {0};
        sprintf(send_buf, "hello, client....%d\n", num++);

        sendto(fd, send_buf, strlen(send_buf) + 1, 0, (struct sockaddr *) &addr, sizeof(addr));
        printf("组播的数据： %s\n", send_buf);
        sleep(1);
    }
    close(fd);
    return 0;
}
```

+ 多播客户端示例代码

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd = socket(PF_INET, SOCK_DGRAM, 0);
    if (fd == -1) {
        perror("socket");
        exit(-1);
    }

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(9999);
    server_addr.sin_addr.s_addr = INADDR_ANY;

    int ret = bind(fd, (struct sockaddr*)&server_addr, sizeof (server_addr));
    if(ret == -1){
        perror("read");
        exit(-1);
    }

    struct ip_mreq op;
    inet_pton(AF_INET, "239.0.0.10", &op.imr_multiaddr.s_addr);
    op.imr_interface.s_addr = INADDR_ANY;

    // 加入多播组
    setsockopt(fd, IPPROTO_IP, IP_ADD_MEMBERSHIP, &op, sizeof(op));

    int num = 0;
    while (1) {
        char buf[128];
        int ans = recvfrom(fd, buf, sizeof(buf), 0, NULL, NULL);
        printf("server say: %s\n", buf);
    }
    close(fd);
    return 0;
}
```

## 套接字通信

> 本地套接字的作用： 本地的进程间通信
> 有关系的进程间的通信
> 没有关系的进程间的通信
> 本地套接字实现和网络套接字类似，一般采用 TCP 的通信流程

```c
// 本地套接字通信的流程
// 服务器端
// 1. 创建监听的套接字 
int lfd = socket(AF_LOCAL, SOCK_STREAM, 0);
// 2. 监听的套接字绑定本地的套接字文件
struct sockaddr_un addr;
bind(lfd, addr, len); // 绑定成功之后，指定 sun_path 中的套接字文件会自动生成
// 3. 监听
listen(lfd, 100);
// 4. 等待并接收连接请求
struct sockaddr_un client_addr;
int cfd = accept(lfd, &client_addr, len);
// 5. 通信， 
// 接收数据 read()/ recv()
// 发送数据： write()/ send()
// 6. 关闭连接
close();

// 客户端的流程
// 1. 创建通信的套接字
int fd = socket(AF_UNIX/ AF_LOCAL, SOCK_STREAM, 0);
// 2. 监听的套接字要绑定本地的 ip 端口
struct sockaddr_un addr;
bind(lfd, addr, len);
// 3. 连接服务器
struct sockadd_un server_addr;
connect(fd, &server_addr);
// 4. 通信
// 5. 关闭连接
// 
```

```c
#include <sys/un.h>
#define UNIX_PATH_MAX 108
struct sockaddr_un{
    sa_family_t sun_family; // 地址族协议， af_local
    char sun_path[UNIX_PATH_MAX]; // 套接字文件的路径，这是一个伪文件，大小永远 = 0
};
```

+ 本地套接字通信 服务器端

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/un.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    unlink("server.sock");
    int lfd = socket(AF_LOCAL, SOCK_STREAM, 0);
    if (lfd == -1) {
        perror("socket");
        exit(-1);
    }

    struct sockaddr_un addr;
    addr.sun_family = AF_LOCAL;
    strcpy(addr.sun_path, "server.sock");
    int ret = bind(lfd, (struct sockaddr *) &addr, sizeof(addr));
    if (ret == -1) {
        perror("bind");
        exit(-1);
    }

    ret = listen(lfd, 100);
    if (ret == -1) {
        perror("listen");
        exit(-1);
    }

    struct sockaddr_un client_addr;
    int len = sizeof(client_addr);
    int cfd = accept(lfd, (struct sockaddr *) &client_addr, &len);
    if (cfd == -1) {
        perror("accept");
        exit(-1);
    }

    printf("client socket filename: %s\n", client_addr.sun_path);

    while (1) {
        char buf[128];
        int len = recv(cfd, buf, sizeof(buf),0);
        if (len == -1) {
            perror("recv");
            exit(-1);
        } else if (len == 0) {
            printf("client close...\n");
            break;
        } else if (len > 0) {
            printf("client say: %s\n", buf);
            send(cfd, buf, len, 0);
        }
    }
    close(cfd);
    close(lfd);
    return 0;
}
```

+ 本地套接字通信 客户端

```c
//
// Created by larry on 23-4-16.
//
#include <stdio.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/un.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    unlink("client.sock");
    int cfd = socket(AF_LOCAL, SOCK_STREAM, 0);
    if (cfd == -1) {
        perror("socket");
        exit(-1);
    }

    // connect
    struct sockaddr_un server_addr;
    server_addr.sun_family = AF_LOCAL;
    strcpy(server_addr.sun_path, "server.sock");
    int ret = connect(cfd, (struct sockaddr *) &server_addr, sizeof(server_addr));
    if (ret == -1) {
        perror("connect");
        exit(-1);
    }

    int num = 0;
    while (1) {
        char buf[128];
        sprintf(buf, "hello, i am client %d\n", num++);
        send(cfd, buf, strlen(buf) + 1, 0);
        printf("client say: %s\n", buf);

        int len = recv(cfd, buf, sizeof(buf), 0);
        if (len == -1) {
            perror("recv");
            exit(-1);
        } else if (len == 0) {
            printf("server close...\n");
            break;
        } else if (len > 0) {
            printf("server say: %s\n", buf);
        }
        sleep(1);
    }

    close(cfd);
    return 0;
}
```
