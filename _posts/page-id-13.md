---
title: '通过Certbot实现使用长期免费的HTTPS数字证书'
date: 2023-11-30 10:41:16
img: https://wangxs020202.gitee.io/pbad/background/lets-encrypt.png
top: false
summary: 本文将介绍如何使用Certbot来实现使用长期免费的HTTPS数字证书。
categories: SSL
tags:
- Certbot
- HTTPS
- SSL
- SSL证书
- Let's Encrypt
- Nginx
---

### 一、写在前面
最近开发应用需要使用https协议，免不了使用ssl数字证书，以前都是用某云的定期免费证书，到期还得重新申请比较麻烦。付费的证书价格比较高，有没有长期免费的数字证书呢，答案就是Let’s Encrypt 。

### 二、Let’s Encrypt 简介

![lets-encrypt_ssl](https://wangxs020202.gitee.io/pbad/new/lets-encrypt_ssl.png)

Let’s Encrypt 是一家免费、开放、自动化的证书颁发机构（CA），为公众的利益而运行。 它是一项由 Internet Security Research Group (ISRG) 提供的服务。

我们以尽可能对用户友好的方式免费提供为网站启用 HTTPS（SSL/TLS）所需的数字证书。 这是因为我们想要创建一个更安全，更尊重隐私的 Web 环境。

您可以通过下载我们的 年度报告 阅读我们最近一年的评论。

Let’s Encrypt的关键原则为：

- 免费： 任何拥有域名的人都可以使用 Let’s Encrypt 免费获取受信的证书。
- 自动化： 运行于服务器上的软件可以与 Let’s Encrypt 直接交互，以便轻松获取证书，安全地配置它，并自动进行续期。
- 安全： Let’s Encrypt 将成为一个推动 TLS 安全最佳实践发展的的平台，无论是作为一个证书颁发机构（CA）还是通过帮助网站运营商正确地保护其服务器。
- 透明： 所有颁发或吊销的证书将被公开记录，供任何人查阅。
- 开放： 自动签发和续订协议 已经发布 作为其他人可以采用的开放标准。
- 乐于合作： 就像互联网底层协议本身一样，Let’s Encrypt 是为了让整个互联网社区受益而做出的共同努力，它不受任何单一组织的控制。

### 三、Certbot 简介

Certbot是一个全功能、可扩展的客户端，用于Let’s Encrypt CA(或任何其他使用ACME协议的CA ),它可以自动执行获取证书和配置web服务器以使用证书的任务。

### 四、开始动手

#### 4.1 安装Nginx

```shell
sudo apt install nginx
```

#### 4.2 安装snap

```shell
sudo apt update
sudo apt install snapd
```

#### 4.3 安装core snap来获取最新的snapd

```shell
sudo snap install core
```

`output`

```shell
core 16-2.60.4 from Canonical✓ installed
```

#### 4.4 验证安装结果

```shell
sudo snap install hello-world
hello-world
```

`output`

```shell
hello-world 6.4 from Canonical✓ installed
Hello World!
```

#### 4.5 确保snapd版本是最新的

```shell
sudo snap refresh core
```

#### 4.6 移除 certbot-auto和任何Certbot OS包

```shell
sudo apt-get remove certbot
```

#### 4.7 安装Certbot

```shell
sudo snap install --classic certbot
```

`output`

```shell
certbot 2.7.4 from Certbot Project (certbot-eff✓) installed
```

#### 4.8 创建软链接

```shell
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

#### 4.9 获取证书

##### 4.9.1 下载和安装证书
无需其他配置，直接对域名进行验证，获取证书并对nginx进行配置。

```shell
sudo certbot --nginx
```

##### 4.9.2 只下载证书

```shell
sudo certbot certonly --nginx
```

##### 4.9.3 测试自动续订

```shell
sudo certbot renew --dry-run
```


### 五、总结

Let’s Encrypt 和 ACME 协议的目标是实现可信数字证书的自动获取，从而简化 HTTPS 服务器部署中的人工操作。 这一目标是由 Web 服务器上的证书管理软件完成的。

整体流程分为两步。首先，管理软件向证书颁发机构证明该服务器拥有域名的控制权。 之后，该管理软件就可以申请、续期或吊销该域名的证书。



