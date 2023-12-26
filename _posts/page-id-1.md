---
title: '通过Channels在Django中释放WebSocket的能力'
date: 2023-05-19 14:32:46
img: https://wangxs020202.gitee.io/pbad/background/django-channels.png
top: true
summary: 本文将介绍如何在Django中使用Channels来实现WebSocket的功能。
categories: Django
tags:
  - Channels
  - WebSocket
  - ASGI
  - Python
  - 聊天室
---

### 一、写在前面

随着Web应用程序需求的增加，越来越多的开发人员转向实时（realtime）Web技术，其中最普遍的技术是WebSocket。然而，在使用Django进行Web开发时，处理实时和异步请求并不是一件容易的事情。
为了克服这个问题，Django引入了Channels，这是一个使Django应用程序能够处理异步和实时Web请求（包括WebSockets）的扩展库。

在本文中，我们将探讨如何使用Django Channels来开发基于WebSocket的应用程序，并将重点介绍Channels如何释放WebSocket的能力。

`作者这里使用的Django版本是4.1.7`

### 二、Channels简介

Channels 是一个采用 Django 并将其能力扩展到 HTTP 之外的项目——处理 WebSocket、聊天协议、IoT 协议等。它建立在称为ASGI 的Python
规范之上。
Channels 建立在 Django 的原生 ASGI 支持之上。虽然 Django 仍然处理传统的 HTTP，但 Channels 让您可以选择以同步或异步方式处理其他连接。

Channels 使 Django 能够处理更多类型的连接，包括 WebSocket、HTTP/2 和 ASGI，而不仅仅是 HTTP。它还允许您在 Django
中使用异步代码，而不是仅限于同步代码。这使您可以使用诸如 asyncio 和 Twisted 之类的库，这些库可以在单个进程中处理更多连接。

