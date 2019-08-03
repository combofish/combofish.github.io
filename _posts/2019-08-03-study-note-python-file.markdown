---
layout: post
title: "Python Basis (4) File. Study Note"
date: 2019-08-03
tag: Python File
---

本文是我在学习期间的笔记，看的书是《python语言及其应用》。      

- [文件输入输出](#orgab4d3c0)
  - [open()](#org53a9e3d)
    - [mode的第一一个字母](#org5d24719)
    - [mode的第二个字母](#orge04deb2)
    - [最后要关闭文件](#org7de945b)
  - [write()](#orgd30cf33)
  - [print()](#org6e95922)
  - [分段写入](#orgebb3b34)
  - [避免重写](#orgb2f97d7)
    - [处理异常](#orgb41f6b0)
  - [read()](#org3c3dd44)
    - [设置最大读入限制](#org31bfc28)
  - [readline()](#orge29d6c2)
  - [使用迭代器](#org357e1e9)
  - [readlines()](#org2768362)
  - [使用 write() 写二进制文件](#orgf4c594b)
    - [分块写二进制数据](#orgc066024)
  - [使用 read() 读二进制文件](#org90d922d)
  - [使用 with 自动关闭文件](#org06254ac)
  - [使用 seek() 改变函数位置](#org9c0ed96)
  - [seek(offset, origin)](#org061ea2d)
- [结构化的文本文件](#org391807a)
  - [csv](#org6522345)
  - [XML](#org9a0388d)
  - [HTML](#orgcb04dc6)
  - [JSON](#orgc1c53d7)
  - [yaml](#org4cd35c1)


<a id="orgab4d3c0"></a>

# 文件输入输出

-   数据持久化最简单的类型是普通文件，有时也叫平面文件(flat file)
-   它仅仅是在一个文件名下的字节流


<a id="org53a9e3d"></a>

## open()

fileob = open(filename, mode)


<a id="org5d24719"></a>

### mode的第一一个字母

-   r 读模式
-   w 写模式，如果文件不存在就创建，存在就重写新的内容
-   x 在文件不存在的情况下创建新文件并写文件
-   a 如果文件存在，就在文件末尾追加内容


<a id="orge04deb2"></a>

### mode的第二个字母

代表文件类型

-   t (或者省去)代表文本类型
-   b 代表二进制文件


<a id="org7de945b"></a>

### 最后要关闭文件


<a id="orgd30cf33"></a>

## write()

-   write() 函数返回写入文件的字节数

```python
poem = '''There was a young lady named Bright,
Whose speed was far faster than light;
She started one day
In a relative way,
And returned on the previous night.'''
print('len(poem)',len(poem))
fout = open('relativity','wt')
print(fout.write(poem))
fout.close()
```


<a id="org6e95922"></a>

## print()

-   也可以使用print() 函数

```python
fout = open('relativity','wt')
print(poem, file=fout)
fout.close()
```

-   print() 会在每个参数后添加空格，在每行结束后添加换行。
-   sep 分割符：默认是一个空格 ’ ‘
-   end 结束字符：默认是一个换行'\n'

```python
fout = open('relativity','wt')
print(poem, file=fout, sep='', end='')
fout.close()
```


<a id="orgebb3b34"></a>

## 分段写入

```python
fout = open('relativity','wt')
size = len(poem)
offset = 0
chunk = 100
while True:
    if offset > size:
        break

    print(fout.write(poem[offset:offset+chunk]))
    offset += chunk

fout.close()
```


<a id="orgb2f97d7"></a>

## 避免重写

```python
fout = open('relativity','xt')
fout.write('stomp stomp stomp')
fout.close()
```


<a id="orgb41f6b0"></a>

### 处理异常

```python
try:
    fout = open('relativity','xt')
    fout.write('stomp stomp stomp')
except FileExistsError:
    print('relativity already exists! That was a close one.')
```


<a id="org3c3dd44"></a>

## read()

-   不带参数的read() 一次读取文件的所有内容

```python
fin = open('relativity','rt')
poem = fin.read()
fin.close()
print(len(poem))
```


<a id="org31bfc28"></a>

### 设置最大读入限制

-   读到文件结尾，再调用 read() 函数会返回空字符串'' ,

```python
poem = ''
fin = open('relativity','rt')
chunk = 100
while True:
    fragment = fin.read(chunk)
    if not fragment:
        break

    poem += fragment
fin.close()
print(len(poem))
```


<a id="orge29d6c2"></a>

## readline()

-   对于文本文件，即使空行也有一个字符长度(换行字符'\n')

```python
poem = ''
fin = open('relativity','rt')
while True:
    line = fin.readline()
    if not line:
        break

    poem += line

fin.close()
print(len(poem))
```


<a id="org357e1e9"></a>

## 使用迭代器

-   iterator

```python
poem = ''
fin = open('relativity','rt')
for line in fin:
    poem += line

fin.close()
print(len(poem))
```


<a id="org2768362"></a>

## readlines()

-   读入所有的行，并返回单行字符串的列表
-   print() 默认在每行结束加上换行

```python
fin = open('relativity','rt')
lines = fin.readlines()
fin.close()
print(len(lines)," lines read")
for line in lines:
    print(line, end='')

print('newline')

for line in lines:
    print(line)
```


<a id="orgf4c594b"></a>

## 使用 write() 写二进制文件

```python
bdata = bytes(range(0,256))
print(len(bdata))

fout = open('bfile','wb')
print(fout.write(bdata))
fout.close()
```


<a id="orgc066024"></a>

### 分块写二进制数据

```python
fout = open('bfile','wb')
size = len(bdata)
offset = 0
chunk = 100
while True:
    if offset > size:
        break

    print(fout.write(bdata[offset:offset+chunk]))
    offset += chunk

fout.close()
```


<a id="org90d922d"></a>

## 使用 read() 读二进制文件

```python
fin = open('bfile','rb')
bdata = fin.read()
print(len(bdata))
fin.close()

```


<a id="org06254ac"></a>

## 使用 with 自动关闭文件

```python
with open('relativity','wt') as fout:
    print(fout.write(poem))

```


<a id="org9c0ed96"></a>

## 使用 seek() 改变函数位置

-   tell() 返回距离文件开始处的字节偏移量
-   seek() 允许跳转到文件其他字节偏移量的位置
-   seek() 同样返回当前的偏移量

```python
fin = open('bfile','rb')
print(fin.tell())
print(fin.seek(255))
bdata = fin.read()
print(len(bdata))
print(bdata[0])
```


<a id="org061ea2d"></a>

## seek(offset, origin)

-   origin = 0, 从开头偏移offset个字节
-   origin = 1, 从当前位置处偏移offset个字节
-   origin = 2, 距离最后结尾处偏移offset个字节

这些值也在os模块中定义

```python
import os
print('os.SEEK_SET :',os.SEEK_SET)
print('os.SEEK_CUR :',os.SEEK_CUR)
print('os.SEEK_END :',os.SEEK_END)
```

-   用不同的方法读取最后一个字节

```python
fin = open('bfile','rb')
print(fin.seek(-1, 2))
print(fin.tell())
bdata = fin.read()
print(len(bdata))
print(bdata[0])
```

-   从当前位置寻找

```python
fin = open('bfile','rb')
print(fin.seek(254,0))
print(fin.tell())
print(fin.seek(1,1))
print(fin.tell())
bdata = fin.read()
print(len(bdata))
print(bdata[0])
```

-   最流行的编码格式（例如UTF-8）每个字符的字节数都不仅相同


<a id="org391807a"></a>

# 结构化的文本文件


<a id="org6522345"></a>

## csv

```python
import csv

villains = [
    ['Docter','No'],
    ['Rosa','Klebb'],
    ['Mister','Big'],
    ['Auric','Goldfinger'],
    ['Ernst','Blfeld'],
]
with open('villains','wt') as fout:
    csvout = csv.writer(fout)
    csvout.writerows(villains)

```

-   test

```shell
cat villains 
```

| Docter | No         |
| Rosa   | Klebb      |
| Mister | Big        |
| Auric  | Goldfinger |
| Ernst  | Blfeld     |

-   重新读入

```python
import csv
with open('villains','rt') as fin:
    cin = csv.reader(fin)
    willains = [row for row in cin]

print(willains)
```

-   读入的数据也可以是字典集合

```python
import csv
with open('villains','rt') as fin:
    cin = csv.DictReader(fin,fieldnames=['first','last'])
    villains = [row for row in cin]

print(villains)    

```

-   用 DictWriter() 重写 CSV 文件

```python
import csv
villains = [
    {'first': 'Doctor','last':'No'},
    {'first':'Rosa','last':'Klebb'},
    {'first':'Mister','last':'Big'},
]
with open('villains','wt') as fout:
    cout = csv.DictWriter(fout,['first','last'])
    cout.writeheader()
    cout.writerows(villains)
```

```shell
cat villains 
```

-   再来读取文件

```python
import csv
with open('villains','rt') as fin:
    cin = csv.DictReader(fin)
    villains = [row for row in cin]

print(villains)    
```


<a id="org9a0388d"></a>

## XML


<a id="orgcb04dc6"></a>

## HTML


<a id="orgc1c53d7"></a>

## JSON


<a id="org4cd35c1"></a>

## yaml

待续。。。