---
layout: post
title: "Tomcat Installation and Use"
date: "2021-08-06 7：35：00"
tag: tomcat java jdk
---
- [Tomcat 安装与使用](#org72edd2c)
  - [下载使用](#orgf5ecefa)
  - [部署项目](#org7f1d1e0)


<a id="org72edd2c"></a>

# Tomcat 安装与使用

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。


<a id="orgf5ecefa"></a>

## 下载使用

去 [Tomcat官网](https://tomcat.apache.org/) 下载所需版本，下载后解压。我安装的是 apache-tomcat-9.0.44。tomcat10 所用的servlet 与tomcat9 有不同，从9开发的工程不能直接在tomcat10下运行。 具体区别见 [Tomcat10的坑](https://blog.csdn.net/qq_34325222/article/details/105922739) 。

-   查看目录结构：
    -   bin：可以执行文件。
    -   conf：tomcat服务器的配置文件
    -   lib：tomcat启动后需要依赖的jar包
    -   logs：tomcat工作之后的日志文件
    -   webapps：是tomcat布暑工程的目录。
    -   work：jsp文件在被翻译之后，保存在当前这个目录下，session对象被序列化之后保存的位置

```sh
>tree -L 1 .

├── bin
├── BUILDING.txt
├── conf
├── CONTRIBUTING.md
├── lib
├── LICENSE
├── logs
├── NOTICE
├── README.md
├── RELEASE-NOTES
├── RUNNING.txt
├── temp
├── webapps
└── work
```

-   修改默认端口

在 conf/server.xml 文件中找到如下代码，替换自己需要用的端口号。

```xml
<Connector port="8090" protocol="HTTP/1.1"
	   connectionTimeout="20000"
	   redirectPort="8443" />
```

-   启动软件

```sh
bin/startup.sh
```

-   查看启动日志

```sh
tail -f logs/catalina.out
```

-   关闭Tomcat的命令

```sh
bin/shutdown.sh
```


<a id="org7f1d1e0"></a>

## 部署项目

-   方法一：

将工程的 war 包复制到 webapps 目录下。 注意事项，复制的war包在tomcat 启动后会自动解压，所以要注意war 包的权限。在里面有可执行文件时，注意加上x权限。 这种方式不被推荐，项目不好管理，而且需要链接加上项目名才能正常访问。

-   方法二：

修改conf/server.xml文件。在<Host>标签体中配置：

```xml
<Context docBase="ProjectDir" path="/dir"/>
```

其中docBase为项目存放路径，path为虚拟目录。

**注意：修改该文件后，必须重启服务器才能生效。**

-   方法三：

在\conf\Catalina\localhost创建任意名称的xml文件，在该文件中编写：

```xml
<Context docBase="ProjectDir"/>
```

此时的虚拟目录就是xml文件的名称；该部署方式是最推荐使用的，很灵活，若将项目卸载，只需修改该xml文件后缀，不需要重启服务器。
