---
layout: post
title: "Linux 环境变量介绍"
date: 2019-08-12
tag: Emacs org-mode org-mind-map 
---

- [Linux 环境变量](#orgf0a8f4b)
  - [分类](#org144424b)
    - [全局变量](#org3d1ecfd)
    - [局部变量](#orgf889f18)
  - [设置环境变量](#org7a24e42)
    - [设置局部环境变量](#orgff6dd9d)
    - [设置全局环境变量](#org4446ca5)
    - [删除环境变量](#org77a9175)
  - [默认环境变量](#org0699ca0)
    - [PATH 环境变量](#org6e5f868)
  - [系统环境变量](#org1d69079)
    - [登陆 shell](#orga9dce43)
    - [非登陆的交互式 shell](#org007bdf6)
    - [作为脚本运行的非交互式 shell](#orgecc8c81)
  - [可变数组](#org5e0cd20)
  - [命令别名 alias](#org9ad7094)


<a id="orgf0a8f4b"></a>

# Linux 环境变量

-   environment variables
-   用于存储有关shell会话和系统环境的信息


<a id="org144424b"></a>

## 分类


<a id="org3d1ecfd"></a>

### 全局变量

-   对shell会话可见
-   对所有shell创建的子进程可见，（对那些子进程中需要获取父进程信息的程序来说非常有用）
-   系统环境变量一律使用大写来区分与普通用户的环境变量

查看环境变量

```shell
printenv | wc -l 
```

-   新启动shell 后，这些环境依然是可见的


<a id="orgf889f18"></a>

### 局部变量

-   只对创建他们的 shell 可见, 即只能在定义他们的进程中使用
-   set 命令 显示为某个特定进程设置的所有环境变量，包括全局变量

```shell
set | wc -l 
```


<a id="org7a24e42"></a>

## 设置环境变量


<a id="orgff6dd9d"></a>

### 设置局部环境变量

-   在环境变量名称，等号和值之间没有空格
-   变量赋值给一个包含空格的字符串时，要有引号
-   默认子shell 中不可用

```shell
test='two string'
echo $test
zsh
echo $test
exit
echo $test

```


<a id="org4446ca5"></a>

### 设置全局环境变量

-   export 导出到全局环境
-   导出环境变量时不需要用$ 符号

```shell
test='two string'
export test
echo $test
zsh
echo $test
exit
echo $test

```


<a id="org77a9175"></a>

### 删除环境变量

-   unset 删除环境变量
-   删除环境变量时不需要用$ 符号

```shell
test='two string'
echo $test
export test
zsh
echo $test
unset test
echo $test
exit
echo $test
unset test
echo $test

```


<a id="org0699ca0"></a>

## 默认环境变量

-   默认情况下bash shell 会用一些特定的环境变量来定义系统环境


<a id="org6e5f868"></a>

### PATH 环境变量

-   定义了命令行输入命令的搜索路径
-   目录之间用冒号分割

1.  添加单点符

    ```shell
    echo $PATH
    PATH=$PATH:.
    echo $PATH
    ```


<a id="org1d69079"></a>

## 系统环境变量

-   Linux System用环境变量在程序和脚本中标识自己
-   启动一个 shell 时，默认在几个启动文件中查找命令
-   bash 检查的启动文件取决与启动 shell的方式


<a id="orga9dce43"></a>

### 登陆 shell

处理文件的次序

-   /etc/profile 系统上每个用户登陆时都会启动这个文件
-   $HOME/.bash<sub>profile</sub>
-   $HOME/.bash<sub>login</sub>
-   $HOME/.profile

$HOME 下的启动文件是提供一个用户专属的启动文件来定义用户专有的环境变量，一般 Linux 发行版只用这三个中的一个

```shell
head $HOME/.bash_profile 

```


<a id="org007bdf6"></a>

### 非登陆的交互式 shell

-   在用户目录下检查 .bashrc 是否存在
    -   作用：
        -   查看 /etc 目录下的共用 bashrc 文件，
        -   为用户提供一个自己的命令别名
-   交互式 shell 的启动文件只会在有新的交互式 shell 启动时才运行，因此任何的子 shell 都会自动执行这个交互式 shell 的启动文件
-   /etc/bashrc 也会执行位于 /etc/profile.d 下的应用专属的启动文件


<a id="orgecc8c81"></a>

### 作为脚本运行的非交互式 shell

系统执行 shell 脚本时所用的 shell

-   BASH<sub>ENV</sub> 环境变量


<a id="org5e0cd20"></a>

## 可变数组

-   环境变量可以作为数组使用
    -   数组的索引从0开始（ **注** 在这个环境下索引从1开始, 环境 /bin/zsh ）
    -   unset 可删除数组，也可删除指定索引

```shell
my_env_var=(one two three four five six)
echo $my_env_var
echo $my_env_var[2]
echo $my_env_var[0]
echo ${my_env_var[3]}
echo $my_env_var[*]
my_env_var[4]=ten
echo ${my_env_var[*]}
```

```shell
my_env_var=(1 2 3 4 5)
echo ${my_env_var[*]}
# unset my_env_var[2]
echo $my_env_var[*]
echo ${my_env_var[2]}
echo ${my_env_var[3]}
unset my_env_var
echo $my_env_var[*]
```

```shell
echo $SHELL
# /bin/zsh
```


<a id="org9ad7094"></a>

## 命令别名 alias

-   命令别名允许为通用命令和参数一起创建一个别名
-   这样能通过最少的键来使用命令了

```shell
alias

```

-   相当于局部变量，通常只在定义他们的shell进程中有效

```shell
alias ll='ls -la --color=auto'
# ll
alias | grep ll
sh
# ll
alias | grep ll
exit 
```
