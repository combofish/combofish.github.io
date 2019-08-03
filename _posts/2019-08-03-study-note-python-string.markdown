---
layout: post
title: "Python Basis (2) Sting, Study Note"
date: 2019-08-03
tag: Python String
---
本文是我在学习期间的笔记，看的书是《python语言及其应用》。

- [字符串](#orgfccb567)
  - [str()](#orgbcd0e58)
  - [使用\\转义](#orgdee654b)
  - [使用+拼接](#orgfcbaf51)
  - [使用\*复制](#orgcdaf9fe)
  - [使用 [] 提取字符](#org1a2e2b4)
  - [使用 [start:end:step]分片](#org56f65aa)
  - [使用 len() 获得长度](#orgc5d0a04)
  - [使用 split() 分割](#orgb179f37)
  - [使用 join() 合并](#orgaabccb4)
  - [熟悉字符串](#orge666415)
  - [大小写与对齐的方式](#org9e3d9d0)
  - [使用 replace() 替换](#orgd20b939)


<a id="orgfccb567"></a>

# 字符串

-   对Unicode的支持使得python3 可以包含世界上任何书面语言以及许多特殊符号，
-   字符串本质是字符序列
-   python的字符串是不可变的，但是可以将字符串的一部分复制到新字符串
-   包裹在一对单引号或者双引号中即可创建字符

-   三元引号可以用于创建多行字符串
-   python 允许空串存在


<a id="orgbcd0e58"></a>

## str()

-   进行类型转换
-   当调用 print() 函数或者进行 字符串差值(string interpolation)时，
-   Python内部会自动使用str()将非字符串对象转换为字符串


<a id="orgdee654b"></a>

## 使用\\转义

```python
palindrome = 'A man,\nA plan, \nA canal:\nPanama.'
print(palindrome)

print('\tabc')
print('a\tbc')
print('ab\tc')

testimony = "\"I did nothing!\" he said. \"Not that either! Or the other thing.\""
print(testimony)

fact = "The world's largest rubber duck was 54'2\" by 65'7\" by 105'"
print(fact)

print('Today we honor our friend, the backslash: \\.')
```


<a id="orgfcbaf51"></a>

## 使用+拼接

-   可以直接将一个字面字符串（非字符串变量）放到另一个后面直接实现拼接
-   拼接时，不会自动添加空格
-   用 print() 打印时，Python会在各个参数之间自动添加空格，并在结尾添加换行符

```python
print('a string''a string')
print('a string' + 'a string')
print('a string','a string')
```


<a id="orgcdaf9fe"></a>

## 使用\*复制

```python
start = 'Na' * 4 + '\n'
middle = 'Hey' * 3 + '\n'
end = 'Goodbye'
print(start + middle + end)

```


<a id="org1a2e2b4"></a>

## 使用 [] 提取字符

```python
letters = 'abcdefghijklmnopqrstuvwxyz'
print(len(letters))
print(letters[0])
print(letters[-1])
```


<a id="org56f65aa"></a>

## 使用 [start:end:step]分片

slice 分片操作，可以从一个字符串中抽取子字符串

-   [:] 提取从开头到结尾的整个字符串
-   [start:] 从start 提取到结尾
-   [:end] 从开头提取到end - 1
-   [start:end] 从 start 提取到 end - 1
-   [start:end:step] 从 start 提取到 end - 1,每 step个字符提取一个

```python
print('letters : ',letters)
print(letters[:])
print(letters[20:])
print(letters[10:])
print('letters[12:15]',letters[12:15])
print('letters[-3:]',letters[-3:])
print(letters[18:-3])
print('letters[-6:-2]',letters[-6:-2])
print(letters[::7])
print(letters[4:20:3])
print('letters[19::4]',letters[19::4])
print('letters[:21:5]',letters[:21:5])
print('letters[-1::-1]',letters[-1::-1])
print('letters[::-1]',letters[::-1])
```

-   分片操作对无效偏移量的容忍程度大于单字符提取操作
-   小于起始位置的偏移量会被当作0,大于终止位置的偏移量会被当作-1

```python
print('letters[-50:]',letters[-50:])
print('letters[-51:-50]',letters[-51:-50])
print('letters[:70]',letters[:70])
print('letters[70:71]',letters[70:71])

```


<a id="orgc5d0a04"></a>

## 使用 len() 获得长度

-   广义函数 len()
-   函数指的是可以执行某些特定操作的有名字的代码

```python
print(len(letters))
empty = ''
print(len(empty))
```


<a id="orgb179f37"></a>

## 使用 split() 分割

-   可以基于分割符将字符串分割成由若干个子串组成的列表
-   默认使用空白字符，换行符，空格，制表符 作为分割符

```python
todos = 'get gloves,get mask,give cat citamins,call ambulance'
print(todos.split(','))
print(todos.split())
test = '   hhh hello world!'
print(test.split())

```


<a id="orgaabccb4"></a>

## 使用 join() 合并

-   str.join(list)

```python
crypto_list = ['Yeti', 'Bigfoot', 'Loch Ness Monster']
crypto_string = ', '.join(crypto_list)
print('Found and signing book deals: ', crypto_string)

```


<a id="orge666415"></a>

## 熟悉字符串

-   str.startswith('str') 是不是以他开头
-   str.endswith('str') 是不是以他结尾
-   str.find('str') 第一次出现单词的偏移量
-   str.rfind('str') 最后一次出现的偏移量
-   str.count('str') 出现多少次
-   str.isalnum('str') 都是字母或数字吗

```python
poem = '''All that doth flow we connot liquid name
Or else would fire and water be the same;
But that is liquid which is moist and wet
Fire that property can never get.
Then 'tis not could thay doth the fire put out
But 'tis the wet that makes it die, no doubt'''

print('poem: ',poem[:13])
print('len(poem)',len(poem))
print("startswith('All')",poem.startswith('All'))
print('endswith("That\'s all, folks!)',poem.endswith('That\'s alll, folks!'))
word = 'the'
print('poem.find("the")',poem.find(word))
print('poem.rfind("the")',poem.rfind(word))
print('poem.count("the")',poem.count(word))
print('poem.isalnum()',poem.isalnum())
test = '12adb'
test2 = '123'
print(test.isalnum(),test2.isalnum())
```

-   test

```python
a = '  hello world!'
print(a.startswith('hello'))
```


<a id="org9e3d9d0"></a>

## 大小写与对齐的方式

-   str.capitalize()
-   str.title()
-   str.upper()
-   str.lower()
-   str.swapcase()
-   str.center(num)
-   str.ljust(num)
-   str.rjust(num)

```python
s = 'a duck goes into A bar...'
print('s.strip("."):',s.strip('.'))
print(s.capitalize())
print(s.title())
print(s.upper())
print(s.lower())
print(s.swapcase())
print(s.center(40))
print(s.ljust(40))
print(s.rjust(40))

```


<a id="orgd20b939"></a>

## 使用 replace() 替换

-   str.replace('str','str',num)
-   num 最多替换几次

```python
print(s.replace('duck','marmoset'))
print(s.replace('a ','a famous',100))
print(s.replace('a','a famous',100))

```