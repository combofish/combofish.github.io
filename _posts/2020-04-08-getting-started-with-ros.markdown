---
layout: post
title: "Getting started with ROS(Study Note)"
date: "2020-04-08 23：33：00"
tag: ROS
---
- [ROS](#orge92d058)
  - [其他介绍](#org997a8b7)
    - [ROS wiki的解释](#org215dfc2)
    - [Brian Gerkey的网上留言](#org833e70c)
  - [ROS 工程结构](#org6b47ae3)
  - [常用指令](#orgc3708a4)
  - [通信架构（计算图级）](#org385087d)
    - [master 节点管理器](#org10b4efa)
    - [node (ROS 中的进程）](#org2632d0f)
    - [Topic](#org7410ca9)
    - [Message](#org8bc4a4e)
    - [Service](#org601a603)
    - [Topic VS Service](#org58c5bff)
    - [Parameter Server](#org695be0d)
    - [Action](#orga418764)
  - [常用工具](#orgdab523d)
    - [Gazebo 仿真](#orge9a502c)
    - [RViz](#org30ae46c)
    - [rqt](#org467e150)
    - [CLI tools](#orgd26a4b1)
    - [专用工具](#org12b09f2)
  - [Client Library](#orgfce7f08)
  - [TF & URDF](#org1fc121a)
    - [TF tree](#orgd06abbf)
    - [URDF](#orgec4625d)
  - [SLAM](#org71d7355)
  - [namespace 和 重映射](#orgcca0614)


<a id="orge92d058"></a>

# ROS

ROS是机器人操作系统（Robot Operating System）的英文缩写。 ROS是用于编写机器人软件程序的一种具有高度灵活性的软件架构。 它包含了大量工具软件、库代码和约定协议， 旨在简化跨机器人平台创建复杂、鲁棒的机器人行为这一过程的难度与复杂度。 ROS的原型源自斯坦福大学的STanford Artificial Intelligence Robot (STAIR) 和 Personal Robotics (PR)项目。2007 正式启动，相当于运行在Linux上的中间件。


<a id="org997a8b7"></a>

## 其他介绍


<a id="org215dfc2"></a>

### ROS wiki的解释

ROS（Robot Operating System，下文简称“ROS”）是一个适用于机器人的开源的元操作系统。 它提供了操作系统应有的服务，包括硬件抽象，底层设备控制，常用函数的实现，进程间消息传递，以及包管理。 它也提供用于获取、编译、编写、和跨计算机运行代码所需的工具和库函数。

ROS 的主要目标是为机器人研究和开发提供代码复用的支持。 ROS是一个分布式的进程（也就是“节点”）框架，这些进程被封装在易于被分享和发布的程序包和功能包中。ROS也支持一种类似于代码储存库的联合系统，这个系统也可以实现工程的协作及发布。这个设计可以使一个工程的开发和实现从文件系统到用户接口完全独立决策（不受ROS限制）。同时，所有的工程都可以被ROS的基础工具整合在一起。


<a id="org833e70c"></a>

### Brian Gerkey的网上留言

我通常这样解释ROS：

  1. 通道：ROS提供了一种发布-订阅式的通信框架用以简单、快速地构建分布式计算系。
  2. 工具：ROS提供了大量的工具组合用以配置、启动、自检、调试、可视化、登录、测试、终止分布式计算系统。
  3. 强大的库：ROS提供了广泛的库文件实现以机动性、操作控制、感知为主的机器人功能。
  4. 生态系统：ROS的支持与发展依托着一个强大的社区。ros.org尤其关注兼容性和支持文档，提供了一套“一站式”的方案使得用户得以搜索并学习来自全球开发者数以千计的ROS程序包。


<a id="org6b47ae3"></a>

## ROS 工程结构

catkin 工作空间： ROS 定制的编译构建系统，对CMake的扩展。 组织和管理功能包的文件夹，以 catkin 工具编译。

```shell
mkdir ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make

# compile
cd ~/catkin_ws/
catkin_make
source ~/catkin_ws/devel/setup.bash
```

catkin workspace :

-   src 源代码
-   build cmake & catkin 缓存和中间文件
-   devel 目标文件存放的目录

src:

-   package1
    -   CMakeLists.txt 规定了catkin 编译的规则
    -   package.xml 定义了包的属性，相当于包的自我描述
-   package2
    -   CMakelists.txt
    -   package.xml

**包是catkin 编译的基本单元**
一个包的目录机构：

-   package
    -   Cmakelists.txt
    -   package.xml
    -   scripts/
        -   \*.py
        -   \*.sh
    -   include/
        -   \*.h
    -   src/
        -   \*.cpp
    -   msg/
    -   srv/
    -   action/
    -   config/\*.yaml
    -   launch/\*.launch


<a id="orgc3708a4"></a>

## 常用指令

```shell
rospack find package_name # 查找某个包的位置
rospack list # 列出本地所有的包
roscd package_name # 跳转到某个包路径下
rosls package_name # 列举某个包下的文件信息
rosed package_name file_name # 编辑某个包下的文件
catkin_create_pkg <pkg_name> [deps] #创建包
rosdep install [pkg_name] #安装某个包所需的依赖
```


<a id="org385087d"></a>

## 通信架构（计算图级）


<a id="org10b4efa"></a>

### master 节点管理器

每个 node 启动时都要向 master 注册。master 管理node 之间的通信。 roscore 程序向节点提供连接信息，以便节点之间可以互相传递信息的服务程序。

```shell
roscore 
# 启动 ros master
# 启动 rosout：日志输出
# 启动 parameter server： 参数服务器
```


<a id="org2632d0f"></a>

### node (ROS 中的进程）

node 是包里可执行文件运行的实例

-   启动 node

```shell
rosrun [pkg_name] [node_name]
rosnode list # 列出当前运行的 node 信息
rosnode info [node_name] # 查看某个node的详细信息
rosnode kill [node_name] 
```

-   启动 node 的另一种方法：roslunch 启动一系列节点的工具。当节点增多，使用rosrun 逐一启动各个节点是不现实的。 操作launch 文件 而不是节点。这些文件有 .launch 后缀。
    
    ```shell
    roslaunch [pkg_name] [file_name.launch]
    # launch 文件也是 xml 格式
    ```


<a id="org7410ca9"></a>

### Topic

ROS 中的异步通信。publish - subscribers模式。一个topic 可以有多个订阅者。 同一个topic也可以有多个发布者。


<a id="org8bc4a4e"></a>

### Message

topic 内容的数据类型，定义在pkg/msg/\*.msg 文件中。 基本的msg可用的类型有：bool,int64,uint,float64,string,array[],array[C]等。 还有 time, duration, header.

-   相关命令

```shell
rostopic list 列出当前所有的topic 
rostopic info /topic_name 显示某个 topic 的属性信息
rostopic echo /topic_name 显示某个 topic 的内容
rostopic pub /topic_name 向某个 topic 发布内容

## rosmsg
rosmsg list 列出系统上所有的msg 
rosmsg show /msg_name 显示某个msg的内容
```

使用 rostopic pub 向小车传递移动信息写法

```shell
rostopic pub -r 10 /cmd_vel geometry_msgs/Twist '{linear: {x: 0.1, y: 0.0, z: 0.0}, angular:{x: 0.0, y: 0.0, z: 0.0}}'
```


<a id="org601a603"></a>

### Service

ROS 中的同步通信方式，Node之间可以通过 request-reply 方式通信。调用时才执行。

```shell
# rosservice 类比与 rostopic
rosservice list # 列出当前所有活跃的 service 
roservice info service_name # 显示某个service 的属性信息
roservice call service_name args # 调用某个service # 不知道参数具体格式时，可以n按TAB键

# rossrv  类比与 rosmsg
rossrv list # 列出系统上所有的srv
rossrv show srv_name # 显示某个srv的内容
```

.srv 文件能嵌套.msg, 但不能嵌套 .srv。


<a id="org58c5bff"></a>

### Topic VS Service

|      | Topic                 | Service            |
|---- |--------------------- |------------------ |
| 通信原理 | 异步通信              | 同步通信           |
| 实现原理 | TCP/IP                | TCP/IP             |
| 通信模型 | Publish-Subscribe     | Request-Reply      |
|      | Publisher-Subscribers | Client-Server      |
|      | 多对多                | 多对一             |
|      | 接收者收到数据会回调（callback） | 远程过程调用（RPC）服务器端的服务 |
| 应用场景 | 连续，高频的数据发布  | 偶尔调用的功能/具体的任务 |
| 举例 | 激光雷达，里程计发布数据 | 开关传感器，拍照，逆解计算 |

**注：定义了 srv, 或者msg 都需要到 package.xml 和 CMakelists.txt 中添加相应的部分。**


<a id="org695be0d"></a>

### Parameter Server

在启动系统时，系统自动启动参数服务器。存储各种参数的字典，可用命令行，launch文件和 node（API）读写。

-   rosparam 一个命令行工具，用于与参数服务器交互。

```shell
rosparam list # 列出当前所有参数
rosparam get param_key # 显示某个参数的值
rosparam set param_key param_value # 设置某个参数的值
rosparam dump file_name # 保存参数到文件, yaml格式
rosparam load file_name # 从文件读取参数
rosparam delete param_key # 删除参数
```

-   在launch 文件中与设置参数相关的标签有 <param> 和 <rosparam>


<a id="orga418764"></a>

### Action

升级版的service, 带有状态反馈的通信方式， 通常用在长时间，可抢占的任务中。action 通信的数据格式定义在 \*.action 中。-> 是客户发给服务端，<- 是服务器返回给请求方。

| Action Client | ROS Topic   | Action Server |
|------------- |----------- |------------- |
|               | goal ->     |               |
|               | cancel ->   |               |
|               | status <-   |               |
|               | result <-   |               |
|               | feedback <- |               |


<a id="orgdab523d"></a>

## 常用工具


<a id="orge9a502c"></a>

### Gazebo 仿真

机器人仿真工具，ODE物理引擎，可用于动力学，导航，感知等任务的模拟。


<a id="org30ae46c"></a>

### RViz

RViz: The Robot Visualization tool 可视化工具，方便监控和调试


<a id="org467e150"></a>

### rqt

可视化工具，常用rqt_graph, rqt_plot, rqt_console。 基于qt开发，可扩展性好。

-   rqt_graph 图可视化工具，可以用来检查软件的通信架构。
-   rqt_plot 绘制曲线
-   rqt_console 查看日志


<a id="orgd26a4b1"></a>

### CLI tools

命令行工具有很多，可以用在终端下输入ros后按tab键查看。再次只介绍rosbag。

-   rosbag 命令行工具，记录和回放数据流, 原理是记录topic 生成 \*.bag 用于回放。

```shell
rosbag record <topic_names> # 记录某些topic 到bag 中
rosbag record -a # 记录所有的topic
rosbag play <bag-files> # 回放bag
```


<a id="org12b09f2"></a>

### 专用工具

Moveit


<a id="orgfce7f08"></a>

## Client Library

和API 类似，并在其上进行了封装，提供ROS 编程的库（接口），例如：建立node, 发布消息， 调用服务。 roscpp 执行效率高, rospy 开发效率高, roslisp，，，

-   rospy

[rospy参考文档](http://docs.ros.org/api/rospy/html/)


<a id="org1fc121a"></a>

## TF & URDF

TF: TransForm 坐标系转换，（位置+ 姿态），坐标系数据维护工具


<a id="orgd06abbf"></a>

### TF tree

类型：第一代 tf/tfMessage.msg, 第二代 tf2_msgs/TFMessage

-   相关命令

```shell
rosrun tf view_frames # 根据当前的tf树创建一个pdf图
rosrun rtq_tf_tree rtq_tf_tree # 产看当前的tf 树
rosrun tf tf_echo [reference_frame] [target_frame] # 查看两个frame之间的变换关系
```


<a id="orgec4625d"></a>

### URDF

Unified Robot Description Format 用来描述机器人的结构

-   link 标签 每个部件
-   joint 标签 连接两个部件的关节


<a id="org71d7355"></a>

## SLAM

-   Simultaneous Localization And Mapping 同时定位和建图

移动机器人的任务通常有：

-   Mapping
-   Localization
-   PathPlanning

针对这些问题，ROS 提供的工具：

-   SLAM 开源算法包：Gmapping SLAM, Karto SLAM等；
-   AMCL（自适应蒙特卡洛定位） 定位算法包；
-   Navigation 导航功能包集，分Global planning(全局规划)和 local planning (局部导航规划)。


<a id="orgcca0614"></a>

## namespace 和 重映射

ROS 可以在不同命名空间中启动同一个节点来避免冲突。 ROS 程序中任何一个用于命名的字符串都可以在运行时重映射。

```shell
./image_view image:=right/image
```

```shell
./camera __ns:=right
```

```shell
./talker __name:=talker1
./talker __name:=talker2
```
