---
title: 探究Celery的分布式任务队列在Python应用中的优越性
date: 2023-04-14 22:38:40
img: https://wangxs020202.gitee.io/pbad/background/celery.png
top: false
summary: Celery是一款高性能、灵活和可靠的分布式任务队列框架，广泛应用于Python应用程序中，充分发挥其异步任务处理、负载均衡、智能调度等功能，提升了应用的性能和稳定性。本篇博客将介绍Celery的基本原理和应用场景，包括集成Flask的Web应用、爬虫和数据处理、邮件发送等方面，并结合实际案例深入探讨分布式任务队列在Python应用中的优越性。
categories: Celery
tags:
 - Celery
 - 分布式任务队列
 - 异步任务处理
 - Python
 - Django
 - Flask
---

### 一、Celery简介

Celery是一款Python语言编写的分布式任务队列框架，它基于消息中间件（如RabbitMQ、Redis等）实现异步任务的处理和调度，支持调度任务的排队、重试、回调以及监控报告等多种功能，解决了复杂业务场景下的处理效率问题。相比于其他同类框架，Celery具有更快的执行速度、更好的容错机制、更灵活的部署方式等优势。

### 二、Celery的基本原理

Celery的核心组件如下图所示：

![Celery的核心组件](https://wangxs020202.gitee.io/images/me/celery.png)

Celery的核心组件如下：

- Task 任务

Task是Celery的核心概念，代表一个需要异步执行的工作单元，开发者可以自行定义Task，并配置Task的一些属性，比如处理逻辑、超时时间、重试次数等。每个Task都必须定义一个run方法，Celery会在异步队列中选取合适的Worker执行该方法，并返回执行结果。

- Broker 中间件

Broker是Celery中任务调度和消息传递的核心组件，其作用是将程序添加到队列中，以便在消费端的Worker中执行。Celery支持多种编程语言的消息中间件，包括RabbitMQ、Redis 、AMQP等。开发者可以根据实际情况选择最合适的Broker，以保证性能和可靠性。

- Worker 工作者

Worker是Celery代码节点的执行者，它从中间件检索任务，将其发送给线程或进程池来处理任务，然后返回执行结果。我们可以使用Celery命令启动多个worker并分别配置不同的concurrency（并发数）和queues（任务队列），以实现更灵活和高效的任务调度。同时，Celery还提供了守护进程等方式运行worker，保证任务的持续运行。

### 三、Celery的应用场景

Celery的应用场景主要包括：

- Web应用

Web应用通常需要处理大量复杂的请求，其中很多操作可能需要跟第三方服务集成（比如文件上传、邮箱验证等）。这样大量的I/O操作会严重阻塞单进程应用的执行效率。通过使用Celery异步任务处理，可以将耗时的操作放到worker中异步执行，使得web应用变得响应更快、请求得到更加迅速的处理。判断与处理输入数据属性值的程序就是一个解释远程调用 Flask + Celery正在演示的场景。

- 爬虫和数据处理

很多爬虫和数据处理需要大量时间和计算资源来完成，当这些任务都阻塞在爬虫进程或数据接口的处理上时，如果采用同步串起来的方式，会让进程直接卡死或数据运算量庞大浪费 CPU 资源，而异步任务队列给了工具箱能力范式以及收获事件驱动的思想，不仅可以减轻内存峰值后会导致启动失败的负担。还能有效减少单进程数据计算的待机等待,同时重试机制也容易解决部分由于网络或者其他原因导致数据处理失败的问题，大幅度提升爬虫和数据处理的性能和可靠性。

- 邮件发送

邮件发送是一项常见且需要高可靠性的任务，但是受限于DNS查询和SMTP连接限制，频繁地发送多个电子邮件可能会导致单进程应用无法从阻塞中恢复。使用Celery异步任务队列和邮件发送插件（如Flower等），可以使得邮件任务全部放到后台worker处理，保证前端应用不卡死并且全面监管邮件本身的发送状态。

### 四、Celery的集成

#### 4.1 集成Flask

Flask是一个基于Python的轻量级Web应用框架，它提供了简单的API来实现Web应用的开发。Celery可以很方便地集成到Flask应用中，实现异步任务的处理和调度。

下面我们以一个简单的使用Flask+Celery搭建任务队列的Web应用为例进行说明，期望通过 Celery 来发挥异步任务在实际业务场景中的各种优势。

首先按照Celery官网的安装指引，安装好必备的依赖之后，在项目目录下编辑main.py文件来定义Celery的运行以及如何处理任务。

```python
import time

#创建celery实例对象
app = Celery("tasks",
broker="redis://localhost:6379/0",
backend="redis://localhost:6379/0")

#定义任务函数，并设置超时参数
@app.task(bind=True, default_retry_delay=60, max_retries=3)
def task_logout(self, user_email):
try:
#执行具体操作，并返回结果信息
print(f"Start to logout for user {user_email} at {time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())}")
......
return "User has been logged out"
except Exception as ex:
    #如果出现异常，则重试
    self.retry(exc=ex)
```

在main.py文件中，我们首先创建了一个Celery实例对象app，并配置了broker和backend。broker用于接收任务，backend用于存储任务执行结果。然后我们定义了一个task_logout的任务函数，该函数接收一个user_email参数，执行具体的操作，并返回执行结果。

#### 4.2 集成Django

Django是一个基于Python的开源Web应用框架，它遵循MVT（Model-View-Template）的设计模式，提供了强大的ORM功能。Celery可以很方便地集成到Django应用中，实现异步任务的处理和调度。

下面我们以一个简单的使用Django+Celery搭建任务队列的Web应用为例进行说明，期望通过 Celery 来发挥异步任务在实际业务场景中的各种优势。


1、安装对应的库

```python
pip3 install celery==4.4.2
pip3 install eventlet==0.25.2
pip3 install Django==2.0.4
```

2、配置settings文件：

```python
#redis
CELERY_BROKER_URL = 'redis://localhost:6379/'
#broker配置redis
CELERY_RESULT_BACKEND = 'redis://localhost:6379/'
#文件格式为：json
CELERY_RESULT_SERIALIZER = 'json'
```

3、在settings文件同级目录创建celery.py

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# 设置环境变量
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'Django项目名称.settings')

