---
layout: post
title: "Macro-to deal with repetitive operations (based on Emacs)"
date: "2021-05-14 17：10：10"
tag: macro Emacs editor repetitive-operations
---
- [应用场景](#org3198c9a)
- [应用举例一：重复性代码](#org3774b9c)
- [应用举例二：重复性命令](#org4542e75)
- [使用步骤](#org61dd93b)
- [保存宏](#orgafc8a58)
- [参考链接](#org52bbfd9)

键盘宏：是把一组按键盘的操作定义下来作为宏。这个宏可以保存下来，一遍一遍执行。


<a id="org3198c9a"></a>

# 应用场景

**重复性操作** 不管是操作的对象是单行还是多行，是单个文件还是多个文件，也不管操作的是不是文本还是输入命令，只要它们有相似性。就可以用宏来简化操作。


<a id="org3774b9c"></a>

## 应用举例一：重复性代码

-   Android 初始化一个 UI 控件

```kotlin
private lateinit var textView1: TextView

private fun initView() {
    textView1 = findViewById(R.id.textView1)
}
```

在 Android 开发中，需要初始化一些 UI 控件，当控件一多，十到二十几个的时候，一个个添加，就有些繁琐了。不过鉴于初始化代码都一样，我们可以录制一个宏。

-   首先准备所需的 控件名字

```kotlin
textView1,textView2,textView3,textView4,textView5,textView6,textView7,textView8,textView9
```

-   录制变量声明和初始化定义的语句

```elisp
(fset 'init-ui-view
   [?\M-f ?\C-d return up ?\C-k ?p ?r ?i ?v ?a ?t ?e ?  ?l ?a ?t ?e ?i ?n ?i ?t ?  ?v ?a ?r ?  ?\C-y ?: ?T ?e ?x ?t ?V ?i ?e ?w return backspace return backspace ?\M-\} return ?\C-y ?  ?= ?  ?f ?i ?n ?d ?V ?i ?e ?w ?B ?y ?I ?d ?\( ?R ?. ?i ?d ?. ?\C-y ?\) ?\M-< ?\C-n])
```

-   操作之后

    ```kotlin
    private lateinit var textView1:TextView
    private lateinit var textView2:TextView
    private lateinit var textView3:TextView
    private lateinit var textView4:TextView
    private lateinit var textView5:TextView
    private lateinit var textView6:TextView
    private lateinit var textView7:TextView
    private lateinit var textView8:TextView
    private lateinit var textView9:TextView
    
    textView8 = findViewById(R.id.textView8)
    textView7 = findViewById(R.id.textView7)
    textView6 = findViewById(R.id.textView6)
    textView5 = findViewById(R.id.textView5)
    textView4 = findViewById(R.id.textView4)
    textView3 = findViewById(R.id.textView3)
    textView2 = findViewById(R.id.textView2)
    textView1 = findViewById(R.id.textView1)
    textView9 = findViewById(R.id.textView9)
    ```


<a id="org4542e75"></a>

## 应用举例二：重复性命令

在编辑一些文本之后，经常需要导出另一种格式。使用导出命令，它又会让我选择格式。没错，就是2步。

-   不过用多了，还是觉的繁琐。
-   上宏，简化成一步。


<a id="org61dd93b"></a>

# 使用步骤

| 快捷键 | 实际调用的函数                                   |
|----- |------------------------------------------------ |
| F3    | kmacro-start-macro-or-insert-counter ARG         |
| F4    | kmacro-end-or-call-macro ARG &optional NO-REPEAT |
| C-x ( | kmacro-start-macro ARG                           |
| C-x ) | kmacro-end-macro ARG                             |


<a id="orgafc8a58"></a>

## 保存宏

```sh
M+x name-last-kbd-macro # 为最后录制的一个宏命名
M+x insert-kbd-macro # 按名字插入宏
```

把自制的宏保存在自己的配置文件中，就可以一直使用了。


<a id="org52bbfd9"></a>

# 参考链接

[emacs 宏操作“神器”](https://blog.csdn.net/lvye1221/article/details/9236399) 
[Emacs录制宏](https://blog.csdn.net/wild46cat/article/details/50823703)
[Emacs中宏的基本使用方法](https://www.cnblogs.com/heart-runner/archive/2012/01/06/2314860.html)
