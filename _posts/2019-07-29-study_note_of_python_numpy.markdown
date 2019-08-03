---
layout: post
title: "(Data analysis part) study note of python NumPy module"
date: 2019-7-29
tag: python NumPy 
---

本文是我在学习期间的笔记，看的书是《Python数据分析基础教程 NumPy学习指南 》第二版。

# Table of Contents

1.  [numypy](#org54ed8c4)
    1.  [向量加法](#org6dc0381)
    2.  [NumPy 基础](#org1d0ed4a)
        1.  [数组对象](#orgc66e265)
        2.  [指定类型](#org7536264)
        3.  [自定义数据类型](#orga46c54e)
        4.  [数组的切片和索引](#org224f7c9)
        5.  [改变数组的维度](#org7c96580)
        6.  [数组的组合](#orgee9d5e8)
        7.  [数组的分割](#orgf2e84da)
        8.  [数组的属性](#orgf99a4a6)
        9.  [数组的转换](#org4a54499)
    3.  [常用函数](#org15f4963)
        1.  [文件读写](#orga7c8c9b)
    4.  [便捷函数](#org8389a49)
    5.  [矩阵和通用函数](#org9cb6ac6)
        1.  [矩阵](#orga398132)
        2.  [从已有矩阵创建新矩阵](#org1a9ff53)
        3.  [通用函数](#orgc7a9a89)
        4.  [算术运算](#org16607d0)
        5.  [模运算](#org37da9fa)


<a id="orgc381657"></a>

# numypy

numpy数组在数值运算方面的效率优于python提供的list容器。


<a id="org4693e2a"></a>

## 向量加法

-   TypeError: 'range' object does not support item assignment
```python
def python_sum(n):
	a = list(range(n))
	b = list(range(n))
	c = []
	
	for i in range(len(a)):
	a[i] = pow(i,2)
	b[i] = pow(i,3)
	c.append(a[i] + b[i])

	return c

def numpy_sum(n):
	import numpy as np
	a = pow(np.arange(n), 2)
	b = pow(np.arange(n), 3)
	c = a + b
	
	return c


def test(size):

# size = int(sys.argv[1])
# size = 1000
	
	from datetime import datetime
	import sys 
	
	start = datetime.now()
	c = python_sum(size)
	delta = datetime.now() - start
	print("The last 2 elements of the sum: ", c[-2:])
	print("PythonSum elapsed time in microseconds: ", delta.microseconds)
	
	start = datetime.now()
	c = numpy_sum(size)
	delta = datetime.now() - start
	print("The last 2 elements of the sum: ", c[-2:])
	print("NumPySum elapsed time in microseconds: ", delta.microseconds)

test(100000)
```
两者的输出形式上有差异：numpysum 的输出不包含逗号，numpy的数组对象以专门的数据结构来存储数值。


<a id="orgcf5a345"></a>

## NumPy 基础


<a id="org3635a5c"></a>

### 数组对象

-   ndarray 是一个多维数组对象由：实际的数据，描述这些数据的元数据构成。

大部分操作仅仅修改元数据，而不改变底层的实际数据。

-   numpy 数组的下标也是从0开始的。
```python
from numpy import *

a = arange(50)
print(a.dtype)
print(a.shape) # 返回一个表示数组维度的tuple

# 创建多维数组

b = array([arange(5), arange(5)])
print(b,"\n",b.dtype,"维度： ",b.shape)
print("选取[1,2]: \n\t", b[1,2])
```

<a id="org46dbb47"></a>

### 指定类型
```python
arange(8,dtype = float64)
```

-   复数是不能转换为整数的。 否则会触发 TypeError
-   数据类型对象是numpy.dtype类的实例。
-   数据类型对象是可以给出单个数组元素在内存中占用的字节数，即dtype类的itemsize属性：
```python
array(6).dtype.itemsize
```

<a id="org5f03474"></a>

### 自定义数据类型


<a id="org8c21aee"></a>

### 数组的切片和索引


<a id="orgd9bff91"></a>

### 改变数组的维度

先定义一个数组
```python
a = arange(24).reshape(2,3,4)
a
```

-   ravel() 函数完成展平操作,只返回数组的视图。
```python
a.ravel()
```

-   flatten() 展平，会请求分配内存来保存结果。
```python
a.flatten()
```

-   两个函数都不改变原数组
```python
a
```

-   用元祖设置维度
```python
a.shape = (6,4)
a
```

-   transpose() 得到转置矩阵
```python
a.transpose()
```

-   resize() 和reshape()函数功能一样，但resize会直接修改原来的数组
```python
a.resize(2,12)
a 
```


<a id="org86585b5"></a>

### 数组的组合

-   先创建一个数组
```python
from numpy import *  
a = arange(9).reshape(3,3)
b = a * 2
print("a = \n",a,"\nb = \n",b)
```

-   hstack((a,b)) 水平组合
```python
hstack((a,b))
```

-   concatenate((a,b),axis = 1) 实现水平组合
```python
concatenate((a,b), axis=1)
```

-   vstack((a,b)) 垂直组合，同样需要构造一个元组作为参数
```python
vstack((a,b))
```

-   concatenate((a,b),axis=0) 实现垂直组合
```python
concatenate((a,b),axis=0)
```

-   dstack((a,b)) 深度组合，将一系列数组沿着纵轴方向进行层叠组合
```python
dstack((a,b))
```

-   column<sub>stack</sub>() 对于一位数组将安照列方向进行组合
-   == 运算符可以来比较两个 NumPy 数组
```python
column_stack((a, b)) == hstack((a, b))
```

-   row<sub>stack</sub>((a,b)) 按行方向进行组合的函数
```python
row_stack((a, b)) == vstack((a, b))
```


<a id="org667f980"></a>

### 数组的分割

1.  水平分割

-   hsplit(a,3)
```python
hsplit(a,3)
```

-   split(a,3,axis=1))
```python
split(a,3,axis=1)
```

2.  垂直分割

-   vsplit(a,3)
```python
vsplit(a,3)
```

-   split(a,3,axis=0)
```python
split(a,3,axis=0)
```

3.  深度分割

-   dsplit(a,3)
```python
c = arange(27).reshape(3,3,3)
print("c = \n", c)
print("\ndsplit = ",dsplit(c,3))
```

<a id="org3eda087"></a>

### 数组的属性

除了shape和dtype属性外，ndarray对象还有其他许多属性。

1.  ndim

-   给出数组的维度
```python
b.ndim
```
2.  size

-   给出数组元素的总个数
```python
b.size
```
3.  itemsize

-   给出数组中元素的被内存中所占的字节数
```python
b.itemsize
```
4.  nbytes

-   整个数组所占的存储空间
```python
b.nbytes
```
5.  T
```python
b.T
```
6.  real,imag
```python
c = array([1.j + 1, 2j + 3])
print("c.real = ",c.real, "\n,c.imag = ",c.imag)
```
7.  flat

-   返回一个Numpy.flatiter对像，
-   这是获得flatiter对象的唯一方式，
-   我们无法访问flaiter的构造函数。
-   遍历数组
```python
f = b.flat
for i in f:
print(i)
```
-   获取对象
-   flat属性是一个可赋值的属性，对flat属性赋值将导致整个数组的元素都被覆盖。
```python
b = arange(4).reshape(2, 2)
print("b = ",b)
print("b.flat[2] = ",b.flat[2])
print("b.flat[[1,3]] = ",b.flat[[1,3]])
b.flat = 7
print("b = ",b)
b.flat[[1,3]] = 1
print("b = ", b)
```

<a id="org151ab2e"></a>

### 数组的转换

1.  tolist()

-   转换成列表
```python    
c = array([ 1 + 1.j, 3. + 2.j])
c.tolist()
```
2.  astype()

-   转换数组时指定数据类型
-   在复数转换为整数的过程中，丢失了虚部，会报错
```python
print("c = ",c)
print("c.astype(int) = ", c.astype(int))
print("c.astype('complex') = ", c.astype('complex'))
```

<a id="org439e5e7"></a>

## 常用函数


<a id="org77b0e9f"></a>

### 文件读写

-   np.eye(n) 创建单位矩阵
-   np.savetxt() 将数据存储到文件中
```python
import os
import numpy as np

i2 = np.eye(2)
print('i2 = ',i2)
np.savetxt('eye2.txt',i2)
print(os.path.exists('eye2.txt'))
```
-   loadtxt() 能方便的读取CSV文件


<a id="org5958fea"></a>

## 便捷函数


<a id="org031867e"></a>

## 矩阵和通用函数

-   通用函数 universal functions 即 ufuncs
-   矩阵是ndarray的子类，可以由专门的字符串格式来创建。
-   二维的

-   mat, matrix, bmat 函数来创建矩阵


<a id="org018c165"></a>

### 矩阵

1.  创建矩阵

-   用字符串创建
```python
from numpy import *
A = mat('1 2 3; 4 5 6; 7 8 9')
print("Creating from string: ", A)
```
-   用numpy数组进行创建
```python
mat(arange(9).reshape(3,3))
```
2.  获取转置矩阵
```python
A.T
```
3.  获取逆矩阵
```python
A.I
```

<a id="org69478ad"></a>

### 从已有矩阵创建新矩阵

-   bmat 即分块矩阵 block matrix
```python
import numpy as np
A = np.eye(2)
print("A",A)
B = 2 * A
print("B", B)
```
-   使用字符串创建复合矩阵
```python
np.bmat("A B; A B")
```

<a id="orgd102294"></a>

### 通用函数

-   通用函数的输入是一组标量，输出也是一组标量。通常可以对应与基本数学运算。
-   zeros<sub>like</sub>(a) 创建一个和a形状相同的，并且元素个数全部为0的数组。
-   frompyfunc(func,1,1) 创建通用函数，指定输入参数的个数为1,输出参数的个数为1。
```python
def ultimate_answer(a):
	result = np.zeros_like(a)
	result.flat = 42

	return result

ufunc = np.frompyfunc(ultimate_answer, 1, 1)
print("The answer : ", ufunc(np.arange(4)))

print("The answer : ", ufunc(np.arange(4).reshape(2,2)))
```
1.  通用函数的方法

-   通用函数并非真的函数，而是能表示函数的对象
-   通用函数有四个方法，对应两个输入参数，输出一个参数的ufunc对象有效。
-   不符合条件时将报出ValueError异常。

1.  reduce

-   在连续的数组元素之间递归调用通用函数，即可得到输入数组的规约（reduce）计算结果
```python
a = np.arange(9)
print("Reduce",np.add.reduce(a))
```
2.  accumulate

-   递归作用于输入数组，但它存储中间结果并返回。    
```python
np.add.accumulate(a)
```
3.  reduceat

-   该函数需要一个数组和一个索引值列表作为参数
```python
print("Reduceat", np.add.reduceat(a, [0, 5, 2, 7]))
# 相当于
print("Reduceat step I", np.add.reduce(a[0:5]))
print("Reduceat step II", a[5])
print("Reduceat step III", np.add.reduce(a[2:7]))
print("Reduceat step IV", np.add.reduce(a[7:]))
```
4.  outer

-   该方法返回一个数组，它的秩（rank）等于两个输入数组的秩值之和。
```python
np.add.outer(np.arange(3), a)
```

<a id="orgafda283"></a>

### 算术运算

-   在numpy中，基本的算术运算符+，-和\*隐式关连这通用函数add,subtract,和multiply。
-   除法涉及到三个通用函数，divide, true<sub>divide,和</sub> floor<sub>divide</sub>，以及两个对应的运算符/和//。

1.  divide, true<sub>divide</sub>, floor<sub>divide</sub>

-   floor<sub>divide</sub>() 函数在整数和浮点数的除法中只保留整数部分。
```python
import numpy as np
a = np.array([2,3,4])
b = np.array([1,2,3])
print("Divide",np.divide(a,b),"\n",np.divide(b,a))
print("True Divide",np.true_divide(a,b),"\n",np.true_divide(b,a))
print("Floor Divide",np.floor_divide(a,b),"\n",np.floor_divide(b,a))

print("/ operator", a/b,"\n",b/a)
print("// operator", a//b, "\n", b//a)
```

<a id="org3e6d5a5"></a>

### 模运算

-   计算模数或者余数
-   可用的函数有 mod, remainder, fmod.
-   或者%运算符

1.  remainder

-   返回两个数组中元素相除后的余数，如果第二个数字为0, 则直接返回0。
```python
a = np.arange(-4,4)
print("a = ",a)
print("Remainder",np.remainder(a,2))
```
2.  mod

-   与 remainder 函数的功能完全一致
```python
np.mod(a,2)
```
3.  %

-   该操作符仅仅是 remainder 函数的缩写
```python
a % 2 
```
4.  fmod

-   所得的余数的正负由被除数决定，与除数的正负无关。
```python
np.fmod(a,2)
```