# 注册Celery的APP
app = Celery('Django项目名称')
# 绑定配置文件
app.config_from_object('django.conf:settings', namespace='CELERY')

# 自动发现各个app下的tasks.py文件
app.autodiscover_tasks()

#如果有需要可以将该任务设置成定时任务
from celery.schedules import crontab
CELERY_BEAT_SCHEDULE = {
    # 周期性任务
    'task-one': {
        'task': 'myapp.tasks.print_test',
        'schedule': 周期时间,
        # 'args': ()
    }
}
```

4、修改settings文件同级目录的init.py文件

```python
from __future__ import absolute_import, unicode_literals
from .celery import app as celery_app

#导包
import pymysql
#初始化
pymysql.install_as_MySQLdb()



__all__ = ['celery_app']
```

5、在应用中创建tasks.py文件

```python
from celery.task import task

# 自定义要执行的task任务
@task
def print_test():
	print("nict try")
	return 'hello'
```

6、在视图页面进行调用

```python
from myapp import tasks

def ctest(request,*args,**kwargs):
    res=tasks.print_test.delay()
    #delay方法就是异步方式请求
    #任务逻辑
    return JsonResponse({'status':'successful','task_id':res.task_id})
```

7、 在manage.py的目录下启动celery服务

```python
celery worker -A mydjango -l info -P eventlet
```

8、 在浏览器中调用异步服务接口

![celery](https://wangxs020202.gitee.io/images/me/celery1.png)

同时也可以在backend中查询任务结果

![celery](https://wangxs020202.gitee.io/images/me/celery2.png)

*****注：redis中的key并不是单纯的task_id，而是需要加上前缀celery-task-meta-

9、最后，如果需要启动定时任务，就需要在manage.py所在的文件夹内单独启动beat服务

```python
celery -A mydjango beat -l info
```

### 五、结语

本文主要介绍了Celery的基本概念、应用场景、使用、集成等内容，希望能够帮助到大家。
