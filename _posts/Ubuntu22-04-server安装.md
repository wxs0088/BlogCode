---
title: Ubuntu 22.04 Server 安装实录：探索 Linux Server 世界的无限可能！
date: 2023-04-25 17:03:28
img: https://wangxs020202.gitee.io/pbad/new/ubuntu.jpg
top: false
summary: 记录一次Ubuntu 22.04 Server的安装实录
categories: Linux
tags:
- Linux
- Ubuntu
- Shell
- Vim
- SSH
---

### 一、写在前面

Ubuntu 22.04 Server是Ubuntu官方推出的最新版本，它的安装过程非常简单，本文将记录一次Ubuntu 22.04 Server的安装实录。

### 二、安装Ubuntu 22.04 Server

#### 2.1 下载Ubuntu 22.04 Server

Ubuntu 22.04 Server的下载地址：[https://ubuntu.com/download/server](https://ubuntu.com/download/server)

#### 2.2 下载Vmware Workstation Pro

Vmware Workstation Pro的下载地址：[https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html](https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html)

#### 2.3 安装Vmware Workstation Pro

安装Vmware Workstation Pro的过程非常简单，一路Next即可。

#### 2.4 安装Ubuntu 22.04 Server

#### 2.4.1 创建虚拟机

创建虚拟机网上教程很多，这里就不再赘述了。

#### 2.4.1 选择语言

安装首页忘记截图了，选择`try or install ubuntu`。

![1_2023-04-25_16-08-08](https://wangxs020202.gitee.io/pbad/new/1_2023-04-25_16-08-08.png)

#### 2.4.3 选择安装类型
这里选择不更新安装器，直接继续安装。
![2_2023-04-25_16-08-25](https://wangxs020202.gitee.io/pbad/new/2_2023-04-25_16-08-25.png)

#### 2.4.4 选择键盘布局

![3_2023-04-25_16-08-34](https://wangxs020202.gitee.io/pbad/new/3_2023-04-25_16-08-34.png)

#### 2.4.5 选择安装类型
这里我们选择最小安装。
![4_2023-04-25_16-08-55](https://wangxs020202.gitee.io/pbad/new/4_2023-04-25_16-08-55.png)

#### 2.4.6 设置网络连接
这可以选择默认的自动获取IP地址。但是如果你的网络环境不支持DHCP，那么你需要手动设置IP地址。
选择 DHCP 会有一个小小的问题，就是你的虚拟机的IP地址可能会变化，这样你就需要重新配置你的虚拟机的IP地址，这样就会很麻烦。所以，我建议你选择手动设置IP地址。
![5_2023-04-25_16-09-15](https://wangxs020202.gitee.io/pbad/new/5_2023-04-25_16-09-15.png)

![6_2023-04-25_16-09-28](https://wangxs020202.gitee.io/pbad/new/6_2023-04-25_16-09-28.png)

![7_2023-04-25_16-11-06](https://wangxs020202.gitee.io/pbad/new/7_2023-04-25_16-11-06.png)

#### 2.4.7 配置镜像源地址
这里我们选择阿里云的镜像源地址。
![9_2023-04-25_16-12-12](https://wangxs020202.gitee.io/pbad/new/9_2023-04-25_16-12-12.png)

#### 2.4.8 磁盘分区
这里不做任何修改，直接继续安装。
![10_2023-04-25_16-12-36](https://wangxs020202.gitee.io/pbad/new/10_2023-04-25_16-12-36.png)

![11_2023-04-25_16-12-49](https://wangxs020202.gitee.io/pbad/new/11_2023-04-25_16-12-49.png)

![12_2023-04-25_16-13-03](https://wangxs020202.gitee.io/pbad/new/12_2023-04-25_16-13-03.png)

#### 2.4.9 创建登录用户

![13_2023-04-25_16-13-42](https://wangxs020202.gitee.io/pbad/new/13_2023-04-25_16-13-42.png)

#### 2.4.10 配置安装openssh-server
配置安装openssh-server，这个可以用来进行远程连接
![14_2023-04-25_16-13-55](https://wangxs020202.gitee.io/pbad/new/14_2023-04-25_16-13-55.png)

#### 2.4.11 配置安装其他额外的软件
这里我们选择不安装额外的软件。
![15_2023-04-25_16-14-10](https://wangxs020202.gitee.io/pbad/new/15_2023-04-25_16-14-10.png)

#### 2.4.12 安装过程
系统安装过程没什么操作，这里就没再截图了。等待一段时间，会提示你安装完成。重新启动就可以了

### 三、配置Ubuntu 22.04 Server

#### 3.1 安装vim编辑器

```shell
sudo apt-get install vim
```

#### 3.2 设置root密码
连续输入两次root密码即可。这里密码是不显示的，所以你需要自己记住密码。
```shell
sudo passwd root
```

#### 3.3 编辑ssh服务的配置文件

```shell
sudo vim /etc/ssh/sshd_configss
```

修改图中两个位置的配置，然后保存退出。

![16_2023-04-25_17-44-15](https://wangxs020202.gitee.io/pbad/new/16_2023-04-25_17-44-15.png)

#### 3.4 重启ssh服务

```shell
sudo service ssh restart
```

#### 3.5 安装网络工具，并查看IP地址

```shell
sudo apt-get install net-tools
ifconfig
```

![17_2023-04-25_17-48-48](https://wangxs020202.gitee.io/pbad/new/17_2023-04-25_17-48-48.png)

#### 3.6 远程客户端登录root用户

![18_2023-04-25_17-49-45](https://wangxs020202.gitee.io/pbad/new/18_2023-04-25_17-49-45.png)
![19_2023-04-25_17-50-03](https://wangxs020202.gitee.io/pbad/new/19_2023-04-25_17-50-03.png)

#### 3.7 更换apt源
```shell
# 备份原有的源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 编辑源文件
sudo vim /etc/apt/sources.list
```
删除原有的源，然后添加新的源，这里我添加的是清华大学源。
```text
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse

deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
```
然后保存退出，更新源
```shell
sudo apt-get update
```

到这里，我们就完成了Ubuntu 22.04 Server的安装和一些基础的配置了。

### 四、结语

这里我们只是简单的安装了Ubuntu 22.04 Server，并且配置了一些基础的配置，比如设置root密码，安装vim编辑器，配置apt源等等。如果你想要安装其他的软件，可以参考Ubuntu官方的文档。

后续我会在这个基础上，会做一些服务的部署，以及docker的安装和使用。敬请期待。
