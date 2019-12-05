---
layout: post
title: "Wget - bulk download"
date: 2019-12-05
tag: wget bulk download
---

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. wget</a>
<ul>
<li><a href="#sec-1-1">1.1. windows proxy set</a></li>
</ul>
</li>
</ul>
</div>
</div>

# wget<a id="sec-1" name="sec-1"></a>

    wget -t5 --timeout=3 -c -i urls.txt

-   t5 &#x2013;tries=数字 设置重试次数为 <数字> (0 代表无限制)
-   timeout=3 设置超时时间
-   c 断点续传
-   i 从文件中读取 urls

## windows proxy set<a id="sec-1-1" name="sec-1-1"></a>

在终端下使用代理，需要在cmd 下输入：

    set http_proxy=http://127.0.0.1:1189
    set https_proxy=http://127.0.0.1:1189

上面的作用是设置环境变量，关闭CMD 窗口后，就会失效。
