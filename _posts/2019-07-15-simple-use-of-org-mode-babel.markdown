---
layout: post
title:  "Simple use of Emacs org-mode babel!"
date:   2019-07-15 16:24:22 +0800
categories: emacs org-mode
---

# Table of Contents

1.  [babel](#org3091edc)
        1.  [ruby](#orgc2df58e)
        2.  [javascript](#org391d42b)
2.  [捕获代码执行结果](#org9951923)
    1.  [:results value (functional mode)](#org4a8bd9c)
    2.  [:results output (scripting mode)](#orgb561cf9)
    3.  [基于会话的代码块](#orgd08cc00)
    4.  [代码块的传参数](#org42db4de)
    5.  [内联(In-line)的代码块](#org4aa3f6c)

前言，最近多了解了下org-mode 的babel, 整理了如下文章，原文参见
[原文](https://brantou.github.io/2017/04/01/babel-intro/)


<a id="org3091edc"></a>

# babel

Bable 可让许多不同的语言工作一起。 编程语言生活在自然语言的 Org-mode 文档的代码块之中。 一个数据片段可从一个表格传递给一个 Pythoh 代码块，然后可能再转移到一个 R 代码块， 最终以数据块被嵌入段落的中间而终结，或者通过 gnuplot 代码块生成图片嵌入在文档中。

通过扩展 Org-mode, 使其具有编辑,导出和执行源代码的功能, Babel 将 Org-mode 变成了文学编程和可重复性研究的工具。

Babel 通过提供以下特性来增强 Org-mode对代码代码块的支持: Babel augments Org-mode support for code blocks by providing:

-   代码块的交互和执行结果导出
-   代码块可像函数一样可参数化，引用其他代码块，可被远程调用
-   拼接，导出源代码到文件支持文学编程。


<a id="orgc2df58e"></a>

### ruby

    require 'date'
    "This file was last evaluated on #{Date.today}"


<a id="org391d42b"></a>

### javascript

    console.log("start")


<a id="org9951923"></a>

# 捕获代码执行结果

Babel 提供了两种根本不同的模式来捕获代码执行的结果： **functional mode** 和 **scripting mode** 。 模式的选择可以通过配置 :results 头参数来指定。


<a id="org4a8bd9c"></a>

## :results value (functional mode)

代码执行的结果是代码块中最后一个语句的值。 在 functional mode 下，代码块是具有返回值的函数。 
一个代码块的返回值可以用作另一代码块的输入，即使是不同语言的输入。 这样的话，Babel成为一种元编程语言。 
如果块返回表格数据（某种类型的向量，数组或表），那么将可以作为 Org-mode 的表格保存在缓冲区中。 
functional mode 是默认设置。

    import time
    print("Hello, today's date is %s" % time.ctime())
    print("Two plus two is")
    return 2 + 2

请注意，在 functional mode 下，输出只由最后一个语句返回，没有其他情况。

    4


<a id="orgb561cf9"></a>

## :results output (scripting mode)

在 scripting mode 中，Babel捕获代码块的文本输出并将其放置在 Org-mode 的缓冲区中。 
它被称为 scripting mode ，因为代码块包含一系列命令，并返回每个命令的输出。 
与功能模式不同，代码块本身除了其包含的命令的输出之外没有返回值。2

观察以下使用 scripting mode 执行代码块的结果。

    import time
    print("Hello, today's date is %s" % time.ctime())
    print('Two plus two is')
    2 + 2

在这里， scripting mode 返回了python写到 stdout 的文本。 
因为代码块不包含最后一个语句 (2 + 2) 的 print() 语句，所以结果中不会出现4。


<a id="orgd08cc00"></a>

## 基于会话的代码块

对于某些语言，例如Python，R，ruby和shell，可以在Emacs中运行一个不完备的交互式会话进程。 
这意味着创建了一个不同源代码块之间共享数据对象的持久化环境。 Babel 支持使用 :session 头参数来指定代码块运行于特定会话中。 
如果头参数被赋予一个值，那么该参数将被用作会话的名称。 因此，可以并发的在不同的会话中运行同一语言的不同代码块。

基于特定会话的代码块对于原型设计和调试特别有用。 函数 org-babel-pop-to-session 可用于切换会话缓冲区。

一旦代码块编辑完成，通常最好在会话之外执行它，因为这样它执行的环境将是确定的。

With R, the session will be under the control of Emacs Speaks Statistics as usual, 
and the full power of ESS is thus still available, both in the R session, and when switching to the R code edit buffer with ​C-c '​

    a = "hello"

    print(a)


<a id="org42db4de"></a>

## 代码块的传参数

    return x * x

    36


<a id="org4aa3f6c"></a>

## 内联(In-line)的代码块

可使用以下语法内联(In-line)的执行代码：

Without header args: or with header args: ,
for example "`100`", where x is a variable existing in the
python session.
代码如下:

`hello world`


