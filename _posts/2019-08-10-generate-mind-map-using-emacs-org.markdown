---
layout: post
title: "用EMACS生成脑图(mind map)"
date: 2019-08-10
tag: Emacs org-mode org-mind-map 
---

- [绘制脑图](#org7da168c)
  - [安装](#org61adace)
    - [emacs 环境](#org3e3b55f)
    - [graphviz 环境](#org8e87aca)
    - [添加配置](#orgb681864)
  - [使用](#org84685f1)


<a id="org7da168c"></a>

# 绘制脑图

-   org-mind-map 是一个创建graphviz有向图的emacs包


<a id="org61adace"></a>

## 安装

org-mind-map 包的github [主页](https://github.com/theodorewiles/org-mind-map) <https://github.com/theodorewiles/org-mind-map>


<a id="org3e3b55f"></a>

### emacs 环境
+ 下载安装EMACS，自带org环境
+ 安装 org-mind-map 包

<a id="org8e87aca"></a>

### graphviz 环境

[ 官网](http://graphviz.org/) <http://graphviz.org/>


<a id="orgb681864"></a>

### 添加配置

-   添加到配置文件里
    
    ```elisp
    ;; This is an Emacs package that creates graphviz directed graphs from
    ;; the headings of an org file
    (use-package org-mind-map
      :init
      (require 'ox-org)
      :ensure t
      ;; Uncomment the below if 'ensure-system-packages` is installed
      ;;:ensure-system-package (gvgen . graphviz)
      :config
      (setq org-mind-map-engine "dot")       ; Default. Directed Graph
      ;; (setq org-mind-map-engine "neato")  ; Undirected Spring Graph
      ;; (setq org-mind-map-engine "twopi")  ; Radial Layout
      ;; (setq org-mind-map-engine "fdp")    ; Undirected Spring Force-Directed
      ;; (setq org-mind-map-engine "sfdp")   ; Multiscale version of fdp for the layout of large graphs
      ;; (setq org-mind-map-engine "twopi")  ; Radial layouts
      ;; (setq org-mind-map-engine "circo")  ; Circular Layout
      )
    ```


<a id="org84685f1"></a>

## 使用

-   在 .org 文档中写

```org
* example tree
** branch A 
** branch B
*** sub-branch 1
*** sub-branch 2 
*** sub-branch 3 
```

[xiaoguotu](/images/mind-map.png)
[效果](./example1.png)
