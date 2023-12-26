---
title: 最大化服务器性能：在Ubuntu 22.04上逐步指南安装Anaconda
date: 2023-04-26 10:32:29
img: https://oss.py.cn/pycn/upload/article/000/000/008/5f3b402bc155f941.jpg
top: false
summary: 本文将指导您在Ubuntu 22.04上安装Anaconda，以最大化服务器性能。
categories: Anaconda
tags:
- Anaconda
- Linux
- Ubuntu
- Python
---

### 一、写在前面

Anaconda是一个用于科学计算的Python发行版，支持 Linux、Mac、Windows系统，提供了包管理与环境管理的功能，可以很方便地解决多版本python并存、切换以及各种第三方包安装问题。

Anaconda利用工具conda来进行包和环境管理。conda是一个开源包管理系统和环境管理系统，用于安装多个版本的软件包及其依赖关系，并在它们之间轻松切换。它在Anaconda发行版中默认包含，但也可以通过Miniconda独立安装。

本文将指导您在Ubuntu 22.04上安装Anaconda，以最大化服务器性能。

### 二、下载Anaconda

Anaconda的下载地址：[官网](https://www.anaconda.com/products/individual)，[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)。

### 三、安装Anaconda

这里我们继续使用上期的Ubuntu 22.04系统，如果您还没有安装，请参考：[Ubuntu 22.04 Server 安装实录](https://sirxs.cn/2023/04/25/ubuntu22-04-server-an-zhuang/)。

#### 3.1 传输文件

将下载好的Anaconda文件传输到服务器上，这里我们使用Termius，如果您还没有安装，也可使用Xftp等工具。

![anaconda_2023-04-26_10-39-37](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-39-37.png)

#### 3.2 安装Anaconda

在服务器上，进入Anaconda文件所在目录，执行以下命令：

```bash
sudo bash Anaconda3-5.3.1-Linux-x86_64.sh -u
```
![anaconda_2023-04-26_10-19-46](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-19-46.png)

-u 参数表示使用默认安装路径，如果不加-u参数，需要手动输入安装路径。

`注意`：这个过程中可能会出现以下错误

```text
Anaconda3-5.3.1-Linux-x86_64.sh: line 353: bunzip2: command not found
```

这是因为服务器上没有安装bzip2，执行以下命令安装即可：

```bash
sudo apt install bzip2
```
![anaconda_2023-04-26_10-19-14](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-19-14.png)

license协议，`yes`继续。

![anaconda_2023-04-26_10-20-19](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-20-19.png)

安装路径，按`Enter`继续。

![anaconda_2023-04-26_10-20-37](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-20-37.png)

添加环境变量，`yes`继续。

![anaconda_2023-04-26_10-21-09](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-21-09.png)

安装Microsoft VSCode，`no`继续。

![anaconda_2023-04-26_10-21-29](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-21-29.png)

安装完成。

下面是整体的安装过程：

![anaconda_2023-04-26_10-22-00](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-22-00.gif)

### 四、创建虚拟环境

#### 4.1 查看本地虚拟环境

```bash
source ~/.bashrc
conda env list
```

![anaconda_2023-04-26_10-21-46](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-21-46.png)

#### 4.2 创建虚拟环境

```bash
conda create -n mDjango python=3.10 -y
```
![anaconda_2023-04-26_10-25-57](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-25-57.png)
![anaconda_2023-04-26_10-28-31](https://wangxs020202.gitee.io/pbad/new/anaconda_2023-04-26_10-28-31.png)

#### 4.3 激活虚拟环境

```bash
conda activate mDjango
```

### 四、anaconda常用命令

```bash
conda --version # 查看版本
conda update conda # 更新conda
conda update anaconda # 更新anaconda
conda update python # 更新python
conda update --all # 更新所有包
conda env list # 查看所有虚拟环境
conda create -n 虚拟环境名称 python=版本号 # 创建虚拟环境
conda activate 虚拟环境名称 # 激活虚拟环境
conda deactivate # 退出虚拟环境
conda env remove -n 虚拟环境名称 # 删除虚拟环境
conda install 软件包名称 # 安装软件包
conda remove 软件包名称 # 删除软件包
conda list # 查看已安装的包
conda search 软件包名称 # 查看可安装的包
conda info 软件包名称 # 查看包的信息
conda config --add channels 清华大学开源软件镜像站地址 # 添加清华大学开源软件镜像站
conda config --set show_channel_urls yes # 显示镜像站地址
conda config --set show_channel_urls no # 不显示镜像站地址
```

### 五、结语

本文是在上篇文章的基础上，继续安装Anaconda，如果您还没有安装Ubuntu 22.04，请参考：[Ubuntu 22.04 Server 安装实录](https://sirxs.cn/2023/04/25/ubuntu22-04-server-an-zhuang/)。

