---
layout: post
title: "Python Basis (1), Study Note"
date: 2019-08-03
tag: Python 
---

本文是我在学习期间的笔记，看的书是《python语言及其应用》。     

- [learn python](#org1b6d34b)
  - [查看对象的属性和方法](#org59c7e73)
  - [TypeError](#org7fb0367)
  - [查看python已经安装了哪些包](#org146bbef)
  - [python的除法和幂运算](#org0516b40)
  - [python字符数字之间的转换函数](#org20b5043)
  - [python数值运算函数](#org7f63bb4)
  - [包和模块](#org418d7ac)
  - [pyc文件](#org42b9baf)
  - [查看帮助文档](#org9221697)
  - [让代码运行的更快](#orgcf1cc89)
  - [标准库](#org58c6de1)
  - [列表和元组](#org078c5e1)
    - [二分搜索](#org625b5af)
    - [用bisect模块在列表中查询最接近的目标的值](#orgb7f050a)
    - [动态数组](#orgc9c7832)
    - [静态数组](#orga6e62b9)
  - [字典和集合](#org18706df)
    - [如何工作的](#org6caf310)



<a id="org1b6d34b"></a>

# learn python


<a id="org59c7e73"></a>

## 查看对象的属性和方法

1.  dir

    -   dir(object)
    
    ```python
    a = [1 ,2]
    print(dir(a))
    print(dir(a)[-12:])
    ```

2.  动态获取属性 Type.FTE 的值

    -   getattr(Type,'FTE')
    -   Type.\_<sub>dict</sub>\_\_['FTE']
    
    -   配合inspect 使用
    
    ```python
    import inspect
    print([i for i in dir(list) if inspect.isbuiltin(getattr(list, i))])
    print([i for i in dir(inspect) if inspect.isfunction(getattr(inspect,i))])
    print([i for i in dir(inspect) if not callable(getattr(inspect, i))])
    print(list.__dict__.keys())
    ```


<a id="org7fb0367"></a>

## TypeError

-   创建整数列表（导致“TypeError: ‘range’ object does not support item assignment”）有时你想要得到一个有序的整数列表， 所以range() 看上去是生成此列表的不错方式。然而，range() 返回的是“range object”，而不是实际的list 值。
    
    ```python
    a = range(5)
    # a = [ i ** 2 for i in a ]
    for i in a:
        a[i] = a[i] - 5
    print(a)
    ```
    
    Traceback (most recent call last): File "<stdin>", line 4, in <module> TypeError: 'range' object does not support item assignment
    
    -   解决方法
    
    a = range(0,N)改为a = list(range(0,N))


<a id="org146bbef"></a>

## 查看python已经安装了哪些包

```sh
pip list 
```


<a id="org0516b40"></a>

## python的除法和幂运算

-   使用“/”运算符时，只要有一个操作数是浮点数，那么产生的结果就是浮点数结果
-   "//"的结果都是整除，如果操作数是浮点数的话，返回一个整除结果转换成浮点数
-   // x与y之整数商，即：不大于 x 与 y 之商的最大整数
    
    ```python
    print(20/2)
    print(20/3)
    print(20.0/3)
    print(20/3.0)
    print(20.0/3.0)
    
    print(20//3)
    print(20.0 // 3)
    print(20 // 3.0)
    print(20.0 // 3.0)
    
    print(2 * 10)
    print(2 ** 10)
    print(pow(2,10))
    print(pow(3.14,10))
    ```


<a id="org20b5043"></a>

## python字符数字之间的转换函数

python中的字符数字之间的转换函数 int(x [,base ]) 将x转换为一个整数 long(x [,base ]) 将x转换为一个长整数 float(x ) 将x转换到一个浮点数 complex(real [,imag ]) 创建一个复数 str(x ) 将对象 x 转换为字符串 repr(x ) 将对象 x 转换为表达式字符串 eval(str ) 用来计算在字符串中的有效Python表达式,并返回一个对象 tuple(s ) 将序列 s 转换为一个元组 list(s ) 将序列 s 转换为一个列表 chr(x ) 将一个整数转换为一个字符 unichr(x ) 将一个整数转换为Unicode字符 ord(x ) 将一个字符转换为它的整数值 hex(x ) 将一个整数转换为一个十六进制字符串 oct(x ) 将一个整数转换为一个八进制字符串

chr(65)='A' ord('A')=65

int('2')=2; str(2)='2'

1.  原文链接

    <https://www.cnblogs.com/wuxiangli/p/6046800.html>

1.  数字转换为字符

    ```python
    print(type("%d" % 12))
    ```
    
    1.  原文连接
    
        <https://www.cnblogs.com/zmlctt/p/4234257.html>


<a id="org7f63bb4"></a>

## python数值运算函数

1.  abs(x)

    -   返回绝对值
    
    ```python
    print(abs(-128))
    ```

2.  divmod(a,b)

    -   （x//y, x%y), 输出为二元组形式
    -   "//" x与y之整数商，即：不大于 x 与 y 之商的最大整数
    
    ```python
    divmod(42,5)
    ```

3.  pow(x,y) pow(x,y,z)

    -   x **\* y 或 x \*** y % z 幂运算
    
    ```python
    print(pow(2,10))
    print(pow(2,10,100))
    ```

4.  round(x) round(x,d)

    -   对 x 四舍五入, 保留d位小数, 无参数 d 则返回四舍五入的整数值
    
    ```python
    import math 
    
    print(math.pi)
    print(round(math.pi))
    print(round(math.pi,3))
    ```

5.  max(x1,x2,&#x2026;xn) min(x1,x2,&#x2026;xn)

    -   max: x1, x2, x3, &#x2026;, xn 的最大值, n没有限制
    -   min: x1, x2, x3, &#x2026;, xn 的最小值, n没有限制
    
    ```python
    print(max(1, 2, 3, 4.55, 745, 23987))
    print(min(1, 2, 3, 4.55, 745, 23987))
    ```


<a id="org418d7ac"></a>

## 包和模块

1.  模块属性

    -   模块属性\_<sub>name</sub>\_\_，它的值由Python解释器设定。
    -   如果脚本文件是作为主程序调用，其值就设为\_<sub>main</sub>\_\_，
    -   如果是作为模块被其他文件导入，它的值就是其文件名。
        
        ```python
        print(__name__)
        
        import numpy
        print(numpy.__name__)
        ```
    
    每个模块都有自己的私有符号表，所有定义在模块里面的函数把它当做全局符号表使用。
    
    模块可以导入其他的模块。通常将import语句放在模块的开头，被导入的模块名字放在导入它的模块的符号表中。
    
    -   from module import names
    
    可以直接从模块中导入名字到符号表，但模块名字不会被导入。
    
    -   from module import \*
    
    可以把模块中的所有名字全部导入，除了那些以下划线开头的名字符号。不建议使用，不清楚导入了什么符号，有可能覆盖自己定义的东西
    
    -   内建函数dir()可以查看模块定义了什么名字（包括变量名，模块名，函数名等）
    -   dir(模块名)，没有参数时返回所有当前定义的名字

2.  模块搜索路径

    -   当导入一个模块时，解释器先在当前包中查找模块，若找不到，
    -   在内置的built-in模块中查找，
    -   找不到则按sys.path给定的路径找对应的模块文件(模块名.py)
    
    sys.path的初始值来自于以下地方：
    
    -   包含脚本当前的路径，当前路径
    -   PYTHONPATH
    -   默认安装路径
    
    sys.path初始化完成之后可以更改
    
    ```python
    import sys
    
    for i in sys.path:
        print(i)
    ```


<a id="org42b9baf"></a>

## pyc文件

1.  什么是pyc文件呢？

    pyc文件就是Python的字节码文件，Python是一种全平台的解释性语言， 全平台其实就是Python文件在经过解释器解释之后（或者称为编译）生成的pyc文件可以在多个平台下运行，这样同样也可以隐藏源代码。 其实，Python是完全面向对象的语言，Python文件在经过解释器解释后生成字节码对象PyCodeObject，pyc文件可以理解为是PyCodeObject对象的持久化保存方式。

2.  什么时候会生成pyc文件呢？

       pyc文件只有在文件被当成模块导入时才会生成。 也就是说，Python解释器认为，只有import进行的模块才需要被重用。 生成pyc文件的好处显而易见，当我们多次运行程序时，不需要重新对该模块进行重新的解释。 主文件一般只需要加载一次不会被其他模块导入，所以一般主文件不会生成pyc文件。

3.  pyc文件的过期时间

    在生成pyc文件的同时，写入了一个Long型的变量，用于记录最近修改的时间； 每次载入之前都先检查一下py文件和pyc文件的最后修改日期，如果不一致则会生成一个新的pyc文件

4.  原文连接

    <https://www.jianshu.com/p/40a42bf1d15d>


<a id="org9221697"></a>

## 查看帮助文档

1.  pydoc

2.  help()

    -   在repl下键入help()


<a id="orgcf1cc89"></a>

## 让代码运行的更快

-   让代码运行的更快最简单的方法是让他做更少的工作。

1.  Cython

    -   会额外要求用一种Python 和C 的混合来编写
    -   编译成C最通用的工具

2.  Shed Skin

    -   一个用于非numpy代码的，自动把Python转换为C的转换器

3.  Pythran

    -   一个用于numpy和非numpy代码的新编译器

4.  Numba

    -   基于LLVM的编译方式
    -   一个专用于numpy代码的新编译器

5.  PyPy

    -   替代虚拟机，包含了内置的即使编译器(JIT)
    -   用于非numpy代码，取代常规Python可执行程序稳定的即使编译器


<a id="org58c6de1"></a>

## 标准库

Python有一套很有用的标准库(standard library)。 标准库会随着Python解释器，一起安装在你的电脑中的。 它是Python的一个组成部分。这些标准库是Python为你准备好的利器，可以让编程事半功倍。

1.  文字处理

    -   Python的string类提供了对字符串进行处理的方法。
    -   通过标准库中的re包，Python可以用正则表达式(regular expression)来处理字符串。
    -   Python标准库还为字符串的输出提供更加丰富的格式， 比如： string包，textwrap包。

2.  数据对象

    -   不同的数据对象，适用于不同场合的对数据的组织和管理。
    -   Python的标准库定义了表和词典之外的数据对象，比如说数组(array)，队列(Queue)。
    -   copy包，以复制对象。

3.  管理时间和日期

    -   datetime和time 标准库

4.  Python运行控制

    -   sys包被用于管理Python自身的运行环境。Python是一个解释器(interpreter), 也是一个运行在操作系统上的程序。
    -   用sys包来控制这一程序运行的许多参数，比如说Python运行所能占据的内存和CPU， Python所要扫描的路径等。
    -   另一个重要功能是和Python自己的命令行互动，从命令行读取命令和参数。

5.  操作系统

    -   os包是Python与操作系统的接口。
    -   我们可以用os包来实现操作系统的许多功能，比如管理系统进程，改变当前路径(相当于’cd‘)，改变文件权限等，建立。
    -   我们通过文件系统来管理磁盘上储存的文件。查找、删除，复制文件，以及列出文件列表等都是常见的文件操作。 这些功能经常可以在操作系统中看到（比如ls, mv, cp等Linux命令），
    -   通过Python标准库中的glob包、shutil包、os.path包、以及os包的一些函数等，在Python内部实现。
    -   subprocess包被用于执行外部命令，其功能相当于我们在操作系统的命令行中输入命令以执行， 比如常见的系统命令'ls'或者'cd'，还可以是任意可以在命令行中执行的程序。

6.  线程与进程

    -   Python支持多线程(threading包)运行和多进程(multiprocessing包)运行。
    -   通过多线程和多进程，可以提高系统资源的利用率，提高计算机的处理速度。
    -   Python在这些包中，附带有相关的通信和内存管理工具。此外，
    -   Python还支持类似于UNIX的signal系统，以实现进程之间的粗糙的信号通信。

7.  基于socket层的网络应用

    -   socket是网络可编程部分的底层。通过socket包，我们可以直接管理socket，
    
    比如说将socket赋予给某个端口(port)，连接远程端口，以及通过连接传输数据。 也可以利用SocketServer包更方便地建立服务器。
    
    -   通过与多线程和多进程配合，建立多线程或者多进程的服务器，可以有效提高服务器的工作能力。
    
    +通过asyncore包实现异步处理，也是改善服务器性能的一个方案。

8.  互联网应用

    -   网络的很多底层细节（比如socket）都是被高层的协议隐藏起来的。
    -   建立在socket之上的http协议实际上更容易也更经常被使用。http通过request/responce的模式建立连接并进行通信，其信息内容也更容易理解。
    -   Python标准库中有http的服务器端和客户端的应用支持(BaseHTTPServer包; urllib包, urllib2包), 并且可以通过urlparse包对URL
    
    （URL实际上说明了网络资源所在的位置）进行理解和操作。

9.  原文链接

    <https://www.cnblogs.com/vamei/archive/2012/07/18/2597212.html#2927936>


<a id="org078c5e1"></a>

## 列表和元组

-   列表是动态的数组
-   元组是静态的数组
-   元组缓存于python运行时环境，每次使用元组时无须访问内核去分配内存

-   元组用于描述一个不会改变的事物的多个属性，而列表可被用于保存多个互相独立对性的数据集合
-   列表和元组都支持混合类型

-   通用代码会比某个特定问题设计的代码慢很多

高效搜索所必须的是\*排序算法\*和\*搜索算法\*


<a id="org625b5af"></a>

### 二分搜索

```python
def binary_search(needle, haystack):
    imin, imax = 0, len(haystack)

    while(True):
        if imin >= imax:
            return -1

        midpoint = (imax + imin) // 2
        if haystack[midpoint] > needle:
            imax = midpoint
        elif haystack[midpoint] < needle:
            imin = midpoint + 1
        else:
            return midpoint

a = [2, 4, 34, 563, 23, 12]
a.sort()
num = 23
ind = binary_search(num, a)
print("a = ",a)
print("Search : ", num," in a[", ind,"]")
```


<a id="orgb7f050a"></a>

### 用bisect模块在列表中查询最接近的目标的值

-   bisect.insort() 在保持排序的同时往列表中添加元素

```python
import bisect
import random

def find_closest(haystack, needle):
    i = bisect.bisect_left(haystack, needle)
    if i == len(haystack):
        return i - 1
    elif haystack[i] == needle:
        return i
    elif i > 0:
        j = i - 1

        if haystack[i] - needle > needle - haystack[j]:
            return j

    return i

important_numbers = []
for i in range(10):
    new_number = random.randint(0, 1000)
    bisect.insort(important_numbers, new_number)

print("important_numbers : ", important_numbers)

closest_index = find_closest(important_numbers, -250)
print("Closest value to -250: ", important_numbers[closest_index]," index : ",closest_index)

closest_index = find_closest(important_numbers, 500)
print("Closest value to 500: ", important_numbers[closest_index]," index : ",closest_index)

closest_index = find_closest(important_numbers, 1000)
print("Closest value to 1000: ", important_numbers[closest_index]," index : ",closest_index)

```


<a id="orgc9c7832"></a>

### 动态数组

-   可以根据需要随时改变其中的内容
-   这是因为动态数组支持resize操作
-   超额分配的大小来确保数据集仍然可以被保存在内存里

```python
num = [5, 8, 2, 1, 0, 19]
num[3] = 8 * num[0]
print("numbers = ", num)
print("len(numbers) = ",len(num))
num.append(42)
print("numbers = ", num)
print("len(numbers) = ",len(num))
```


<a id="orga6e62b9"></a>

### 静态数组

-   元组固定且不可变
-   内容无法被修改，大小也无法被改变
-   避免和操作系统大打交道

```python
t = (1, 5, 6, 9)
t[0] = 9
```

-   但两个元组可以合并成一个元组
-   返回一个新分配的元组

```python
t1 = (1, 2, 4, 6)
t2 = (6, 7, 8, 3)
print("t1 + t2 = ", t1 + t2)
```

-   初始化列表和元组的时间对比

```python
import time
t1 = time.time()
l = [0,1,2,3,4,5,6,7,8,9]
t2 = time.time()
t = (0,1,2,3,4,5,6,7,8,9)
t3 = time.time()
print("list cost : ",t2 - t1," tuple cost : ",t3 - t2)
print("list : ",l," tuple : ",t)
print("times : ", (t2 - t1) / (t3 - t2))
```

-   初始化列表比初始化元组慢近5倍


<a id="org18706df"></a>

## 字典和集合

-   字典和集合十分依赖他们的散列函数
-   适合用于存储能够被键索引的程序

-   列表查询电话薄

```python
def find_phonenumber(phonebook, name):
    for n,p in phonebook:
        if n == name:
            return p

    return None

phonebook = {("John","555-555-2222"),("Albert Einstein", "212-555-5555")}
print("John's phone number is ",find_phonenumber(phonebook,"John"))

phonebook2 = {"John":"555-555-2222","Albert Einstein":"212-555-5555"}
print("John's phone number is ",phonebook2["John"])
```

-   用列表和集合查询不同的名字

```python
def list_unique_names(phonebook):
    unique_names = []
    for name, phonenumber in phonebook:
        first_name, last_name = name.split(' ', 1)
        for unique in unique_names:
            if unique == first_name:
                break
        unique_names.append(first_name)

    print("list: unique_names : ",unique_names)
    return len(unique_names)

def set_unique_names(phonebook):
    unique_names = set()
    for name, phonenumber in phonebook:
        first_name, last_name = name.split(' ',1)
        unique_names.add(first_name)

    return len(unique_names)

phonebook = [
    ("Jhon Doe","555-555-5555"),
    ("Albert Einstein", "212-555-5555"),
    ("Jhon Murphey","202-555-5555"),
    ("Elaine Bodian","301-555-5555")]

print("Number of unique names from set method: ", set_unique_names(phonebook))
print("Number of unique names from list method: ", list_unique_names(phonebook))
```


<a id="org6caf310"></a>

### 如何工作的

-   字典和集合使用散列表来获得O(1)的查询和插入
-   通过将一个数据的键转化成某种可以被用作索引的东西，我们就得到了跟列表一样的性能
-   hash() 返回的是整型

```python
def index_sequence(key, mask=0b111, PERTURB_SHIFT = 5):
    perturb = hash(key)
    i = perturb & mask
    yield i
    while True:
        i = ((i << 2) + i + perturb + 1)
        perturb >>= PERTURB_SHIFT
        yield i & mask
```

-   自定义散列函数

```python
class City(str):
    def __hash__(self):
        return ord(self[0])

data = {
    City("Rome"): 4,
    City("San Francisco"): 3,
    City("New York"): 5,
    City("Barcelona"): 2
}    
```

1.  删除

2.  改变大小

    -   字典或集合默认的最小长度是8
    -   改变大小仅发生在插入时

3.  散列函数和熵

    -   最优双字母列表函数
    
    ```python
    def twoletter_hash(key):
        offset = ord('a')
        k1, k2 = key
        return (ord(k2) - offset) + 26 * (ord(k1) - offset)
    ```
    
    -   好坏散列函数的时间区别
    
    ```python
    import string
    import timeit
    
    class BadHash(str):
        def __hash__(self):
            return 42
    
    class GoodHash(str):
        def __hash__(self):
            return ord(self[1]) + 26 * ord(self[0]) - 2619
    
    baddict = set()
    gooddict = set()
    for i in string.ascii_lowercase:
        for j in string.ascii_lowercase:
            key = i + j
            baddict.add(BadHash(key))
            gooddict.add(GoodHash(key))
    
    print("baddict : ", len(baddict))
    print("gooddict : ", len(gooddict))
    
    # badtime = timeit.repeat(
    #     "key in baddict",
    #     setup = "form __main__ import baddict, BadHash; key = BadHash('zz')",
    #     repeat = 3,
    #     number = 1000000,
    # )
    
    # goodtime = timeit.repeat(
    #     "key in gooddict",
    #     setup = "form __main__ import gooddict, GoodHash; key = GoodHash('zz')",
    #     repeat = 3,
    #     number = 1000000,
    # )
    
    # print("Min look up for baddict: ", min(badtime))
    # print("Min look up for gooddict: ", min(goodtime))
    ```

4.  字典和命名空间

    -   在不必要的时候使用字典查询会让代码变慢
    -   python在命名空间管理上过渡的使用字典来进行查询
    
    当python访问一个变量或者函数或模块时，都有一个体系来决定它去那里查找这些对象
    
    -   首先查找locals()数组，保存了所有本地变量的条目
    -   如果不再本地变量里，就会搜索globals()字典。
    -   再搜索\_<sub>builtin</sub>\_<sub>对象</sub>，是模块对象。
    
    1.  命名空间查询
    
        ```python
        import dis
        import math
        from math import sin
        
        def test1(x):
            return math.sin(x)
        
        def test2(x):
            return sin(x)
        
        def test3(x, sin = math.sin):
            return sin(x)
        
        print("dis.dis(test1) : ")
        dis.dis(test1)
        print("dis.dis(test2) : ")
        dis.dis(test2)
        print("dis.dis(test3) : ")
        dis.dis(test3)
        
        ```
    
    2.  循环内命名空间查询的降速效果
    
        ```python
        import time
        from math import sin
        
        def tight_loop_slow(iterations):
            result = 0
            for i in range(iterations):
                result += sin(i)
        
            return result
        
        def tight_loop_fast(iterations):
            result = 0
            local_sin = sin
            for i in range(iterations):
                result += local_sin(i)
        
            return result
        
        
        l = 10000000 # 千万
        t1 = time.time()
        res1 = tight_loop_fast(l)
        t2 = time.time()
        res2 = tight_loop_slow(l)
        t3 = time.time()
        
        fast_time = t2 - t1
        low_time = t3 - t2
        print("result : ", res1, "tight_loop_fast: ",t2 - t1)
        print("result : ", res2, "tight_loop_slow: ",t3 - t2)
        print("times : slow / fast : ", (t3 - t2) / ( t2 - t1))
        print("fast : ",round((low_time - fast_time) / low_time * 100,2), "%")
        ```
        
        -   可以看出本地化函数能获的7.36%的速度提升


