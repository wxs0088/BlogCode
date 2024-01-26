---
title: 美多商城项目实战EP03：JSON Web Tokens
date: 2024-01-26 15:49:02
img: https://wangxs020202.gitee.io/pbad/background/mdmall.png
top: false
summary: 美多商城项目实战EP03：JSON Web Tokens
categories: 美多商城
tags:
  - 美多商城
  - Python
  - Django
  - DRF
  - JWT
---

### 一、写在前面

这篇文章主要是对`JSON Web Tokens`的介绍，以及在美多商城项目中的使用。

### 二、JSON Web Tokens

#### 2.1 什么是JSON Web Tokens

`JSON Web Tokens`是一个开放标准（RFC 7519），它定义了一种紧凑且自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

`JSON Web Tokens`可以使用HMAC算法或者是RSA的公私钥对进行签名。

#### 2.2 为什么使用JSON Web Tokens

在传统的应用中，当用户登录成功后，服务器会返回一个session_id给客户端，客户端会将session_id保存在cookie中，当用户再次访问服务器时，会将cookie中的session_id发送给服务器，服务器会根据session_id去数据库中查询用户的信息，如果查询到了，说明用户已经登录，否则用户未登录。
这种方式在单机的应用中没有问题，但是在分布式的应用中，就会有问题，因为在分布式的应用中，每台服务器都会有自己的session_id，当用户登录成功后，服务器A会返回一个session_id给客户端，客户端会将session_id保存在cookie中，当用户再次访问服务器B时，会将cookie中的session_id发送给服务器B，服务器B会根据session_id去数据库中查询用户的信息，但是由于服务器A和服务器B的session_id不一样，所以服务器B查询不到用户的信息，所以会认为用户未登录。
`JSON Web Tokens`的出现就是为了解决这个问题，当用户登录成功后，服务器会返回一个`JSON Web Tokens`给客户端，客户端会将`JSON Web Tokens`保存在cookie中，当用户再次访问服务器时，会将cookie中的`JSON Web Tokens`发送给服务器，服务器会对`JSON Web Tokens`进行解码，如果解码成功，说明用户已经登录，否则用户未登录。

#### 2.3 JSON Web Tokens的结构

`JSON Web Tokens`由三部分组成，分别是`Header`、`Payload`、`Signature`，它们之间使用`.`进行分割。

##### 2.3.1 Header

`Header`是一个JSON对象，它描述了`JWT`的元数据，例如`JWT`的类型、加密算法等等。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

##### 2.3.2 Payload

`Payload`是一个JSON对象，它包含了`JWT`的声明，例如`iss`（签发者）、`exp`（过期时间）、`sub`（主题）、`aud`（受众）等等。

```json
{
  "iss": "MDmall",
  "exp": 1622121600,
  "sub": "1",
  "aud": "web"
}
```

##### 2.3.3 Signature

`Signature`是对`Header`和`Payload`的签名，它使用`Header`中指定的加密算法进行签名，然后使用`Base64Url`编码。



### 三、美多商城项目中的使用

#### 3.1 安装依赖

```bash
pip install pyjwt
```

#### 3.2 在`base.py`文件中添加配置

```python
# jwt配置
JWT_AUTH = {
    # JWT加密所用的密钥
    'JWT_SECRET_KEY': 'mdmall',
    # JWT加密使用的算法，常用的有 HS256 和 RS256
    'JWT_ALGORITHM': 'HS256',
    # 是否验证Token过期时间，默认为True
    'JWT_VERIFY_EXPIRATION': True,
    # Token有效期，默认设置为1小时
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=7200),
}
```

#### 3.3 创建`utils`文件夹

```python
mkdir utils
```

#### 3.4 在`utils`文件夹下创建`JwtTools.py`文件

```python
import jwt
from django.conf import settings
from datetime import datetime, timedelta


class JWTTool:
    @staticmethod
    def create_token(payload):
        payload['exp'] = datetime.utcnow() + timedelta(hours=settings.JWT_AUTH['JWT_EXPIRATION_DELTA'].seconds // 3600)
        return jwt.encode(payload, key=settings.JWT_AUTH['JWT_SECRET_KEY'],
                          algorithm=settings.JWT_AUTH['JWT_ALGORITHM'])

    @staticmethod
    def validate_token(token):
        try:
            result = jwt.decode(token, key=settings.JWT_AUTH['JWT_SECRET_KEY'],
                                algorithms=[settings.JWT_AUTH['JWT_ALGORITHM']])
            if 'exp' in result and datetime.utcfromtimestamp(result['exp']) > datetime.utcnow():
                return result
        except:
            pass
        return None
```

#### 3.5 创建中间件文件

```bash
touch middlewares.py
```

#### 3.6 在`middlewares.py`文件中添加代码

```python
from django.http import JsonResponse
from django.utils.deprecation import MiddlewareMixin
from rest_framework import status
from MDmall_Backend.utils.JwtTools import JWTTool


class ProcesstMinddleware(MiddlewareMixin):
    # 在视图函数之前执行
    def process_request(self, request):
        # 获取请求的接口
        path = request.path
        filter_path = []  # 不需要验证token的接口
        request.META['REMOTE_ADDR'] = request.META.get('REMOTE_ADDR', 'unknown')
        if path.startswith('/admin'):
            return
        # 判断是否是登录接口
        if path not in filter_path:
            # 获取请求头中的token
            token = request.META.get("HTTP_AUTHORIZATION")
            res = {}
            # 判断token是否存在
            if not token:
                res["code"] = 508
                res["message"] = "token不存在"
                res["status"] = status.HTTP_200_OK
                return JsonResponse(res)
            # 验证token
            payload = JWTTool.validate_token(token)
            if not payload:
                res["code"] = 514
                res["message"] = "token验证失败,请重新登录"
                res["status"] = status.HTTP_200_OK
                return JsonResponse(res)
            # 将用户信息保存到request中
            request.user_id = payload["user_id"]

    # 在视图函数之后执行
    # def process_response(self, request, response):
    #     print(response)
    #     return response
    #
    # # 在视图函数之前，process_request 方法之后执行的
    # def process_view(self, request, view_func, view_args, view_kwargs):
    #     print("md1  process_view 方法！")
    #     print(view_func)

    # 在视图函数之后，process_response 方法之前执行的
    # def process_exception(self, request, exception):
    #     print("md1  process_exception 方法！")
    #     print(exception)
```

#### 3.7 在`base.py`文件中添加中间件

```python

MIDDLEWARE = [
    # ...
    'middlewares.ProcesstMinddleware',
]
```

