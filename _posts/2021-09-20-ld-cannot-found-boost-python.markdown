---
layout: post
title: "/usr/bin/ld: cannot found -lboost_python"
date: "2021-09-20 23：09：00"
tag: python g++ boost-numpy boost-python python.hpp numpy.hpp
---

- [/usr/bin/ld: cannot found -lboost_python](#orge7a48d8)
  - [错误情况](#orgd171cde)
  - [解决方法](#org51a2eec)
- [其他错误](#orgcf0c7a3)
    - [fatal error: pyconfig.h: 没有那个文件或目录](#orgd7b51f9)
    - [如果提示找不到 "numpy.hpp", "python.hpp"](#org6ef9bcd)


<a id="orge7a48d8"></a>

# /usr/bin/ld: cannot found -lboost_python


<a id="orgd171cde"></a>

## 错误情况

编译CPP工程时，提示找不到 boost_python 和 boost_numpy。

```shell
$ make              
[ 50%] Linking CXX shared library segment.so
/usr/bin/ld: cannot -lboost_python
/usr/bin/ld: cannot found -lboost_numpy
collect2: error: ld returned 1 exit status
make[2]: *** [CMakeFiles/segment.dir/build.make:84：segment.so] 错误 1
make[1]: *** [CMakeFiles/Makefile2:76：CMakeFiles/segment.dir/all] 错误 2
make: *** [Makefile:84：all] 错误 2
```

-   查看 CMakeLists.txt , 发现依赖到了这两个链接库

```shell
set(BOOST_LIBS "-lboost_system -lboost_python -lboost_numpy") 
```

-   搜索相关安转包, 发现已经安装。

```shell
sudo apt-cache search libboost-python
sudo apt-cache search libboost-numpy 
sudo apt-get install libboost-numpy-dev libboost-python-dev   

# RESULT
libboost-numpy-dev 已经是最新版 (1.71.0.0ubuntu2)。
libboost-python-dev 已经是最新版 (1.71.0.0ubuntu2)。
```

-   去ubuntu 官网查看该软件包 libboost-python。

具体版本是：libboost-python1.71.0_1.71.0-6ubuntu6_arm64.deb，发现该软件包包含如下软件：

```shell
/usr/lib/aarch64-linux-gnu/libboost_python38.so.1.71.0
/usr/share/doc/libboost-python1.71.0/changelog.Debian.gz
/usr/share/doc/libboost-python1.71.0/copyright
/usr/share/lintian/overrides/libboost-python1.71.0
```

-   猜测： 动态链接库的名字不是 boost_python, 而是 boost_python38。更改 CMakeLists.txt。

```shell
set(BOOST_LIBS "-lboost_system -lboost_python38 -lboost_numpy38") 
```

-   重新编译：

```shell
$ make      
[ 50%] Building CXX object CMakeFiles/segment.dir/segment_py.cpp.o
[100%] Linking CXX shared library segment.so
[100%] Built target segment
```


<a id="org51a2eec"></a>

## 解决方法

-   确保系统安转相关的软件包，libboost-python 和 libboost-numpy。

```shell
sudo apt-get install libboost-numpy-dev libboost-python-dev
```

-   确保正确引用动态链接库的名字。

```shell
set(BOOST_LIBS "-lboost_system -lboost_python38 -lboost_numpy38") 
```

-   查看动态链接库是否到环境变量。

```shell
$ ldconfig -p | grep boost_python            
libboost_python38.so.1.71.0 (libc6,x86-64) => /lib/x86_64-linux-gnu/libboost_python38.so.1.71.0
libboost_python38.so.1.67.0 (libc6,x86-64) => /lib/x86_64-linux-gnu/libboost_python38.so.1.67.0
libboost_python38.so (libc6,x86-64) => /lib/x86_64-linux-gnu/libboost_python38.so
libboost_python27.so.1.67.0 (libc6,x86-64) => /lib/x86_64-linux-gnu/libboost_python27.so.1.67.0
$ ldconfig -p | grep boost_numpy 
libboost_numpy38.so.1.71.0 (libc6,x86-64) => /lib/x86_64-linux-gnu/libboost_numpy38.so.1.71.0
libboost_numpy38.so (libc6,x86-64) => /lib/x86_64-linux-gnu/libboost_numpy38.so
```


<a id="orgcf0c7a3"></a>

# 其他错误


<a id="orgd7b51f9"></a>

### fatal error: pyconfig.h: 没有那个文件或目录

-   搜索 pyconfig.h 的位置。

```shell
$ sudo find / -name "pyconfig.h"
/home/larry/anaconda3/pkgs/python-3.8.11-h12debd9_0_cpython/include/python3.8/pyconfig.h
/home/larry/anaconda3/include/python3.7m/pyconfig.h
/usr/include/python3.8/pyconfig.h
/usr/include/x86_64-linux-gnu/python3.8/pyconfig.h
```

-   选一个所需的环境路径，添加到环境变量即可。

```shell
export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/include/python3.8
```


<a id="org6ef9bcd"></a>

### 如果提示找不到 "numpy.hpp", "python.hpp"

-   编译结果

```shell
$ make                          
Scanning dependencies of target segment
[ 50%] Building CXX object CMakeFiles/segment.dir/segment_py.cpp.o
/home/larry/GitProjects/selective_search_py/segment_py.cpp:4:10: fatal error: boost/numpy.hpp: 没有那个文件或目录
    4 | #include <boost/numpy.hpp>
      |          ^~~~~~~~~~~~~~~~~
compilation terminated.
make[2]: *** [CMakeFiles/segment.dir/build.make:63：CMakeFiles/segment.dir/segment_py.cpp.o] 错误 1
make[1]: *** [CMakeFiles/Makefile2:76：CMakeFiles/segment.dir/all] 错误 2
make: *** [Makefile:84：all] 错误 2
```

-   fatal error: boost/numpy.hpp: 没有那个文件或目录
-   查看C语言源代码

```c
#include <boost/python.hpp>
#include <boost/numpy.hpp>
```

-   搜索一下 `numpy.hpp` 与 `python.hpp`，发现这两个文件都在 */usr/include/boost/* 目录下，所以编译时，如果找不到头文件可以在 g++ 编译器后添加参数 **-I/usr/include/boost/python/** 。

`numpy.hpp` 在 */usr/include/boost/python* 下，所以使用 `numpy.hpp` 的头文件包含应该是  **include <boost/python/numpy.hpp>** 。


```shell
$ sudo find / -name "numpy.hpp"     
/usr/include/vtk-7.1/vtkdiy/io/numpy.hpp
/usr/include/boost/python/numpy.hpp
$ sudo find / -name "python.hpp"
/usr/include/boost/mpi/python.hpp
/usr/include/boost/python.hpp
/usr/include/boost/parameter/python.hpp
```

-   修改源文件

```c
#include <boost/python.hpp>
#include <boost/python/numpy.hpp>
```
