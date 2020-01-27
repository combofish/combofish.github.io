---
layout: post
title: "Python: Web Crawl(Study Note)"
date: "2020-01-28 02：27：00"
tag: BeautifulSoup Scrapy Python requests
---
- [Python Web Crawl](#orgdc61ff7)
  - [requests 库](#org497e978)
    - [Request 对象](#org8619198)
    - [Response 对象的属性](#orgf4403ee)
    - [获取网页内容-代码示例](#org2706d5b)
    - [Requests 库的异常](#org7c7d705)
    - [通用代码框架](#org8c1be47)
    - [HTTP](#org3fb1baf)
    - [requests.request](#org551f572)
  - [网络爬虫带来的问题](#orgc2b614e)
    - [网页爬虫的尺寸](#org248b771)
    - [网络爬虫的限制](#org83d3514)
    - [Robots 协议](#org19ebe55)
  - [网络爬虫示例](#org863977b)
    - [爬取一个京东页面](#org3e4eaa7)
    - [爬取一个亚马逊页面](#orgc48af55)
    - [搜索引擎关键词提交](#orgba8822d)
    - [网络图片的爬取](#org9707942)
    - [IP地址归属地的自动查询](#orgcffd8e3)
  - [Beautiful Soup](#org7a14e97)
    - [初识](#org26856a2)
    - [对 BeautifulSoup 的理解](#org743d67f)
    - [基于 bs4 的HTML 内容的遍历方法](#org3a77f5d)
    - [基于 bs4 的HTML 内容输出](#orgeb5769b)
  - [信息组织与提取方法](#org64d0cee)
    - [信息的标记](#org4cb5d3b)
    - [信息提取的一般方法](#orgd992996)
    - [提取HTML 中所有的URL 连接(find<sub>all</sub> 函数)](#org09c94c9)
  - [网络爬虫示例-定向爬取中国大学排名](#orgf23d023)
    - [从网络上获取网页内容](#org1a7784c)
    - [提取网页信息到合适的数据结构中](#org8596e2a)
    - [利用数据结构展示并输出结果](#orgc472c02)
    - [调用代码](#org01ca4b9)
    - [美化输出](#orgb8998d8)
  - [网络爬虫示例-当当商品定向比价](#org7519b30)
  - [网络爬虫示例-股票数据定向爬虫](#orga7d2d72)
    - [程序实现](#org6895b10)
    - [优化](#orgb32ed2f)
  - [Scrapy 网络爬虫框架](#org6579d4c)
    - [requests vs. Scrapy](#orga0a561b)
    - [Scrapy 命令行](#orge0fe917)
    - [Scrapy 使用](#org592273b)
    - [yield 关键字](#org98a9d36)
    - [Scrapy 爬虫的使用](#org2dae41e)
    - [Scrapy 爬虫的数据类型](#org08c27b6)
  - [网络爬虫示例-Scrapy 获取股票名称和交易信息](#org4a63409)
    - [步骤](#org0f4bd42)
    - [程序优化](#org7aab748)


<a id="orgdc61ff7"></a>

# Python Web Crawl

The Website is the API. 本文是学习时做的笔记，原文写于 Org Format。 围绕 **定向网络数据爬取** 和 **网页解析** 。能教大家爬取一些简单的 HTML 网页。

**注：带有实战的例子，其中有两个爬取股票数据的示例不能正常运行，因网页api变动，不过数据处理逻辑仍然有借鉴之处。**


<a id="org497e978"></a>

## requests 库

-   requests.request()
-   requests.get(url,params=None,\*\*kwargs)
-   requests.head()
-   requests.post()
-   requests.put() 提交，能将原有邪恶信息覆盖掉
-   requests.delete()
-   requests.patch() 局部更改，节省网络带宽


<a id="org8619198"></a>

### Request 对象

构造一个向服务器请求资源的Request 对象,内部生成。

```python
r = requests.get(url)
```

函数返回 Response 对象


<a id="orgf4403ee"></a>

### Response 对象的属性

-   r.status<sub>code</sub>
-   r.text
-   r.encoding 从HTTP header 中猜测的响应内容编码方式

-   r.apparent<sub>encoding</sub> 从内容中分析出的响应内容编码方式(备选编码方式)

-   r.content


<a id="org2706d5b"></a>

### 获取网页内容-代码示例

```python
import requests

r = requests.get('http://www.baidu.com')
print(r.status_code)
r.encoding = 'utf-8'
print(r.text)
print(r.headers)
print(type(r))
print(r.content)
```

200

<class 'requests.models.Response'>


<a id="org7c7d705"></a>

### Requests 库的异常

-   requests.ConnectionError
-   requests.HTTPError
-   requests.URLRequired
-   requests.TooManyRedirects
-   requests.ConnectTimeout
-   requests.Timeout 请求超时

```python
r.raise_for_status()
```


<a id="org8c1be47"></a>

### 通用代码框架

```python
import requests

def getHTMLText(url):
    try:
	r = requests.get(url, timeout=30)
	r.raise_for_status()
	r.encoding = r.apparent_encoding
	return r.text
    except:
	return "Error"

url = "http://www.baidu.com"
print(getHTMLText(url))
print(getHTMLText("www.baidu.com"))

```


<a id="org3fb1baf"></a>

### HTTP

HTTP Hypertext Transfer Protocol 超文本传输协议,是一个基于“请求与响应”模式的无状态的应用层协议. URL 格式 http :// host [:port] [path]

1.  POST

    ```python
    import requests
    payload = { 'key1': 'value1', 'key2':'value2'}
    r = requests.post('http://httpbin.org/post', data = payload)
    print(r.text)
    
    ```
    
    ```python
    r = requests.post("http://httpbin.org/post",data = 'ABC')
    print(r.text)
    ```

2.  PUT

    ```python
    payload = {'key1': 'value1', 'key2':'value2'}
    r = requests.put('http://httpbin.org/put',data = payload)
    print(r.text)
    ```


<a id="org551f572"></a>

### requests.request

```python
requests.request(method.url,**kwargs)
```

\*\*kwargs: 控制访问的参数，均可为选项

-   params:
-   data: 字典，字节序列或文件对象，作为 Request 的内容。
-   json: JSON 格式数据，作为Request 的内容。
-   headers: 字典，HTTP定制头
-   cookies: 字典或CookieJar, Request中的cookie
-   auth: 元组， 支持HTTP认证功能
-   files: 字典类型，传输文件
-   timeout: 设定超时时间，秒为单位
-   proxies: 字典类型，设定访问代理服务器，可以增加登陆认证
-   allow<sub>redirects</sub>: True/False, 默认为True, 重定向开关
-   stream: True/False, 默认为True，获取内容立即下载开关
-   verify: True/False, 默认为True,认证SSL证书开关
-   cert: 本地SSL 证书路径
    
    ```python
    import requests
    
    kv = { 'key1':'value1', 'key2':'value2'}
    r = requests.request('GET','http://python123.io/ws',params=kv)
    print(r.url)
    
    body = 'main content'
    r = requests.request('POST','http://python123.io/ws',data=body)
    
    kv = {'key1':'value1'}
    r = requests.request('POST','http://python123.io/ws',json=kv)
    
    hd = {'user-agent':'Chrome/10'}
    r = requests.request('POST','http://python123.io/ws',headers=hd)
    
    fs = {'file': open('data.xls','rb')}
    r = requests.request('POST','http://python123.io/ws',files=fs)
    
    r = requests.request('POST','http://python123.io/ws',timeout=10)
    
    pxs = {'http':'http://user:pass@10.10.10.1:1234'
           'https':'https://10.10.10.1:4321'}
    r = requests.request('GET'，'http://www.baidu.com', proxies=pxs)
    ```


<a id="orgc2b614e"></a>

## 网络爬虫带来的问题

对服务器性能的骚扰问题，内容层面的法律问题，个人隐私泄漏问题。


<a id="org248b771"></a>

### 网页爬虫的尺寸

| 规模 |              |           | robots 协议 |
|--- |------------ |--------- |--------- |
| 小规模 | 数据量小，爬取网页 | Requests库 |           |
| 中规模 | 数据规模大，爬去速度敏感 | Scrapy库  | 商业利益：必须遵守 |
| 大规模 | 搜索引擎，爬取全网 | 定制开发  | 必须遵守  |


<a id="org83d3514"></a>

### 网络爬虫的限制

1.  来源审查

    判断 User-Agent 进行限制， 检查来访HTTP协议头的User-Agent 域，只响应浏览器或者友好爬虫的访问。

2.  发布公告

    Robots 协议，告知所有爬虫网站的爬去策略，要求爬虫遵守。


<a id="org19ebe55"></a>

### Robots 协议

Robots Exclusion Standard 网络爬虫排除标准 在网站根目录下的 robots.txt 文件，告知网络爬虫那些页面可以抓取，那些不可以。 网络爬虫： 自动或人工识别robots.txt,再进行内容爬去

例： 京东的 robots 协议。

```python
import requests

r = requests.get('https://www.jd.com/robots.txt')
print(r.text)

```

1.  基本语法

    ```shell
    # 注释， * 代表所有， /代表根目录
    User-agent: * 
    Disallow: /
    ```


<a id="org863977b"></a>

## 网络爬虫示例


<a id="org3e4eaa7"></a>

### 爬取一个京东页面

```python
import requests

url = 'https://item.jd.com/62234535959.html'
try:
    r = requests.get(url)
    r.raise_for_status()
    print(r.status_code)
    r.encoding = r.apparent_encoding
    print(r.text[:1000])
except:
    print('Error!')

```


<a id="orgc48af55"></a>

### 爬取一个亚马逊页面

```python
import requests

url = 'https://www.amazon.com/-/zh/dp/B07V3224V3/ref=lp_18637575011_1_1?srs=18637575011&ie=UTF8&qid=1580015328&sr=8-1'
r = requests.get(url)
print(r.status_code)
r.encoding = r.apparent_encoding
print(r.text)

```

503, 未能正常访问

```python
r.request.headers
```

```shell
{'Connection': 'keep-alive', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'User-Agent': 'python-requests/2.18.4'}

```

1.  修改头部信息

    ```python
    kv = { 'User-Agent':'Mozilla/5.0'}
    r = requests.get(url,headers=kv)
    print(r.status_code)
    print(r.request.headers)
    print(r.text[:1000])
    
    ```

2.  完整代码

    ```python
    import requests
    
    url = 'https://www.amazon.com/-/zh/dp/B07V3224V3/ref=lp_18637575011_1_1?srs=18637575011&ie=UTF8&qid=1580015328&sr=8-1'
    try:
        kv = {'user-agent': 'Mozilla/5.0'}
        r = requests.get(url,headers=kv)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        print(r.text[:1000])
    except:
        print('Error')
    ```


<a id="orgba8822d"></a>

### 搜索引擎关键词提交

1.  百度搜索

    百度：<http://www.baidu.com/s?wd=keyword>
    
    ```python
    import requests
    
    kv = {'wd':'Python'}
    r = requests.get('http://www.baidu.com/s',params=kv)
    print(r.status_code)
    print(r.request.url)
    print(len(r.text))
    
    ```

2.  360搜索

    360搜索： <http://www.so.com//s?q=keyword>
    
    ```python
    import requests
    
    keyword = 'Python'
    try:
        kv = {'q':keyword}
        r = requests.get('http://www.so.com/s',params=kv)
        print(r.request.url)
        r.raise_for_status()
        print(len(r.text))
    except:
        print("Error")
    
    ```


<a id="org9707942"></a>

### 网络图片的爬取

```python
import requests

path = '/home/larry/Downloads/down_photo.jpg'
url = 'http://img.netbian.com/file/2020/0122/small6ffe5839d7c23612b8f23e303211890c1579622447.jpg'
r = requests.get(url)
print(r.status_code)
with open(path,'wb') as f:
    f.write(r.content)


```

= >>> >>> >>> >>> 200 =

1.  完整代码

    ```python
    import requests
    import os
    
    url = 'http://img.netbian.com/file/2020/0122/small6ffe5839d7c23612b8f23e303211890c1579622447.jpg'
    root = '/home/larry/Downloads/pics/'
    path = root + url.split('/')[-1]
    try:
        if not os.path.exists(root):
    	os.mkdir(root)
    
        if not os.path.exists(path):
    	r = requests.get(url)
    	with open(path,'wb') as f:
    	    f.write(r.content)
    
    	print('file saved!')
        else:
    	print('File exist already')
    
    except:
        print('Error') 
    
    
    ```


<a id="orgcffd8e3"></a>

### IP地址归属地的自动查询

-   网站： www.ip138.com
-   URL: <http://m.ip138.com/ip.asp?ip=address>
    
    ```python
    import requests
    
    url = 'http://m.ip138.com/ip.asp?ip='
    
    try:
        r = requests.get(url + '202.204.80.112')
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        print(r.text[-500:])
    except: 
        print("Error")
    ```


<a id="org7a14e97"></a>

## Beautiful Soup


<a id="org26856a2"></a>

### 初识

-   安装

```shell
pip install beautifulsoup4
```

-   使用

```python
from bs4 import BeautifulSoup
```

```python
import requests

try:
   r = requests.get("http://python123.io/ws/demo.html")
   demo = r.text
except:
    print('Error')

```

-   BeautifulSoup 使用

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(demo, 'html.parser') # 解析器
print(soup.prettify())
```


<a id="org743d67f"></a>

### 对 BeautifulSoup 的理解

Beautiful Soup 库是解析，遍历，维护“标签树”的功能库。标签树 <-> BeautifulSoup 类

| 解析器       | 使用方法                        | 条件                 |
|------------ |------------------------------- |-------------------- |
| bs4的HTML解析器 | BeautifulSoup(mk,'html.parser') | bs4                  |
| lxml的HTML解析器 | BeautifulSoup(mk,'lxml')        | pip install lxml     |
| lxml的XML解析器 | BeautifulSoup(mk,'xml')         | pip install lxml     |
| html5lib的解析器 | BeautifulSoup(mk,'html5lib')    | pip install html5lib |

1.  Beautiful Soup 类的基本元素

    -   Tag 标签，最基本的信息组织单元
    -   Name 标签的名字， <tag>.name
    -   Attributes 标签的属性，字典形式组织 <tag>.attrs
    -   NavigableString 标签内非属性的字符串 <tag>.string
    -   Comment 标签内字符串的注释部分，一种特殊的Comment类型
    
    ```python
    import requests
    from bs4 import BeautifulSoup
    
    url = ''
    try:
        r = requests.get(url)
        r.raise_for_status()
        demo = r.text
    except:
        print('Error')
    
    soup = BeautifulSoup(demo,'html.parser')
    print(soup.title)
    print(soup.a)
    
    # Name
    print(soup.a.name)
    print(soup.a.parent.name)
    print(soup.a.parent.parent.name)
    
    # Attribute
    tag = soup.a
    print(tag.attrs)
    print(tag.attrs['class'])
    print(tag.attrs['href'])
    print(type(tag.attrs))
    print(type(tag))
    
    print(soup.a, soup.a.string)
    print(soup.p, soup.p.string)
    ```
    
    ```python
    newsoup = BeautifulSoup("<b><!--This is a comment--></b><p>This is not a comment</p>",'html.parser')
    print(newsoup.b.string,type(newsoup.b.string))
    print(newsoup.p.string, type(newsoup.p.string))
    ```


<a id="org3a77f5d"></a>

### 基于 bs4 的HTML 内容的遍历方法

-   上行遍历，下行遍历，平行遍历

1.  标签树的下行遍历

    -   .contents
    
    子节点的列表，将<tag> 所有子节点存入列表
    
    -   .children
    
    子节点的迭代类型， 与.contents类似， 用于循环遍历子节点
    
    -   .descendants
    
    子孙节点的迭代类型，包含所有子孙节点，用于循环遍历
    
    ```python
    import requests
    from bs4 import BeautifulSoup
    
    try:
        r = requests.get("http://python123.io/ws/demo.html")
        r.raise_for_status()
        demo = r.text
    
    except:
        print('Error')
    
    soup = BeautifulSoup(demo,'html.parser')
    print(soup.prettify())
    print(soup.head)
    print(soup.head.contents)
    print(soup.body.contents)
    
    ```

2.  标签树的上行遍历

    -   .parent 节点的父亲标签
    -   .parents 节点先辈标签的迭代类型，用于循环遍历先辈节点
    
    ```python
    soup = BeautifulSoup(demo, 'html.parser')
    print(soup.title.parent)
    print(soup.html.parent)
    print(soup.parent)
    ```
    
    -   上行遍历
    
    ```python
    soup = BeautifulSoup(demo, 'html.parser')
    for parent in soup.a.parents:
        if parent is None:
    	print(parent)
        else:
    	print(parent.name)
    ```

3.  标签树的平行遍历

    平行遍历发生在同一个父亲节点下的各节点间
    
    -   .next<sub>sibling</sub>
    
    返回按照HTML文本顺序的下一个平行节点标签
    
    -   .previous.sibling
    
    返回按照HTML文本顺序的上一个平行节点标签
    
    -   .next<sub>siblings</sub>
    
    返回按照HTML文本顺序的后续所有平行节点标签
    
    -   .previous<sub>siblings</sub>
    
    返回按照HTML文本顺序的前续所有平行节点标签
    
    ```python
    soup = BeautifulSoup(demo, 'html.parser')
    print(soup.prettify())
    print(soup.a.next_sibling)
    print(soup.a.next_sibling.next_sibling)
    print(soup.a.previous_sibling)
    print(soup.a.previous_sibling.previous_sibling)
    print(soup.a.parent)
    ```
    
    ```python
    for sibling in soup.a.next_siblings:
        print(sibling)
    
    for sibling in soup.a.previous_siblings:
        print(sibling)
    
    ```


<a id="orgeb5769b"></a>

### 基于 bs4 的HTML 内容输出

```python
import requests
from bs4 import BeautifulSoup

try:
   r = requests.get("http://python123.io/ws/demo.html")
   demo = r.text
except:
   print('Error')

soup = BeautifulSoup(demo,'html.parser')
print(soup.prettify())
print(soup.a.prettify())

```

-   bs4 编码 utf-8

```python
soup = BeautifulSoup('<p>中文</p>','html.parser')
print(soup.p.string)
print(soup.p.prettify())
```


<a id="org64d0cee"></a>

## 信息组织与提取方法


<a id="org4cb5d3b"></a>

### 信息的标记

标记后的信息可形成信息的组织结构，增加了信息的维度。

-   HTML

-   XML eXtensible Markup Language

-   JSON JavaScript Object Notation

健值对的组合

-   yaml YAML Ain't Markup Language

无类型的健值对 缩进表示所属关系，-表达并列关系，#表示注释，|表达整块数据。

1.  对比

    -   XML 最早的通用信息标记语言，可扩展性好，但繁琐。
    
    Internet 上的信息交互与传递
    
    -   JSON 信息类型，适合程序处理(js),较XML 简洁。程序代码的一部分。
    
    移动应用云端和节点的信息通信，无注释
    
    -   YAML 各种系统的配置文件，有注释易读。


<a id="orgd992996"></a>

### 信息提取的一般方法

-   完整解析信息的标记形式，再提取关键信息，需要解析器，优点信息解析准确，缺点提取过程繁琐，速度慢。
-   无视标记形式，直接搜索关键信息，对信息的文本查找函数即可。提取过程简洁，速度较快。提取结果准确性与信息内容相关。
-   融合方法。


<a id="org09c94c9"></a>

### 提取HTML 中所有的URL 连接(find<sub>all</sub> 函数)

<>.find<sub>all</sub>(name,attrs,recursive,string,\*\*kwargs) 返回一个列表类型，存储查找的结果。

-   name 对标签名称的检索字符串
-   attrs 对标签属性值的检索字符串，可标注属性检索
-   recursive 是否对子孙全部检索，默认为True
-   string 对字符串区域的检索字符串

1.  find<sub>all</sub> 函数

    ```python
    from bs4 import BeautifulSoup
    
    soup = BeautifulSoup(demo, 'html.parser')
    for link in soup.find_all('a'):
        print(link['href'])
        print(link.get('href'))
    
    for link in soup.find_all(['a','b']):
        print(link)
    
     for link in soup.find_all(True):
        print(link.name)
    ```

2.  显示 以 b 开头的标签信息

    ```python
    import re
    
    for tag in soup.find_all(re.compile('b')):
        print(tag.name)
    
    ```

3.  包含属性查找

    ```python
    soup.find_all('p','course')
    ```

4.  属性中域查找

    ```python
    print(soup.find_all(id='link1'))
    print(soup.find_all(id='link'))
    print(soup.find_all(id=re.compile('link')))
    ```

5.  不查找孙节点及以后的标签

    ```python
    print(soup.find_all('a'))
    print(soup.find_all('a',recursive=False))
    
    ```

6.  string 参数使用

    ```python
    print(soup)
    print(soup.find_all(string='Basic Python'))
    print(soup.find_all(string=re.compile('python')))
    
    ```

7.  简写形式

    -   <tag>(..) 等价于 <tag>.find<sub>all</sub>(..)
    -   soup(..) 等价于 soup.find<sub>all</sub>(..)

8.  扩展方法

    -   <>.find()
    -   <>.find<sub>parents</sub>()
    -   <>.find<sub>parent</sub>()
    -   <>.find<sub>next</sub><sub>siblings</sub>()
    -   <>.find<sub>next</sub><sub>sibling</sub>()
    -   <>.find<sub>previous</sub><sub>siblings</sub>()
    -   <>.find<sub>previous</sub><sub>sibling</sub>()


<a id="orgf23d023"></a>

## 网络爬虫示例-定向爬取中国大学排名

程序结构设计：


<a id="org1a7784c"></a>

### 从网络上获取网页内容

getHTMLText()

```python
import requests
from bs4 import BeautifulSoup
import bs4

def getHTMLText(url):
    try:
	r = requests.get(url,timeout=30)
	r.raise_for_status()
	print(r.encoding)
	r.encoding = r.apparent_encoding
	print(r.encoding)
	return r.text
    except:
	print("Error")
	return ""

```


<a id="org8596e2a"></a>

### 提取网页信息到合适的数据结构中

fillUnivList()

```python
def fillUnivList(ulist,html): 
    soup = BeautifulSoup(html,'html.parser')
    # print(soup.find_all('div'))
    for tr in soup.find('tbody').children:
	if isinstance(tr, bs4.element.Tag):
	    tds = tr('td')
	    ulist.append([tds[0].string,tds[1].string,tds[2].string])

```


<a id="orgc472c02"></a>

### 利用数据结构展示并输出结果

printUnivList()

```python
def printUnivList(ulist, num):
    print("{:^10}\t{:^6}\t{:^10}".format("排名","学校名称","总分"))
    for i in range(num):
	#print(ulist[i])
	u = ulist[i]
	print("{:^10}\t{:^6}\t{:^10}".format(u[0],u[1],u[2]))

```


<a id="org01ca4b9"></a>

### 调用代码

```python
url = 'http://www.zuihaodaxue.com/zuihaodaxuepaiming2019.html'
uinfo = []

html = getHTMLText(url)
fillUnivList(uinfo, html)
printUnivList(uinfo, 20)
```


<a id="orgb8998d8"></a>

### 美化输出

```python
def printUnivList(ulist, num):
    tplt = "{0:^6}\t{1:{5}^10}\t{2:^6}\t{3:^6}\t{4:^6}"
    print(tplt.format("排名","学校名称","省市","总分","指标得分",chr(12288)))
    for i in range(num):
	#print(ulist[i])
	u = ulist[i]
	print(tplt.format(u[0],u[1],u[2],u[3],u[4],chr(12288)))


```


<a id="org7519b30"></a>

## 网络爬虫示例-当当商品定向比价

程序的结构设计

-   提交商品请求，循环获取页面
-   对每个页面，提取商品名称和价格信息
-   将信息输出到屏幕

```python
#!/usr/bin/env python
# author: combofish
# Filename: crawler_dangdang_goods.py

import requests
import re

def getHTMLText(url):
    try:
	r = requests.get(url,timeout=30)
	r.raise_for_status()
	r.encoding = r.apparent_encoding
	# rint(">>>>>>>>>>>")
	# print(r.text)
	return r.text
    except:
	return ""

def parsePage(ilt, html):
    try:
	# search_now_price">&yen;32.90</span>
	# class="search_discount">&nbsp;(6.72折) </s
	plt = re.findall(r'now\_.*?[\d\.]+', html)
	tlt = re.findall(r'nbsp\;\([\d\.]*', html)
	print(plt)
	print(tlt)

	for i in range(len(plt)):
	    price = eval(plt[i].split(';')[1])
	    title = eval(tlt[i].split('(')[1])
	    ilt.append([title, price])

    except:
	print('function- parsePage Error')


def printGoodsList(ilt):
    tplt = '{:4}\t{:16}\t{:8}'
    print(tplt.format('序号','商品名称','价格'))
    count = 0
    for g in ilt:
	count = count + 1
	if count < 20:
	    print(tplt.format(count, g[0], g[1]))


if __name__ == '__main__':

    goods = '数学之美'
    depth = 2
    start_url = "http://search.dangdang.com/?key=" + goods + "&act=input" 
    infoList = []
    for i in range(depth):
	try:
	    # http://search.dangdang.com/?key=%E6%95%B0%E5%AD%A6%E4%B9%8B%E7%BE%8E&act=input&page_index=3
	    url = start_url + '&page_index=' + str(i+1)
	    html = getHTMLText(url)
	    parsePage(infoList, html)
	except:
	    print('Error')
	    continue

    printGoodsList(infoList)


```


<a id="orga7d2d72"></a>

## 网络爬虫示例-股票数据定向爬虫

目标：获取上交所和深交所所有股票的名称和交易信息

程序结构设计：

-   获取股票列表
-   根据股票列表逐个获取个股信息
-   保存结果到文件


<a id="org6895b10"></a>

### 程序实现

```python
import requests
import re
from bs4 import BeautifulSoup
import traceback


def getHTMLText(url, code='utf-8'):
    try:
	r = requests.get(url,timeout=30)
	r.raise_for_status()
	# r.encoding = r.apparent_encoding
	r.encoding = code
	return r.text

    except:
	return ""


def getStockList(lst, stockURL):
    html = getHTMLText(stockURL, code='GB2312')
    soup = BeautifulSoup(html,'html.parser')
    a = soup.find_all('a')
    for i in a:
	try:
	    href = i.attrs['href']
	    lst.append(re.findall(r'[s][hz]\d{6}',href)[0])

	except:
	    continue


def getInfoList(lst, stockURL, fpath):
    count = 0

    for stock in lst:
	url = stockURL + stock + '.html'
	html = getHTMLText(url)
	try:
	    if html = '':
		continue

	    infoDict = {}
	    soup = BeautifulSoup(html, 'html.parser')
	    stockInfo = soup.find('div', attrs={'class':'stock-bets'})

	    name = stockInfo.find_all(attrs={'class':'bets-name'})[0]
	    infoDict.update({'股票名称': name.text.split()[0]})

	    keyList = stockInfo.find_all('dt')
	    valueList = stockInfo.find_all('dd')
	    for i in range(len(keyList)):
		key = keyList[i].text
		value = valueList[i].text
		infoDice[key] = value

	    with open(fpath, 'a', encoding='utf-8') as f:
		f.write(str(infoDict) + '\n')
		count = count + 1
		print("\r当前进度: {:.2f} %".format(count*100/len(lst)),end="")   

	except:
	    count = count + 1
	    print("\r当前进度: {:.2f} %".format(count*100/len(lst)),end="")   
	    # traceback.print_exec()
	    continue


def main():
    stock_list_url = 'http://quote.eastmoney.com/stocks'
    stock_info_url = 'http://quote.eastmoney.com/stocks'
    output_file = '/home/larry/Downloads/StockInfo.txt'
    slist = []
    getStockList(slist, stock_list_url)
    getStockInfo(slist, stock_info_url, output_file)


main()


```


<a id="orgb32ed2f"></a>

### 优化

-   提前设定网页编码方式，减少判断编码方式的时间

```python
def getHTMLText(url, code = 'utf-8'):
    r.encoding = code

```

-   动态显示当前进度 "\r" 打印后的光标提到当前行的头部

```python
count = count + 1
print('\r当前速度: {:.2f}%'.format(count*100 / len(lst)), end ='')

```


<a id="org6579d4c"></a>

## Scrapy 网络爬虫框架

爬虫框架是实现爬虫功能的一个软件结构和功能组件集合，5+2 结构

```shell
pip install scrapy

```

Scrapy 爬虫： 具有企业级专业爬虫的扩展性（7\*24 高可靠性)， 千万级 URL 爬取管理与部署， 足够一般

5个主要部分： SPIDERS, ENGINE, SCHEDULER, DOWNLOADER, ITEM PIPELINES， 2个中间件。 用户需要编写的是 SPIDERS, ITEM PIPELINES,

-   ENGINE 不需要用户修改，控制所有模块之间的数据流，根据条件触发事件
-   DOWNLOADER 根据请求下载网页，不需要用户修改
-   Scheduler 对所有爬去请求进行调度管理，不需要用户修改
-   Downloader Middleware 实施 Engine, Scheduler 和 Downloader 之间进行用户可配置的控制，修改，丢弃，新增请求或者响应
-   Spider 解析 Downloader 返回的响应 (Response), 产生爬取项 (scraped item), 产生额外的爬去请求 (Request)
-   Item Pipelines
    -   以流水线方式处理 Spider 产生的爬取项
    -   由一组操作顺序组成，类似流水线，每个操作是一个 Item Pipelines 类型
    -   可能操作包括：清理，检验和查重爬取项中的HTML数据，将数据存储到数据库
-   Spider Middleware 对请求和爬取项再处理，修改，丢弃，新增请求或者爬取项


<a id="orga0a561b"></a>

### requests vs. Scrapy

-   共同点： 都可以进行页面请求和爬取，Python 爬虫的两个重要技术路线

两者都没有处理js，提交表单，应对验证码等功能（可扩展）。

| requests    | Scrapy        |
|----------- |------------- |
| 页面级爬虫  | 网站级爬虫    |
| 功能库      | 框架          |
| 并发行考虑不足，性能差 | 并发行好，性能较高 |
| 重点在于页面下载 | 重点在于爬虫结构 |
| 定制灵活    | 一般定制灵活，深入定制困难 |


<a id="orge0fe917"></a>

### Scrapy 命令行

Scrapy 是为持续运行设计的专业爬虫框架，提供 Scrapy 命令行。scrapy shell 进入

常用命令

```shell
startproject 创建一个新工程        scrapy startproject <name> [dir]
genspider    运行一个爬虫          scrapy genspider [options] <name> <domain>
settings     获得爬虫配置信息       scrapy settings [options]
crawl        运行一个爬虫           scrapy crawl <spider>
list         列出工程中所有的爬虫    scrapy list
shell        启动URL 调试CLI       scrapy shell [url]

```


<a id="org592273b"></a>

### Scrapy 使用

建立工程

```shell
scrapy startproject python123demo

```

目录结构：

-   scrapy.cfg 部署Scrapy爬虫的配置文件
-   python123demo
    -   init.py
    -   items.py
    -   middlewares.py
    -   pipelines.py
    -   settings.py
    -   spiders/ 代码模板目录(继承类)

```shell
scrapy genspider demo python123.io

```

会在spiders/ 中生成 demo.py , 文件修改如下

```python
# -*- coding: utf-8 -*-
import scrapy


class DemoSpider(scrapy.Spider):
    name = 'demo'
    # allowed_domains = ['python123.io']
    start_urls = ['http://python123.io/ws/demo.html']

    def parse(self, response):
	fname = response.url.split('/')[-1]
	with open(fname, 'wb') as f:
	    f.write(response.body)

	self.log('Saved file %s.' % fname)

```

完整版代码

```python
# -*- coding: utf-8 -*-
import scrapy


class DemoSpider(scrapy.Spider):
    name = 'demo'

    def start_requests(self):
	urls = [
	    'http://python123.io/ws/demo.html'
	]
	for url in urls:
	    yield scrapy.Request(url=url, callback=self.parse)


    def parse(self, response):
	fname = response.url.split('/')[-1]
	with open(fname, 'wb') as f:
	    f.write(response.body)

	self.log('Saved file %s.' % fname)

```


<a id="org98a9d36"></a>

### yield 关键字

yield <-> 生成器

-   生成器是一个不断产生值的函数
-   包含 yield 语句的函数是一个生成器
-   生成器每次产生一个值， yield 语句， 函数被冻结， 被唤醒后再产生一个值

1.  生成器写法

    ```python
    def gen(n):
        for i in range(n):
    	yield i**2
    
    
    for i in gen(5):
        print(i)
    
    ```

2.  普通写法

    ```python
    def square(n):
        ls = [ i**2 for i in range(n) ]
        return ls
    
    
    for i in square(5):
        print(i)
    
    ```

3.  优点

    当一个列表比较大，而又不是一次就要使用时，用 生成器 会占用更少的资源


<a id="org2dae41e"></a>

### Scrapy 爬虫的使用

使用步骤

-   创建一个工程和 Spider 模板
-   编写 Spider
-   编写 Item Pipeline
-   优化配置策略


<a id="org08c27b6"></a>

### Scrapy 爬虫的数据类型

1.  Request 类

    class scrapy.http.Request()
    
    -   Request 对象表示一个 HTTP 请求
    -   由 Spider 生成， 由 Downloader 执行
    
    常用属性或方法
    
    -   url
    -   method 对应的请求方法
    -   headers 字典类型风格的请求头
    -   body 请求内容主题，字符串类型
    -   meta 用户添加的扩展信息，在 Scrapy 内部模块间传递信息使用
    -   copy() 复制该请求

2.  Response 类

    class scrapy.http.Response()
    
    -   Response 对象表示一个 HTTP 请求
    -   由 Downloader 生成， 由 Spider 执行
    
    常用属性或方法
    
    -   url
    -   status HTTP 的状态码，默认 200
    -   headers 字典类型风格的请求头
    -   body Response 内容主题，字符串类型
    -   flags 一组标记
    -   request 产生 Response 类型对应的 Request 对象
    -   copy() 复制该请求

3.  Item 类

    class scrapy.item.Item()
    
    -   Item 对象表示一个从 HTML 页面中提取的信息内容
    -   由 Spider 生成， 由 Item Pipeline 执行
    -   Item 类似字典类型， 可以按照字典类型操作

4.  Scrapy 爬虫信息提取方法

    Scrapy 爬虫支持多种 HTML 信息提取方法： BeautifulSoup, lxml, re, XPath Selector, CSS Selector.
    
    -   CSS Selector 使用方法
    
    ```python
    <html>.css('a:attr(href)').extract()
    
    ```


<a id="org4a63409"></a>

## 网络爬虫示例-Scrapy 获取股票名称和交易信息

-   获取股票列表
-   获取个股的详细信息


<a id="org0f4bd42"></a>

### 步骤

建立工程和 Spider 模板， 编写 Spider, 编写 ITEM Pipelines

```shell
scrapy startproject stockscraw
cd stockscraw
scrapy genspider stocks app.finance.ifeng.com

```

编写 Spider, 即修改 spiders/stocks.py

```python
# -*- coding: utf-8 -*-
import scrapy
import re

class StocksSpider(scrapy.Spider):
    name = 'stocks'
    start_urls = [
	'http://app.finance.ifeng.com/list/stock.php?t=ha'
    ]

    def parse(self, response):
	for href in response.css('a::attr(href)').extract():
	    try:
		stock = re.findall(r'[s][hz]\d{6}', href)[0]
		url = 'http://finance.ifeng.com/app/hq/stock/sh'+ stock + '/index.shtml'
		yield scrapy.Request(url, callback=self.parse_stock)

	    except:
		continue

    def parse_stock(self, response):
	infoDict = {}
	stockInfo = response.css('.stock-bets')
	name = stockInfo.css('.bets-name').extract()[0]
	keyList = stockInfo.css('dt').extract()
	valueList = stockInfo.css('dd').extract()
	for i in range(len(keyList)):
	    key = re.findall(r'>/*</dt>', keyList[i])[0][1:-5]
	    try:
		val = re.findall(r'\d+\.?.*</dd>', valueList[i])[0][0:-5]
	    except:
		val = '--'

	    infoDict[key] = value

	infoDict.update(
	    {'股票名称': re.findall(r'\s.*\(',name)[0].split()[0] + \
	     re.findall('>.*\<', name)[0][1:-1]})
	yield

```

编写 pipelines, 配置 pipelines.py 文件，定义对爬取项(Scraped Item)的处理类

-   pipelines.py

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html


class StockscrawPipeline(object):
    def process_item(self, item, spider):
	return item

class StocksInfoPipeline(object):
    def open_spider(self, spider):
	self.f = open('StockInfo.txt', 'w')

    def close_spider(self, spider):
	self.f.close()

    def process_item(self, item, spider):
	try:
	    line = str(dict(item)) + '\n'
	    self.f.write(line)

	except:
	    pass

	return item

```

配置 settings.py 中的 ITEM<sub>PIPELINES</sub> ， 让爬虫能找到新类

```python
# Configure item pipelines
# See https://doc.scrapy.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
     'stockscraw.piplines.StocksInfoPipeline':300
#    'stockscraw.pipelines.StockscrawPipeline': 300,
}

```

执行

```shell
scrapy crawl stocks

```


<a id="org7aab748"></a>

### 程序优化

1.  配置并发连接选项

    settings.py
    
    -   CONCURRENT<sub>REQUESTS</sub> Downloader 最大并发请求下载的数量， 默认32
    -   CONCURRENT<sub>ITEMS</sub> Item Pipeline 最大并发 ITEM 处理数量， 默认 100
    -   CONCURRENT<sub>REQUESTS</sub><sub>PRE</sub><sub>DOMAIN</sub> 每个目标域最大的并发请求数量， 默认8
    -   CONCURRENT<sub>REQUEST</sub><sub>PRE</sub><sub>IP</sub> 每个目标IP最大的并发请求数量， 默认0, 非0有效
    
    根据需要进行修改
