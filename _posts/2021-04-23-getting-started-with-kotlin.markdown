---
layout: post
title: "Getting started with Kotlin"
date: "2021-04-23 11：23：00"
tag: Kotlin Java Jvm Android
---
- [Kotlin](#orgd9bfae6)
  - [简介](#orgd95fc21)
  - [基础语法](#org160ec76)
    - [包声明](#orgf981051)
    - [定义常量与变量](#org56b2e80)
    - [注释](#org56d06d2)
    - [? 可以为空(null)](#orgc84e9e5)
    - [类型检测](#orgfbf96af)
    - [Int to String, String to Int](#orga57dd15)
    - [比较](#org691a53d)
    - [多行字符串](#org7b6d9a5)
    - [字符串模板](#org3f6db40)
  - [函数](#org7608c70)
    - [函数定义](#org741c9dc)
    - [无返回值的函数(类似Java中的void)：](#org6e92b0e)
    - [可变长参数函数](#org9d0c940)
    - [lambda(匿名函数)](#orgf36ce2c)
    - [函数的便捷写法](#org7a56e63)
    - [默认参数 & 具名参数](#org08457e4)
  - [流程控制](#org64c2c74)
    - [When 表达式](#org73819bb)
    - [返回和跳转](#org5506f4c)
    - [Break 和 Continue 标签](#org9af424b)
  - [数据结构](#orge0ffaf0)
    - [定义列表](#orgb6b4696)
    - [数组](#org68872cd)
    - [map使用](#org3f20087)
    - [递归](#org5b4b45d)
    - [尾递归优化](#org3d2e4c1)
  - [异常](#org93c6e61)
    - [Exception 与 Java 类似](#orgb58e2d5)
  - [面向对象](#org9941ab7)
    - [抽象类](#org7e6b4c9)
    - [类的初始化代码块](#org88f17eb)
    - [内部类](#orgdeedfdd)
    - [类的修饰符](#org38f2301)
    - [Any 类](#org93f2bf8)
    - [泛型](#org4a3be39)
    - [super的泛型使用](#org683e433)
    - [接口](#org2b6d0a1)
    - [扩展](#orgead3f6d)
    - [单例模式](#org48998d5)
    - [接口代理,委托](#orgb1e3854)
    - [Sealed Class](#org032d245)
    - [枚举类](#orga12a60b)


<a id="orgd9bfae6"></a>

# Kotlin


<a id="orgd95fc21"></a>

## 简介

-   Kotlin (科特林)是一个用于现代多平台应用的静态编程语言，由 JetBrains 开发，2016年推出 Kotlin V1.0。
-   Kotlin可以编译成Java字节码，也可以编译成JavaScript，方便在没有JVM的设备上运行。
-   在Google I/O 2017中，Google 宣布 Kotlin 成为 Android 官方开发语言。

```kotlin
package start
fun main(args: Array<String>) {   
    println("Hello World!") 
}
```


<a id="org160ec76"></a>

## 基础语法


<a id="orgf981051"></a>

### 包声明

-   kotlin源文件不需要相匹配的目录和包，源文件可以放在任何文件目录。


<a id="org56b2e80"></a>

### 定义常量与变量

```kotlin
// 可变变量定义：var 关键字
var <标识符> : <类型> = <初始化值>
// 不可变变量定义：val 关键字，只能赋值一次的变量(类似Java中final修饰的变量)
val <标识符> : <类型> = <初始化值>
// 常量与变量都可以没有初始化值,但是在引用前必须初始化
```

-   编译器支持自动类型判断,即声明时可以不指定类型,由编译器判断。


<a id="org56d06d2"></a>

### 注释

```kotlin
// 这是一个单行注释
/* 这是一个多行的
 块注释。 */
```


<a id="orgc84e9e5"></a>

### ? 可以为空(null)

-   Kotlin的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式，字段后加!!像Java一样抛出空异常。
-   另一种字段后加?可不做处理返回值为 null或配合?:做空判断处理

```kotlin
//类型后面加?表示可为空
var age: String? = "23" 
//抛出空指针异常
val ages = age!!.toInt()
//不做处理返回 null
val ages1 = age?.toInt()
//age为空返回-1
val ages2 = age?.toInt() ?: -1
// 当一个引用可能为 null 值时, 对应的类型声明必须明确地标记为可为 null。
```


<a id="orgfbf96af"></a>

### 类型检测

-   is 运算符检测一个表达式是否某类型的一个实例(类似于Java中的instanceof关键字)。

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
	// 做过类型判断以后，obj会被系统自动转换为String类型
	return obj.length 
    }
```


<a id="orga57dd15"></a>

### Int to String, String to Int

```kotlin
var a = "13"
var b = 13
println(b.toString())
println(a.toInt())
```


<a id="org691a53d"></a>

### 比较

-   在 Kotlin 中，三个等号 `=` 表示比较对象地址，两个 == 表示比较两个值大小。


<a id="org7b6d9a5"></a>

### 多行字符串

-   String 可以通过 trimMargin() 方法来删除多余的空白。

```kotlin
fun main(args: Array<String>) {
    val text = """
    |多行字符串
    |多行字符串
    """.trimMargin()
    println(text)    // 前置空格删除了
}
```

-   默认 | 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 trimMargin(">")。


<a id="org3f6db40"></a>

### 字符串模板

```kotlin
var str = "World"
println("Hello${str}, and str can be ${str.replace("World","Duck")}")
```

-   输出$

```kotlin
val price = """
${'$'}9.99
"""
println(price)  // 求值结果为 $9.99
```


<a id="org7608c70"></a>

## 函数


<a id="org741c9dc"></a>

### 函数定义

```kotlin
fun 函数名(参数名:参数类型)：返回值类型{
}
```


<a id="org6e92b0e"></a>

### 无返回值的函数(类似Java中的void)：

```kotlin
fun printSum(a: Int, b: Int): Unit { 
    print(a + b)
}
```

-   如果是返回 Unit类型，则可以省略(对于public方法也是这样)：

```kotlin
public fun printSum(a: Int, b: Int) { 
    print(a + b)
}
```


<a id="org9d0c940"></a>

### 可变长参数函数

-   函数的变长参数可以用 vararg 关键字进行标识

```kotlin
fun vars(vararg v:Int){
    for(vt in v){
	print(vt)
    }
}
```


<a id="orgf36ce2c"></a>

### lambda(匿名函数)

```kotlin
fun main(args: Array<String>) {
    val sumLambda: (Int, Int) -> Int = {x,y -> x+y}
    println(sumLambda(1,2))  // 输出 3
}
```


<a id="org7a56e63"></a>

### 函数的便捷写法

```kotlin
var i = {a:Int,b:Int -> a+b}
var j:(Int,Int) -> Int = {x,y -> x+y}
    println(i(2,3))
}

fun add(a: Int, b: Int): Int {
    return a + b
}

fun add2(a: Int, b: Int): Int = a + b
```


<a id="org08457e4"></a>

### 默认参数 & 具名参数

```kotlin
println(getCircleArea(r = 3.9f))

fun getCircleArea(PI:Float = 3.14f,r:Float):Float{
    return PI * r * r
}
```


<a id="org64c2c74"></a>

## 流程控制


<a id="org73819bb"></a>

### When 表达式

-   when 将它的参数和所有的分支条件顺序比较，直到某个分支满足条件。
-   when 既可以被当做表达式使用也可以被当做语句使用。如果它被当做表达式，符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。
-   when 类似其他语言的 switch 操作符。其最简单的形式如下：

```kotlin
when (x) {
    1 -> print("x == 1")
    2,3 -> print("x == 2")
    in 4..9 -> print("x in 4-8")
    else -> { // 注意这个块
	      print("x 不是 1 ，也不是 2")
    }
}
```


<a id="org5506f4c"></a>

### 返回和跳转

Kotlin 有三种结构化跳转表达式：

-   return。默认从最直接包围它的函数或者匿名函数返回。
-   break。终止最直接包围它的循环。
-   continue。继续下一次最直接包围它的循环。


<a id="org9af424b"></a>

### Break 和 Continue 标签

在 Kotlin 中任何表达式都可以用标签（label）来标记。 标签的格式为标识符后跟 @ 符号，例如：abc@、fooBar@都是有效的标签。 要为一个表达式加标签，我们只要在其前加标签即可。

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
	if (……) break@loop
    }
}
```

标签限制的 break 跳转到刚好位于该标签指定的循环后面的执行点。 continue 继续标签指定的循环的下一次迭代。


<a id="orge0ffaf0"></a>

## 数据结构


<a id="orgb6b4696"></a>

### 定义列表

```kotlin
var nums = 1 until 100 // [1,100)
var nums == 1 .. 16
for (num in nums step 2){}
var num3 = nums.reversed()
num3.count()

var lists = listOf<String>("dog","egg","duck")
println(lists)

for (list in lists)
println(list)

for ((i,e) in lists.withIndex())
println("$i:$e")

for (i in 4 downTo 1 step 2) print(i) // 输出“42”

// 使用 until 函数排除结束元素
for (i in 1 until 10) {   // i in [1, 10) 排除了 10
    println(i)
}
```


<a id="org68872cd"></a>

### 数组

数组用类 Array 实现，并且还有一个 size 属性及 get 和 set 方法，由于使用 [] 重载了 get 和 set 方法.

-   数组的创建两种方式：一种是使用函数arrayOf()；另外一种是使用工厂函数。

```kotlin
fun main(args: Array<String>) {
    //[1,2,3]
    val a = arrayOf(1, 2, 3)
    //[0,2,4]
    val b = Array(3, { i -> (i * 2) })
    //读取数组内容
    println(a[0])    // 输出结果：1
    println(b[1])    // 输出结果：2
}
```


<a id="org3f20087"></a>

### map使用

```kotlin
var map = TreeMap<String, String>()
map["l1"] = "good"
map["l2"] = "well"
println(map["l1"])
```


<a id="org5b4b45d"></a>

### 递归

```kotlin
println(fact(5))
println(fact(3))
println(fact(10))
println(fact(100))
println(factBig(BigInteger("30")))

fun fact(n:Int):Int = if(n == 1) 1 else n*fact(n-1)
fun factBig(n:BigInteger):BigInteger = if (n==BigInteger.ONE) BigInteger.ONE else n*factBig(n- BigInteger.ONE)
```


<a id="org3d2e4c1"></a>

### 尾递归优化

```kotlin
fun ollAdd(n:Int):Int = if(n==1) 1 else n+ollAdd(n-1)
tailrec fun ollAdd2(n:Int,res:Int):Int = if (n ==1) 1 else ollAdd2(n-1,res+n)
tailrec fun ollAdd3(n:Int,res:Int):Int {
    println(res+n)
    if (n ==1) {
	return 1
    } else {
	return ollAdd3(n-1,res+n)
    }
}
```


<a id="org93c6e61"></a>

## 异常


<a id="orgb58e2d5"></a>

### Exception 与 Java 类似

```kotlin
try {
    var num = "1a"!!.toInt()
    println(num)
} catch (e: Exception) {
    println("err")
}
```


<a id="org9941ab7"></a>

## 面向对象


<a id="org7e6b4c9"></a>

### 抽象类

抽象是面向对象编程的特征之一，类本身，或类中的部分成员，都可以声明为abstract的。抽象成员在类中不存在具体的实现。

注意：无需对抽象类或抽象成员标注open注解。

```kotlin
abstract class Food {
    abstract fun f()
}
```


<a id="org88f17eb"></a>

### 类的初始化代码块

-   主构造器中不能包含任何代码，初始化代码可以放在初始化代码段中，初始化代码段使用 init 关键字作为前缀。

```kotlin
class Person constructor(firstName: String) {
    init {
	println("FirstName is $firstName")
    }
}
```


<a id="orgdeedfdd"></a>

### 内部类

-   内部类使用 inner 关键字来表示。
-   内部类会带有一个对外部类的对象的引用，所以内部类可以访问外部类成员属性和成员函数。

```kotlin
class Outer {
    private val bar: Int = 1
    var v = "成员属性"
    /**嵌套内部类**/
    inner class Inner {
	fun foo() = bar  // 访问外部类成员
	fun innerTest() {
	    var o = this@Outer //获取外部类的成员变量
	    println("内部类可以引用外部类的成员，例如：" + o.v)
	}
    }
}
```


<a id="org38f2301"></a>

### 类的修饰符

-   类的修饰符包括 classModifier 和<sub>accessModifier</sub>\_:

```kotlin
classModifier: 类属性修饰符，标示类本身特性。

abstract    // 抽象类  
final       // 类不可继承，默认属性
enum        // 枚举类
open        // 类可继承，类默认是final的
annotation  // 注解类
accessModifier: 访问权限修饰符

private    // 仅在同一个文件中可见
protected  // 同一个文件中或子类可见
public     // 所有调用的地方都可见
internal   // 同一个模块中可见
```


<a id="org93f2bf8"></a>

### Any 类

-   Kotlin 中所有类都继承该 Any 类，它是所有类的超类，对于没有超类型声明的类是默认超类：

```kotlin
class Example // 从 Any 隐式继承

//  Any 默认提供了三个函数：
equals()
hashCode()
toString()

//注意：Any 不是 java.lang.Object。
```


<a id="org4a3be39"></a>

### 泛型

泛型，即 "参数化类型"，将类型参数化，可以用在类，接口，方法上。

-   声明一个泛型类:

```kotlin
class Box<T>(t: T) {
    var value = t
}
// 创建类的实例时我们需要指定类型参数:

val box: Box<Int> = Box<Int>(1)
// 或者
val box = Box(1) // 编译器会进行类型推断，1 类型 Int，所以编译器知道我们说的是 Box<Int>。

```

-   **注** 定义泛型类型变量，可以完整地写明类型参数，如果编译器可以自动推定类型参数，也可以省略类型参数。
    
    Kotlin 泛型函数的声明与 Java 相同，类型参数要放在函数名的前面：

```kotlin
fun <T> boxIn(value: T) = Box(value)

// 以下都是合法语句
val box4 = boxIn<Int>(1)
val box5 = boxIn(1)     // 编译器会进行类型推断
```


<a id="org683e433"></a>

### super的泛型使用

-   如果有多个相同的方法（继承或者实现自其他类，如A、B类），则必须要重写该方法，使用super范型去选择性地调用父类的实现。

```kotlin
open class A {
    open fun f () { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } //接口的成员变量默认是 open 的
    fun b() { print("b") }
}

class C() : A() , B{
    override fun f() {
	super<A>.f()//调用 A.f()
	super<B>.f()//调用 B.f()
    }
}

fun main(args: Array<String>) {
    val c =  C()
    c.f();
}
```

C 继承自 a() 或 b(), C 不仅可以从 A 或则 B 中继承函数，而且 C 可以继承 A()、B() 中共有的函数。此时该函数在中只有一个实现，为了消除歧义，该函数必须调用A()和B()中该函数的实现，并提供自己的实现。

-   输出结果为: AB


<a id="org2b6d0a1"></a>

### 接口

Kotlin 接口与 Java 8 类似，使用 interface 关键字定义接口，允许方法有默认实现：

```kotlin
interface MyInterface {
    fun foo() {  //已实现
		 // 可选的方法体
		 println("foo")
    }
}

//一个类或者对象可以实现一个或多个接口。
class Child : MyInterface {
    override fun bar() {
	// 方法体
    }
}
```


<a id="orgead3f6d"></a>

### 扩展

-   Kotlin 可以对一个类的属性和方法进行扩展，且不需要继承或使用 Decorator 模式。
-   扩展是一种静态行为，对被扩展的类代码本身不会造成任何影响。

1.  扩展函数

    -   扩展函数可以在已有类中添加新的方法，不会对原类做修改，扩展函数定义形式：
    -   若扩展函数和成员函数一致，则使用该函数时，会优先使用成员函数。
    
    ```kotlin
    class C {
        fun foo() { println("成员函数") }
    }
    
    fun C.foo() { println("扩展函数") }
    
    fun main(arg:Array<String>){
        var c = C()
        c.foo()
    }
    ```
    
    -   实例执行输出结果为：成员函数


<a id="org48998d5"></a>

### 单例模式

-   Kotlin 使用 object 关键字来声明一个对象。
-   Kotlin 中我们可以方便的通过对象声明来获得一个单例。


<a id="orgb1e3854"></a>

### 接口代理,委托

-   通过关键字 by 指定委托方


<a id="org032d245"></a>

### Sealed Class

子类类型有限的类型


<a id="orga12a60b"></a>

### 枚举类

-   枚举类最基本的用法是实现一个类型安全的枚举。
-   枚举常量用逗号分隔,每个枚举常量都是一个对象。

```kotlin
fun main() {
    println(Week.星期一)
    println(Week.星期一.ordinal)
}

enum class Week{
    星期一,星期二
}

enum class Color{
    RED,BLACK,BLUE,GREEN,WHITE
}

//每一个枚举都是枚举类的实例，它们可以被初始化：
enum class Color(val rgb: Int) {
    RED(0xFF0000),GREEN(0x00FF00),BLUE(0x0000FF)
}

//默认名称为枚举字符名，值从0开始。若需要指定值，则可以使用其构造函数：
enum class Shape(value:Int){
    circle(2),rectangle(200)
}
```
