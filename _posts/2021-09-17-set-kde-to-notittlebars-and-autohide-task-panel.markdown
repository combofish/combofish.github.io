---
layout: post
title: "Setting KDE to No Titlebars and Auto Hide task panel"
date: "2021-09-17 21：17：00"
tag: kde plasma
---
- [KDE 桌面环境配置](#orgdb0564f)
  - [取消标题栏](#org43b61ce)
  - [自动隐藏任务栏](#org2a06ff2)


<a id="orgdb0564f"></a>

# KDE 桌面环境配置

Ubuntu 自带的桌面总有标题栏，以及左侧的 Dock 栏。这样造成了屏幕使用效率低。本文记录了 在 plasma 上设置取消标题栏和自动隐藏任务栏的设置。
+ 总体效果如下：
![OverView](/images/Kde-Set-To-NoTittleBars-AutoHide-TaskPanel/OverView.png)

<a id="org43b61ce"></a>

## 取消标题栏

找到 System setting -> Workspace -> Windows Management -> Windows Rules.

-   选择新建，在 description 中添加名字，如 NoTitleBars。
-   在 Windows Clas 中，选择 Regular Expression, 内容填写 .\* 。
-   在 Appearance & Fixes 中选择 “No titlebar and frame”Force Yes。

![在这里设置取消标题栏](/images/Kde-Set-To-NoTittleBars-AutoHide-TaskPanel/NoTittleBarsSetting.png)

参考链接：[how-to-hide-title-bars-in-kde-plasma-5](https://medium.com/@CodyReichert/how-to-hide-title-bars-in-kde-plasma-5-348e0df4087f)

<a id="org2a06ff2"></a>

## 自动隐藏任务栏

在任务栏右键，选择 Edit panel, 找到 More Setting, 在 Visibility 中选择 Auto Hide。
![在这里设置自动隐藏任务栏](/images/Kde-Set-To-NoTittleBars-AutoHide-TaskPanel/AutoHideTaskPanel.png)
