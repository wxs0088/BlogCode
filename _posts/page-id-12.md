---
title: 释放 Gunicorn、Nginx 和 Daphne 在服务器中对 Django 部署中的威力
date: 2023-08-17 19:14:41
img: https://wangxs020202.gitee.io/pbad/background/page_12.png
top: false
summary: 通过使用Gunicorn、Nginx 和 Daphne 来部署 Django 项目，可以大大提高网站的性能。
categories: Django
tags:
    - Django
    - Gunicorn
    - Nginx
    - Daphne
    - scoket
    - 部署
    - python
---
### 序幕

使用Gunicorn部署Django项目可以达到很好的效果，但是无法获取到后端的静态资源很是烦恼，本文将Gunicorn、Daphne做成系统的服务，再配合上Nginx，可以很好的解决这个问题。
接下来我们将一步步的来实现这个功能。


## 后端部署

### 一、安装依赖

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 二、修改后端配置文件

`/Settings/base.py`

```bash
DEBUG = False
IS_LINUX = True
```

### 三、swagger页面

1. 启动后端（由于Gunicorn找不到静态CSS内容，这里使用manage启动）

   ```bash
   python manange.py runserver 0.0.0.0:8000
   ```

2. 错误解决（由于版本不兼容需要手动修改swagger）

   ![p1](https://wangxs020202.gitee.io/pbad/new/p1.png)

   找到安装目录

   ```bash
   pip show django-rest-swagger
   ```

   修改index.html文件 第二行的`staticfiles`修改为`static`

   ```bash
   cd /usr/local/lib/python3.9/site-packages
   cd rest_framework_swagger/templates/rest_framework_swagger
   vim index.html
   ```

   保存静态文件（后续nginx代理会用到）

   ```bash
   python manage.py collectstatic
   ```

3. 重启服务

   ```bash
   python manange.py runserver 0.0.0.0:8000
   ```

4. 查看效果（/api/docs）

   ![p2](https://wangxs020202.gitee.io/pbad/new/p2.png)

###  四、为 Gunicorn 创建 systemd Socket 和服务文件

1. 以`sudo` 的权限为 Gunicorn 创建并打开一个 systemd 套接字文件

   ```bash
   sudo vim /etc/systemd/system/gunicorn.socket
   ```

2. 在文件中，创建一个`[Unit]` 部分来描述套接字，一个`[Socket]` 部分来定义套接字的位置，还有一个`[Install]` 部分来确保套接字在正确的时间被创建。

   ```txt
   [Unit]
   Description=gunicorn socket
   
   [Socket]
   ListenStream=/run/gunicorn.sock
   
   [Install]
   WantedBy=sockets.target
   ```

3. 以`sudo` 的权限为 Gunicorn 创建并打开一个 systemd 服务文件。服务文件的名称应与扩展名的套接字文件名称一致。

   ```bash
   sudo vim /etc/systemd/system/gunicorn.service
   ```

4. `[Unit]` 用于指定元数据和依赖关系。需要在这里写上你的服务的描述，并告诉init系统只有在达到联网目标后才启动这个服务。因为该服务依赖于套接字文件中的套接字，所以需要加入一个`Requires` 指令来表明这种关系。

   ```txt
   [Unit]
   Description=gunicorn daemon
   Requires=gunicorn.socket
   After=network.target
   ```

5. `[Service]` 部分。需指定该进程的用户和组。必须指定Gunicorn可执行文件的完整路径。将进程绑定到在`/run` 目录中创建的Unix套接字上，以便进程能够与Nginx通信。将所有数据记录到标准输出，以便`journald` 进程能够收集Gunicorn日志。还可以在这里指定任何可选的Gunicorn调整。下面是一个指定三个工作进程的例子。

   ```txt
   [Unit]
   Description=gunicorn daemon
   Requires=gunicorn.socket
   After=network.target
   
   [Service]
   User=root
   Group=apache
   WorkingDirectory=/BackendDir
   ExecStart=gunicorn \
             --access-logfile - \
             -k uvicorn.workers.UvicornWorker \
             --workers 3 \
             --bind unix:/run/gunicorn.sock \
             BackendApp.asgi:application
   ```

6. `[Install]` 部分。这将告诉systemd，如果让这个服务在开机时启动，应该把它链接到什么地方。

   ```txt
   [Unit]
   Description=gunicorn daemon
   Requires=gunicorn.socket
   After=network.target
   
   [Service]
   User=root
   Group=apache
   WorkingDirectory=/BackendDir
   ExecStart=gunicorn \
             --access-logfile - \
             -k uvicorn.workers.UvicornWorker \
             --workers 3 \
             --bind unix:/run/gunicorn.sock \
             BackendApp.asgi:application
   [Install]
   WantedBy=multi-user.target
   ```

7. 现在可以启用Gunicorn套接字。这将在`/run/gunicorn.sock` ，并在启动时创建套接字文件。 当有连接进入该套接字时，systemd 会自动启动`gunicorn.service` 来处理。

   ```bash
   sudo systemctl start gunicorn.socket
   sudo systemctl enable gunicorn.socket
   ```

### 五、检查Gunicorn套接字文件的存在

1. 检查Gunicorn套接字文件的情况。首先，检查进程的状态，了解它是否能够启动。

   ```bash
   sudo systemctl status gunicorn.socket
   ```

   `output`

   ```bash
   ● gunicorn.socket - gunicorn socket
        Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; preset: disabled)
        Active: active (running) since Mon 2023-08-14 15:40:25 CST; 17h ago
         Until: Mon 2023-08-14 15:40:25 CST; 17h ago
      Triggers: ● gunicorn.service
        Listen: /run/gunicorn.sock (Stream)
        CGroup: /system.slice/gunicorn.socket
   
   Aug 14 15:40:25 localhost systemd[1]: Listening on gunicorn socket.
   ```

2. 检查`/run` 目录下的`gunicorn.sock` 文件是否存在。

   ```bash
   file /run/gunicorn.sock
   ```

   `output`

   ```bash
   /run/gunicorn.sock: socket
   ```

3. 如果`systemctl status` 命令表明发生了错误，或者在该目录中没有找到`gunicorn.sock` 文件，这表明Gunicorn套接字没有被正确创建。通过检查Gunicorn套接字的日志。

   ```bash
   sudo journalctl -u gunicorn.socket
   ```

   **在继续之前确保以上的步骤没有任何问题。**

### 六、测试套接字的激活

1. 只启动了`gunicorn.socket` 单元，`gunicorn.service` 还没有被激活，因为套接字还没有收到任何连接。可以通过输入以下命令来检查这一点。

   ```bash
   sudo systemctl status gunicorn
   ```

   `output`

   ```bash
   ● gunicorn.service - gunicorn daemon
      Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
      Active: inactive (dead)
   ```

2. 通过输入`curl` ，向套接字发送一个连接。

   ```bash
   curl --unix-socket /run/gunicorn.sock localhost
   ```

   应该在终端中收到来自你的应用程序的HTML输出。这表明Gunicorn已经启动并且能够为你的Django应用程序提供服务。

3. 再次查看Gunicorn服务是否在运行。

   ```bash
   sudo systemctl status gunicorn
   ```

   `output`

   ![p3](https://wangxs020202.gitee.io/pbad/new/p3.png)

4. 如果`curl` 的输出或`systemctl status` 的输出表明发生了问题，请检查日志。

   ```bash
   sudo journalctl -u gunicorn
   ```

   检查你的`/etc/systemd/system/gunicorn.service` 文件是否有问题。如果对`/etc/systemd/system/gunicorn.service` 文件进行了修改，重新加载守护进程以重读服务定义。

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart gunicorn
   ```

   **在继续之前确保以上的步骤没有任何问题。**

### 七、配置Nginx以代理传递给Gunicorn

至此Gunicorn已经设置好了，接下来需要配置Nginx将流量传递给该进程。

1. 在`Nginx`的配置目录下创建代理配置`proxy_params`文件（如果已存在跳过此步）

   ```bash
   vim /etc/nginx/proxy_params
   ```

   输入以下内容

   ```txt
   proxy_set_header Host $http_host;
   
   proxy_set_header X-Real-IP $remote_addr;
   
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   
   proxy_connect_timeout 30;
   
   proxy_send_timeout 60;
   
   proxy_read_timeout 60;
   
   proxy_buffering on;
   
   proxy_buffer_size 32k;
   
   proxy_buffers 4 128k;
   ```

2. 创建后端配置文件`backend.conf`

   ```bash
   vim /etc/nginx/conf.d/backend.conf
   ```

3. 指定这个块应该监听正常的端口，并且应该响应服务器的域名或IP地址。

   ```txt
   server {
       listen 89;
       server_name localhost;
   }
   ```

4. 忽略任何寻找favicon的问题，配置后端静态资源访问路径

   ```txt
   server {
       listen 89;
       server_name localhost;
   
       location = /favicon.ico { access_log off; log_not_found off; }
       location /static {
           root /BackendDir;
       }
   ```

5. 创建一个`location / {}` 块，以匹配所有其他请求。在这个位置，将包括Nginx的`proxy_params` 文件，然后你将把流量直接传递给Gunicorn套接字。

   ```txt
   server {
       listen 89;
       server_name localhost;
   
       location = /favicon.ico { access_log off; log_not_found off; }
       location /static {
           root /BackendDir;
       }
   
       location / {
           include proxy_params;
           proxy_pass http://unix:/run/gunicorn.sock;
       }
   }
   ```

6. 测试Nginx配置是否有语法错误。

   ```bash
   sudo nginx -t
   ```

   `output`

   ```bash
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successfu
   ```

   如果遇到错误，检查`/etc/nginx/conf.d/backend.conf`语法是否正确

7. 重启`Nginx`

   ```bash
   sudo systemctl restart nginx
   ```

   现在应该可以域名或IP地址进入服务器，查看`django-admin`以及`swagger`页面是否可正常访问。

   ![p4](https://wangxs020202.gitee.io/pbad/new/p4.png)

   ![p5](https://wangxs020202.gitee.io/pbad/new/p5.png)

   如果页面还是无法正常访问，请检查项目目录下`static/`相关的静态文件是否存在。可通过以下命令查看`Nginx`的错误日志。

   ```bash
   sudo less /var/log/nginx/error.log
   ```

8. 检测接口是否可以正常访问

   ![p6](https://wangxs020202.gitee.io/pbad/new/p6.png)

### 八、为 Daphne创建服务文件

1. 以`sudo` 的权限为 daphne 创建并打开一个 systemd 套接字文件

   ```bash
    sudo vim /etc/systemd/system/daphne.service
   ```

2. 格式与Gunicorn一致，**ExecStart**使用`daphne`启动

   ```txt
   [Unit]
   Description=WebSocket Daphne Service
   After=network.target
   
   [Service]
   User=root
   Group=root
   WorkingDirectory=/BackendDir
   ExecStart=daphne -u /tmp/daphne.sock BackendApp.asgi:application
   Restart=on-failure
   
   [Install]
   WantedBy=multi-user.target
   ```

3. 启动`Daphne`配置文件

   ```bash
   sudo systemctl start daphne.service
   sudo systemctl status daphne.service
   ```

   `output`

   ```bash
   ● daphne.service - WebSocket Daphne Service
        Loaded: loaded (/etc/systemd/system/daphne.service; disabled; preset: disabled)
        Active: active (running) since Mon 2023-08-14 17:40:20 CST; 17h ago
      Main PID: 56416 (daphne)
         Tasks: 2 (limit: 45339)
        Memory: 50.6M
           CPU: 20.584s
        CGroup: /system.slice/daphne.service
                └─56416 /usr/bin/python3 /usr/local/bin/daphne -u /tmp/daphne.sock BackendApp.asgi:application
   
   Aug 14 17:40:20 localhost systemd[1]: Started WebSocket Daphne Service.
   ```

### 九、配置Nginx以代理Daphne

1. 修改后端`nginx`配置文件

   ```bash
   sudo vim /etc/nginx/conf.d/backend.conf
   ```

2. 添加`Daphne`代理配置

   ```txt
   server {
       listen 89;
       server_name localhost;
   
       location = /favicon.ico { access_log off; log_not_found off; }
       location /static {
           root /BackendDir;
       }
   
       location / {
           include proxy_params;
           proxy_pass http://unix:/run/gunicorn.sock;
       }
       
       location /ws/ {
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_redirect off;
            proxy_pass http://unix:/tmp/daphne.sock;
        }
   }
   ```

3. 测试Nginx配置是否有语法错误。

   ```bash
   sudo nginx -t
   ```

   `output`

   ```bash
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successfu
   ```

   如果遇到错误，检查`/etc/nginx/conf.d/backend.conf`语法是否正确

4. 重启`Nginx`

   ```bash
   sudo systemctl restart nginx
   ```

### 十、对后端代码进行编译加密

1. 查看脚本参数

   ```bash
    python BackendDir/compile_pyc.py -h
   ```

   `output`

   ```bash
   在命令行中使用以下命令:
           python compile_pyc.py clean_pyc PATH            #清除当前项目中所有的pyc文件
           python compile_pyc.py compile_to_pyc PATH               #生成pyc文件
           python compile_pyc.py remove_py PATH            #删除py文件
           python compile_pyc.py move_to_path PATH         #移动所有pyc文件至原位置
           python compile_pyc.py replace_name PATH         #修改文件名文件
           python compile_pyc.py remove_pycache_dir PATH           #清除当前项目中所有的__pycache__文件夹
           python compile_pyc.py one_step PATH             #清除已有的pyc => 加密 => 删除py => 移动至原位置 => 重命名 => 清除__pycache__文件夹
   ```

2. 使用`one_step`对后端文件进行编译加密

   ```bash
   python BackendDir/compile_pyc.py one_step BackendDir/
   ```

### 十一、故障排查

以下日志可能会有帮助。

- 通过检查Nginx进程日志。`sudo journalctl -u nginx`
- 通过检查Nginx的访问日志。`sudo less /var/log/nginx/access.log`
- 通过检查Nginx的错误日志。`sudo less /var/log/nginx/error.log`
- 通过检查Gunicorn应用程序的日志。`sudo journalctl -u gunicorn`
- 通过检查Gunicorn套接字的日志。`sudo journalctl -u gunicorn.socket`
- 通过检查Daphne应用程序的日志。`sudo journalctl -u daphne`

更新配置或应用程序时，需要重新启动这些进程。

如果更新了Django应用程序，可以通过重启Gunicorn进程。

```Bash
sudo systemctl restart gunicorn
```

如果改变了Gunicorn的套接字或服务文件，请重新加载守护进程，并重启该进程。

```Bash
sudo systemctl daemon-reloadsudo systemctl restart gunicorn.socket gunicorn.service
```

如果改变了Nginx服务器块的配置，测试配置，然后通过键入Nginx。

```Bash
sudo nginx -t && sudo systemctl restart nginx
```

## 前端部署

### 一、打包前端为静态文件

**在打包之前确保已安装并可执行前端代码的前端环境**

1. 在前端项目目录下，运行以下命令

   ```bash
   npm run build:prod
   ```

2. 打包结束后会在项目目录下生成`dist`目录，目录结构大概如下

   ```bash
   [root@localhost dist]# tree -d
   .
   ├── driver_png
   ├── icon
   ├── software_png
   └── static
       ├── css
       ├── fonts
       ├── img
       └── js
   ```

### 二、使用`Nginx`配置前端静态文件

1. 修改`/etc/nginx/conf.d/default.conf`文件（也可以在新增配置文件中添加）。监听正常的端口，并且应该响应服务器的域名或IP地址。

   ```bash
   server {
       listen       88;
       server_name  localhost;
   }
   ```

2. 创建`location / {}` 块，代理前端静态文件，同时指定首页以及定义`try_files`。

   ```txt
   server {
       listen       88;
       server_name  localhost;
   
       location / {
           root   /FrontendDistDir;
           index  index.html index.htm;
           try_files $uri $uri/ /index.html;
       }
   }
   ```

3. 配置接口代理，由于前端在打包时不会打包代理需要在`nginx`配置代理

   ```txt
   server {
       listen       88;
       server_name  localhost;
   
       location / {
           root   /FrontendDistDir;
           index  index.html index.htm;
           try_files $uri $uri/ /index.html;
       }
   
       location ^~ /prod-api/fog/ {
           proxy_pass http://localhost/fog/;
       }
   
       location ^~ /prod-api/ {
           proxy_pass http://localhost:89/api/;
       }
   }
   ```

4. 测试Nginx配置是否有语法错误。

   ```bash
   sudo nginx -t
   ```

   `output`

   ```bash
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successfu
   ```

   如果遇到错误，检查`/etc/nginx/conf.d/default.conf`语法是否正确

5. 重启`Nginx`

   ```bash
   sudo systemctl restart nginx
   ```

   ![p7](https://wangxs020202.gitee.io/pbad/new/p7.png)

   
