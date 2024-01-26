---
title: 美多商城项目实战EP01：项目介绍，项目架构，项目准备
date: 2024-01-26 10:32:03
img: https://wangxs020202.gitee.io/pbad/background/mdmall.png
top: top
summary: 美多商城项目实战EP01：项目介绍，项目架构，项目准备
categories: 美多商城
tags:
  - 美多商城
  - Python
  - Django
  - Vue.js
  - 项目实战
  - 项目准备
  - 项目介绍
  - 项目架构
---

### 一、写在前面

美多商城是一个综合性的B2C平台，包含了商品展示、购物车、订单管理、支付功能。最近也学习了一些其他的技术，所以打算用这个项目来实战一下，把之前学习的知识点串联起来，也算是对自己学习的一个总结。
这里我还是以[美多商城项目课件](https://luojun0115.github.io/meiduo2018/index.html)为基础，后续会加入一些自己的想法，比如加入一些新的功能，或者使用新的技术等等。

### 二、项目介绍

美多商城项目是一个B2C模式的电商网站，采用前后端分离的开发模式，前端采用vue框架，后端采用DRF框架。

### 三、项目架构

项目采用前后端分离的应用模式
前端使用Vue.js
后端使用Django REST framework

![img.png](https://luojun0115.github.io/meiduo2018/images/%E9%A1%B9%E7%9B%AE%E6%95%B4%E4%BD%93%E6%9E%B6%E6%9E%84.png)


### 四、项目准备

#### 4.1 环境准备

##### 4.1.1 后端环境准备

这里我是用的使用anaconda来管理Python环境，大家可以参考[最大化服务器性能：在Ubuntu 22.04上逐步指南安装Anaconda](https://sirxs.cn/2023/04/26/ubuntu-an-zhuang-anaconda/)这篇文章来安装anaconda。这里我使用的是Python3.10版本。

##### 4.1.2 前端环境准备

这里我推荐参考[菜鸟教程](https://www.runoob.com/w3cnote/vue2-start-coding.html)的Vue.js安装教程，由于我并不是专门做前端的，所以我这里使用的是Vue2。

##### 4.1.3 数据库环境准备

这里我推荐参考我之前写的一篇文章[在Ubuntu 20.04玩转Mysql8.0：安装、配置、使用、删除](https://sirxs.cn/2024/01/26/page-id-16/)的Mysql安装教程，这里我使用的是Mysql8.0版本。

#### 4.2 后端项目准备

##### 4.2.1 创建项目目录

```bash
mkdir meiduo_mall
cd meiduo_mall
```

##### 4.2.2 创建虚拟环境

```bash
conda create -n meiduo python=3.10
```

##### 4.2.3 激活虚拟环境

```bash
conda activate meiduo
```

##### 4.2.4 安装Django

```bash
pip install django==5.0.1
```

##### 4.2.5 创建Django项目

```bash
django-admin startproject MDmall_Backend
```

##### 4.2.6 创建Django应用

```bash
cd MDmall_Backend
python manage.py startapp users # 创建用户应用
```

##### 4.2.7 启动Django项目

```bash
python manage.py runserver
```

#### 4.3 前端项目准备

##### 4.3.1 创建项目目录

```bash
cd meiduo_mall
```

##### 4.3.2 安装Vue

```bash
npm install -g @vue/cli
```

##### 4.3.3 创建Vue项目

```bash
vue init webpack MDmall_Frontend
```

##### 4.3.4 运行Vue项目

```bash
cd MDmall_Frontend
npm run serve
```




