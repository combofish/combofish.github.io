---
layout: post
title: "Java Web and Servlet(Study Note)"
date: "2021-04-02 17：58：00"
tag: Java JavaWeb Servlet HTTP
---
- [Java Web](#orgb0a3824)
  - [Web 资源的分类:按实现技术和呈现效果](#org2424668)
  - [常见服务器](#org998d699)
  - [安装](#org6520810)
  - [Tomcat 服务器目录解绍](#org1b8d83e)
  - [默认端口](#org251b116)
  - [部署 web 工程到 tomcat](#orgc720410)
  - [ROOT 工程的访问以及 默认 index.html 页面的访问](#orgc73a302)
  - [工程目录结构](#orgc376439)
  - [IDEA 工程设置](#org7d98d77)
  - [Tomcat10 库名更改](#orge74c004)
- [Servlet](#org33cf052)
  - [创建 Servlet 程序](#org40c5cdd)
  - [web.xml config(添加资源路径) 斜杠表示工程路径（约定大于配置）](#orgdbc3d84)
  - [servlet 生命周期](#orge85e75e)
  - [Servlet 请求的分发](#orgf946ac0)
  - [通过继承 HttpServlet 实现 Servlet 程序](#org64c25cc)
  - [IDEA 直接新建 Servlet 实现 Servlet 程序](#orgb63b3fb)
  - [ServletConfig 类](#org8629145)
    - [获取 Servlet 程序的别名 servlet-name 的值](#org48eb540)
    - [获取初始化参数 init-param](#orgf690154)
    - [获取 ServletContext 对象](#org17e4daa)
  - [ServletContext 类](#org24b00f2)
    - [域对象](#org4d13a31)
    - [ServletContext 类的四个常见作用](#orgff66387)
- [HTTP 协议](#org9b1e6ee)
  - [请求的格式](#org418c7d5)
    - [GET](#orgeeed389)
    - [POST](#org9484ee7)
    - [GET & POST 请求有哪些](#org008dcac)
  - [响应的格式](#orgce16cbd)
  - [常用的响应码](#org6defd80)
  - [MIME类型说明](#orgf1256ec)
- [HttpServletRequest 类](#org4a5bf7d)
  - [常用 request 方法](#org65a9843)
  - [代码实现](#org182ef1b)
  - [获取请求的参数](#org6f25a03)
  - [请求的转发](#orgeeead2e)
    - [请求转发的特点](#org7c0650e)
    - [JAVA 代码实现](#org5e96d8c)
    - [请求转发 html 实现跳转](#org62f667f)
    - [请求转发 Servlet 实现跳转页面](#orgf2469a7)
  - [斜杠的不同意义](#orgee04fe9)
- [HttpServletResponse 类](#org27b9173)
  - [字节流 & 字符流](#org1db2ea5)
  - [往客户端回传字符串数据](#orgdfe2de3)
  - [回传数据的中文显示问题](#org0639aef)
  - [请求重定向](#orge723de9)
  - [请求响应的特点](#org3af1d38)
  - [请求重定向的第二种方案 sendRedirect()](#org5b0615b)
- [END](#org5f89333)


<a id="orgb0a3824"></a>

# Java Web

-   Java web 是指所有通过 java 语言开发并可以同过浏览器访问的程序的总称，叫 Java Web。
-   Java Web 是基于请求和响应来开发的。
-   请求和响应成对出现。


<a id="org2424668"></a>

## Web 资源的分类:按实现技术和呈现效果

-   静态： html, css, js, txt, mp4。
-   动态: JSP Page, Servlet 程序， PHP, Asp。


<a id="org998d699"></a>

## 常见服务器

-   Tomcat Apache
-   Jboss
-   GlassFish
-   Resin
-   WebLogic


<a id="org6520810"></a>

## 安装

-   解压即可


<a id="org1b8d83e"></a>

## Tomcat 服务器目录解绍

-   bin 可执行文件
-   conf 配置文件
-   lib 服务器的 jar 包
-   logs 运行时输出的日志信息
-   temp 存储临时数据
-   webapps 存放部署的web工程
-   work 工作目录，存放jsp 运行时jsp 翻译为 Servlet 的源码和 Session 钝化的目录。


<a id="org251b116"></a>

## 默认端口

-   MySQL 默认端口 3306
-   Tomcat 默认端口 8080


<a id="orgc720410"></a>

## 部署 web 工程到 tomcat

-   第一种方法：把工程放到 ”webapps/工程名字“ 目录下。
-   第二种方法： conf 目录下写配置文件。/etc/tomcat9/Catalina/localhost/

```xml
<!-- Context 表示一个工程上下文
     path    访问路径
     docBase 工作目录的位置
-->
<Context path="/web01" docBase="/home/larry/"/>
```


<a id="orgc73a302"></a>

## ROOT 工程的访问以及 默认 index.html 页面的访问

-   访问ROOT目录下的工程，不需要带工程名。&#x2013;> ROOT/index.html


<a id="orgc376439"></a>

## 工程目录结构

-   src 存放自己编写的 java 源代码。
-   web 用来存放 web 工程的资源文件，比如 html page, css, js file。
    -   WEB-INF 是一个受服务器保护的目录，浏览器无法直接访问到此目录的内容。
        -   web.xml 它是整个动态 web 工程的配置部署描述文件，可以在这里配置 web 工程的组件：如 Servlet 程序，Filter 过滤器，Listener 监听器，Session 超时。
        -   lib 存放第三方的 jar 包，（idea） 还需要自己配置导入。


<a id="org7d98d77"></a>

## IDEA 工程设置

-   设置热修改，可以避免每一值修改后都手动更新服务器。 On frame deactivation: Update classes and resources.


<a id="orge74c004"></a>

## Tomcat10 库名更改

Tomcat 更新到10 后，部署服务，发现 javax.servlet. 包，原来是包名改变了。

```java
// Tomcat 10 用以下的 import
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
//----------------------------------------
// Tomcat 9 用以下的 import
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
```


<a id="org33cf052"></a>

# Servlet

-   Servlet 是 JAVA EE 的规范之一，是一个接口。
-   JAVA Web 三大组件之一，Servlet 程序, Filter 过滤器, Listen 监听器。
-   Tomcat 相当于一个容器，运行 Servlet。Servlet 可以接收客户端发来的请求，并响应给客户端。


<a id="org40c5cdd"></a>

## 创建 Servlet 程序

-   编写一个类去实现 Servlet 接口。
-   实现 servlet 方法，处理请求，并响应方法。
-   在 web.xml 中配置 servlet 程序的访问地址。


<a id="orgdbc3d84"></a>

## web.xml config(添加资源路径) 斜杠表示工程路径（约定大于配置）

-   servlet ： 标签 Tomcat 配置 Servlet 程序
    -   servlet-name tag ： Servlet 程序起一个別名（一般是类名）
    -   servlet-class tag ：servlet 程序的全类名，包名在前，类名在后。
-   servlet-mapping tag : 给 Servlet 程序配置访问地址
    -   servlet-name tag : 当前配置的地址给那个 Servlet 程序使用
    -   url-pattern tag : 配置访问地址, **注** 要以斜杠打头

```xml
<web-app>
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.hhh.hello</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-mapping>/helloServlet</url-mapping>
  </servlet-mapping>
</web-app>
```


<a id="orge85e75e"></a>

## servlet 生命周期

-   执行 Servlet 构造器方法, Servlet 中Service 方法是专门用来处理请求和响应的
-   执行 init 初始化方法
-   执行 Service 服务方法
-   执行 destroy 销毁方法

```java
package com.example.demo4;

import javax.servlet.*;
import java.io.IOException;
import java.io.PrintWriter;

public class HelloServlet3 implements Servlet {
    public HelloServlet3() {
	System.out.println("1: 构造器方法");
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
	System.out.println("2：初始化方法");
    }

    @Override
    public ServletConfig getServletConfig() {
	return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
	System.out.println("3: 服务器方法");

    }

    @Override
    public String getServletInfo() {
	return null;
    }

    @Override
    public void destroy() {
	System.out.println("4： Destroy方法");
    }
}
```

Servlet 构造方法和 init 方法创建时执行，Service 方法每次访问时执行, Destroy 方法停止时执行。


<a id="orgf946ac0"></a>

## Servlet 请求的分发

GET 与 POST 请求分离。

-   **注** alt + Enter : idea 快捷键: Introduce local variable.

```java
public void service(ServletRequest arg0, ServletResponse arg1) throws ServletException, IOException {
    // TODO Auto-generated method stub
    System.out.println("3. service");

    HttpServletRequest httpServletRequest  = (HttpServletRequest) arg0;
    String method = httpServletRequest.getMethod();
    System.out.println(method);
    if ("GET".equals(method)) {
	doGet();
    }else if ("POST".equals(method)) {
	doPost();
    }

    PrintWriter pw = arg1.getWriter();
    pw.println("Hello world");

}

public void doGet() {
    System.out.println("GET request!");
}

public void doPost() {
    System.out.println("POST request!");

}
```

前端模拟 post 请求

```html
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
  <head>
    <title>JSP - Hello World</title>
  </head>
  <body>
    <form action="http://localhost:8080/demo4_war_exploded/hello3" method="post">
      <input type="submit">
    </form>
  </body>
</html>
```


<a id="org64c25cc"></a>

## 通过继承 HttpServlet 实现 Servlet 程序

-   编写一个类去继承 HttpServlet 类，（HttpServlet 类，抽象类，实现了 service 方法，并实现了请求的分发。
-   根据业务需要去 **重写** doGet 或 doPost 方法
-   到 web.xml 中配置 Servlet 程序的访问

```java
package com.example.demo4;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class HelloServlet4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	//super.doGet(req, resp);
	System.out.println("HelloServlet4 doGet function!");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	//super.doPost(req, resp);
	System.out.println("HelloServlet4 doPost function!");
    }
}
```


<a id="orgb63b3fb"></a>

## IDEA 直接新建 Servlet 实现 Servlet 程序

idea -> new Servlet -> web.xml(config servlet-mapping)


<a id="org8629145"></a>

## ServletConfig 类

Servlet 程序和 ServletConfig 对象都是由 Tomcat 负责创建，我们负责使用。

-   Servlet 程序默认是第一次访问的时候创建。
-   ServletConfig 是每个 Servlet 程序创建时，就创建一个对应的 ServletConfig 对象。


<a id="org48eb540"></a>

### 获取 Servlet 程序的别名 servlet-name 的值

```java
@Override
public void init(ServletConfig servletConfig) throws ServletException {
    System.out.println("Servlet 别名: " + servletConfig.getServletName());
}
```


<a id="orgf690154"></a>

### 获取初始化参数 init-param

-   先配置

```xml
<servlet>
  <servlet-name>HelloServlet2</servlet-name>
  <servlet-class>com.example.demo4.HelloServlet2</servlet-class>
  <init-param>
    <param-name>username</param-name>
    <param-value>root</param-value>
  </init-param>
  <init-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql:/localhost:3306/test</param-value>
  </init-param>
</servlet>

<servlet-mapping>
  <servlet-name>HelloServlet2</servlet-name>
  <url-pattern>/hello2</url-pattern>
</servlet-mapping>
```

-   代码调用

```java
public void init(ServletConfig servletConfig) throws ServletException {
     System.out.println("Servlet 别名: " + servletConfig.getServletName());
     System.out.println("Init param 初始化参数 username is : " + servletConfig.getInitParameter("username"));
     System.out.println("Init param 初始化参数 url is : " + servletConfig.getInitParameter("url"));
 }
```


<a id="org17e4daa"></a>

### 获取 ServletContext 对象

```java
public class HelloServlet2 implements Servlet {
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
	System.out.println(servletConfig.getServletContext());
    }
}
```

-   result: org.apache.catalina.core.ApplicationContextFacade@2dbcc9b2


<a id="org24b00f2"></a>

## ServletContext 类

-   ServletContext 是一个接口，它表示 Servlet 上下文对象。
-   一个 web 工程，只有一个 ServletContext 对象实例。
-   ServletContext 对象是一个域对象。


<a id="org4d13a31"></a>

### 域对象

是可以像 Map 一样存取数据的对象，叫域对象。这里的域指的是数据存取的操作范围。

| type | 存数据         | 取数据         | 删除数据          |
|---- |-------------- |-------------- |----------------- |
| Map  | put()          | get()          | remove()          |
| 域对象 | setAttribute() | getAttribute() | removeAttribute() |


<a id="orgff66387"></a>

### ServletContext 类的四个常见作用

1.  获取 web.xml 中配置的上下文参数 context-param

    context-param tag 是整个上下文参数，它属于整个 web 工程。
    
    ```xml
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    	 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
    	 version="4.0">
      <context-param>
        <param-name>username</param-name>
        <param-value>larry</param-value>
      </context-param>
      <context-param>
        <param-name>password</param-name>
        <param-value>1234</param-value>
      </context-param>
    </web-app>
    ```
    
    -   代码获取
    
    ```java
    package com.example.demo4;
    
    import javax.servlet.*;
    import javax.servlet.http.*;
    import java.io.IOException;
    
    public class ContextServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    	ServletContext context = getServletConfig().getServletContext();
    	String username = context.getInitParameter("username");
    	String password = context.getInitParameter("password");
    
    	System.out.println("context-param 参数： username : " + username);
    	System.out.println("context-param 参数： password : " + password);
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
        }
    }
    ```

2.  获取当前的工程路径：格式： /工程路径

    -   获取工程部署后在服务器硬盘上的绝对路径
    
    ```java
    package com.example.demo4;
    
    import javax.servlet.*;
    import javax.servlet.http.*;
    import java.io.IOException;
    
    public class ContextServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    	ServletContext context = getServletConfig().getServletContext();
    	System.out.println("当前工程路径： " + context.getContextPath());
    	System.out.println("工程部署的路径是： " + context.getRealPath("/"));
    	/**
    	 * 当前工程路径： /demo4_war_exploded
    	 * 工程部署的路径是： /home/larry/IdeaProjects/demo4/target/demo4-1.0-SNAPSHOT/
    	 */
        }
    
        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
        }
    }
    
    ```

3.  像 Map 一样存取数据

    ```java
    package com.example.demo4;
    
    import javax.servlet.*;
    import javax.servlet.http.*;
    import java.io.IOException;
    
    public class ContextServlet1 extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    	// 获取 ServletContext 对象
    	ServletContext context = getServletContext();
    
    	context.setAttribute("key1","value1");
    	System.out.println("Context1 中获取数据 key1 的值是： " + context.getAttribute("key1"));
        }
    }
    ```
    
    -   创建 contextServlet2.java, 从里面访问 key1
    
    ```java
    package com.example.demo4;
    
    import javax.servlet.*;
    import javax.servlet.http.*;
    import java.io.IOException;
    
    public class ContextServlet2 extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    	ServletContext context = getServletContext();
    	System.out.println("从 ContextServlet2 中获取 key1: " + context.getAttribute("key1"));
        }
    }
    ```
    
    **注** 先访问 Context2, 再访问 Context1, 最后再访问 Context2。
    
    -   从 ContextServlet2 中获取 key1: null
    -   Context1 中获取数据 key1 的值是： value1
    -   从 ContextServlet2 中获取 key1: value1
    
    **注** 在两个 Servlet 中打印 context 对象: context 值一样。
    
    ```java
    /**
     * 从 ContextServlet2 中获取 key1: null
     * context in ContextServlet2 : org.apache.catalina.core.ApplicationContextFacade@2afe71aa
     * Context1 中获取数据 key1 的值是： value1
     * context in ContextServlet1 : org.apache.catalina.core.ApplicationContextFacade@2afe71aa
     * 从 ContextServlet2 中获取 key1: value1
     * context in ContextServlet2 : org.apache.catalina.core.ApplicationContextFacade@2afe71aa
     */
    ```


<a id="org9b1e6ee"></a>

# HTTP 协议

HTTP 协议中的数据又叫报文。


<a id="org418c7d5"></a>

## 请求的格式


<a id="orgeeed389"></a>

### GET

-   请求行
    -   请求的方式 GET
    -   请求的资源路径[+?+请求参数]
    -   请求的协议和版本号: HTTP/1.1
-   请求头：key:value 组成，不同的键值对，表示不同的含义。
    -   Accept: 告诉服务器，客户端可以接收的数据类型。
    -   Accept-Language: zh<sub>CN</sub> en<sub>US</sub>.
    -   User-Agent: 浏览器的信息
    -   Accept-Encoding: 告诉服务器，客户端可以接收的数据编码（压缩）格式。
    -   Host: 表示服务器的ip和端口号。
    -   Connection: 告诉服务器请求链接如何处理：
        -   Keep-Alive： 告诉服务器回传数据不要马上关闭，保持一小段时间的链接。
        -   Closed: 马上关闭。


<a id="org9484ee7"></a>

### POST

-   请求行
    -   请求的方式 POST
    -   请求的资源路径[+?+请求参数]
    -   请求的协议和版本号: HTTP/1.1
-   请求头：key:value 组成，不同的键值对，表示不同的含义。
    -   Accept: 告诉服务器，客户端可以接收的数据类型。
    -   Accept-Language: zh<sub>CN</sub> en<sub>US</sub>.
    -   Referer: 表示请求发起时，浏览器地址栏中的地址（从哪来）
    -   User-Agent: 浏览器的信息
    -   Content-Type: 表示发送的数据的类型。
        -   application/x-www-form-urlencoded
            -   表示提交的数据格式是：name=value&name=value, 然后对其进行 url 编码。
            -   url 编码把非英文的内容转换为： %xx%xx。
        -   multipart.form-data
            -   表示以多段的形式提交数据给服务器（以流的形式提交，用于上传）
    -   Content-Length: 发送的数据的长度。
    -   Cache-Control: 表示如何控制缓存：no-cache: 不缓存。
-   **空行**
-   请求体：发送给服务器的数据。action=login&username=root


<a id="org008dcac"></a>

### GET & POST 请求有哪些

1.  GET 请求

    -   html tag, method=get
    -   a tag
    -   link tag 引入 css
    -   Script tag 引入 js file
    -   img tag 引入图片
    -   iframe 引入 html 页面
    -   浏览器地址栏中的地址

2.  POST 请求

    -   html tag, method=post


<a id="orgce16cbd"></a>

## 响应的格式

-   响应行
    -   响应的协议和版本号 HTTP/1.1
    -   响应状态码 200
    -   响应状态描述符 OK
-   响应头：key:value 组成，不同的键值对，表示不同的含义。
    -   Server:
    -   Content-Type: 响应体的数据类型
    -   Content-Length: 响应体的长度
    -   Date: 请求响应的时间（这是格林时间）

**空行**

-   响应体：


<a id="org6defd80"></a>

## 常用的响应码

-   200 表示请求成功
-   302 表示请求重定向
-   404 表示服务器已经收到了，但你要的数据不存在
-   500 表示服务器已经收到了，但服务器内部错误(代码错误)


<a id="orgf1256ec"></a>

## MIME类型说明

-   { ".txt", "text/plain" } 等等


<a id="org4a5bf7d"></a>

# HttpServletRequest 类

每次只要有请求进入 Tomcat 服务器， Tomcat 服务器就会把请求过来的 HTTP 协议信息解析好装到 Request 对象中。 然后传递到 Service 方法中（doGet() 和 doPost() ）给我们用。我们可以通过 HttpServletRequest 对象来获取到所有请求信息。


<a id="org65a9843"></a>

## 常用 request 方法

```java
request.getRequestURI();        // 获取请求的资源路径
request.getRequestURL();        // 获取请求的统一资源定位符（绝对路径）
request.getRemoteHost();        // 获取客户的 ip 地址
request.getHeader();            // 获取请求头
request.getParameter();         // 获取请求的参数（多个值的时候使用）
request.getParameterValues();   // 获取请求的参数（多个值的时候使用）
request.getMethod();            // 获取请求的方式
request.setAttribute();         // 设置域数据
request.getAttribute();         // 获取域数据
request.getRequestDispatcher(); // 获取请求转发对象
```


<a id="org182ef1b"></a>

## 代码实现

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.IOException;

public class RequestAPIServlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	System.out.println("URI: " + request.getRequestURI());
	System.out.println("URL: " + request.getRequestURL());
	System.out.println("客户端 IP 地址： " + request.getRemoteHost());
	System.out.println("获取请求头： " + request.getHeader("User-Agent"));
	System.out.println("请求的方式： " + request.getMethod());
	/**
	 * URI: /demo4_war_exploded/request
	 * URL: http://localhost:8080/demo4_war_exploded/request
	 * 客户端 IP 地址： 0:0:0:0:0:0:0:1
	 * 获取请求头： Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36
	 * 请求的方式： GET
	 */
    }
}
```


<a id="org6f25a03"></a>

## 获取请求的参数

-   html page: form.html

```html
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/html">
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  <body>

    <form action="http://localhost:8080/demo4_war_exploded/parameter" method="get">
      用户名： <input type="text" name="username"> <br/>
      密码：  <input type="password" name="password"> <br/>
      兴趣爱好： <input type="checkbox" name="hobby" value="cpp"> CPP
      <input type="checkbox" name="hobby" value="java"> JAVA
      <input type="checkbox" name="hobby" value="php"> PHP
      <input type="submit">
    </form>

    <form action="http://localhost:8080/demo4_war_exploded/parameter" method="post">
      POST:用户名： <input type="text" name="username"> <br/>
      密码：  <input type="password" name="password"> <br/>
      兴趣爱好： <input type="checkbox" name="hobby" value="cpp"> CPP
      <input type="checkbox" name="hobby" value="java"> JAVA
      <input type="checkbox" name="hobby" value="php"> PHP
      <input type="submit">
    </form>
  </body>
</html>
```

-   java code
-   **注** 中文乱码时要设置字符集

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;
import java.lang.reflect.Array;
import java.util.Arrays;

public class ParameterServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	String username = request.getParameter("username");
	String password = request.getParameter("password");
	String hobby = request.getParameter("hobby");
	String[] hobbies = request.getParameterValues("hobby"); // 当选择多个值时

	System.out.println("Username : " + username);
	System.out.println("Password : " + password);
	System.out.println("Hobby : " + hobby);
	System.out.println("Hobbies : " + Arrays.asList(hobbies));
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	request.setCharacterEncoding("UTF-8");
	String username = request.getParameter("username");
	String password = request.getParameter("password");
	String hobby = request.getParameter("hobby");
	String[] hobbies = request.getParameterValues("hobby"); // 当选择多个值时

	System.out.println("Username : " + username);
	System.out.println("Password : " + password);
	System.out.println("Hobby : " + hobby);
	System.out.println("Hobbies : " + Arrays.asList(hobbies));
    }
}
```


<a id="orgeeead2e"></a>

## 请求的转发

服务器收到请求后，从一个资源跳到另一个资源的操作叫转发。Servlet1 -> Servlet2: 两个 Servlet 共同完成一个业务逻辑功能。


<a id="org7c0650e"></a>

### 请求转发的特点

-   浏览器地址没发生变化
-   是一次请求
-   它们共享 Request 域中的数据
-   可以转发到 WEB-INF 目录下 // can't


<a id="org5e96d8c"></a>

### JAVA 代码实现

-   Servlet1

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class Servlet1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	String username = request.getParameter("username");
	System.out.println("在 Servlet1 查看参数： " + username);

	// 传递到 Servlet2
	request.setAttribute("key","柜台1的章");
	/**
	 * 请求转发必须以斜杠打头， / 斜杠表示地址为 http://localhost:8080/工程名/ ， 映射到IDEA代码的 Web 目录下。
	 */
	//RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");
	RequestDispatcher requestDispatcher = request.getRequestDispatcher("/WEB-INF/form1.html"); //Can't
	requestDispatcher.forward(request,response);
    }
}
```

-   Servlet2

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.IOException;

public class Servlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	String username = request.getParameter("username");
	System.out.println("在 Servlet2 中看参数 " + username);

	// 查看 柜台1 是否盖章
	Object key = request.getAttribute("key");
	System.out.println("柜台1 是否有章： " + key);

	// 处理自己的业务
	System.out.println("Servlet2 处理自己的业务。");
    }
}
```


<a id="org62f667f"></a>

### 请求转发 html 实现跳转

-   index.jsp

```html
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
  <head>
    <title>JSP - Hello World</title>
  </head>
  <body>
    这是 web 下的 index.html 页面。<br/>
    <a href="a/b/c.html">跳转 a/b/c.html </a>
  </body>
</html>
```

-   a/b/c.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  <body>
    这是 a/b/c.html <br/>
    <a href="../../index.jsp"> Return to index.html</a>
  </body>
</html>
```


<a id="orgf2469a7"></a>

### 请求转发 Servlet 实现跳转页面

-   base 标签可以设置当前页面中所有相对路径工作时，参照哪个路径来进行跳转回去，如不设置，则以浏览器访问地址作为参考。

-   a/b/c.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <base href="http://localhost:8080/demo4_war_exploded/a/b/c.html">
    <title>Title</title>
  </head>
  <body>
    这是 a/b/c.html <br/>
    <a href="../../index.jsp"> Return to index.html</a>
  </body>
</html>
```

-   index.jsp

```html
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
  <head>
    <title>JSP - Hello World</title>
  </head>
  <body>
    这是 web 下的 index.html 页面。<br/>
    <a href="a/b/c.html">跳转 a/b/c.html </a>
    <a href="http://localhost:8080/demo4_war_exploded/forwardc">请求跳转 a/b/c.html </a>
  </body>
</html>
```

-   ForwardCServlet.java

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class FrowardC extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	System.out.println("经过 forward C 程序。");
	request.getRequestDispatcher("a/b/c.html").forward(request,response);
    }
}
```


<a id="orgee04fe9"></a>

## 斜杠的不同意义

-   斜杠如果被浏览器解析，表示： <http://localhost:8080/>

```html
<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
  <head>
    <title>JSP - Hello World</title>
  </head>
  <body>
    <a href="/">斜杠</a>
  </body>
</html>
```

-   斜杠被服务器解析，表示： <http://ip:port/project_name>

```html
<url-pattern>/servlet1</url-pattern>
```

-   如一下服务器端代码

```java
servletContext.getRealPath("/");
request.getRequestDispatcher("/");
```

-   特殊情况: 请求重定向：把斜杠发送给浏览器解析。

```java
response.sendRedirect("/"); 
```


<a id="org27b9173"></a>

# HttpServletResponse 类

和 HttpServletRequest 类一样，每次请求进来， Tomcat 都创建一个 Response 对象递给 Servlet 程序去使用。

-   HttpServletRequest 表示请求过来的信息， HttpServletResponse 表示所有响应的信息。


<a id="org1db2ea5"></a>

## 字节流 & 字符流

-   字节流 getOutputStream() 常用于下载，传递二进制数据。
-   字符流 getWriter() 常用于回传字符串。
-   两个流只能用一个。


<a id="orgdfe2de3"></a>

## 往客户端回传字符串数据

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;
import java.io.PrintWriter;

public class ResponseIOServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	PrintWriter writer = response.getWriter();
	writer.write("Response's content!");
    }
}
```


<a id="org0639aef"></a>

## 回传数据的中文显示问题

-   设置后端发送的字符集
-   在响应头设置浏览器显示的字符集

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;
import java.io.PrintWriter;

public class ResponseIOServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	// 方法一：
	// System.out.println(response.getCharacterEncoding());
	// response.setCharacterEncoding("UTF-8");
	// System.out.println(response.getCharacterEncoding());
	// response.setHeader("Content-Type","text/html; charset=UTF-8");

	// 方法二：
	// 它会同时设置服务器和客户端都使用 UTF-8 字符集， 还设置列响应头。
	// 此方法一定要在获取流之前调用才有效。
	response.setContentType("text/html; charset=UTF-8");
	System.out.println(response.getCharacterEncoding());
	PrintWriter writer = response.getWriter();
	writer.write("Response's content! 中文");
    }
}
```


<a id="orge723de9"></a>

## 请求重定向

请求重定向是指，客户端给服务器发送一个请求，服务器端告诉客户端说，我给你一个新地址，你去新地址访问。（因为之前的地址可能已经废弃）。

-   响应码 302
-   响应头 Location 新地址。

-   ResponseServlet1

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class ResponseServlet1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	System.out.println("明到此一游 in Servlet Response 1.");
	request.setAttribute("key1","username");
	response.setStatus(302);
	response.setHeader("Location","http://localhost:8080/demo4_war_exploded/response2");
    }
}
```

-   ResponseServlet2

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class ResponseServlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	response.getWriter().write("Response'2 result! And key1:  " + request.getAttribute("key1"));
    }
}
```


<a id="org3af1d38"></a>

## 请求响应的特点

-   浏览器地址栏发生变化
-   两次请求
-   不共享Request 域中的数据
-   不能访问 WEB-INF 目录下的资源
-   可以访问工程之外的资源，比如 <http://www.baidu.com>


<a id="org5b0615b"></a>

## 请求重定向的第二种方案 sendRedirect()

```java
package com.example.demo4;

import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class ResponseServlet1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	System.out.println("明到此一游 in Servlet Response 1.");
	response.sendRedirect("http://localhost:8080/demo4_war_exploded/response2");
    }
}
```


<a id="org5f89333"></a>

# END

本文是我看视频 [尚硅谷最新版JavaWeb全套教程,java web零基础入门完整版](https://www.bilibili.com/video/BV1Y7411K7zz) 后的笔记。如有错误欢迎指正。
