---
title: 打造自己的博客天地：在腾讯云服务器上部署Hexo博客
date: 2023-04-21 14:48:23
img: https://titangene.github.io/images/cover/hexo.jpg
top: false
summary: 本文将介绍如何在腾讯云服务器上部署Hexo博客，以及如何使用Nginx反向代理，实现域名访问。
categories: Hexo
tags:
- Git
- Nginx
- 腾讯云
- 博客
---

### 一、写在前面

这个博客自创建以来也有差不多一周了，之前一直在使用GitHub Pages来搭建博客，但是由于GitHub Pages的访问速度比较慢，所以最近决定将博客迁移到腾讯云服务器上使用Nginx来做代理，这样可以实现域名访问，访问速度也会更快。

### 二、腾讯云服务器部署Hexo博客

#### 2.1 购买腾讯云服务器

首先需要购买腾讯云服务器，这里推荐购买腾讯云的云服务器，因为腾讯云的云服务器性价比比较高，而且还有免费的流量，可以满足日常使用的需求。

#### 2.2 购买域名

接下来需要购买一个域名，这里也可以使用腾讯云的域名服务，购买域名的时候需要注意一下域名的后缀，如果你购买的是`.cn`的域名，那么需要在腾讯云的域名服务中进行备案，备案需要一定的时间，所以在购买域名的时候需要注意一下。

`服务器与域名也可以在其他云服务商处购买。`

#### 2.3 配置域名解析

在购买好域名之后，需要将域名解析到腾讯云服务器的公网IP上，这样才能通过域名访问到博客。

这些网上都有教程，这里就不再赘述了。

#### 2.4 配置腾讯云服务器

##### 2.4.1 安装Git、Nginx

```bash
sudo yum install -y git nginx
```

##### 2.4.2 配置Git

`注意：如果本地没有配置ssh，先配一个ssh。这里因为我已经有了，就不再配置了。`

开始前先切到`root`用户下,执行一下命令：

```bash
useradd git # 添加一个新用户
passwd git # 设置git用户密码
su git # 切换用户进行后续操作
cd /home/git/
mkdir -p projects/blog # 把项目目录建立起来
mkdir repos && cd repos
git init --bare blog.git # 创建仓库
cd blog.git/hooks
```

在`blog.git/hooks`目录下创建一个`post-receive`文件，内容如下：

```bash
#!/bin/bash
git --work-tree=/home/git/projects/blog --git-dir=/home/git/repos/blog.git checkout -f
```

退出vim，继续进行用户相关的操作：

```bash
chmod +x post-receive # 添加可执行权限
```

退出git用户，切换到root用户，执行以下命令：

```bash
chown -R git:git /home/git/repos/blog.git # 给git用户添加权限
```

在我们本地测试一下是否成功：

```bash
git clone git@server_ip:/home/git/repos/blog.git
```

如果成功，会在当前目录下生成一个`blog`文件夹，目前里面是空的，因为我们还没有往里面添加文件。

##### 2.4.3 配置Nginx

修改`/etc/nginx/nginx.conf`文件，修改以下内容：

开头

```bash
user root;
```

server

```bash
server {
        listen       80;
        server_name  your_domain;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        root /home/git/projects/blog;
        index index.html index.htm;
        }

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

`注意：这里的server_name需要改成你自己的域名。`

##### 2.4.4 配置Hexo

在本地博客目录下执行以下命令：

```bash
npm install --save hexo-deployer-git
```

修改`_config.yml`文件，添加以下内容：

```yaml
url: https://your_domain
deploy:
  type: git
  repo: git@server_ip:/home/git/repos/blog.git
  branch: master
```

`注意：这里的your_domain需要改成你自己的域名,repo需要改成你自己的服务器地址。`

##### 2.4.5 部署博客

在本地博客目录下执行以下命令：

```bash
hexo clean
hexo g
hexo d
```

如果成功，会在服务器上生成一个`blog`文件夹，里面就是我们的博客了。

##### 2.4.6 访问博客

启动Nginx服务并访问该博客网站，如果能正常访问，说明部署成功。

```bash
sudo systemctl enable nginx.service # 设置开机启动
sudo systemctl start nginx.service  # 启动服务
sudo systemctl stop nginx.service   # 停止服务
sudo systemctl restart nginx.service # 重启服务
sudo systemctl status nginx.service # 查看服务状态
```

### 三、总结

通过本文，读者可以了解如何利用腾讯云的便捷服务快速搭建自己的Hexo博客，并将其部署到互联网上，与大家分享自己的思考和经验。期望本文对那些对博客写作有兴趣的人提供一些又新又有用的参考。
