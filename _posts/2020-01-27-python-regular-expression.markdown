---
layout: post
title: "Python: Regular Expression(Study Note)"
date: "2020-01-27 00：54：00"
tag: regex re Python  
---
- [正则表达式](#orgda05641)
  - [正则表达式规则](#org9b8c0ad)
  - [经典正则表达式](#org8c19773)
  - [IP地址的正则表达式](#orgab5c75b)
  - [re 库的基本使用](#orge60f686)
    - [Re 库主要功能函数](#orgba32278)
    - [re.search()](#orga194ec9)
    - [re.match()](#org6163760)
    - [re.findall()](#orga89cd84)
    - [re.split()](#org0aad166)
    - [re.finditer()](#orgba1361f)
    - [re.sub()](#org5742a20)
    - [RE 库的另一种等价用法](#org7ee0318)
  - [RE 库的 match 对象](#org502adad)
    - [match 对象的属性](#org2ec6b33)
    - [match 对象的方法](#org6d22815)
    - [示例代码](#org1916605)
  - [RE 库的贪婪匹配和最小匹配](#org467acd6)
    - [默认匹配](#orgb660e27)
    - [最小匹配](#org6707512)
    - [最小匹配操作符号](#org4c933e9)


<a id="orgda05641"></a>

# 正则表达式

RE, regex, Regular expression 用简洁形式表示了一组字符串的 **特征** 或 **模式** ，通用的字符串表达框架。

-   表达文本类型的特征
-   判断某个字符串的特征归属
-   同时查找或替换一组字符串
-   匹配字符串的全部或部分

用前需要编译，正则表达式由字符和操作符组成。


<a id="org9b8c0ad"></a>

## 正则表达式规则

```shell
. 表示任何单个字符
[] 字符集，对单个字符给出取值范围
[^] 非字符集，对单个字符给出排除范围
* 前一个字符0次或者无限次扩展
+ 前一个字符1次或者无限次扩展
? 前一个字符0次或者1次扩展
| 左右表达式任意一个
{m} 扩展前一个字符m次
{m,n} 扩展前一个字符m至n次，含n
^ 匹配字符串开头
$ 匹配字符串结尾
() 分组标记，内部只能使用|操作符
\d 数字，等价于[0-9]
\w 单个字符，等价于[A-Za-z0-9]

```


<a id="org8c19773"></a>

## 经典正则表达式

```shell
^[A-Za-z]+$ 由26个字母组成的字符串
^[A-Za-z0-9]+$ 由26个字母和数字组成的字符串
^-?\d$ 整数形式的字符串
^[0-9]*[1-9][0-9]*$ 正整数形式的字符串
[1-9]\d{5} 中国境内邮政编码，6位
[\u4e00-\u9fa5] 匹配中文字符
\d{3}-\d{8}|\d{4}-\d{7} 国内电话号码，010-68912536

```


<a id="orgab5c75b"></a>

## IP地址的正则表达式

```shell
\d.\d.\d.\d
\d{1,3}.\d{1,3}.\d{1,3},\d{1,3} # 不够准确
255.255.255.255 
0-99: [1-9]?[0-9]     [1-9]?\d      
100-199: 1[0-9]{2}    1\d{2}
200-249: 2[0-4][0-9]  2[0-4]\d
250-255: 25[0-5]      
四位 (.){3}()
(( [1-9]?\d | 1\d{2} | 2[0-4]\d | 25[0-5] ).){3}( [1-9]?\d | 1\d{2} | 2[0-4]\d | 25[0-5] )
```


<a id="orge60f686"></a>

## re 库的基本使用

Python 的标准库 正则表达式的表示类型: 原生字符串 raw string : r'text' 原生字符串是不包含转译字符的字符串，当正则表达式包含 **转义字符** 时，使用raw string.


<a id="orgba32278"></a>

### Re 库主要功能函数

-   re.search(pattern,string,flags=0) 在一个字符串中搜索匹配正则表达式的第一个位置，返回match 对象
-   re.match() 从一个字符串的开始位置起匹配正则表达式，返回match 对象
-   re.findall() 搜索字符串，以列表类型返回全部能匹配的字符串
-   re.split( pattern,string,maxsplit,flags) 返回列表类型
-   re.finditer() 搜索字符串，返回一个匹配结果的迭代类型，每个迭代元素是match 对象
-   re.sub(pattern,repl,string,count=0,flags=0)
    
    pattern: 正则表达式的字符串或原生字符串表示 maxsplit: 最大分割数，剩余部分作为最后一个元素输出

1.  常用标记：

    -   re.I re.IGNORECASE 忽略正则表达式中的大小写
    -   re.M re.MULTILINE 正则表达式中的<sup>操作符能够将给定字符串的每行当作匹配开始</sup>
    -   re.S re.DOTALL 正则表达式中的.操作符能够匹配所有字符，默认匹配除换行外的所有字符


<a id="orga194ec9"></a>

### re.search()

```python
import re

match = re.search(r'[1-8]\d{5}','BIT 188888')
if match:
    print(match.group(0))

```


<a id="org6163760"></a>

### re.match()

```python
match = re.match(r'[1-9]\d{5}','BIT 100299')
if match:
    pring(match.group(0))

print(match.group(0))

```

-   match 是一个空变量
    
    ```python
    match = re.match(r'[1-9]\d{5}','123456 BIT')
    print(match.group(0))
    
    ```


<a id="orga89cd84"></a>

### re.findall()

```python
ls = re.findall(r'[1-9]\d{5}','BIT 123456 234567')
print(ls)
```


<a id="org0aad166"></a>

### re.split()

```python
re.split(r'[1-9]\d{5}','BIT 788222 JIH 892789 hi')

```

-   添加 maxsplit 约束
    
    ```python
    re.split(r'[1-9]\d{5}','BIT 123456 JI 234556',maxsplit=1)
    ```


<a id="orgba1361f"></a>

### re.finditer()

```python
for m in re.finditer(r'[1-9]\d{5}', 'BIT 190290 YUB223423'):
    if m:
	print(m.group(0))

```


<a id="org5742a20"></a>

### re.sub()

```python
re.sub(r'[1-9]\d{5}',':zipcode', 'BIT123456 TSU100022')
```


<a id="org7ee0318"></a>

### RE 库的另一种等价用法

上述是 函数式用法，一次性操作。 面向对象用法，编译后多次操作

-   regex = re.compile(pattern,flags=0) 将正则表达式的字符串形式编译成正则表达式对象，仍有 search(), match(), split(), findall(), finiter(), sub() 函数。
    
    ```python
    pat = re.compile(r'[1-9]\d{5}')
    rst = pat.search('BIJ 100821')
    print(rst.group(0))
    
    ```


<a id="org502adad"></a>

## RE 库的 match 对象

match 对象，一次匹配的结果


<a id="org2ec6b33"></a>

### match 对象的属性

-   .string 待匹配的文本
-   .re 匹配时使用的pattern 对象
-   .pos 正则表达式搜索文本的开始位置
-   .endpos 正则表达式搜索文本的结束位置


<a id="org6d22815"></a>

### match 对象的方法

-   group(0) 获得匹配后的字符串
-   start() 匹配字符串在原始字符串的开始位置
-   end() 匹配字符串在原始字符串的结束位置
-   span() 返回 (.start(), .end())


<a id="org1916605"></a>

### 示例代码

```python
import re

m = re.search(r'[1-9]\d{5}','BIJ 123456 HUI123456')
if m:
    print(match.group(0))

print(m.string)
print(m.re)
print(m.pos)
print(m.endpos)

print(m.start())
print(m.end(),m.span())
```


<a id="org467acd6"></a>

## RE 库的贪婪匹配和最小匹配

RE 库默认采用贪婪匹配，即输出匹配最长的子串。


<a id="orgb660e27"></a>

### 默认匹配

```python
m = re.search(r'PY.*N', 'PYTHONNNN')
if m:
    print(m.group(0))

```


<a id="org6707512"></a>

### 最小匹配

```python
m = re.search(r'PY.*?N', 'PYANBNCNDN')
if m:
    print(m.group(0))

```


<a id="org4c933e9"></a>

### 最小匹配操作符号

```shell
*? 前一个字符0次或无限次扩展，最小匹配
+? 前一个字符1次或无限次扩展，最小匹配
?? 前一个字符0次或1次扩展，最小匹配
{m,n}? 扩展前一个字符m 至 n次(含n),最小匹配

```
