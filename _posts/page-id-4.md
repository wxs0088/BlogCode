---
title: 利用DockerHub在Centos7.6环境下部署Nginx反向代理Gunicorn+Flask独立架构
date: 2023-05-28 17:41:37
img: https://wangxs020202.gitee.io/pbad/background/page_4.png
top: false
summary: 本文将介绍如何在Centos7.6环境下利用DockerHub部署Nginx反向代理Gunicorn+Flask独立架构。
categories: Docker
tags:
  - Docker
  - Gunicorn
  - Flask
  - Python
  - Centos
---

### 序幕

**书接上回，上回说到**，[在Win10系统下利用Docker部署Gunicorn+Flask打造独立镜像](https://sirxs.cn/2023/05/27/page-id-3/)，今天我们来讲一讲利用DockerHub在Centos7.6环境下部署Nginx反向代理Gunicorn+Flask独立架构

### 准备

1. 首先你需要有自己的云服务。这里我使用的是阿里云的服务器，如果你没有服务器，可以通过这个链接申请一台

   [https://free.aliyun.com/?spm=5176.14145661.J_3598540520.ace-channel-latest-activity-card.3eb418759BoljH](https://free.aliyun.com/?spm=5176.14145661.J_3598540520.ace-channel-latest-activity-card.3eb418759BoljH)

   阿里云白嫖服务器

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUucG5n?x-oss-process=image/format,png)

2. 其次你需要将本地项目`push`到DockerHub(仓库)

    - 首先激活账号，创建仓库

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxLnBuZw?x-oss-process=image/format,png)

      **这里的创建仓库与Github类似**

    - 填写仓库信息具体为仓库名称、描述以及是否公开或者私有。

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUyLnBuZw?x-oss-process=image/format,png)

    - 创建成功之后，它就会出现在镜像列表中

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUzLnBuZw?x-oss-process=image/format,png)

3. 此时我们需要对本地的镜像重命名，这里重命名为herosir/flask_back。因为要与dockerhub上的仓库对应。如果名称不对应是无法将本地镜像push到线上仓库中。

   ```python
   docker tag myflask herosir/flask_back
   ```

4. 结果

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnU0LnBuZw?x-oss-process=image/format,png)

5. 之后在命令行输入命令

   ```python
   docker login
   ```

6. 用DockerHub的账号和密码登录

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnU1LnBuZw?x-oss-process=image/format,png)

7. 登录成功之后，用命令把本地镜像push到DockerHub中

   ```python
   docker push herosir/flask_back
   ```

8. 注意这里的镜像名称必须和hub中的仓库名称一致，否则将会抛出错误。

9. 上传成功后，就可以在DockerHub中看到它了，此时就能随意pull操作了

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnU2LnBuZw?x-oss-process=image/format,png)

### 部署

1. 前置操作已经完毕，此时，登录你的云服务器，这里以阿里云的Centos7.6为例子，进入服务器后安装Docker服务

   ```python
   #升级yum
   sudo yum update
   #卸载旧版本docker
   sudo yum remove docker  docker-common docker-selinux docker-engine
   #安装依赖
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   #设置源
   sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   sudo yum makecache fast
   #安装docker
   sudo yum install docker-ce
   
   #启动服务
   sudo systemctl start docker
   ```

2. 安装完成后输入 docker -v

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnU3LnBuZw?x-oss-process=image/format,png)

   *返回Docker版本号说明没有问题*

3. 拉取我们之前打包并且上传到hub的Flask镜像

   ```python
   docker pull herosir/flask_back
   ```

4. 下载成功后，会展示在镜像库里

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnU4LnBuZw?x-oss-process=image/format,png)

5. 运行项目，这里我们可以采用后台守护进程的模式起服务

   ```python
   sudo docker run -d -p 5000:5000 --name testflask herosir/flask_back
   ```

6. 使用docker ps命令可以看到是否运行成功。

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnU5LnBuZw?x-oss-process=image/format,png)

7. 使用服务器的ip访问一下Flask服务，这里有个小坑，不论是腾讯云、阿里云还是百度云亦或是各种乱七八糟的云，都需要在安全组策略中开放你需要访问的端口，比如这里我用的阿里云。

    - 解决安群组策略

    - 找到服务器的`配置规则`

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxMC5wbmc?x-oss-process=image/format,png)

    - `手动添加`下图信息

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxMS5wbmc?x-oss-process=image/format,png)

8. 设置完成之后通过服务器`公网IP`进行访问

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxMi5wbmc?x-oss-process=image/format,png)

   OK，访问没有问题

9. 接下来，我们同样利用Docker来安装Nginx服务

   ```python
   docker pull nginx
   ```

10. 随后启动Nginx测试一下

    ```python
    docker run -d -p 80:80 nginx
    ```

    - 这里也需要设置安群组

      操作同上，内容如下

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxMy5wbmc?x-oss-process=image/format,png)

11. 直接访问`公网IP`

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxNC5wbmc?x-oss-process=image/format,png)

12. 现在，我们将运行Nginx容器里的配置文件copy到宿主机里面

    ```python
    docker cp 容器id:/etc/nginx/conf.d/default.conf /root/default.conf
    ```

    **前面是容器的路径 后面是宿主机的路径**

    *容器id可以通过docker ps命令查看*

13. 复制出来之后，输入命令修改这个nginx配置

    ```python
    vim /root/default.conf
    ```

14. 将Gunicorn配置加到里面(**更改**)

    ```python
    server {
        listen       80;
        listen  [::]:80;
        server_name  localhost;
    
        #charset koi8-r;
        #access_log  /var/log/nginx/host.access.log  main;
    
        location / {
    
            proxy_pass http://你的服务器公网IP:5000; # 这里是指向 gunicorn host 的服务地址
    
            proxy_set_header Host $host;
    
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
        }
    
        #error_page  404              /404.html;
    
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    
        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}
    
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}
    
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
    ```

15. 修改完配置文件之后，关掉运行的nginx服务容器，并且删掉它

    ```python
    docker stop 容器id
    docker rm $(docker ps -a -q)
    ```

16. 随后再次启动Nginx容器，不过这次和上次不同之处就是需要用到 -v 进行挂载了，挂载简单理解就是将宿主机的文件替换Docker容器内部的文件，达到修改的效果。

    ```python
    docker run --name mynginx -d -p 80:80 -v /root/default.conf:/etc/nginx/conf.d/default.conf nginx
    ```

    **这里-v参数也遵循冒号左侧为宿主机右侧为容器的原则。**

17. 新启动成功后，访问服务器ip

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvYWxpeXVuYnUxNS5wbmc?x-oss-process=image/format,png)

    **OK，部署完成**

### 结语

部署完成，轻松且愉快
