---
title: "Step-by-Step Guide: 在Ubuntu上安装Python3，为强大的编程提供支持"
date: 2023-06-16 22:56:53
img: https://wangxs020202.gitee.io/pbad/background/page_6.png
top: false
summary: 本文将介绍在Ubuntu上安装Python3的方法，以及更换pip源的方法。
categories: 笔记
tags:
  - Python
  - Ubuntu
  - 安装
  - 教程
  - 笔记
---

### 序幕

由于Ubuntu安装Python的方式与其他Linux系统有所不同，所以本文将介绍在Ubuntu上安装Python3的方法。

### 正文

#### 一、 安装Python3

1. 切换的国内源
    ```shell
      $ sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
      $ sudo vim /etc/apt/sources.list
    ```

   将原有的源注释掉，添加如下内容：

    ```shell
    #清华源
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
    ```
2. 更新apt-get
    ```shell
    sudo apt-get update
    ```
3. 安装build-essential
    ```shell
    sudo apt-get install build-essential
    ```
4. 安装Python3.8
    ```shell
    sudo apt-get install python3.8
    sudo apt install python3-pip
    ```
5. 查看Python是否安装成功
    ```shell
    python3 -V
    ```
   输出版本号Python 3.8.5即表示安装成功。

#### 二、 更换pip源

1. 临时更换

   ```shell
   pip install *** -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
   ```
   在安装包的时候加上-i参数，指定pip源为清华源，这样就可以临时更换pip源了。
2. 永久更换
   - 新建.pip隐藏文件夹
      ```shell
      mkdir ~/.pip
      ```
   - 在.pip文件夹下新建pip.conf文件
     ```shell
     touch ~/.pip/pip.conf
     vim ~/.pip/pip.conf
     ```
   - 在pip.conf文件中添加如下内容：
     ```shell
     [global]
     index-url = https://pypi.tuna.tsinghua.edu.cn/simple
     [install]
     trusted-host = https://pypi.tuna.tsinghua.edu.cn
     ```
   这样就可以永久更换pip源了。
