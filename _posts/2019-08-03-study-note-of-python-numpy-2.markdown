---
layout: post
title: "(Data analysis part) Study note of python NumPy module (2)"
date: 2019-08-03
tag: Python NumPy 
---

本文是我在学习期间的笔记，看的书是《Python数据分析基础教程 NumPy学习指南 》第二版。

- [读入csv文件](#org9a2478a)
  - [平均值](#orga430ce7)
    - [vwap 成交量加权平均成绩](#org03a17de)
    - [np.mean() 计算算术平均值](#orgcca2343)
    - [时间加权平均价格](#org50878b7)
  - [取值范围](#orgafbd1f9)
  - [统计分析](#org1ba3b26)
    - [获取收盘价](#org4e38d9a)
    - [np.median() 获取中位数](#orgea77500)
    - [np.msort() 对数组排序](#orgaf5ce9e)
    - [np.var(c) 方差](#orga6fe622)
- [专用函数](#orga087df2)
  - [对复数进行排序](#org788205e)
  - [搜索](#orgf877faa)
  - [数组元素抽取](#org21f3384)
  - [金融函数](#orgc062fd7)


<a id="org9a2478a"></a>

# 读入csv文件

-   cvs Comma-Separated Value,逗号分割符
-   unpack= True 分拆存储不同的数据


<a id="orga430ce7"></a>

## 平均值


<a id="org03a17de"></a>

### vwap 成交量加权平均成绩

```python
import numpy as np

def str2num(s):
    return float(s[1:-1])

def str2num_m(st):
    return float(st[1:-2])

c, v = np.loadtxt('data.csv',delimiter=',', usecols=(1,5),converters={1:str2num,5:str2num_m},unpack=True)
vwap = np.average(c, weights = v)
print("VWAP = ",vwap)
```


<a id="orgcca2343"></a>

### np.mean() 计算算术平均值

```python
print("收盘价的平均值为： ",np.mean(c))
print("成交量的平均值是： ",np.mean(v))
```


<a id="org50878b7"></a>

### 时间加权平均价格

```python
t = np.arange(len(c))
print("TWAP = ", np.average(c, weights=t))
```


<a id="orgafbd1f9"></a>

## 取值范围

```python
high_price, low_price = np.loadtxt('data.csv',delimiter=',', usecols=(3,4),converters={3:str2num,4:str2num},unpack=True)
print("Highest = ", np.max(high_price))
print("Lowest = ", np.min(low_price))
```

-   ptp() 返回数组最大值与最小值之间的差值

```python
print("Spread high price: ", np.ptp(high_price))
print("Spread low price: ", np.ptp(low_price))
```


<a id="org1ba3b26"></a>

## 统计分析

```python
print(high_price)
print(high_price.ndim)
print(len(high_price))
print(high_price.shape)
print(type(high_price))
```


<a id="org4e38d9a"></a>

### 获取收盘价

```python
closing_price = np.loadtxt('data.csv',delimiter=',',usecols=(1,),converters={1:str2num})
```


<a id="orgea77500"></a>

### np.median() 获取中位数

```python
print(np.median(closing_price))
```


<a id="orgaf5ce9e"></a>

### np.msort() 对数组排序

```python
sorted_close = np.msort(closing_price)
print("sorted = ", sorted_close)

N = len(closing_price)
print("middle = ", sorted_close[int((N - 1) / 2)])

print("average middle = ",(sorted_close[int(N/2)] + sorted_close[int((N - 1)/2)]) / 2)
```


<a id="orga6fe622"></a>

### np.var(c) 方差

```python
print("variance = ", np.var(closing_price))

print("Variance from definition = ", np.mean((closing_price - np.mean(closing_price)) ** 2))
```


<a id="orga087df2"></a>

# 专用函数

```python
import numpy as np
import time

# 对应数据格式 ""2019年2月3日""
def datestr2num(s):
    s = s[1:-1]
    fmt = "%Y年%m月%d日"
    t1 = time.strptime(s,fmt)
    return time.strftime("%Y%m%d",t1)

dates = np.loadtxt("data.csv",delimiter = ',', usecols=(0,),converters={0:datestr2num},unpack=True)
print(date)

```

-   >>> 未完成


<a id="org788205e"></a>

## 对复数进行排序

-   sort<sub>complex</sub> 函数对生成的复数进行排序

```python
import numpy as np

np.random.seed(42)
complex_numbers = np.random.random(5) + 1j * np.random.random(5)
print("Complex numbers\n ",complex_numbers)

print("Sorted\n", np.sort_complex(complex_numbers))
```


<a id="orgf877faa"></a>

## 搜索

-   np.argmax() 返回数组中最大值对应的下标
-   np.nanargmax() 返回数组中最大值对应的下标，忽略Nan的值
-   np.argmin()
-   np.nanargmin()
-   np.argwhere() 根据条件搜索非0的元素，并分组返回对应的下标

```python
a = np.array([2,3,4,1,3,5,0,6])
print("index of the min number: ",np.argmin(a))
print("index of the max number: ",np.argmax(a))

b = np.array([np.nan,3,5,6,7,8])
print("index of the min number: ",np.argmin(a))
print("index of the max number: ",np.argmax(a))

print("a = ",a)
print("np.argwhere(a <= 4): ",np.argwhere(a <= 4))

```

-   searchsorted() 为指定的插入值返回一个在有序数组中的索引位置，从这个位置插入可以保持数组的有序性
-   np.insert()

```python
a = np.arange(9)
indices = np.searchsorted(a,[-2,6])
print("a = ",a)
print("indices :",indices)
print("The full array",np.insert(a,indices,[-2,6]))

```


<a id="org21f3384"></a>

## 数组元素抽取

-   extract() 根据某个条件在数组中抽取元素
-   nonzero() 抽取数组中的非0元素

```python
a = np.arange(9)
condition = (a % 2) == 0
print("Even numbers: ",np.extract(condition, a))

print("non zero: ",np.nonzero(a))

```


<a id="orgc062fd7"></a>

## 金融函数

-   >>> 未完成
