---
layout: post
title: "How to Use HC-05 Bluetooth Modules"
date: "2020-02-08 14：11：00"
tag: Arduino HC-05 Bluetooth 
---
- [ESP-01 WIFI模块](#orgf67137d)
  - [简介](#orgf5e8b7e)
  - [正常使用时的接线](#org539dfce)
    - [参考链接](#org1eaea5b)
  - [三种编程方式](#orgba8a717)
  - [三种工作模式](#orga482825)
  - [AT 指令要点](#org73df30f)
  - [模式一：Station（client）模式](#orgdf93de7)
  - [模式二：AP（sever）模式](#org9f9e7bb)
  - [模式三：AP+Station（server+client）模式](#orga641395)
  - [其他：UDP 传输](#org86c312c)
  - [示例设置一：TCP 透传](#orgdc4dea5)
  - [示例设置二：多连接模式](#org6d415f3)
  - [Arduino 连接](#org3fec003)
    - [接线](#org766992c)
    - [示例代码](#orgd95ee2c)
  - [Arduino 安装ESP8266 固件编程](#org8c5881a)


<a id="orgf67137d"></a>

# ESP-01 WIFI模块


<a id="orgf5e8b7e"></a>

## 简介

-   官方 文档 <https://docs.ai-thinker.com/esp8266>
-   引脚定义
    
    | 从模块背侧看    |     |      |
    |--------------- |--- |---- |
    | 3v3             | RX  |      |
    | RST             | IO0 | 天线方向 |
    | CH<sub>PD</sub> | IO2 |      |
    | TX              | GND |      |
    
    -   一点参数
        -   ESP chip version: ESP8266
        -   Flash size: 1M
        -   Onboard USB-TTL converter: No
        -   GPIO's broken out/available to free use: 0, 2
        -   Power supply information: 3.3VDC
        -   Antenna: onboard PCB antenna
    
    -   ESP01有以下几个特点：
        -   支持IIC总线
        -   支持UART
        -   可支持一个数字输入/输出口
        -   不能直接接入模拟输入/输出设置


<a id="org539dfce"></a>

## 正常使用时的接线

VCC, GND, URXD, UTXD, *CH_PD* 这5个io 口就可以了。

-   *CH_PD* 正常使用时接高电平
-   区分 下载模式和运行模式，就是GPIO0高低电平问题。
    -   下载模式：CH_PD(EN)、RST、GPIO2、（接3.3V），GPIO0接地。
    -   运行模式：CH_PD(EN)、RST、GPIO2、（接3.3V），GPIO0接3.3V。


<a id="org1eaea5b"></a>

### 参考链接

<https://jingyan.baidu.com/article/0a52e3f4d03556bf63ed7253.html>


<a id="orgba8a717"></a>

## 三种编程方式

-   1、使用AT指令进行操作：

这是最常见的方式，也是最简单是一种方式。无需编程，使用PC端的串口助手配合简单的指令就可以实现，也可以配合单片机发送指令使用。

-   2、LUA语言编程：

这是一种单独8266编程的方式，可以不依靠单片机和串口调试软件，直接把程序编写到8266内部。

-   3、Arduino 开发环境编程：

这个接触过Arduino的都会比较熟悉。可以直接在Arduino ide的环境下使用Arduino的开发方式进行开发。个人比较推荐这种方式，因为比较容易接受与理解。arduino相关资料也比较多。


<a id="orga482825"></a>

## 三种工作模式

-   STA(Station) 模式：

ESP8266 模块通过路由器连接互联网，手机或电脑通过互联网实现对设备的远程控制。

-   AP 模式：

ESP8266 模块作为热点，相当于普通路由器，手机或电脑直接与模块连接，实现局域网无线控制。

-   STA+AP 模式：

两种模式的共存模式，即可以通过互联网控制可实现无缝切换，方便操作。

**就是说模块可以当成一个设备（client）连接区域网内的路由，也可以设置成是一个路由（sever），也可以既作为局域网里面的client同时又是其他client的sever。**


<a id="org73df30f"></a>

## AT 指令要点

-   **AT 指令要求以新行 (CR LF) 结尾**
-   **AT 指令必须为大写英文字母;**
-   **AT指令默认响应为OK**
-   **一般而言,当我们用USB转TTL模块与Esp8266连接之后,波特率默认为115200**
-   **用串口软件输入指令时,每条AT指令后面都要加一个回车键再发送,即串口助手记得勾选”发送新行”**

```shell
AT # 测试指令
AT+GMR # 查看固件
AT+CWMODE=? # 响应返回当前可支持哪些模式
AT+CWMODE? # 响应当前处于哪种模式
AT+CWMODE=2 # 1-Station 模式，2-AP 模式，3-AP 兼 Station 模式

AT+CIPSTATUS # 查看当前连接
# 说明： <id>:连接的 id 号 0-4
# <type>:字符串参数，类型 TCP 或 UDP
# <addr>:字符串参数， IP 地址
# <port>:端口号
# <tetype>: 0-本模块做 client 的连接， 1-本模块做 server 的连接
```


<a id="orgdf93de7"></a>

## 模式一：Station（client）模式

```shell
AT+CWMODE=1 # 指令原型为：AT+CWMODE=<mode>；其中<mode>:1-Station模式，2-AP模式，3-AP兼Station模式。
AT+RST # 配置好模式后需要重启生效。
AT+CWMODE? # 这条指令可以不要，这是查询当前模式的指令，模式返回是1，说明是Station模式。
AT+CWLAP # 可以让模块搜索周围的信号了，列出可以连接的热点（中文显示为乱码）。
AT+CWJAP="HOTPOINT","1234" # 指令原型为：AT+CWJAP=<ssid>,<pwd>）,ssid就是wifi的名字，pwd就是wifi的密码。
AT+CWJAP? # 查询一下当前连接的AP。
AT+CIFSR # 查看下模块的IP地址。
# 在通过路由器查看下模块的IP地址时。如果模块之前我们设置成了AP和Station共存模式，则会出现两个IP，
# 上面的APIP是作为无线AP的IP地址。
# 下面的STAIP是它作为客户端从路由器获取到的IP 地址。

AT+CIPSTA_CUR=”192.168.6.100”,"192.168.6.1","255.255.255.0" # 这条指令可以不要，这是分配固定ip。
AT+CIPSTART="TCP","192.168.1.100",8080 # 连接到服务器，/192.168.1.100为服务器IP地址；8080为端口。
AT+CIPSEND=4 # 发送四个字节的数据。
```

-   **小提示**
    -   通过路由器管理界面查看到模块的MAC地址为：18-FE-34-9C-8B-9D，可以调整一下路由器的IP分发策略，让这个mac地址获得固定的ip，这样每次连接路由器之后就不用查看ip了。配置后提示要重启路由器才生效，没关系以后有时间再重启。
    -   测试连接服务器时，可以在电脑（电脑也需连接同一个路由器）上建立一个TCP服务器，然后用我们刚刚连接到路由器的ESP8266模块连接到我们建立的这个TCP服务器上，然后在这个模块与服务器之间转输数据。 用到的工具是网络助手NetAssist.exe，运行后在网络协议类型：TCP Server ，然后点“连接”按扭，成为一台TCP服务器。


<a id="org9f9e7bb"></a>

## 模式二：AP（sever）模式

```shell
AT+CWMODE=2 # 指令原型为：AT+CWMODE=<mode>；其中<mode>:1-Station模式，2-AP模式，3-AP兼Station模式。
AT+RST # 配置好模式后需要重启生效。
AT+CWMODE? # 这是查询当前模式的指令，模式返回是2，说明是AP模式。
AT+CWSAP="ESP8266","0123456789",11,0
# 指令原型为：AT+ CWSAP=<ssid>,<pwd>,<chl>, <ecn>；
# 其中<ssid>:字符串参数，接入点名称；
# <pwd>:字符串参数，密码最长64字节，ASCII；
# <chl>:通道号；
# < ecn >:0-OPEN，1-WEP，2-WPA_PSK，3-WPA2_PSK，4-WPA_WPA2_PSK。

# 然后现在就可以在你的手机或者是电脑通过无线网卡连接到ESP8266上了。
AT+CWLIF # 查看已接入设备的 IP
# AT+CIFSR # 查看本模块的 IP 地址,AP 模式下无效！会造成死机现象！

AT+CIPMUX=1
# 开启多连接模式，因为只有在开启多连接模式的时候才能开启服务器模式。注意：透传只能在单连接模式下进行。
# 模块最多允许5个客户端连接（每个客户端对应一个id号，0--4）
AT+CIPSERVER=1,8080 # 建立SERVER（0关闭，1开启），设置端口为8080
# AT+CIPSERVER=1  # 默认的端口333 
AT+CIPSEND=0,10 # 下一次输入字符串，0是通道号，10是数据长度。
# 每个连接进来的TCPClient都会有一个ID,默认从0开始增加
# 多连接模式下，第一个参数指明第几个Client(客户端),第二个参数指明发送几个字节,这个用的10,超过后只发送前20字节。
# 单路连接时，只指定发送长度即可。
```

最后，我们就可以通过网络调试助手来通过“TCP Client”模式下添加“IP：192.168.4.1（模块默认的IP）,端口8080（第6步设置的）” 值得一提的是，ESP8266当服务器的时候，客户端如果没有数据传输，隔一段时间会自动断开连接，可通过AT+CIPSTO=<time>命令设置超时时间（说明：<time>:服务器超时时间，0~2880，单位为s）。


<a id="orga641395"></a>

## 模式三：AP+Station（server+client）模式

```shell
AT+CWMODE=3 # 指令原型为：AT+CWMODE=<mode>；其中<mode>:1-Station模式，2-AP模式，3-AP兼Station模式。
AT+RST # 配置好模式后需要重启生效。
AT+CWSAP="ESP8266","0123456789",11,0
# 指令原型为：AT+ CWSAP=<ssid>,<pwd>,<chl>, <ecn>；
# <ssid>:字符串参数，接入点名称；
# <pwd>:字符串参数，密码最长64字节，ASCII；
# <chl>:通道号；
# < ecn >:0-OPEN，1-WEP，2-WPA_PSK，3-WPA2_PSK，4-WPA_WPA2_PSK。
# 然后现在就可以在你的手机或者是电脑通过无线网卡连接到ESP8266上了。
# 打开手机上的有人网络助手，TCP server→配置→激活→能看到此时手机的IP和端口号，要记下，下面要用。

AT+CIPMODE=1 # 开启透传模式。
AT+CIPMUX=0 # 开启单路模式。
AT+CIPSTART="TCP","192.168.4.2",8080
# 192.168.4.2为服务器IP地址；8080为端口。填刚才记下的手机IP和端口号
# 这时手机已经能向模块发信息了，但模块不能发。
AT+CIPSEND=4 # AT+CIPSEND=(要发送的字节长度)
# AT+CIPSEND # ESP8266发送数据至手机。 
```


<a id="org86c312c"></a>

## 其他：UDP 传输

UDP&#x2013;分为发送端和接收端，面向无连接的通信（速度快），只向指定的ip（每一台电脑都有自己的ip地址，向指定的ip地址发数据，数据就发送到了指定的电脑）端口发送,不管你接没接收到，只管发。

-   UDP传输,此时Esp8266不分Server和Client

```shell
AT+CWMODE=3
AT+CWJAP="SSID","password"
AT+CIPMUX=1
```

由于没有客户端和服务器,我们可以设置是否通信的一端可变,就是设置远端是否可变,在不可变的情况下我们分配给连接的另一端一个ID,可变的情况下当有多个连接时就会发送到最近一个连接的设备

-   远端固定,”AT+CIPSTART”最后参数为0

```shell
AT+CIPSTART=4,"UDP","192.168.43.113",333,8080,0
# 这里的4表示给对方的ID,
# 192.168.43.113对方IP
# 333表示对方的端口
# 8080设置自己的端口(随意设置0~65535)
# 0表示远端固定

AT+SEND=4,5 # 发送数据,给定4表示对方的ID,5表示自己的长度
```

-   远端可变,”AT+CIPSTART”最后参数为2

```shell
AT+CIPSTART="UDP","192.168.43.113",333,8080,2 # 然后就可以发送数据了
AT+CIPSEND=6,"192.168.43.111",333 # 发送给指定的UDP地址:字节长度,对方IP,对方端口
```


<a id="orgdc4dea5"></a>

## 示例设置一：TCP 透传

TCP&#x2013;分为服务器和客户端，与udp不同的是双方建立正常的连接后，才能通信，每次通信都会检测连接正不正常。（通信可靠，速度比udp慢） 注意只有在单连接的时候才可以设置为透传模式&#x2013;就是串口的数据直接发送到网络,网络的数据直接发送到串口

```shell
AT+CWMODE=3
AT+CWJAP="SSID","PASSWD"
AT+CIFSR    # 查询自己的IP地址 
AT+CIPSTART="TCP","192.168.xxx.xxx",8000 # 作为 TCP client 链接到上述服务器
AT+CIPMODE=1 # 使能透传模式
AT+CIPSEND # 向服务器发送数据
```


<a id="org6d415f3"></a>

## 示例设置二：多连接模式

```shell
AT+CWMODE=3
AT+RST # 重启生效
AT+CWSAP="ESP8266_TEST","1234",1,3 # 名字，密码，通道和密码模式
AT+CIPMUX=1 # 启动多连接
AT+CIPSERVER=1,8000 # 建立SERVER（0关闭，1开启）, 端口8000,
```

上面的配置成多连接，重启，配置名字和密码只要配置一次就好， 即使之后断电再上电也不用重复配置，每次上电后只要输入AT+CIPMUX=1和AT+CIPSERVER=1,8080即可。


<a id="org3fec003"></a>

## Arduino 连接


<a id="org766992c"></a>

### 接线

-   GND,VCC
-   TX,,RX
-   CH_PD


<a id="orgd95ee2c"></a>

### 示例代码

```c
#include <SoftwareSerial.h>
//#include <string.h>
#define TIMEOUT 5000 // mS
#define LED 5
SoftwareSerial mySerial(2, 3); // RX, TX
const int button = 11;
int button_state = 0;

void setup()
{
  pinMode(LED, OUTPUT);
  pinMode(button, INPUT);

  Serial.begin(9600);
  mySerial.begin(115200);
  SendCommand("AT+RST", "Ready");
  delay(5000);
  SendCommand("AT+CWMODE=1", "OK");
  SendCommand("AT+CIFSR", "OK");
  SendCommand("AT+CIPMUX=1", "OK");
  SendCommand("AT+CIPSERVER=1,80", "OK");
}

void loop() {
  button_state = digitalRead(button);

  if (button_state == HIGH) {
    mySerial.println("AT+CIPSEND=0,23");
    mySerial.println("<h1>Button was pressed!</h1>");
    delay(1000);
    SendCommand("AT+CIPCLOSE=0", "OK");
  }

  String IncomingString = "";
  boolean StringReady = false;

  while (mySerial.available()) {
    IncomingString = mySerial.readString();
    StringReady = true;
  }

  if (StringReady) {
    Serial.println("Received String: " + IncomingString);

    if (IncomingString.indexOf("LED=ON") != -1) {
      digitalWrite(LED, HIGH);
    }

    if (IncomingString.indexOf("LED=OFF") != -1) {
      digitalWrite(LED, LOW);
    }
  }
}

boolean SendCommand(String cmd, String ack) {
  mySerial.println(cmd); // Send "AT+" command to module
  if (!echoFind(ack)) // timed out waiting for ack string
    return true; // ack blank or ack found
}

boolean echoFind(String keyword) {
  byte current_char = 0;
  byte keyword_length = keyword.length();
  long deadline = millis() + TIMEOUT;
  while (millis() < deadline) {
    if (mySerial.available()) {
      char ch = mySerial.read();
      Serial.write(ch);
      if (ch == keyword[current_char])
	if (++current_char == keyword_length) {
	  Serial.println();
	  return true;
	}
    }
  }
  return false; // Timed out
}
```

-   效果

通过连接服务器端口，发送 “LED=ON” 可以打开小灯，发送 “LED=OFF” 可以关闭小灯。


<a id="org8c5881a"></a>

## Arduino 安装ESP8266 固件编程

编写固件时可以直接使用GPIO2

```c
int PIN = 2;
```

```c
void setup() {
  // initialize digital esp8266 gpio 2 as an output.
  pinMode(2, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(2, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);              // wait for a second
  digitalWrite(2, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);              // wait for a second
}
```
