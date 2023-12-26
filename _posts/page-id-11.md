---
title: "网络通畅，远程自如：UTM上安装CentOS 7.9后，轻松配置网络与SSH"
date: 2023-06-26 11:13:56
img: https://wangxs020202.gitee.io/pbad/background/page_11.png
top: false
summary: 本文将介绍如何在UTM上安装CentOS 7.9后，如何轻松配置网络与SSH。
categories: Mac
tags:
  - UTM
  - CentOS
  - 网络
  - SSH
---

### 序幕

上篇文章我们解决了[在MacBook上安装CentOS 7.9](https://sirxs.cn/2023/06/26/page-id-10/)的问题，本文将介绍如何在UTM上安装CentOS 7.9后，如何轻松配置网络与SSH。

### UTM上安装CentOS 7.9

- 在UTM上安装CentOS 7.9，参考上一篇文章：[在MacBook上安装CentOS 7.9](https://sirxs.cn/2023/06/26/page-id-10/)。

### 配置网络

1. 配置虚拟机的网络为桥接模式

   ![image-20230310111001050](https://wangxs020202.gitee.io/pbad/new/image-20230310111001050.png)

2. 进入虚拟机查看是否有网，发现ping不通说明没有网络

   ![image-20230310111238247](https://wangxs020202.gitee.io/pbad/new/image-20230310111238247.png)

3. 使用 `nmcli` 命令查看一下网络设置，该命令会返回 4 列数据. 分别是 `NAME(联网代号)`, `UUID(识别码)`, `TYPE网卡类型`, `DEVICE(网卡名称)`.

   ```shell
   nmcli connection show
   ```

   ![image-20230310111342185](https://wangxs020202.gitee.io/pbad/new/image-20230310111342185.png)

4. 修改网络配置文件，将onboot改为yes

   ```shell
   sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
   ```

   ![image-20230310111801979](https://wangxs020202.gitee.io/pbad/new/image-20230310111801979.png)

5. 重启网卡，再次ping发现可以ping同了

   ```
   systemctl restart network
   ```

   ![image-20230310112110801](https://wangxs020202.gitee.io/pbad/new/image-20230310112110801.png)

6. 安装`openssh-server&openssl`

   ![image-20230310112602994](https://wangxs020202.gitee.io/pbad/new/image-20230310112602994.png)

7. 修改ssh配置文件

   ![image-20230310112812350](https://wangxs020202.gitee.io/pbad/new/image-20230310112812350.png)![image-20230310112837191](https://wangxs020202.gitee.io/pbad/new/image-20230310112837191.png)

8. 查看虚拟机ip并进行外部连接

   ![image-20230310112318919](https://wangxs020202.gitee.io/pbad/new/image-20230310112318919.png)

   ![image-20230310113606195](https://wangxs020202.gitee.io/pbad/new/image-20230310113606195.png)