**tips**:
这里我们需要redis作为Channels的后端，所以需要安装redis，作者这里直接使用docker安装redis，具体安装方法可以参考[这里](https://www.runoob.com/docker/docker-install-redis.html)。

### 三、Channels的使用

#### 3.1 安装django-channels和其依赖库

```shell
pip install channels
pip install channels-redis
```

#### 3.2 配置settings.py

```python
INSTALLED_APPS = [
        ...
        'channels',
    ]
    
    # myapp是存放asgi.py的应用名
    ASGI_APPLICATION = 'myproject.asgi.application'
    
    CHANNEL_LAYERS = {
        "default": {
            "BACKEND": "channels_redis.core.RedisChannelLayer",
            "CONFIG": {
                "hosts": [('127.0.0.1', 6379)],
            },
        },
    }
```

这里将Redis作为Channel layer的后端。可以使用其他后端，比如In-Memory Layer或者asgi-redis。这里的application属性是我们自己编写的ASGI应用程序的位置。

#### 3.3 在项目的app中新建一个asgi.py文件，编写ASGI应用程序（有的话可以直接修改）

```python
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
from upp.routing import websocket_urlpatterns

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": AuthMiddlewareStack(
        URLRouter(
            websocket_urlpatterns
        )
    ),
})
```

这里我们使用get_asgi_application()
函数获取Django的默认ASGI应用程序，同时使用channels.routing模块中的ProtocolTypeRouter来指定HTTP请求和Websocket请求的处理方式。URLRouter将Websocket请求路由到相应的Consumer中。

#### 3.4 在Django项目中新建一个routing.py文件，编写WebSocket路由

```python
from django.urls import path
from . import consumers

websocket_urlpatterns = [
    path('ws/chat/', consumers.ChatConsumer.as_asgi()),
]
```

这里的写法与作用和Django的路由是一样的，只不过这里的路由是用来处理WebSocket请求的。

#### 3.5 在Django项目中新建一个consumers.py文件，编写Websocket Consumer（聊天室）

```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        await self.channel_layer.group_add(
            "chat",
            self.channel_name,
        )

        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(
            "chat",
            self.channel_name,
        )

    async def receive(self, text_data):
        data = json.loads(text_data)
        message = data["message"]

        await self.channel_layer.group_send(
            "chat",
            {
                "type": "chat.message",
                "message": message,
            },
        )

    async def chat_message(self, event):
        message = event["message"]

        await self.send(text_data=json.dumps({
            "message": message,
        }))
```

在这里，我们定义了一个名为ChatConsumer的Consumer类，处理来自“chat”频道的Websocket消息。
当一个用户连接到ChatConsumer时，我们将其添加到“chat”组中。当一个用户断开连接时，我们将其从“chat”组中删除。当一个用户发送一个消息时，我们将其发送到“chat”组中的所有用户。

#### 3.6 在Django项目中新建一个consumers2.py文件，编写Websocket Consumer（定时任务）

在`3.5`中我们实现了一下类似聊天室的功能，这里我们实现一个定时任务的功能，每隔一秒向客户端发送一个消息。

```python
import asyncio, json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async


# 定义一个异步函数，用于定时发送单个装机状态软件以及软件安装状态

class AttimeConsumer(AsyncWebsocketConsumer):
    # 接收连接
    async def connect(self):
        # 将当前连接加入到group_name组中
        await self.channel_layer.group_add("group_name", self.channel_name)
        # 接受连接
        await self.accept()
        # 创建一个任务，用于定时发送消息
        asyncio.create_task(self.send_messages())

    # 断开连接
    async def disconnect(self, close_code):
        # 将当前连接从group_name组中移除
        await self.channel_layer.group_discard("group_name", self.channel_name)
        print("断开连接")

    # 接收客户端发送的消息
    async def receive(self, text_data):
        pass

    # 发送消息
    async def send_message(self, event):
        await self.send(text_data=json.dumps(event["message"]))

    async def send_messages(self):
        # 所有装机状态以及软件安装百分比
        while True:
            await asyncio.sleep(1)  # 定时发送消息
            await self.channel_layer.group_send("group_name", {"type": "send.message", "message": 'hello'})
```

当一个用户连接到AttimeConsumer时，我们将其添加到“group_name”组中。每隔一秒，我们将一个消息发送到“group_name”组中的所有用户。
当然`asyncio.create_task(self.send_messages())`定时任务可以放在`receive`中，这样就可以根据客户端发送的消息来决定是否开启定时任务。
**同样将`AttimeConsumer`注册到routing中**

```python
from django.urls import path
from . import consumers

websocket_urlpatterns = [
    path('ws/chat/', consumers.ChatConsumer.as_asgi()),
    path('ws/attime/', consumers2.AttimeConsumer.as_asgi()),
]
```

#### 3.7 启动Django项目

这里我们使用到了ASGI服务器，所以不能使用Django自带的服务器，需要使用到`uvicorn`，安装`uvicorn`：

```shell
pip install uvicorn[standard]
```

`uvicorn`的具体用法可以参考[官方文档](https://www.uvicorn.org/)，这里我们使用`uvicorn`启动Django项目：

```shell
uvicorn myproject.asgi:application --port 8000
```

这里的`myproject.asgi:application`就是我们在`3.2`中配置的`ASGI_APPLICATION`，`--port 8000`指定端口号为8000。

### 四、效果展示

这里我们使用`ApiFox`来模拟前端发送消息，

启动界面：
![django-channels](https://wangxs020202.gitee.io/pbad/new/django-channels1.png)

ApiFox界面：
![django-channels](https://wangxs020202.gitee.io/pbad/new/django-channels.gif)

### 五、总结

相对于Flask-SocketIO，Django-Channels的使用更加简单，而且支持异步，性能更好，但是Django-Channels的文档相对于Flask-SocketIO的文档来说比较少，所以在使用的过程中可能会遇到一些问题，这里推荐一个Django-Channels的中文文档：[Django Channels 中文文档](https://channels.readthedocs.io/en/stable/)
，这里面有很多例子，可以参考。
