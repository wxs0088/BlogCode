---
title: 在Win10系统下利用Docker部署Gunicorn+Flask打造独立镜像
date: 2023-05-27 12:41:33
img: https://wangxs020202.gitee.io/pbad/background/page_3.png
top: false
summary: 本文将介绍如何在Win10系统下利用Docker部署Gunicorn+Flask打造独立镜像。
categories: Docker
tags:
    - Docker
    - Gunicorn
    - Flask
    - Python
    - Windows
---

### 序幕

**书接上回，上回说到**，[Windows下安装docker](https://sirxs.cn/2023/05/26/page-id-2/)，今天我们来讲一讲将本地项目打包到docker

#### 什么是docker镜像

Docker 包含三个基本概念，分别是镜像（Image）、容器（Container）和仓库（Repository）。镜像是 Docker 运行容器的前提，仓库是存放镜像的场所，可见镜像更是Docker的核心。

回到正题，Docker 镜像可以看作是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

### 整理

1. 首先我们先看下项目的整体结构

   ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rangucG5n?x-oss-process=image/format,png)

2. `manage.py`是项目入口文件

3. 接下里我们使用Gunicorn+gevent来运行Flask项目

4. 安装相应的库

   ```python
   pip install gunicorn gevent --user
   ```

5. 编辑项目目录下的gunicorn.conf.py

   ```python
   workers = 3    # 进程数
   worker_class = "gevent"   # 异步模式
   bind = "0.0.0.0:5000"
   ```

6. 由于Gunicorn并不支持Windows环境，所以只需要写好配置，不需要运行。

7. 编辑项目目录下的requirements.txt文件，这里面都是我们项目所依赖的库

   使用

   ```python
   pip freeze > requirements.txt
   ```

   导出`pip list`并生成`requirements.txt`文件

   ```python
   alembic==1.2.1
   aliyun-python-sdk-core==2.13.15
   autobahn==20.4.3
   Automat==0.8.0
   bcrypt==3.1.7
   billiard==3.6.3.0
   celery==4.4.2
   certifi==2019.9.11
   cffi==1.12.3
   channels==2.4.0
   chardet==3.0.4
   Click==7.0
   constantly==15.1.0
   cryptography==2.8
   daphne==2.5.0
   dnspython==1.16.0
   dwebsocket==0.5.12
   eventlet==0.25.2
   Flask==1.1.1
   Flask-Cors==3.0.8
   Flask-Migrate==2.1.1
   Flask-MySQLdb==0.2.0
   Flask-Script==2.0.6
   Flask-SocketIO==4.3.0
   Flask-SQLAlchemy==2.4.1
   Flask-Uploads==0.2.1
   gevent==1.4.0
   greenlet==0.4.15
   gunicorn==20.0.4
   html5lib==1.0.1
   hyperlink==19.0.0
   idna==2.8
   importlib-metadata==1.6.0
   incremental==17.5.0
   itsdangerous==1.1.0
   jmespath==0.9.5
   jsonify==0.5
   monotonic==1.5
   mysqlclient==1.4.4
   npm==0.1.1
   numpy==1.18.1
   opencv-contrib-python==4.1.2.30
   opencv-python==4.1.2.30
   optional-django==0.1.0
   paramiko==2.7.1
   paypalrestsdk==1.13.1
   pbr==5.4.3
   pdfminer3k==1.3.4
   Pillow==6.2.1
   ply==3.11
   pyasn1==0.4.8
   pyasn1-modules==0.2.8
   pycodestyle==2.6.0
   pycparser==2.19
   pycryptodome==3.9.7
   pycryptodomex==3.9.4
   PyHamcrest==1.9.0
   PyJWT==1.7.1
   pymongo==3.10.1
   PyMySQL==0.9.3
   PyNaCl==1.3.0
   pyOpenSSL==19.1.0
   pysnowflake==0.1.3
   python-alipay-sdk==2.0.1
   python-dateutil==2.8.0
   python-docx==0.8.10
   python-editor==1.0.4
   python-engineio==3.13.0
   python-socketio==4.6.0
   qiniu==7.2.8
   redis==3.3.11
   requests==2.22.0
   selenium==3.141.0
   service-identity==18.1.0
   soupsieve==1.9.5
   SQLAlchemy==1.3.10
   sqlparse==0.3.1
   stevedore==1.31.0
   tornado==6.0.4
   Twisted==20.3.0
   txaio==20.4.1
   upyun==2.5.5
   urllib3==1.25.6
   vine==1.3.0
   virtualenv==16.7.7
   virtualenv-clone==0.5.3
   virtualenvwrapper==4.8.4
   virtualenvwrapper-win==1.2.5
   webencodings==0.5.1
   Werkzeug==0.16.0
   Whoosh==2.7.4
   yapf==0.30.0
   zipp==3.1.0
   zope.interface==4.7.1
   ```

   **由于只需要本项目的包，大家可以酌情删除多余的包**

