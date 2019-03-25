---
layout: post
title:  "Simple use of linux command"
date:   2019-03-25
tag: linux command
---
# stat
## 获取特定文件或文件系统的详细状态。  
可以看到修改（modify）和变更（change）以及访问（access）,还有文件的许可权限等。  
## 也可以查看目录的详细信息
可以看到关于链接（links）数量的额外详情。
## 查看一个文件系统的详细信息
加上 -f 参数，可以显示文件系统的状态。  

# strace
## 监视进程和Linux内核之间的交互。 
常用参数:  
+ -c 将统计结果简洁的显示出来。
+ -t 在每个输出行显示时间戳。
+ -e 显示某个特定的系统调用，如 -e open 
+ -o 将输出保存到内存中，如 -o output.txt 
+ -p 对正在运行的进程使用 strace 工具 



