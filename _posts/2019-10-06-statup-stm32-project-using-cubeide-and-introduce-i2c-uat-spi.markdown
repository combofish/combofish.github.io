---
layout: post
title: "STM32 cube工程建立和I2C, SPI, UART的简单介绍"
date: 2019-10-06
tag: STM32 cube SPI I2C UART 
---
- [STM32 cube工程建立和I2C, SPI, UART的简单介绍](#orge98533e)
  - [linux 平台下stm32串口烧录方法](#orgb7bc31e)
    - [stm32flash](#org6a7c0d3)
    - [为命令加点包装](#org6ce3097)
  - [STM32cubeIDE setting](#orgdbb7234)
    - [添加hex output](#org9fe7ee4)
    - [project setting](#org16d5296)
    - [参考链接](#org1c630b9)
  - [usart](#org0cec171)
    - [uart 直接发送](#org941e6bf)
    - [uart 接收后就发送](#org5f0d453)
    - [接收中断并发送](#org0a958d8)
    - [中断接收，先接收完再去处理](#org74e9007)
    - [参考连接](#org8a8550c)
  - [spi](#orge9f35be)
    - [4 条总线](#orgb4bc8b8)
    - [特性](#org718fa80)
    - [cube 配置](#org1e610d3)
    - [主要函数](#org0c13d14)
    - [参考链接](#orgd22b9a5)
  - [I2C](#org27f2b43)
    - [介绍](#org13a666a)
    - [常用函数](#orgc90ce88)
    - [参考链接](#org97e20cc)


<a id="orge98533e"></a>

# STM32 cube工程建立和I2C, SPI, UART的简单介绍


<a id="orgb7bc31e"></a>

## linux 平台下stm32串口烧录方法

-   Rx, Tx 的跳线帽需要连接, 因为使用了板载的串口模块。
-   另外，烧录时 BOOT 的B0 需要置高， 本机所用版本(stm32f103rct6)


<a id="org6a7c0d3"></a>

### stm32flash

```sh
stm32flash -w filename -v -g 0x0 /dev/ttyS0
```


<a id="org6ce3097"></a>

### 为命令加点包装

-   创建文件 flash<sub>stm32.sh</sub>

```sh
#!/bin/sh
# author:combofish
# email:combofish49@gmail.com 
# Filename: flash_stm32.sh

port="/dev/ttyUSB0"

stm32flash -w $1 -v -g 0x0 $port
```

-   之后默认以命令加 xx.hex 文件烧录

```cpp
flash_stm32 f1rrct6.hex
```


<a id="orgdbb7234"></a>

## STM32cubeIDE setting


<a id="org9fe7ee4"></a>

### 添加hex output

你先得在工程属性里配置输出hex文件，才能在Debug/Release文件夹下找到对应工程名的hex文件 右击工程名，属性->C/C++ Build->IAR Linker for ARM->Output Converter->勾选Generate additional output，并在output format下拉框里选择Intel extended


<a id="org16d5296"></a>

### project setting

-   Add necessary library files as reference in the toolchain project configuration file.
-   Generate peripheral initialization as a pair of '.c/ .h' files per peripheral.


<a id="org1c630b9"></a>

### 参考链接

<https://www.cnblogs.com/ChurF-Lin/p/10793111.html>


<a id="org0cec171"></a>

## usart

-   通用异步收发传输器 (Universal Asynchronous Receiver/Transmitter)
-   USART（Universal Synchronous Asynchronous Receiver and Transmitter通用同步异步收发器） USART相当于UART的升级版，USART支持同步模式， 因此USART 需要同步始终信号USART<sub>CK</sub>（如STM32 单片机），通常情况同步信号很少使用， 因此一般的单片机UART和USART使用方式是一样的，都使用异步模式。因为USART的使用方法上跟UART基本相同，所以在此就以UART来讲该通信协议了。


<a id="org941e6bf"></a>

### uart 直接发送

```c
int main() {
  unsigned char uTx_Data[5] = {0x41, 0x42, 0x43, 0x44, 0x45};    //数组内十六进制代表“ABCDE”
  //
  while (1)  {
    //
    HAL_UART_Transmit($huart2, uTx_Data, sizeof(uTx_Data), 0xffff);
  }
}
```

-   uart 发送字符串
-   在头文件添加声明
    
    ```c
    void USER_UART_SendString(UART_HandleTypeDef* uartHandle, unsigned char* uData);
    ```
-   在对应的 .c 文件添加 函数
    
    ```c
    void USER_UART_SendString(UART_HandleTypeDef* uartHandle, unsigned char* uData){
            while (*uData){
                    HAL_UART_Transmit(uartHandle, uData, 1, 1000);
                    uData++;
            }
    }
    ```

-   在 main() 函数的主循环中调用
    
    ```c
    USER_UART_SendString(&huart2, uTx_Data2);
    USER_UART_SendString(&huart3, uTx_Data2);
    HAL_Delay(500);
    ```


<a id="org5f0d453"></a>

### uart 接收后就发送

-   接受字符串
-   接受后就发送
-   这种接收方式是直接在main函数里的while循环里不断接收，会严重占用程序的进程，且接收较长的数据时，会发生接收错误
    
    ```c
    int main() {
      unsigned char uRx_Data = 0;
      //
      if(HAL_UART_Receive(&huart2, &uRx_Data, 1, 1000) == HAL_OK){
        HAL_UART_Transmit(&huart2, &uRx_Data, 1, 1000);
      }
    }
    ```


<a id="org0a958d8"></a>

### 接收中断并发送

-   在 HAL<sub>UART</sub><sub>MspInit</sub>() 中使能中断
    
    ```c
    
    void HAL_UART_MspInit(UART_HandleTypeDef* uartHandle)
    {
    
      GPIO_InitTypeDef GPIO_InitStruct = {0};
      if(uartHandle->Instance==USART2)
        {
          /* USER CODE BEGIN USART2_MspInit 0 */
    
          /* USER CODE END USART2_MspInit 0 */
          /* USART2 clock enable */
          __HAL_RCC_USART2_CLK_ENABLE();
    
          __HAL_RCC_GPIOA_CLK_ENABLE();
          /**USART2 GPIO Configuration    
             PA2     ------> USART2_TX
             PA3     ------> USART2_RX 
          */
          GPIO_InitStruct.Pin = GPIO_PIN_2;
          GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
          GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
          HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
    
          GPIO_InitStruct.Pin = GPIO_PIN_3;
          GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
          GPIO_InitStruct.Pull = GPIO_NOPULL;
          HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
    
          /* USART2 interrupt Init */
          HAL_NVIC_SetPriority(USART2_IRQn, 0, 0);
          HAL_NVIC_EnableIRQ(USART2_IRQn);
          /* USER CODE BEGIN USART2_MspInit 1 */
          //HAL_UART_Receive_IT(&huart2, rData, 1);
          __HAL_UART_ENABLE_IT(uartHandle, UART_IT_RXNE);
    
          /* USER CODE END USART2_MspInit 1 */
        }
    }
    ```
-   在 USART1<sub>IRQHandler</sub>() 中处理数据
    
    ```c
    void USART2_IRQHandler(void)
    {
      /* USER CODE BEGIN USART2_IRQn 0 */
            unsigned char uRx_Data = 0;
            HAL_UART_Receive(&huart2, &uRx_Data, 1, 1000);
            HAL_UART_Transmit(&huart2, &uRx_Data, 1, 0xffff);
    
      /* USER CODE END USART2_IRQn 0 */
      HAL_UART_IRQHandler(&huart2);
      /* USER CODE BEGIN USART2_IRQn 1 */
    
      /* USER CODE END USART2_IRQn 1 */
    }
    ```


<a id="org74e9007"></a>

### 中断接收，先接收完再去处理

-   在 HAL<sub>UART</sub><sub>MspInit</sub>() 中使能中断
    
    ```c
    
    void HAL_UART_MspInit(UART_HandleTypeDef* uartHandle)
    {
    
      GPIO_InitTypeDef GPIO_InitStruct = {0};
      if(uartHandle->Instance==USART2)
        {
          /* USER CODE BEGIN USART2_MspInit 0 */
    
          /* USER CODE END USART2_MspInit 0 */
          /* USART2 clock enable */
          __HAL_RCC_USART2_CLK_ENABLE();
    
          __HAL_RCC_GPIOA_CLK_ENABLE();
          /**USART2 GPIO Configuration    
             PA2     ------> USART2_TX
             PA3     ------> USART2_RX 
          */
          GPIO_InitStruct.Pin = GPIO_PIN_2;
          GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
          GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
          HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
    
          GPIO_InitStruct.Pin = GPIO_PIN_3;
          GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
          GPIO_InitStruct.Pull = GPIO_NOPULL;
          HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
    
          /* USART2 interrupt Init */
          HAL_NVIC_SetPriority(USART2_IRQn, 0, 0);
          HAL_NVIC_EnableIRQ(USART2_IRQn);
          /* USER CODE BEGIN USART2_MspInit 1 */
          //HAL_UART_Receive_IT(&huart2, rData, 1);
          __HAL_UART_ENABLE_IT(uartHandle, UART_IT_RXNE);
    
          /* USER CODE END USART2_MspInit 1 */
        }
    }
    ```
-   在 USART1<sub>IRQHandler</sub>() 中处理数据
    
    ```c
    void USART2_IRQHandler(void)
    {
      /* USER CODE BEGIN USART2_IRQn 0 */
    
      // 中断接收和发送
      /*	unsigned char uRx_Data = 0;
            HAL_UART_Receive(&huart2, &uRx_Data, 1, 1000);
            HAL_UART_Transmit(&huart2, &uRx_Data, 1, 0xffff);*/
    
      // 中断接收完再发送
      static unsigned char uRx_Data[1024] = {0};
      static unsigned char* pRx_Data = uRx_Data;
      static unsigned char uLength = 0;
    
      HAL_UART_Receive(&huart2, pRx_Data, 1, 1000);
    
      if (*pRx_Data == '\n'){
        HAL_UART_Transmit(&huart2, uRx_Data, uLength, 0xffff);
      }else {
        uLength++;
        pRx_Data++;
      }
    
      /* USER CODE END USART2_IRQn 0 */
      HAL_UART_IRQHandler(&huart2);
      /* USER CODE BEGIN USART2_IRQn 1 */
    
      /* USER CODE END USART2_IRQn 1 */
    }
    ```


<a id="org8a8550c"></a>

### 参考连接

<https://www.cnblogs.com/ChurF-Lin/p/10809000.html> <https://www.cnblogs.com/ChurF-Lin/p/10809000.html> <https://www.cnblogs.com/ChurF-Lin/category/1457221.html>


<a id="orge9f35be"></a>

## spi

SPI 协议 Serial Peripheral Interface 串行外围设备接口，是一种高速全双工的通信总线。主要用在MCU与FLASH\ADC\LCD等模块之间的通信。


<a id="orgb4bc8b8"></a>

### 4 条总线

-   SS Slave Select

片选信号线，当有多个SPI 设备与 MCU 相连时，每个设备的这个片选信号线是与 MCU 单独的引脚相连的， 而其他的 SCK、MOSI、MISO 线则为多个设备并联到相同的 SPI 总线上，低电平有效。

-   SCK Serial Clock

时钟信号线，由主通信设备产生，不同的设备支持的时钟频率不一样，如 STM32 的 SPI 时钟频率最大为 f PCLK /2。

-   MOSI Master Output Slave Input

设备输出 / 从设备输入引脚。主机的数据从这条信号线输出，从机由这条信号线读入数据，即这条线上数据的方向为主机到从机。

-   MISO Maste Input Slave Output

主设备输入 / 从设备输出引脚。主机从这条信号线读入数据，从机的数据则由这条信号线输出，即在这条线上数据的方向为从机到主机。


<a id="org718fa80"></a>

### 特性

-   单次传输可选择为 8 或 16 位。
-   波特率预分频系数(最大为 fPCLK/2) 。
-   时钟极性(CPOL)和相位(CPHA)可编程设置。
-   数据顺序的传输顺序可进行编程选择，MSB 在前或 LSB 在前。 注：MSB(Most Significant Bit)是“最高有效位”，LSB(Least Significant Bit)是“最低有效位”。
-   可触发中断的专用发送和接收标志。
-   可以使用 DMA 进行数据传输操作。


<a id="org1e610d3"></a>

### cube 配置

-   配好时钟源
-   SPI mode Full-Duplex Master
    -   Hardware NSS Signal Disable
-   PA4 gpio<sub>output</sub> Label SPI<sub>CS</sub>


<a id="org0c13d14"></a>

### 主要函数

```c
HAL_StatusTypeDef  HAL_SPI_Transmit(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout);//发送数据
HAL_StatusTypeDef  HAL_SPI_Receive(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout);//接收数据
```


<a id="orgd22b9a5"></a>

### 参考链接

<http://news.eeworld.com.cn/mcu/2019/ic-news021243150.html> <https://blog.csdn.net/u014470361/article/details/79015712>


<a id="org27f2b43"></a>

## I2C


<a id="org13a666a"></a>

### 介绍

-   I2C最少只需要两根线，和异步串口类似，但可以支持多个slave设备。
-   和SPI不同的是，I2C可以支持mul-master系统，允许有多个master并且每个master都可以与所有的slaves通信

（master之间不可通过I2C通信，并且每个master只能轮流使用I2C总线）。

-   I2C的数据传输速率位于串口和SPI之间，大部分I2C设备支持100KHz和400KHz模式。
-   使用I2C传输数据会有一些额外消耗：每发送8bits数据，就需要额外1bit的元数据（ACK或NACK）。
-   I2C支持双向数据交换，由于仅有一根数据线，故通信是半双工的。

1.  地址

    -   I2C 总线上的每个设备都有自己的独立地址，主机发起通讯时，通过 SDA 信号线发送设备地址(SLAVE<sub>ADDRESS</sub>)来查找从机。
    -   I2C 协议规定设备地址可以是7 位或10 位，实际中7 位的地址应用比较广泛。
    -   数据方向
        -   数据方向位在设备地址后一位
        -   “1”——读，“0”——写

2.  参考链接

    <https://blog.csdn.net/la_fe_/article/details/100315073>
    
    ```cpp
    #include <stdio.h>
    #include <string.h>
    
    int main(){
      unsigned char a[12] = {0};
      printf("%s", sizeof(a));
    
      return 0;
    }
    ```


<a id="orgc90ce88"></a>

### 常用函数

```c
/* 第1个参数为I2C操作句柄
   第2个参数为从机设备地址
   第3个参数为从机寄存器地址
   第4个参数为从机寄存器地址长度
   第5个参数为发送的数据的起始地址
   第6个参数为传输数据的大小
   第7个参数为操作超时时间 */
HAL_I2C_Mem_Write(&hi2c2,salve_add,0,0,PA_BUFF,sizeof(PA_BUFF),0x10);
HAL_I2C_Mem_Write_IT()；
HAL_I2C_Mem_Read();
HAL_I2C_Mem_Read_IT();
HAL_I2C_Mem_Read_DMA();
HAL_I2C_Mem_Write_DMA();

HAL_I2C_Master_Receive（）；// STM32 主机接收，不需要用到寄存器地址
HAL_I2C_Master_Receive_IT（）；//中断IIC接收
HAL_I2C_Master_Receive_DMA（）；// DMA 方式的IIC接收
HAL_I2C_Master_Transmit_IT();　　//中断IIC发送
HAL_I2C_Master_Transmit_DMA(); 　　// DMA 方式的IIC发送
HAL_I2C_Master_Transmit(&hi2c2,salve_add,PA_BUFF,sizeof(PA_BUFF),0x10); //STM32 主机发送

HAL_I2C_Slave_Receive();// STM32 从机机接收，不需要用到寄存器地址
HAL_I2C_Slave_Transmit();// STM32 从机机发送，不需要用到寄存器地址
HAL_I2C_Slave_Receive_IT();
HAL_I2C_Slave_Receive_DMA();
HAL_I2C_Slave_Transmit_IT();
HAL_I2C_Slave_Transmit_DMA();

//举个调用 HAL_I2C_Mem_Write（）函数读取16个字节的使用例子
HAL_I2C_Mem_Read(&hi2c2,U9_Save_Read_Add,ADC_Result_Add,I2C_MEMADD_SIZE_8BIT,Read_buff,2,0xff);

//再举一个 HAL_I2C_Mem_Read( ) 函数写16个字节的使用例子
uint8_t  Configuration_config[2]={0x09,0xc0};

//设置U9的Configuration寄存器为 0x09 0xc0
HAL_I2C_Mem_Write(&hi2c2,U9_Save_Write_Add,ADC_Configuration_Add,I2C_MEMADD_SIZE_8BIT,Configuration_config,2,0xff); 
```

-   参考链接

<https://www.cnblogs.com/xingboy/p/9566247.html>


<a id="org97e20cc"></a>

### 参考链接

<http://www.eemaker.com/stm32-hal-i2c-24c02.html>
