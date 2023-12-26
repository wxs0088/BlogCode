---
title: "突破界限：通过UTM克服兼容性障碍，在MacBook上安装CentOS 7.9的逐步指南"
date: 2023-06-26 10:59:48
img: https://wangxs020202.gitee.io/pbad/background/page_10.png
top: false
summary: 本文将介绍如何通过UTM克服兼容性障碍，在MacBook上安装CentOS 7.9。
categories: Mac
tags:
    - UTM
    - 虚拟机
    - CentOS
    - Mac
---

### 序幕(背景)

最近在m1芯片的mac上使用Parallels Desktop安装centos7.9发现一个问题。在安装Centos7.9，开始安装的时候，显示这样的，enter了install Centos 7进去之后还会跳回这个界面，甚是无语。

![image-20230310103517510](https://wangxs020202.gitee.io/pbad/new/image-20230310103517510.png)

### 解决方案

在网上找了一圈，发现有人说是因为centos7.9的内核版本太低了，不支持m1芯片，需要升级内核版本。但是我在网上找了一圈，发现没有找到升级内核版本的方法，所以我就想到了用UTM来安装centos7.9，因为UTM是一个虚拟机，可以安装很多系统，而且可以自定义内核版本，所以我就想到了用UTM来安装centos7.9。

### 准备工作

镜像：[CentOS-7-aarch64-Everything-2009.iso ](http://ftp.yz.yamagata-u.ac.jp/pub/linux/centos-altarch/7.9.2009/isos/aarch64/)

工具：[UTM](https://mac.getutm.app/)

### 步骤

1. 创建虚拟机，并使用模拟

   ![image-20230310103615602](https://wangxs020202.gitee.io/pbad/new/image-20230310103615602.png)

   ![image-20230310103633950](https://wangxs020202.gitee.io/pbad/new/image-20230310103633950.png)

2. 选择操作系统，这里我们把之前下载的ISO导入

   ![image-20230310103649707](https://wangxs020202.gitee.io/pbad/new/image-20230310103649707.png)

   ![image-20230310103722908](https://wangxs020202.gitee.io/pbad/new/image-20230310103722908.png)

3. 硬件这里选择与ISO镜像对应的架构（我这里下载的是ARM64）

   ![image-20230310104117521](https://wangxs020202.gitee.io/pbad/new/image-20230310104117521.png)

4. 设置虚拟机名称

   ![image-20230310103757135](https://wangxs020202.gitee.io/pbad/new/image-20230310103757135.png)

5. 创建完成后启动虚拟机

   ![image-20230310103841339](https://wangxs020202.gitee.io/pbad/new/image-20230310103841339.png)

6. 启动完成后选择install Centos 7直接进行安装，之后会进入系统语言选择的界面

   ![image-20230310104349733](https://wangxs020202.gitee.io/pbad/new/image-20230310104349733.png)

7. 接下来会对一些配置进行检测，这里只需等待即可，当**开始安装**可以点击后，之间键入

   ![image-20230310104424687](https://wangxs020202.gitee.io/pbad/new/image-20230310104424687.png)

8. 之后就是对系统的安装了，过程有点缓慢。我们可以对**root密码以及用户**做一些设置，等待即可。

   ![image-20230310104657926](https://wangxs020202.gitee.io/pbad/new/image-20230310104657926.png)

9. 接下来直接重启

   ![image-20230310110219703](https://wangxs020202.gitee.io/pbad/new/image-20230310110219703.png)

10. 若重启后还会进入系统安装的界面，将虚拟机内的磁盘进行调换或者删除第一个即可

    ![image-20230310110335738](https://wangxs020202.gitee.io/pbad/new/image-20230310110335738.png)

    ![image-20230310110357905](https://wangxs020202.gitee.io/pbad/new/image-20230310110357905.png)

11. 进入界面

    ![image-20230310110702245](https://wangxs020202.gitee.io/pbad/new/image-20230310110702245.png)

12. 进入系统后，我们可以看到内核版本是3.10.0-1160.11.1.el7.aarch64，这个版本是可以正常使用的，不会出现之前的问题。

至此，我们就完成了在MacBook上安装CentOS 7.9的过程。
