---
title: 在Ubuntu 20.04玩转Mysql8.0：安装、配置、使用、删除
date: 2024-01-26 09:59:42
img: https://wangxs020202.gitee.io/pbad/background/mysql.png
top: false
summary: 本文将介绍如何在Ubuntu 20.04上安装、配置、使用、删除Mysql8.0。
categories: Mysql
tags:
  - Mysql
  - Ubuntu
  - 数据库
  - 数据库管理
  - 数据库安装
  - 数据库配置
  - 数据库使用
---

### 一、写在前面

`Mysql`是一种关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。`Mysql`是最流行的关系型数据库管理系统之一，它的主要功能是基于
SQL 的关系型数据库管理系统，支持多用户、多线程使用，可以在多种不同的操作系统上使用。本文将介绍如何在Ubuntu
20.04上安装、配置、使用、删除Mysql8.0。

### 二、安装Mysql8.0

#### 2.1 安装Mysql8.0

```bash
sudo apt update
sudo apt install mysql-server
```

### 三、配置Mysql8.0

#### 3.1 安全配置MySQL

```bash
sudo mysql_secure_installation
```

`mysql_secure_installation`脚本设置的东西：更改root密码、移除MySQL的匿名用户、禁止root远程登录、删除test数据库和重新加载权限。
询问是否要更改root密码时，看情况是否需要更改；**循环是否禁止远程登录时，要输入NO，否则会导致后面用navicat远程连接mysql失败。** 
其余的问题直接回车，使用上面的这些选项可以提高MySQL的安全。

### 四、使用Mysql8.0

#### 4.1 登录Mysql

```bash
sudo mysql -u root -p
```

如果您之前没有为`root`用户设置密码，则`-p`选项可能不需要。

#### 4.2 更改 root 用户的认证方法

默认情况下，MySQL 使用`auth_socket`插件对`root`进行身份验证，这可能会阻止远程登录。要改为使用密码认证，请执行以下命令：

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH 'mysql_native_password' BY 'SSKS#dd';
FLUSH PRIVILEGES;
```

#### 4.3 创建一个允许远程访问的 root 用户

接下来，创建一个与您的 root 用户相同权限的用户，但允许从任何主机连接。

```bash
CREATE USER 'root'@'%' IDENTIFIED BY 'SSKS#dd';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

#### 4.4 退出Mysql

```bash
exit
```

#### 4.5 修改 MySQL 配置以允许远程连接：

编辑 MySQL 配置文件，通常在 /etc/mysql/mysql.conf.d/mysqld.cnf 或 /etc/mysql/my.cnf。

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

找到`bind-address`和`mysqlx-bind-address`项，把`127.0.0.1`更改为`0.0.0.0`或注释掉这两行，以允许从任何 IP 地址连接。

```bash
bind-address = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```

保存并关闭文件，然后重新启动 MySQL 服务。

```bash
sudo systemctl restart mysql
```

### 五、防火墙设置

#### 5.1 查看防火墙状态

如果是刚建好的虚拟机，应该是不带防火墙的，需要先安装

```bash
sudo apt install firewalld
```

#### 5.2 开放3306端口

```bash
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

#### 5.3 重启防火墙

```bash
sudo firewall-cmd --reload
```

### 5.4 查看端口开放状态

```bash
sudo firewall-cmd --list-ports
```

### 六、测试Mysql

#### 6.1 远程连接Mysql

从另一台计算机上，尝试使用新设置的`root`用户和密码远程连接到`MySQL`服务器。

```bash
mysql -h [服务器IP地址] -u root -p
```

### 七、删除Mysql8.0

#### 7.1 停止服务

```bash
sudo systemctl stop mysql
```

#### 7.2 卸载软件包

使用`apt-get`命令卸载 MySQL 服务器及其相关软件包。

```bash
sudo apt-get remove --purge mysql-server mysql-client mysql-common
sudo apt-get autoremove
sudo apt-get autoclean
```

#### 7.3删除配置和数据文件

删除 MySQL 的配置文件和数据目录。这一步是必要的，因为`apt-get remove --purge`可能不会删除所有文件。

```bash
sudo rm -rf /etc/mysql /var/lib/mysql
sudo rm -rf /var/log/mysql
```

#### 7.4 删除用户和组

```bash
sudo deluser mysql
sudo delgroup mysql
```

#### 7.5 删除残留文件

还可以使用`find`命令来查找系统中可能遗留的与 MySQL 相关的其他文件，并手动删除它们。

```bash
sudo find / -iname 'mysql*' -exec rm -rf {} \;
```



