---
layout: post
title: "Setup Python Environment Path on Windows(新手篇)"
date: 2019-08-05
tag: Path Python
---
- [PATH](#org9961a2b)
  - [Python 安装位置](#org206aa28)
  - [Python 自带脚本的位置](#org06fba56)
  - [pip 安装的脚本的位置](#org6aaccff)
    - [site-packages](#orga550333)
    - [Scripts](#orgf56d661)
  - [总结](#org53228d3)
  - [查看安装的第三方模块的位置](#org2ea23f3)
    - [用包管理器 pip 查看](#org74b3412)
    - [使用 sys 模块查看](#org4318f69)
    - [使用 sys.modules 函数查看](#org99cc604)


<a id="org9961a2b"></a>

# Path

博主电脑安装Python时以默认路径安装的Python


<a id="org206aa28"></a>

## Python 安装位置

C:\Users\Administrator\AppData\Local\Programs\Python\Python37

-   包含 python.exe 也就是我们常用在终端中打开Python的命令


<a id="org06fba56"></a>

## Python 自带脚本的位置

C:\Users\Administrator\AppData\Local\Programs\Python\Python37\Scripts

-   包含 pip.exe, easy<sub>install.exe</sub> Python包管理的位置


<a id="org6aaccff"></a>

## pip 安装的脚本的位置

C:\Users\Administrator\AppData\Roaming\Python\Python37

-   这个文件夹下有两个文件夹 site-packages, Scripts


<a id="orga550333"></a>

### site-packages

-   安装的库的位置 比如 ipython 会在此目录下有一个文件夹


<a id="orgf56d661"></a>

### Scripts

-   安装的可执行脚本的位置，比如 ipython.exe, jupter.exe


<a id="org53228d3"></a>

## 总结

-   所以要想在终端下使用Python，就需要添加三个位置，即三个有可执行文件的目录
    -   C:\Users\Administrator\AppData\Roaming\Python\Python37\Scripts
    -   C:\Users\Administrator\AppData\Local\Programs\Python\Python37\Scripts
    -   C:\Users\Administrator\AppData\Local\Programs\Python\Python37


<a id="org2ea23f3"></a>

## 查看安装的第三方模块的位置


<a id="org74b3412"></a>

### 用包管理器 pip 查看

```shell
pip show ipython # pip show module_name
```


<a id="org4318f69"></a>

### 使用 sys 模块查看

```python
import sys

for path in sys.path:
    print(path)
```


<a id="org99cc604"></a>

### 使用 sys.modules 函数查看

```python
import sys

for k in sys.modules.items():
    print(k)
```