8. 随后在项目目录下创建一个 Dockerfile 文件，这个文件可以理解为打包镜像的脚本，你需要这个镜像做什么，就把任务写到脚本中，Docker通过执行这个脚本来打包镜像

   ```dockerfile
   FROM python:3.6
   WORKDIR /Project/flask_back
   
   COPY requirements.txt ./
   RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
   
   COPY . .
   ENV LANG C.UTF-8
   CMD ["gunicorn", "manage:app", "-c", "./gunicorn.conf.py"]
   ```

9. 可以看到，我们项目的镜像首先基于python3.6这个基础镜像，然后声明项目目录在/Project/flask_back中，拷贝依赖表，之后安装相应的依赖，这里在安装过程中我们指定了国内的源用来提高打包速度，最后利用gunicorn运行项目，值得一提的是，ENV LANG C.UTF-8是为了声明Docker内部环境中的编码，防止中文乱码问题。

10. 最后我们就可以愉快的打包整个项目了，在项目根目录下执行

    ```python
    docker build -t 'flask_back' .
    ```

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rangyLnBuZw?x-oss-process=image/format,png)

    **这里一定要指定Docker的下载源，否则速度会非常缓慢，打包好的镜像文件大概有1g左右。**

11. 下载结束之后，可以看到myflask这个镜像已经静静躺在镜像库中了，运行

    ```python
    docker images
    ```

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rangzLnBuZw?x-oss-process=image/format,png)

    我们可以看到`flask_back`以及打包成功

12. 然后我们就可以利用这个镜像来通过容器跑Flask项目了，运行命令

    ```python
    docker run -it --rm -p 5000:5000 flask_back
    ```

13. 我们可以看到docker内部的端口5000映射到宿主机的5000端口上

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rang0LnBuZw?x-oss-process=image/format,png)

14. 通过网址访问一下，这里注意一点，就是Windows系统下，访问Docker容器需要通过分配的ip来访问，而不是我们常用的localhost。

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rang1LnBuZw?x-oss-process=image/format,png)

15. 接下来我们启动本地`flask`项目与`docker`内部镜像对比

    ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rang3LnBuZw?x-oss-process=image/format,png)

16. 对比

    - docker

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rang2LnBuZw?x-oss-process=image/format,png)

    - 本地

      ![docker](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93YW5neHMwMjAyMDIuZ2l0ZWUuaW8vaW1hZ2VzL25vdGUvZG9rang4LnBuZw?x-oss-process=image/format,png)

### 结语

**可以看到启动docker与本地项目并不会影响**

到这里我们的 Docker+Flask + Gunicorn就部署完毕了，将这个镜像上传Dockerhub仓库，在任何时间、任何地点、任何系统上，只要连着网、只要我们想，就都可以在短短1分钟之内部署好我们的项目，这就是Docker技术对开发人员最好的馈赠。

下期`利用DockerHub在Centos7.7环境下部署Nginx反向代理Gunicorn+Flask独立架构`
