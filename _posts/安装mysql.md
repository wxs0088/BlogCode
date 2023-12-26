---
title: 从零开始：在CentOS Stream 9上安装MySQL 8并启用远程root连接
date: 2023-04-22 18:04:55
img: https://cache.yisu.com/upload/admin/Ueditor/2020-04-28/1588043464.jpg
top: false
summary: 本文将介绍如何在CentOS Stream 9上安装MySQL 8并启用远程root连接。
categories: MySQL
tags:
- MySQL
- CentOS
---

### 一、写在前面

MySQL是一款开源的关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle公司。MySQL是最流行的关系型数据库管理系统之一，在WEB应用方面MySQL是最好的RDBMS（关系数据库管理系统）应用软件之一。

本文将介绍如何在CentOS Stream 9上安装MySQL 8并启用远程root连接。

### 二、安装MySQL 8

1. 加MySQL 8 Yum源

    在命令行中执行以下命令：
    ```shell
    dnf install https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
    ```

2. 安装MySQL 8
    
    在命令行中执行以下命令：
    ```shell
    dnf install mysql-community-server
    ```
   
3. 启动MySQL 8,并设置开机启动

    在命令行中执行以下命令：
    ```shell
    systemctl start mysqld
    systemctl enable mysqld
    ```
   
4. 查看MySQL 8的初始密码

    在命令行中执行以下命令：
    ```shell
    grep 'temporary password' /var/log/mysqld.log
    ```
    输出如下：
    ```shell
    2021-04-22T10:04:55.000000Z 1 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: 5q5q5q5q5q
    ```
    从输出中可以看到MySQL 8的初始密码为`5q5q5q5q5q`。

5. 登录MySQL 8
    
    在命令行中执行以下命令：
    ```shell
    mysql -u root -p
    ```
    输入密码`5q5q5q5q5q`，回车，登录成功。

6. 修改MySQL 8的root密码

    在命令行中执行以下命令：
    ```shell
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
    ```
    将`new_password`替换为你想要的密码，回车，修改成功。
    `注意: MySQL 8的密码策略要求密码必须包含大小写字母、数字和特殊字符，且长度不能小于8位。`

7. 允许root用户从任何主机连接到MySQL数据库
    
    在命令行中执行以下命令(添加一个可以任何主机连接的用户)：
    ```shell
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'new_password' WITH GRANT OPTION;
    ```
    将`new_password`替换为你想要的密码，回车，修改成功。
    
    还有另一种方法，直接修改本身的root用户，允许root用户从任何主机连接到MySQL数据库，执行以下命令：
    ```shell
    use mysql;
    update user set host='%' where user='root';
    ```
    `注意: 这种方法会导致root用户可以从任何主机连接到MySQL数据库，不安全。`
    
    最后别忘了在命令行中执行以下命令(刷新权限)：
    ```shell
    flush privileges;
    ```
   
8. 退出MySQL 8，重启MySQL 8
    
    在命令行中执行以下命令：
    ```shell
    exit
    systemctl restart mysqld
    ```

9. 修改MySQL 8的配置文件

    在命令行中执行以下命令：
    ```shell
    vim /etc/my.cnf
    ```
    在`[mysqld]`下面添加如下配置：
    ```shell
    bind-address=0.0.0.0
    skip-name-resolve
    ```
    保存退出，重启MySQL 8。
    ```shell
    systemctl restart mysqld
    ```

10. 防火墙放行MySQL 8的端口
    
    在命令行中执行以下命令：
    ```shell
    firewall-cmd --zone=public --add-port=3306/tcp --permanent
    firewall-cmd --reload
    ```
    
    `注意: 如果你的MySQL 8的端口不是3306，那么需要将上面的命令中的3306替换为你的MySQL 8的端口。`

### 三、连接MySQL 8

1. 从本机连接MySQL 8

    在命令行中执行以下命令：
    ```shell
    mysql -u root -p
    ```
    输入密码`new_password`，回车，登录成功。

2. 从远程主机连接MySQL 8
   使用可视化工具连接MySQL 8，如Navicat、DBeaver等。

   `注意: 如果你的MySQL 8的端口不是3306，那么需要将上面的命令中的3306替换为你的MySQL 8的端口。`

   ![1682159126427](https://wangxs020202.gitee.io/pbad/new/1682159126427.jpg)

   ![1682159271082](https://wangxs020202.gitee.io/pbad/new/1682159271082.jpg)


### 四、总结

MySQL 8较之前的版本有很多变化，比如密码策略、默认的认证方式等，大体安装过程和之前的版本差不多，但是在配置方面有很多变化，需要注意。

性能优化方面，MySQL 8的性能优化方面有很多变化，比如InnoDB的默认页大小从16KB变为16MB，这样可以减少磁盘IO，提高性能，但是会占用更多的内存，所以需要根据实际情况进行调整。
