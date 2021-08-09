---
layout: post
title: "Python script to Windows executable program --- Installation and Use of Cxfreeze"
date: "2021-08-09 11：27：00"
tag: cr_freeze python exe msi pip
---
- [Python脚本到Windows可执行程序——Cxfreeze的安装与使用](#org3e910b1)
  - [下载安装](#org509726e)
  - [打包程序](#org05e611b)
    - [通过 命令行 简单使用](#org5461202)
    - [使用 setup.py 详细配置](#orgcd884c9)


<a id="org3e910b1"></a>

# Python脚本到Windows可执行程序——Cxfreeze的安装与使用

写好的 Python 程序，分发到 Windows 用户的时候，如果再在每一台电脑上配置执行 Python 程序的解释器和相应的依赖库，就会比较繁琐。 所以可以将 Python 程序打包程可执行的 .exe 文件。可以用的库有：cx_freeze，py2exe，PyInstaller。在此介绍 cx_freeze。


<a id="org509726e"></a>

## 下载安装

```sh
pip install cx_freeze
cxfreeze -h
```


<a id="org05e611b"></a>

## 打包程序


<a id="org5461202"></a>

### 通过 命令行 简单使用

```sh
cxfreeze main.py [可选参数如下]
# --target-dir 是打包后的程序路径
# --target-name 是打包后的程序名
# --base-name=win32gui 程序用到图形化界面后，配置可使程序打开时隐藏黑窗口
# --include-modules 是要包含的模块或库
# --icon 是打包后的程序图标。（我使用的时候，这一项不起作用，还不知道为什么）
```

如果常用，可以工程目录下建一个 build.bat 文件，存放上述命令。


<a id="orgcd884c9"></a>

### 使用 setup.py 详细配置

-   编写 setup.py 文件

```python
from cx_Freeze import setup, Executable

build_exe_options = {
    'packages': [], # 默认可不填，程序会自动寻找依赖，如果运行时，提示有缺少的包，可以在这里添加
    'excludes': [],
    "include_files": ["config.ini"]  # 可以添加程序用到的其他文件
}

setup(
    name="",
    version="1.0",
    description="",
    author="Combofish",
    options={"build_exe":build_exe_options},
    executables=[Executable(script="main.py",base="win32gui",icon="XX.ico")])
```

-   需要打包的时候，进入到相应的目录，执行命令：

```shell
python setup.py build
```

-   可选的打包方式：生成 .msi 格式的 windows 安装包

```shell
python setup.py bdist_msi
```

-   两种方法的区别
    
    -   build 会在当前目录下生成目录，存放可执行的文件以及依赖，目录结构如下：
    
    ```sh
    lib\
    python3.dll
    python38.dll
    main.exe
    ```
    
    -   bdist_msi, 想当于把这些压缩打包程一个文件，并且可以安装。分发时单个文件会比较方便。
