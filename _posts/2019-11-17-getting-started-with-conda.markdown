---
layout: post
title: "Getting started with conda"
date: 2019-11-17
tag: conda anaconda python virtualenv 
---
- [conda](#orgab6e9bb)
  - [使用之前](#orgf4bd99f)
  - [基本使用](#orgcd0b0e0)
  - [conda 安装特定版本](#orga829a49)
  - [使用 Conda 管理 Python 虚拟环境](#org7bd04d0)
  - [创建虚拟环境](#orgd3d1cb2)
  - [激活和退出虚拟环境](#orgede8e0f)
  - [删除虚拟环境](#org9e59a26)
  - [列出安装的虚拟环境](#org9a5b0a9)
  - [更好的安装方式](#orgd0bc46e)
  - [复制一个虚拟环境](#org0383c59)
  - [导出一个环境](#org93b7f56)
  - [requirement.txt](#org3a516a9)
  - [参考](#org7a18e3f)


<a id="orgab6e9bb"></a>

# conda

Conda 是一个开源的软件包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。


<a id="orgf4bd99f"></a>

## 使用之前

-   添加清华的镜像源 详细步骤看这里 <https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/>
-   总之是在用户目录下的 .condarc 文件里添加 镜像源的频道。

-   要添加官方的源

```shell
conda config --add channels bioconda
conda config --add channels conda-forge
```

-   显示安装的频道

```shell
conda config --set show_channel_urls yes 
```

-   查看已经添加的channels

```shell
conda config --get channels
```


<a id="orgcd0b0e0"></a>

## 基本使用

```shell
conda search gatk # 搜索软件包
conda list # 查看已安装软件
conda update gatk # 更新指定软件
conda remove gatk # 卸载指定软件
```


<a id="orga829a49"></a>

## conda 安装特定版本

如需要安装特定的版本:

```shell
conda install gatk=3.7
conda install 软件名=版本号
```

这时conda会先卸载已安装版本，然后重新安装指定版本。

```shell
conda install tensorflow=1.15
```


<a id="org7bd04d0"></a>

## 使用 Conda 管理 Python 虚拟环境

Python 中的虚拟环境是将一个项目所依赖的库文件独立存放在某个地方的工具。 > 可选， virtualenv 来创建虚拟环境

```shell
# conda env --help
conda-env -h
```


<a id="orgd3d1cb2"></a>

## 创建虚拟环境

```shell
conda create --name py-35
# conda-env create -n py-35 # 需要指定environment.yaml 文件
```


<a id="orgede8e0f"></a>

## 激活和退出虚拟环境

```shell
# source activate py-35 
conda activate py-35
# source deactivate
```

在这个虚拟环境下，我们所用的python 环境（解释器）是默认的，但是库文件已经是只是在这个虚拟环境里的了。

```shell
conda install click
```


<a id="org9e59a26"></a>

## 删除虚拟环境

```shell
conda remove --name py-35 --all
conda remove -n py-35 --all
# conda-env remove --name py-53
```


<a id="org9a5b0a9"></a>

## 列出安装的虚拟环境

```shell
conda env list
# conda-env list
```


<a id="orgd0bc46e"></a>

## 更好的安装方式

-   在建立虚拟环境时直接指定 Python 的版本

```shell
# conda-env create -n test-py35 python=3.5
conda create -n test-py35 python=3.5
```

```shell
conda-env list
```


<a id="org0383c59"></a>

## 复制一个虚拟环境

```shell
conda create --name test-py352 --clone test-py35
```


<a id="org93b7f56"></a>

## 导出一个环境

```shell
source activate test-py35
conda env export > environment.yaml
# conda-env create -f environment.yml  # 复制出虚拟环境
```


<a id="org3a516a9"></a>

## requirement.txt

我们可以通过 freeze 环境包前当前的状态来保证环境的一致性

```shell
pip freeze > requirement.txt
# pip install -r requirement.txt
```


<a id="org7a18e3f"></a>

## 参考

[conda的安装与使用（2019-6-28更新）](https://www.jianshu.com/p/edaa744ea47d) 《python 机器学习实战》
