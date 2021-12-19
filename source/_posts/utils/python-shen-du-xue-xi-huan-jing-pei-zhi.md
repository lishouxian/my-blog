---
title: 'Python深度学习环境配置'
date: 2019-12-02 20:01:57
tags: [深度学习]
categories: utils
published: true
hideInList: false
feature: 
isTop: false
---
# 软件安装

## Anaconda

[Anaconda](https://www.anaconda.com/)是一款非常好用的python多环境包管理工具，用了这个你就不用再管系统中的python版本，系统中的环境变量啊等等，同时在这个软件里的任何操作均不会对系统内的python版本有任何影响，对于需要多python版本并存的深度学习者来说实在是居家必备。非常推荐安装。

<!-- more -->

## Pycharm

[Pycharm](https://www.jetbrains.com/pycharm)是一款非常好用的Python IDE，能够帮助我们在编写代码时提高效率。网上提供的有专业版和社区版即免费版之分。我所使用的是教育版，相比于社区版有更多优秀的功能。

# 环境配置

## python环境

在安装Anaconda时就已经安装了一个版本的python，其中Anaconda2019.2版本的python版本为3.7.4。同时就已经安装好了许多常用的库，例如科学计算库numpy,数据管理pandas,绘图matplotlib等等。在安装时按照默认安装即可。

## 多环节配置

在安装完Anaconda后就已经完成了base环境的安装，版本就是默认的版本，但是对于我们学习过程中会遇到许多不同的环境，包括不同版本的python和不同版本的包，此时建立多个不同的环境就是最优的解决办法。
下面介绍关于conda虚拟环境配置的一些命令：

```angular2
conda create -n env_name list_of_packages
conda create -n my_env numpy
conda create -n py3 python=3.3

source activate my_env
source deactivate (conda deactivate)

conda env export > environment.yaml (将环境中各个包的版本保存到environment.yaml)

根据环境文件复制环境：conda env create -f environment.yaml
列出所有的环境：conda env list
去除环境：conda env remove -n env_name
```

## 在Pycharm应用

为工程启用对应的环境。具体方式有两种，分别是在创立工程的时候就选择相对应的环境，第二种是在工程创立完成后更改或启用相对应的环境。两种方法都需要首先在pycharm中找到创建的环境。
![](https://i.postimg.cc/h41TVRCb/pycharm.png)
在pycharm中具体位置为file-setting-project-interpreter中，选择小齿轮-add后
![](https://i.postimg.cc/k5h6T70L/environment.png)
选择conda environment-existing environment后在interpreter中选择新环境的python.exe的路径即可。
多环境的配置就完成了。