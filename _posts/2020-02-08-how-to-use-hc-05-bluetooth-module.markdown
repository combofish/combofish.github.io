---
layout: post
title: "How to Use HC-05 Bluetooth Modules"
date: "2020-02-08 14：11：00"
tag: Arduino HC-05 Bluetooth 
---
- [BlueTooth HC-05蓝牙模块使用教程](#org32eafa6)
  - [模块简介](#org46aa93b)
    - [命令响应模式（AT模式）](#orgfb79a02)
    - [自动链接模式](#org92eba40)
  - [连接串口通信模块](#orgb6b5a50)
  - [常用 AT 命令如下](#org013044d)
  - [从机设置的AT代码](#org1afbd1a)
  - [主机设置的AT代码](#org01800d7)
  - [Arduino 通信示例代码](#org50aa9ed)
  - [使用步骤](#org42d46a1)
  - [示例：蓝牙控制小灯](#org3b7452b)


<a id="org32eafa6"></a>

# BlueTooth HC-05蓝牙模块使用教程


<a id="org46aa93b"></a>

## 模块简介

嵌入式蓝牙串口通讯模块（简称蓝牙模块）具有两种工作模式：命令响应工作模式和自动连接工作模式。


<a id="orgfb79a02"></a>

### 命令响应模式（AT模式）

首先按住蓝牙模块上的复位键然后再上电，看见蓝牙模块上的LED灯以2s间隔闪烁，就表示进入了AT模式 记住串口助手的波特率设置为38400,停止位1位,数据为8位,奇偶校验无,另外一定要勾上“发送新行”！

当模块处于命令响应工作模式（或者AT模式）时能才能执行 AT 命令，用户可向模块发送各种 AT指令，为模块设定控制参数或发布控制命令。（AT指令就是我们PC与一些终端设备（例如蓝牙，WiFi模块）之间进行通信的，配置这些终端设备参数的一套指令。）


<a id="org92eba40"></a>

### 自动链接模式

我们发送AT+RESET之后,当模块LED以0.5s间隔闪烁时表示进入自动连接模式。 在自动连接工作模式下模块又可分为主（Master）、从（Slave）和回环（Loopback）三种工作角色。 当模块处于自动连接工作模式时，将自动根据事先设定的方式连接的数据传输。

-   主模式：该模块可以主动搜索并连接其它蓝牙模块并接收发送数据。
-   从模式：只能被搜索被其它蓝牙模块连接进行接收发送数据。
-   回环：蓝牙模块就是将接收的数据原样返回给远程的主设备。


<a id="orgb6b5a50"></a>

## 连接串口通信模块

HC05的VCC连接+5V，GND连接GND，TXD连接Pin10，RXD连接Pin11。 HC-05模块是蓝牙SPP（串行端口协议）模块，这意味着它通过串行通信与Arduino通信。 我使用的这个模块可以在3.6到6V的电压下工作，因为它带有一个包含电压调节器的分线板。但是，数据引脚的逻辑电压电平为3.3V。 因此，Arduino的TX（具有5V输出的发送引脚）和蓝牙模块RX（仅支持3.3V的接收引脚）之间的线路需要通过分压器连接，以免烧毁模块。 另一方面，蓝牙模块TX引脚和Arduino RX引脚之间的线路可以直接连接，因为来自蓝牙模块的3.3V信号足以被Arduino开发板上的高逻辑识别。


<a id="org013044d"></a>

## 常用 AT 命令如下

```shell
AT+ORGL    # 恢复出厂模式
AT+NAME=<Name>    # 设置蓝牙名称
AT+ROLE=0    # 设置蓝牙为从模式
AT+CMODE=1    # 设置蓝牙为任意设备连接模式
AT+PSWD=<Pwd>    # 设置蓝牙匹配密码

## Other AT command
AT+VERSION? # 查看版本信息
AT+ADDR? # 查看蓝牙地址
AT+UART? # 查看串口参数
AT : Ceck the connection.
AT+ROLE: See role of bt module(1=master/0=slave)
AT+RESET : Reset and exit AT mode
```

正常情况下，命令发送后，会返回 OK ，如果没有返回任何信息，请检查接线是否正确， 蓝牙模块是否已经进入 AT 模式，如果上述两点都没有问题，可能是蓝牙模块的问题， 可以找蓝牙模块供应商咨询。设置完毕后，断开电源，再次通电， 这时，蓝牙模块指示灯会快速闪烁，这表明蓝牙已经进入正常工作模式。


<a id="org1afbd1a"></a>

## 从机设置的AT代码

```c
AT returns OK
AT+NAME=HC05-Slave
AT+UART=9600,0,0
AT+ROLE=0
```


<a id="org01800d7"></a>

## 主机设置的AT代码

```c
AT returns OK
AT+NAME=HC05-Master
AT+UART=9600,0,0
AT+ROLE=1
```


<a id="org50aa9ed"></a>

## Arduino 通信示例代码

```c
#include <SoftwareSerial.h> 
// Pin10为RX，接HC05的TXD
// Pin11为TX，接HC05的RXD
SoftwareSerial BT(10, 11);
char val;
void setup() {
  Serial.begin(38400);
  Serial.println("BT is ready!");
  // HC-05默认，38400
  BT.begin(38400);
}
void loop() {
  if (Serial.available()) {
    val = Serial.read();
    BT.print(val);
  }
  if (BT.available()) {
    val = BT.read();
    Serial.print(val);
  }
}
```

示例二：效果一样

```c
#include <SoftwareSerial.h>

#define rxPin 2
#define txPin 3
#define baudrate 38400

String msg;
SoftwareSerial hc05(rxPin ,txPin);

void setup(){
  pinMode(rxPin,INPUT);
  pinMode(txPin,OUTPUT);
  Serial.begin(9600);
  Serial.println("ENTER AT Commands:");
  hc05.begin(baudrate);
}
void loop(){
  readSerialPort();
  if(msg!="") hc05.println(msg);
  if (hc05.available()>0){
    Serial.write(hc05.read());
  }
}
void readSerialPort(){
  msg="";
  while (Serial.available()) {
    delay(10);  
    if (Serial.available() >0) {
      char c = Serial.read();  //gets one byte from serial buffer
      msg += c; //makes the string readString
    }
  }
}
```

示例三：主从机通信

-   Arduino 连接蓝牙主机代码
    
    ```c
    #include <SoftwareSerial.h>
    SoftwareSerial ArduinoSlave(2,3);
    String answer;
    String msg;
    void setup(){
      Serial.begin(9600);
      Serial.println("ENTER Commands:");
      ArduinoSlave.begin(9600);
    }
    void loop(){
      //Read command from monitor
      readSerialPort();
      //Read answer from slave
      while (ArduinoSlave.available()) {
        delay(10);  
        if (ArduinoSlave.available() >0) {
          char c = ArduinoSlave.read();  //gets one byte from serial buffer
          answer += c; //makes the string readString
        }
      }
      //Send data to slave
      if(msg!=""){
        Serial.print("Master sent : ");
        Serial.println(msg);
        ArduinoSlave.print(msg);
        msg="";
      }
      //Send answer to monitor
      if(answer!=""){
        Serial.print("Slave recieved : ");
        Serial.println(answer);
        answer="";
      }
    }
    void readSerialPort(){
      while (Serial.available()) {
        delay(10);  
        if (Serial.available() >0) {
          char c = Serial.read();  //gets one byte from serial buffer
          msg += c; //makes the string readString
        }
      }
      Serial.flush();
    }
    ```
    
    -   Arduino 连接蓝牙从机代码
    
    ```c
    #include <SoftwareSerial.h>
    SoftwareSerial ArduinoMaster(2,3);
    String msg;
    void setup(){
      Serial.begin(9600);
      ArduinoMaster.begin(9600);    
    }
    void loop(){
      readSerialPort();
      // Send answer to master
      if(msg!=""){
        Serial.print("Master sent : " );
        Serial.println(msg);
        ArduinoMaster.print(msg);
        msg=""; 
      }
    }
    void readSerialPort(){
      while (ArduinoMaster.available()) {
        delay(10); 
        if (ArduinoMaster.available() >0) {
          char c = ArduinoMaster.read();  //gets one byte from serial buffer
          msg += c; //makes the string readString
        }
      }
      ArduinoMaster.flush();
    }
    ```


<a id="org42d46a1"></a>

## 使用步骤

写入代码后，将 Arduino 断电，然后按着蓝牙模块上的黑色按钮，再让 Arduino 通电， 如果蓝牙模块指示灯按2秒的频率闪烁，表明蓝牙模块已经正确进入 AT 模式。 打开 Arduino IDE 的串口监视器，选择正确的端口，将输出格式设置为 Both: NL & CR ， 波特率设置为 38400 ，可以看到串口监视器中显示 BT is ready! 的信息。 然后，输入 AT ，如果一切正常，串口显示器会显示 OK。接下来，我们即可对蓝牙模块进行设置。


<a id="org3b7452b"></a>

## 示例：蓝牙控制小灯

```c
#include <SoftwareSerial.h>

// Pin3为RX，接HC05的TXD
// Pin2为TX，接HC05的RXD
SoftwareSerial BT(3, 2);
char val;
int ledPin=5;

void setup() {
  Serial.begin(38400);
  Serial.println("BT is ready!");
  // HC-05默认，38400
  BT.begin(38400);

  pinMode(ledPin, OUTPUT);
}

void loop() {
  if (Serial.available()) {
    val = Serial.read();
    BT.print(val);
  }
  if (BT.available()) {
    val = BT.read();
    Serial.print(val);

    if (val == '1')
      {
	// 返回到手机调试程序上
	Serial.write("Serial--ledPin--high");
	digitalWrite(ledPin, HIGH);
      }
    if (val == '2')
      {
	Serial.write("Serial--ledPin--low");
	digitalWrite(ledPin, LOW);
      }
  }
}
```
