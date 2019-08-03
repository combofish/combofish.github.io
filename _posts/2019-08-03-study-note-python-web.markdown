---
layout: post
title: "Python Basis (5) Web. Study Note"
date: 2019-08-03
tag: Python Web
---
本文是我在学习期间的笔记，看的书是《python语言及其应用》。     

- [剖析Web](#orgdbaca80)
  - [python的标准Web库](#org5cda78c)
    - [使用标准库来获取网页内容](#orgecc8715)
    - [使用requests](#org95494f3)
  - [Web服务端](#org586f00a)
    - [最简单的python Web服务器](#org3a0be30)
    - [WSGI (WEB 服务器网关接口)](#orgd5951a0)
    - [框架](#org848bf31)
  - [Web 服务和自动化](#org853061e)
  - [习题测试](#orga2a36fa)
    - [flask code](#org5c3327b)
    - [template code](#orge99ba54)
    - [test](#orgf26d9a7)


<a id="orgdbaca80"></a>

# 剖析Web

-   HTTP 超文本传输协议
-   HTML 超文本标记语言
-   URL 统一资源定位符

Web大致有3层，客户端，服务端，Web API和服务。 互联网最底层的网络传输使用的是传输控制协议/因特网协议，更常用的叫法是TCP/IP


<a id="org5cda78c"></a>

## python的标准Web库

-   http 会处理所有客户端-服务器http请求的具体细节：
    -   client 处理客户端的部分
    -   server 协助编写python Web服务器程序
    -   cookies cookiejar 会处理cookie, cookie 可以在请求中存储数据

-   urllib 基于http的高层库：
    -   request 处理客户端请求
    -   response 处理服务器端响应
    -   parse 会解析URL


<a id="orgecc8715"></a>

### 使用标准库来获取网页内容

```python
import urllib.request as ur

url = 'https://cn.bing.com'
conn = ur.urlopen(url)
print("conn : ",conn)
data = conn.read()
print("len(data): ",len(data))
print("conn.status : ",conn.status)

for key,value in conn.getheaders():
    print(key,value)
```


<a id="org95494f3"></a>

### 使用requests

```python
import requests
url = 'https://cn.bing.com'
resp = requests.get(url)
print('resp : ',resp)
print(len(resp.text))

```


<a id="org586f00a"></a>

## Web服务端


<a id="org3a0be30"></a>

### 最简单的python Web服务器

```shell
python -m http.server
```

-   可以指定端口号

```shell
python -m http.server 9999
```


<a id="orgd5951a0"></a>

### WSGI (WEB 服务器网关接口)


<a id="org848bf31"></a>

### 框架

-   至少具有处理客户端请求和服务器端相应能力

特性：

-   路由
-   模板
-   认证和授权
-   Session

1.  Bottle

    ```python
    from bottle import route,run
    
    @route('/')
    def home():
        return "It isn't fancy, but it's my home page."
    
    run(host='localhost',port = 9999)
    
    ```
    
    -   我们创建一个xx.html文件。
    
    ```shell
    echo "My <b>new</b> and <i>improved</i> home page!!!" >> index-test.html
    cat index-test.html
    ```
    
    -   让页面请求时返回HTML内容
    
    ```python
    from bottle import run, route, static_file
    
    @route('/')
    def home():
        return static_file('index-test.html',root='.')
    
    run(host='localhost',port=9999)
    
    ```
    
    -   指定url中的参数，并使用他们
    
    ```python
    from bottle import run, route, static_file
    
    @route('/')
    def home():
        return static_file('index-test.html',root='.')
    
    @route('/echo/<thing>')
    def echo(thing):
        return "Say hello to my little friend %s!" % thing
    
    run(host='localhost',port=9999)
    
    ```
    
    -   测试刚才的代码能正常工作
    
    ```python
    import requests
    
    resp = requests.get('http://localhost:9998/echo/Jhon')
    if resp.status_code == 200 and \
       resp.text == 'Say hello to my little friend Jhon!':
        print("It worked! That almost never happens!")
    else:
        print('Argh, got this:', resp.text)
    
    ```
    
    -   bottle 的其他特性
        -   debug=True 如果出现HTTP错误，会创建一个调试页面
        -   reloader=True 如果修改了任何py代码，浏览器的页面会重新载入

2.  Flask

    -   Flask 的默认文件静态目录是static
    
    ```python
    from flask import Flask
    
    app = Flask(__name__,static_folder='.',static_url_path='')
    
    @app.route('/')
    def home():
        return app.send_static_file('./index-test.html')
    
    @app.route('/echo/<thing>')
    def echo(thing):
        return "Say hello to my little friend: %s" % thing
    
    app.run(port=9999,debug=True)
    
    ```
    
    -   FileName: flask3.py
    
    ```python
    from flask import Flask, render_template, request
    
    app = Flask(__name__,static_folder='.',static_url_path='')
    
    @app.route('/')
    def home():
        return app.send_static_file('./index-test.html')
    
    @app.route('/echo/')
    def echo():
        thing = request.args.get('thing')
        place = request.args.get('place')
        return render_template('flask2.html',thing=thing,place=place)
    
    app.run(port=9999,debug=True)
    ```
    
    -   test
    
    ```restclient
    GET http://localhost:9999/echo?thing=li&place=NewYork
    ```
    
    -   使用字典的\*\*操作符来一次性传入字典的多个值
    -   FIleName: flask4.py
    
    ```python
    from flask import Flask, render_template, request
    
    app = Flask(__name__,static_folder='.',static_url_path='')
    
    @app.route('/')
    def home():
        return app.send_static_file('./index-test.html')
    
    @app.route('/echo/')
    def echo():
        kwargs = {}
        kwargs['thing'] = request.args.get('thing')
        kwargs['place'] = request.args.get('place')
        return render_template('flask2.html',**kwargs)
    
    app.run(port=9999,debug=True)
    ```
    
    -   test
    
    ```restclient
    GET http://localhost:9999/echo?thing=li&place=NewYork
    ```

3.  非python的web服务器

    -   apache 加上mod<sub>wsgi模块</sub>
    -   nginx 加上uWSGI模块

4.  其他框架

    -   django 有ORM功能，可在网页中自动应用典型的数据库CRUD功能。
    -   web2py
    -   pyramid
    -   turbogears 支持ORM，多种数据库，以及多个模板语言
    -   wheezy.web 专为性能而生
    
    -   >>> install
    
    ```python
    import os
    
    pkgs = ("django","pyramid","wheezy.web")
    for pkg in pkgs:
        os.system('pip install %s --user $USER' % pkg )
    ```


<a id="org853061e"></a>

## Web 服务和自动化

1.  webbrowser模块

    -   webbrowser 可以完全控制浏览器
    
    ```python
    import webbrowser
    url = 'http://www.python.org'
    webbrowser.open(url)
    webbrowser.open_new(url)
    webbrowser.open_new_tab(url)
    
    ```

2.  Web API和表述性状态传递

    -   表述性状态传递(REST)是Roy Fielding在他的博士论文中提出的概念。
    -   HEAD 获取资源信息
    -   GET 从服务器取回资源的数据
    -   POST 更新服务器上的资源
    -   PUT 创建一个新的资源
    -   DELETE

3.  抓取数据

    -   >>> install scrapy use shell
    
    ```shell
    pip install scrapy --user $USER
    ```

4.  用BeautifulSoup来抓取数据

    ```python
    def get_links(url):
        from bs4 import BeautifulSoup as soup
        import requests
        result = requests.get(url)
        page = result.text
        doc =  soup(page)
        links = [ element.get('href') for element in doc.find_all('a')]
        return links
    
    if __name__ == '__main__':
        url = 'http://www.baidu.com'
        print('Links in ',url)
        for num, link in enumerate(get_links(url), start=1):
            print(num,link)
    
        print()
    
    ```


<a id="orga2a36fa"></a>

## 习题测试


<a id="org5c3327b"></a>

### flask code

```python
#!/usr/bin/env python
# coding:utf-8
# Filename: answer-flask.py

from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/')
def home():
    return "It's alive!"

@app.route('/home')
def anotherhome():
    kwargs = {}
    kwargs["thing"] = request.args.get('thing')
    kwargs["height"] = request.args.get('height')
    kwargs["color"] = request.args.get('color')
    return render_template('home.html',**kwargs)

app.run(port=5000,debug=True)
```


<a id="orge99ba54"></a>

### template code

```html
<html>
  <head>
    <tiele>It's alive!</tiele>
  </head>

  <body>
    I'm of course referring to {{ thing }}, which is {{ height }} feet tall and {{ color }}.
  </body>
</html>
```


<a id="orgf26d9a7"></a>

### test

-   '/'测试

```restclient
GET http://localhost:5000/
```

-   '/home'测试

```restclient
GET http://localhost:5000/home?thing=Octothorpe&height=7&color=green
```