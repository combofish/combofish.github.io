---
layout: post
title:  "How to use Emacs org-mode"
date:   2019-07-14 22:55:38 +0800
categories: emacs org-mode
---

# Table of Contents

1.  [org-mode](#org0668dbe)
2.  [用大纲来组织文件结构](#orgb8096a3)
    1.  [标题跳转](#org290c33d)
    2.  [插入及编辑](#orgfded5ff)
    3.  [显示方式](#org5939021)
    4.  [超链接](#org01fb103)
        1.  [链接类型](#orga1cc09e)
        2.  [文件链接](#orge68b15d)
        3.  [编辑链接](#org38b2b81)
    5.  [字体](#org70604f4)
    6.  [表格](#orgde0bbff)
    7.  [段落](#org8205f7b)
    8.  [列表](#orgfa1f3e3)
3.  [进阶](#org4950f8d)
    1.  [标签定义](#org815e226)
        1.  [标签搜索](#orgbc7d4e8)
    2.  [导出和发布](#org9f8d8d7)
        1.  [元数据设置](#org27fbb28)



<a id="org0668dbe"></a>

# org-mode

org-mode 是emacs的亮点之一，不过一直没有详细的学过怎么使用。在此整理了一些相关的知识点，供大家学习。
[整理原文](https://www.jianshu.com/p/78ef59327e2d)


<a id="orgb8096a3"></a>

# 用大纲来组织文件结构

org使用\*号来列提纲的标题。使用\* 号标记，位于行首，之后跟一个空格再输入标题。与md的#类似。
最多支持10及的标题。

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">S-Tab</td>
<td class="org-left">所在标题展开</td>
</tr>


<tr>
<td class="org-left">Tab</td>
<td class="org-left">光标所在标题展开</td>
</tr>
</tbody>
</table>


<a id="org290c33d"></a>

## 标题跳转

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">C-c C-n/p</td>
<td class="org-left">上/下标题</td>
</tr>


<tr>
<td class="org-left">C-c C-f/b</td>
<td class="org-left">上/下标题，仅同一标题</td>
</tr>


<tr>
<td class="org-left">C-c C-j</td>
<td class="org-left">跳转到</td>
</tr>


<tr>
<td class="org-left">C-c C-u</td>
<td class="org-left">跳转到上一级标题</td>
</tr>
</tbody>
</table>


<a id="orgfded5ff"></a>

## 插入及编辑

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">M-Ret</td>
<td class="org-left">插入同级标题</td>
</tr>


<tr>
<td class="org-left">M-S-Ret</td>
<td class="org-left">插入同一级的todo标题</td>
</tr>


<tr>
<td class="org-left">M-Left/Right</td>
<td class="org-left">将当前标题升/降级</td>
</tr>


<tr>
<td class="org-left">M-S-Left/Right</td>
<td class="org-left">将当前标题及子树升级/降级</td>
</tr>


<tr>
<td class="org-left">M-S-Up/Down</td>
<td class="org-left">将当前标题及子树上/下移</td>
</tr>


<tr>
<td class="org-left">C-c \*</td>
<td class="org-left">将本行设置为标题</td>
</tr>


<tr>
<td class="org-left">C-c C-w</td>
<td class="org-left">将子树及区域移动至另一标题处</td>
</tr>


<tr>
<td class="org-left">C-x n s/w</td>
<td class="org-left">只显示当前标题及子树</td>
</tr>


<tr>
<td class="org-left">C-c C-x h</td>
<td class="org-left">查询</td>
</tr>
</tbody>
</table>


<a id="org5939021"></a>

## 显示方式

-   使用M-x org-indent-mode 切换带缩进的显示方式。
-   如果想在打开时进入该模式，在文档头部增加#+STARTUP：indent
-   如果希望所有org文件都以此种方式打开，在.emacs中增加（setq org-startup-indented t）


<a id="org01fb103"></a>

## 超链接

符合超链接的规则的内容，自动视为超链接
例如：
<https://www.baidu.com/> baidu
<home>


<a id="orga1cc09e"></a>

### 链接类型

Possible completions are:
bbdb:   bibtex:     docview:    doi:    elisp:  file+emacs:
file+sys:   file:   ftp:    gnus:   http:   https:
info:   irc:    mailto:     message:    mhe:    news:
rmail:  shell:


<a id="orge68b15d"></a>

### 文件链接

-   未整理


<a id="org38b2b81"></a>

### 编辑链接

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">C-c C-l</td>
<td class="org-left">编辑链接</td>
</tr>


<tr>
<td class="org-left">C-c C-o</td>
<td class="org-left">打开链接</td>
</tr>
</tbody>
</table>


<a id="org70604f4"></a>

## 字体

-   **粗体**
-   *斜体*
-   <del>删除线</del>
-   <span class="underline">下划线</span>


<a id="orgde0bbff"></a>

## 表格

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">C-c</td>
<td class="org-left">创建表格</td>
</tr>


<tr>
<td class="org-left">C-c C-c</td>
<td class="org-left">重新调整表格缩进</td>
</tr>


<tr>
<td class="org-left">Tab</td>
<td class="org-left">移动至下一个区域</td>
</tr>


<tr>
<td class="org-left">S-Tab</td>
<td class="org-left">移动至上一个区域</td>
</tr>


<tr>
<td class="org-left">S-Ret</td>
<td class="org-left">移动至下一行</td>
</tr>


<tr>
<td class="org-left">M-Left/Right</td>
<td class="org-left">移动列</td>
</tr>


<tr>
<td class="org-left">M-Up/Down</td>
<td class="org-left">移动行</td>
</tr>


<tr>
<td class="org-left">M-S-Left/Right</td>
<td class="org-left">删除/插入行</td>
</tr>


<tr>
<td class="org-left">M-S-Up/Dowm</td>
<td class="org-left">删除/插入列</td>
</tr>


<tr>
<td class="org-left">C-c -</td>
<td class="org-left">插入水平分割线</td>
</tr>


<tr>
<td class="org-left">C-c Ret</td>
<td class="org-left">插入水平分割线并跳到下一行</td>
</tr>


<tr>
<td class="org-left">C-c ^</td>
<td class="org-left">根据当前列排序</td>
</tr>
</tbody>
</table>


<a id="org8205f7b"></a>

## 段落

对于单个回车的文本，org认为是在同一个段落，导出时会转化成不同行的同一段落。如果要起新的段落，请敲空白行。


<a id="orgfa1f3e3"></a>

## 列表

-   无序列表，以 - + \* 开头
-   有序列表， 以1.或1）开头
-   描述列表， 以::将描述隔开

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">Tab</td>
<td class="org-left">折叠列表项</td>
</tr>


<tr>
<td class="org-left">M-Ret</td>
<td class="org-left">插入项</td>
</tr>


<tr>
<td class="org-left">M-S-Ret</td>
<td class="org-left">插入带复选框的项</td>
</tr>


<tr>
<td class="org-left">M-S-Up/Down</td>
<td class="org-left">移动列表项</td>
</tr>


<tr>
<td class="org-left">M-left/Right</td>
<td class="org-left">升/降级列表项，不包括子项</td>
</tr>


<tr>
<td class="org-left">M-S-left/Right</td>
<td class="org-left">升/降级列表项，包括子项</td>
</tr>


<tr>
<td class="org-left">C-c C-c</td>
<td class="org-left">改变复选框状态</td>
</tr>


<tr>
<td class="org-left">C-c -</td>
<td class="org-left">更换列表标记</td>
</tr>
</tbody>
</table>


<a id="org4950f8d"></a>

# 进阶


<a id="org815e226"></a>

## 标签定义

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">C-c C-q</td>
<td class="org-left">创建标签</td>
</tr>


<tr>
<td class="org-left">C-c C-c</td>
<td class="org-left">在标题上使用，创建标签</td>
</tr>
</tbody>
</table>

在配置文件中使用 org-tag-alist 定义标签


<a id="orgbc7d4e8"></a>

### 标签搜索

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">快捷键</th>
<th scope="col" class="org-left">描述</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">C-c a m</td>
<td class="org-left">按标签搜索多个文件，需要把文件加入到全局agenda</td>
</tr>


<tr>
<td class="org-left">C-c / m 或者 C-c \\</td>
<td class="org-left">标准检索，按照tag进行检索</td>
</tr>
</tbody>
</table>


<a id="org9f8d8d7"></a>

## 导出和发布

-   C-c C-e 导出
-   M-x org-md-export-as-markdown


<a id="org27fbb28"></a>

### 元数据设置

-   \#+TITLE：
-   \#+AUTHOR：
-   \#+EMAIL:
-   \#+KEYWORDS：

如果遇到段落导出无法换行，在开头加上#OPTIONS: \n:t



