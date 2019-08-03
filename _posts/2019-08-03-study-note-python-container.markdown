---
layout: post
title: "Python Basis (3) Container: list,tuple,dict,set. Study Note"
date: 2019-08-03
tag: Python Container List Tuple Dict Set 
---
本文是我在学习期间的笔记，看的书是《python语言及其应用》。


- [list](#org05b0de7)
  - [list() 将其他类型转换成列表](#org60aec9e)
  - [[offset] 获取元素](#org35f577b)
  - [包含列表的列表](#orgc14d404)
  - [使用 offset 修改元素](#orgd200431)
  - [指定范围内使用切片提取元素](#orgd1dbcda)
  - [append() 添加元素至尾部](#orgcc0428f)
  - [extend() 或 += 合并列表](#org456271b)
  - [insert() 在指定位置插入元素](#orge2ca053)
  - [del 删除指定位置的元素](#orga825a86)
  - [remove() 删除具有指定值的元素](#orgd31e636)
  - [pop() 获取并删除指定位置的元素](#orgf7f609d)
  - [index() 查询具有特定值的元素的位置](#org24d6072)
  - [in 判断值是否存在](#org0783226)
  - [count() 记录特定的值出现的次数](#orgc859a72)
  - [join() 转换为字符串](#orgf7df79a)
  - [sort() 重新排列元素](#org82a75f6)
  - [len() 获取长度](#orgf5f12d0)
  - [= 赋值， copy() 复制](#org1d36c5b)
- [tuple](#org552daa5)
  - [使用 () 创建元组](#orgc6c4773)
- [dictionary](#orge09a0b6)
  - [使用 {} 创建字典](#org49fa956)
  - [dict() 转换为字典](#orga3703c8)
  - [[key] 添加或者修改元素](#org2935f90)
  - [update() 合并字典](#org3fe5396)
  - [del 删除具有指定键的元素](#org07c9ceb)
  - [clear() 删除所有的元素](#org1e513d9)
  - [in 判断键是否存在](#orgc86692b)
  - [[key] 获取元素](#org6f0af89)
  - [keys() 获取所有的键, values() 获取所有的值](#orgcf937b1)
  - [items() 获取所有的键值对](#org79ca2ba)
  - [= 赋值， copy() 复制](#orgbf3af78)
- [set](#org19c1027)
  - [set() 创建集合](#org522f13e)
  - [set() 将其他类型转换成集合](#orgaf1cd93)
  - [in 测试值是否存在](#orgcdf6ff2)
  - [合并及运算符](#org86bdda7)
  - [其他一些运算符](#org9474333)
  - [比较几种数据结构](#orge29d25a)
  - [建立大型数据结构](#org37dbf94)
- [练习](#org954d947)

在编程中，最常见的工作就是将数据进行拆分和合并，将其加工为特定的形式， 而数据结构就是用以切分数据的钢锯以及合并数据的粘和合枪。

-   如果经常需要判断一个值是否存在一个列表中，但并不关心列表中元素之间的顺序，

使用Python的集合进行存储和查找会是更好的选择


<a id="org05b0de7"></a>

# list

-   列表非常适合利用顺序和位置定位某一元素，尤其时当元素的顺序或者内容经常发生改变时
-   相同的元素允许出现多次
-   list() 可以创建一个空列表
-   [] 可以创建一个空列表

```python
empty_list = []
weekendys = ['Monday','Tuesday','Wednesday','Thursday','Friday']
big_birds = ['emu','ostrich','cassowary']
first_names = ['Graham','Jhon','Terry','Terry','Michael']
another_empty_list = list()
print(empty_list,weekendys,another_empty_list)
```


<a id="org60aec9e"></a>

## list() 将其他类型转换成列表

```python
print(list('cat'))
a_tuple = ('ready','fire','aim')
print(list(a_tuple))
birthday = '1/6/1952'
print(birthday.split('/'))
splitme = 'a/b//c/d///e'
print(splitme.split('/'))
print(splitme.split('//'))
```


<a id="org35f577b"></a>

## [offset] 获取元素

-   当指定的偏移量小于起始位置或者大于末尾位置时会产生异常

```python
marxes = ['Grouco','Chico','Harpo']
print(marxes[0],marxes[1],marxes[2],marxes[-1],marxes[-2],marxes[-3])
print(marxes[5])
```


<a id="orgc14d404"></a>

## 包含列表的列表

```python
small_birds = ['hummingbird','finch']
extinct_birds = ['dodo','passenger pigeon','Norwegian Blue']
carol_birds = [3,'French hens',2,'turtledoves']
all_birds = [small_birds, extinct_birds, carol_birds]
print('all_birds',all_birds)
print('all_birds[0]',all_birds[0])
print('all_birds[1]',all_birds[1])
print('all_birds[1][0]',all_birds[1][0])

```


<a id="orgd200431"></a>

## 使用 offset 修改元素

```python
marexs = ['Groucho','Chico','Harpo']
marexs[2] = 'Wanda'
print('marexs',marexs)
```


<a id="orgd1dbcda"></a>

## 指定范围内使用切片提取元素

```python
marexs = ['Groucho','Chico','Harpo']
print('marexs[0:2]',marexs[0:2])
print('marexs[::2]',marexs[::2])
print('marexs[::-2]',marexs[::-2])
print('marexs[::-1]',marexs[::-1])
```


<a id="orgcc0428f"></a>

## append() 添加元素至尾部

```python
marexs.append('Zeppo')
print(marexs)

```


<a id="org456271b"></a>

## extend() 或 += 合并列表

```python
marexs = ['Groucho','CHico','Harpo','Zeppo']
others = ['Gummo','Karl']
marexs.extend(others)
print('marexs',marexs)
marexs += others
print('marexs',marexs)
marexs = ['Groucho','CHico','Harpo','Zeppo']
others = ['Gummo','Karl']
marexs.append(others)
print('marexs',marexs)

```


<a id="orge2ca053"></a>

## insert() 在指定位置插入元素

-   偏移量为0可以插入列表头部
-   偏移量超过尾部，则会插入到列表最后

```python
marxes = ['Groucho','Chico','Harpo','Zeppo']
print(marxes)
marxes.insert(3,'Karl')
print(marxes)
```


<a id="orga825a86"></a>

## del 删除指定位置的元素

-   del 是Python 语句

```python
print('marxes',marxes)
del marxes[-1]
print('marxes',marxes)

```


<a id="orgd31e636"></a>

## remove() 删除具有指定值的元素

```python
print('marxes',marxes)
marxes.remove('Chico')
print('marxes',marxes)

```


<a id="orgf7f609d"></a>

## pop() 获取并删除指定位置的元素

-   pop(-1) 默认删除列表的尾元素

```python
print('marxes',marxes)
marxes.pop()
print('marxes',marxes)
marxes.pop(0)
print('marxes',marxes)
```


<a id="org24d6072"></a>

## index() 查询具有特定值的元素的位置

```python
marxes = ['Groucho','Chico',"Harpo",'Zeppo']
print(marxes.index('Chico'))

```


<a id="org0783226"></a>

## in 判断值是否存在

```python
print('marxes',marxes)
print("'Groucho' in marxes",'Groucho' in marxes)
words = ['a','deer','a','famale','deer']
print("'deer' in words",'deer' in words)

```


<a id="orgc859a72"></a>

## count() 记录特定的值出现的次数

```python
print('marxes',marxes)
print(marxes.count('Harpo'))
print(marxes.count('Bob'))

snl_skit = ['cheeseburger','cheeseburger','cheeseburger']
print(snl_skit.count('cheeseburger'))

```


<a id="orgf7df79a"></a>

## join() 转换为字符串

-   join() 实际上是一个字符串方法

```python
friends = ['Harry','Hermione','Ron']
separator = ' * '
joined = separator.join(friends)
print('joined',joined)
separated = joined.split(separator)
print('separated',separated)
print('separated == friends',separated == friends)
```


<a id="org82a75f6"></a>

## sort() 重新排列元素

-   列表方法 sort() 会对原列表进行排序，改变原列表内容
-   通用函数 sorted() 会返回排好的列表副本，原列表内容不变
-   数字，默认从小到大顺序
-   字符串，默认字母表顺序
-   reverse=True 参数可以改为降序排列

```python
marxes = ['Groucho','Chico','Harpo']
sorted_marxes = sorted(marxes)
print('sorted_marxes',sorted_marxes)
print('marxes',marxes)
marxes.sort()
print('marxes',marxes)

numbers = [2,1,3,2,9.0,3.4]
numbers.sort()
print('numbers',numbers)
numbers.sort(reverse=True)
print('numbers',numbers)
```


<a id="orgf5f12d0"></a>

## len() 获取长度

```python
print('marxes',marxes)
print('len(marxes)',len(marxes))

```


<a id="org1d36c5b"></a>

## = 赋值， copy() 复制

```python
a = [1,2,3]
b = a
print('a',a,'b',b)
a[0] = 'surprise'
print('a',a,'b',b)
b[0] = 'I hate surprise'
print('a',a,'b',b)
```

-   copy()
-   list() 转换函数
-   列表分片[:]

```python
a = [1,2,3]
b = a.copy()
c = list(a)
d = a[:]
a[0] = 'integer lists are boring'
print('a',a,'b',b,'c',c,'d',d)
```


<a id="org552daa5"></a>

# tuple

-   相当于常量列表
-   函数的参数是以元组形式传递的
-   命名元组可以当作对象的替代
-   元组占用的空间比列表小


<a id="orgc6c4773"></a>

## 使用 () 创建元组

-   每一个元素后面都要跟着一个逗号
-   定义元组真正靠的是每个元素的后缀逗号
-   元组解包， 一次将元组赋值给多个变量
-   tuple() 将其他类型转换成元组

```python
empty_tuple = ()
print('empty_tuple',empty_tuple)
one_marx = 'Groucho',
print(one_marx)
marx_tuple = 'Groucho','Chico','Harpo'
print(marx_tuple)
a, b, c = marx_tuple
print('a',a,'b',b,'c',c)

passwd = 'swoedfish'
icecream = 'tuttifrtti'
passwd, icecream = icecream, passwd
print('passwd',passwd,'icecream',icecream)
marx_list = ['Groucho','Chico','Harpo']
print('tuple(marx_list)',tuple(marx_list))
```


<a id="orge09a0b6"></a>

# dictionary

-   可变的，可以增加，删除，修改其中的键值对
-   Python 允许在列表，字典，或者元组的最后一个元素后面添加逗号


<a id="org49fa956"></a>

## 使用 {} 创建字典

```python
empty_dict = {}
print('empty_dict',empty_dict)

bierve = {
    'day':'A period f twenty-four hours, mostly misspent',
    'positive':"Mistaken at the top of one's voice",
    'misfortune':'The kind of fortune that never misses'}

print('bierve',bierve)
```


<a id="orga3703c8"></a>

## dict() 转换为字典

```python
lol = [['a','b'],['c','d'],['e','f']]
print('lol',dict(lol))

lot = [('a','b'),('c','d'),('e','f')]
tol = (['a','b'],['c','d'],['e','f'])
los = ['ab','cd','ef']
tos = ('ab','cd','ef')
print(dict(lot),dict(tol),dict(los),dict(tos))
```


<a id="org2935f90"></a>

## [key] 添加或者修改元素

```python
pythons = {
    'Chapman':'Graham',
    'Cleese':'Jhon',
    "Idle":'Eric',
    "Jones":'Terry',
    'Palin':'Michael'}
print('pythons',pythons)
pythons['Gilliam'] = 'Gerry'
print('pythons',pythons)
pythons['Gilliam'] = 'Terry'
print('pythons',pythons)

```


<a id="org3fe5396"></a>

## update() 合并字典

```python
print('pythons',pythons)
others = {'Marx':'Groucho','Howard':'Moe'}
pythons.update(others)
print('pythons',pythons)

first = {'a':1,'b':2}
second = {'b':'platypus'}
first.update(second)
print('first',first)
```


<a id="org07c9ceb"></a>

## del 删除具有指定键的元素

```python
print('pythons',pythons)
del pythons['Marx']
del pythons['Howard']
print('pythons',pythons)
```


<a id="org1e513d9"></a>

## clear() 删除所有的元素

```python
print('pythons',pythons)
pythons.clear()
print('pythons',pythons)
```


<a id="orgc86692b"></a>

## in 判断键是否存在

```python
pythons = {'Chapman':'Graham','Cleese':'Jhon','Jones':'Terry','Palin':'Michael'}
print("'Chapman' in pythons", 'Chapman' in pythons)
print("'Jhon' in pythons", 'Jhon' in pythons)
```


<a id="org6f0af89"></a>

## [key] 获取元素

-   直接获取不包含会报错
-   get() 获取可以指定默认值
-   get() 获取不到会返回 None

```python
pythons = {'Chapman':'Graham','Cleese':'Jhon','Jones':'Terry','Palin':'Michael'}
print("'Palin' in pythons", 'Palin' in pythons)
print('pythons.get("Marx")',pythons.get('Marx'))
print('pythons.get("Marx")',pythons.get('Marx','Not a Python'))
print('pythons.get("Cleese")',pythons.get('Cleese'))
```


<a id="orgcf937b1"></a>

## keys() 获取所有的键, values() 获取所有的值

```python
print('pythons',pythons)
print('pythons.keys()',pythons.keys())
print('pythons.values()',pythons.values())
print('list(pythons.values())',list(pythons.values()))
```


<a id="org79ca2ba"></a>

## items() 获取所有的键值对

```python
print('pythons',pythons)
print('pythons.items()',pythons.items())
print('list(pythons.items())',list(pythons.items()))
```


<a id="orgbf3af78"></a>

## = 赋值， copy() 复制

```python
signals = {'green':'go','yellow':'go faster','red':'smile for the camera'}
save_signals = signals
signals['blue'] = 'confuse everyone'
print('signals',signals)
print('save_signals',save_signals)

original_signals = signals.copy()
signals['black'] = 'like dark'
print('signals',signals)
print('save_signals',original_signals)
```


<a id="org19c1027"></a>

# set

-   键与键之间也不允许重复


<a id="org522f13e"></a>

## set() 创建集合

```python
empty_set = set()
print('empty_set',empty_set)
even_numbers = {0,2,4,6,8}
odd_numbers = {1,3,5,7,9}
print('even_numbers',even_numbers)
print('odd_numbers',odd_numbers)
```


<a id="orgaf1cd93"></a>

## set() 将其他类型转换成集合

-   字典作为参数传入 set() 时，只有键会被使用

```python
s = 'letters'
print('set',set(s))
print(set(['Dasher','Dancer','Prance','Mason-Dixon']))
print(set(('Ummagumma','Echoes','Atom Heart Mother')))
print(set({'apple':'red','orange':'orange','cherry':'red'}))
```


<a id="orgcdf6ff2"></a>

## in 测试值是否存在

```python
drinks = {
    'martini':{'vodka','vermouth'},
    'black russian':{'vodka','kahlua'},
    'white russian':{'cream','kahlua','vodka'},
    'manhattan':{'rye','vermouth','bitters'},
    'screwdriver':{'orange juice','vodka'}
}

for name, contents in drinks.items():
    if 'vodka' in contents:
        print('name',name)

print('other choose')
for name, contents in drinks.items():
    if 'vodka' in contents and not ('vermouth' in contents or 'cream' in contents):
        print('name',name)
```


<a id="org86bdda7"></a>

## 合并及运算符

-   & 交集运算符

```python
for name, contents in drinks.items():
    print(name, contents)
    if contents & {'vermouth', 'orange juice'}:
        print(name)
```

```python
for name, contents in drinks.items():
    if 'vodka' in contents and not contents & {'vermouth','cream'}:
        print('name',name)
```

-   union()
-   difference() 差集，出现在第一个集合，但不出现在第二个集合
-   intersection()

```python
bruss = drinks['black russian']
wruss = drinks['white russian']
print('bruss',bruss)
print('wruss',wruss)

a = {1,2}
b = {2,3}

print('a & b', a & b)
print('a.intersection(b)',a.intersection(b))
print('bruss & wruss', bruss & wruss)

print('a | b', a | b)
print('a.union(b)',a.union(b))
print('bruss | wruss', bruss | wruss)

print('a - b', a - b )
print('a.difference(b)',a.difference(b))
print('bruss - wruss', bruss - wruss)
print('wruss - bruss', wruss - bruss)
```


<a id="org9474333"></a>

## 其他一些运算符

```python
print('a',a)
print('b',b)
print('bruss',bruss)
print('wruss',wruss)

print('a ^ b', a ^ b)
print('a.symmetric_difference(b)',a.symmetric_difference(b))
print('bruss ^ wruss', bruss ^ wruss)

print('a <= b', a <= b)
print('a.issubset(b)', a.issubset(b))
print('bruss <= wruss',bruss <= wruss)

print('a < b', a < b)
print('a < a', a < a)
print('bruss < wruss',bruss < wruss)

print('a >= b', a >= b)
print('a.issuperset(b)',a.issuperset(b))
print('wruss >= bruss', wruss >= bruss)
print('a >= a', a >= a)

print('a > b', a > b )
print('wruss > bruss', wruss > bruss)
```


<a id="orge29d25a"></a>

## 比较几种数据结构

```python
marx_list = ['Groucho','Chico','Harpo']
marx_tuple = 'Groucho','Chico','Harpo'
marx_dict = {'Groucho':'banjo','Chico':'piano','Harpo':'harp'}
print(marx_dict['Harpo'])
print(marx_list[2])
print(marx_tuple[2])
```


<a id="org37dbf94"></a>

## 建立大型数据结构

-   列表， 字典， 集合不能作为字典的键

```python
marexs = ['Groucho','Chico','Harp']
pythons = ['Chapamn','Cleese','Gilliam','Jones','Palin']
stooges = ['Moe','Curly','Larry']

tuple_of_lists = marexs,pythons,stooges
print('tuple_of_lists',tuple_of_lists)

list_of_lists = [marexs,pythons,stooges]
print('list_of_lists',list_of_lists)

dict_of_lists = {'Marexs':marexs,'pythons':pythons,'stooges':stooges}
print('dict_of_lists',dict_of_lists)
```

-   元组可以作为字典的键

```python
houses = {
    (44.79,-99.33, 238):'My house',
    (23.22,-23.23,233):'The White House'}

print('house',houses)
```


<a id="org954d947"></a>

# 练习

```python
year_list = [1980,1981,1982,1983,1984,1985]
print('year_list[3]',year_list[3])
print('oldest year ', year_list.index(max(year_list)))

things = ['mozzarella','cinderella','salmonella']
# for name in things:
#     name.capitalize()

# print('things',things)    

# for name in things:
#     name.replace('rella','RELLA')

# print('things',things)

# for name in things:
#     name.upper()

# print('things',things)

things = ['mozzarella','cinderella','salmonella']
things = [ name.capitalize() for name in things ]
print('things',things)

things = ['mozzarella','cinderella','salmonella']
things = [ name.upper() for name in things ]
print('things',things)

things = ['mozzarella','cinderella','salmonella']
things = [ name.replace('rella','RELLA') for name in things ]
print('things',things)

things = ['mozzarella','cinderella','salmonella']
things.remove('cinderella')
print('things',things)
```

-   列表操作

```python
surprise = ['Groucho','Chico','Harpo']
surprise[-1] = surprise[-1][::-1].capitalize()
print('surprise',surprise)

```

```python
e2f = {'dog':'chien','cat':'chat','walrus':'morse'}
print('e2f',e2f)
print('e2f["walrus"]',e2f["walrus"])
print('e2f.get("walrus")',e2f.get('walrus','Not found'))

f2e = {}
for key, value in e2f.items():
    f2e[value] = key

print('f2e',f2e)
print('f2e.get("chien")',f2e.get("chien"))
print('word set:', set(f2e.keys()))

```

-   多级字典

```python
life = {}
life['animals'] = {'cats':['Henri','Grumpy','Lucy']}
life['plants'] = {}
life['others'] = {}

print('life',life)
print('life fist level key: ',list(life.keys()))
print('life["animals"] keys: ',list(life["animals"].keys()))
print('life["animals"]["cats"]: ',life["animals"]['cats'])
```