---
layout: post
title: "Getting started with Raspbarry Pi Camera"
date: "2020-02-06 18：27：00"
tag: raspbarrypi camera raspistill raspivid
---
- [Raspberry Pi Camera](#org9af6b00)
  - [命令行工具raspistill和raspivid。](#org6662c2b)
    - [raspistill](#orgf4f672c)
    - [raspivid](#orgf904192)
  - [Python picamera 控制相机](#orgd0ecabb)
  - [以视频流方式使用](#org160cc38)
    - [方法一：mjpg-streamer](#org8832532)
    - [方法二：VLC 视频软件](#orgb3cb0a9)


<a id="org9af6b00"></a>

# Raspberry Pi Camera

树莓派官方提供了小型摄像头,（CMOS Sensor Interface(CSI) 接口摄像头），用于拍照和录制视频。 连接好摄像头后，进入 raspi-config 打开摄像头。


<a id="org6662c2b"></a>

## 命令行工具raspistill和raspivid。


<a id="orgf4f672c"></a>

### raspistill

拍摄静态图片

```shell
raspistill -o image.jpg
# 常用参数
-v：调试信息查看
-w：图像宽度
-h：图像高度
-rot：图像旋转角度，只支持 0、90、180、270 度（这里说明一下，测试发现其他角度的输入都会被转换到这四个角度之上）
-o：图像输出地址，例如image.jpg，如果文件名为“-”，将输出发送至标准输出设备
-t：获取图像前等待时间，默认为5000，即5秒
-tl：多久执行一次图像抓取

# 示例2
raspistill -o image%d.jpg -rot 180 -w 1024 -h 768 -t 20000 -tl 5000 -v
# 图片的大小，1024 * 768
# 图片的总捕获时间 20s
# 图像旋转180度
```


<a id="orgf904192"></a>

### raspivid

录制视频

```shell
raspivid -o video.h264 -t 10000 # 获取10s的视频
raspivid -o video2.h264 -t 10000 -w 1280 -h 720 # 获取10s的视频,并设置宽高'
```

获得10秒H.264压缩格式的视频，存入到文件video.h264。

1.  视频格式转换

    raspivid 通常会将录制的视频保存为 .h264 格式的文件，而我们使用的很多播放器可能无法正常播放该格式的视频文件。 这就需要我们将生成的 .h264 格式的文件封装到播放器能够识别的视频容器格式中（比如封装为 mp4 格式）。 有很多视频处理软件可以达到这个目的，可以直接在树莓派上进行封装。这里介绍的是“gpac”中的“MP4Box”。
    
    安装gpac
    
    ```shell
    apt-get install gpac
    ```
    
    将.h264的文件转换成.mp4的文件
    
    ```shell
    MP4Box -add video.h264 video.mp4
    ```


<a id="orgd0ecabb"></a>

## Python picamera 控制相机

先安装 picamera 库

```shell
pip install picamera 
```

开始使用

```python
from picamera import PiCamera
from time import sleep

camera = PiCamera()
camera.start_preview(alpha=200)
sleep(10)
camera.capture('test.jpg')
camera.end_preview()
```

常用设置

```python
camera.start_preview(alpha=200) # 还可设置画面透明度, 其中alpha 的值可为0-255 的任何值
camera.capture() # 拍摄图片
camera.start_recording('path_to_save.h264') # 开始录制视频
camera.stop_recording() # 停止录制视频

# 设置分辨率
camera.resolution = (2592,1944)  # (1920,1080)  # min (64 * 64)
camera.framerate = 15 # 刷新率

camera.annotate_text = 'annotate' # 在图片中添加文字
camera.brightness = 50 # (0-100) default 50.
camera.rotation = 180 # 旋转摄像头,角度可为0，90，180，270 
```


<a id="org160cc38"></a>

## 以视频流方式使用


<a id="org8832532"></a>

### 方法一：mjpg-streamer

-   MJPG-streamer 适用于从webcam摄像头采集图像,把他们以流的形式通过基于Ip 的网络传输到拥有浏览器的移动设备,通过mjpg-streamer软件可 以实现将摄像头中采集到的图像通过网络传输到浏览器网页上显 示。
    
    -   安装
    
    ```shell
    git clone https://github.com/jacksonliam/mjpg-streamer
    ```
    
    安装依赖，编译并安装 **看readme.md**
    
    -   启动
    
    ```shell
    ./mjpg_streamer -o "output_http.so -w ./www" -i "input_raspicam.so"
    ```
    
    -   使用
    
    在浏览器打开IP<sub>address</sub>:8080，或者 <http://IP_address:8080/?action=stream> 亲测也可以用 VLC 打开。


<a id="orgb3cb0a9"></a>

### 方法二：VLC 视频软件

-   安装 VLC

```shell
apt-get install vlc
```

在树梅派上启动 VLC 服务端，并运行。

```shell
sudo raspivid -o - -t 0 -w 640 -h 360 -fps 25|cvlc -vvv stream:///dev/stdin --sout '#standard{access=http,mux=ts,dst=:8090}' :demux=h264
# 或sudo raspivid -o - -rot 180 -t 0 -fps 30|cvlc -vvv stream:///dev/stdin --sout '#standard{access=http,mux=ts,dst=:8080}' :demux=h264
```

打开VLC软件，然后打开媒体-》网络串流-》输入 <http://PI的IP地址:8090> 就能查看实时的网络监控了。
