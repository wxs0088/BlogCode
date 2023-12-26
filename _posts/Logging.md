---
title: 在Python世界中游刃有余：如何灵活运用Django日志记录功能
date: 2023-04-24 14:35:18
img: https://www.sentinelone.com/wp-content/uploads/2022/02/Django_logging_with_Scalyr_colors.png
top: false
summary: 本文将介绍如何在Django中使用日志记录功能，以及如何使用Django自带的日志记录功能。
categories: Django
tags:
- Python
- Django
- 日志记录
- Logging
---

### 一、写在前面

Django 是一个流行的 Python Web 框架，并且在开发过程中，日志记录是很重要的一部分。随着应用程序变得越来越复杂，我们不可避免地需要更细致、更高级的调试和追踪工具。
在Django的世界里，logging（日志记录）成了一个非常重要并且强大的工具。它可以帮助我们记录所有应用程序运行过程中的事件，从而让开发者能够更好地理解应用程序以及其中代码的工作方式。

那么，在Django中如何使用logging来分析我们的代码运行时出现的问题呢？接下来，我将会向您介绍如何利用Django 4.1.7
中的logging来提高开发质量和效率。

### 二、什么是 logging

logging到底是什么：简单地说，logging就是一种把程序运行时的信息（调试信息、错误、警告等）输出到指定位置的技术。当应用程序崩溃或者在运行上出现其他问题时，这些信息就会被日志记录器捕获，并显示给开发者来分析和理解问题原因。

logging日志还可以在开发者的控制台、文件甚至是邮件中显示，从而让开发者快速地了解应用程序的运行情况。

### 三、Django中的logging

Django提供了一组可定制的日志记录方式，以便开发者能够追踪他们的应用程序从框架到应用程序本身所打印的日志记录。Django提供的logging配置会输出所有的错误、警告和信息消息。如果你想在调试过程中获取更详细的数据，那么logging就是您非常重要的一个工具。

1. Django日志记录器的默认配置：

   根日志级别 (Root Logger Level)：WARNING
   在指定文件中保存（Rotate）日志
   固定格式，包含日期时间戳、日志分类、模块名称和日志等级
   这里是一个简单的例子，演示如何启用日志系统并记录一些信息。

    ```python
    import logging
    
    logger = logging.getLogger(__name__)
    
    def my_view(request, arg1, arg):
        # do something interesting 
        logger.info('Receiving request from %s', request)
        try:
            result = some_function(arg1, arg2)
        except Exception:
            logger.exception("Error when calling some_function()")
        return HttpResponse(result)
    ```
   上面的例子顺序地展示了创建一个日志记录器实例 logger = logging.getLogger(__name__) 。 然后，在 my_view 函数中使用它来记录信息。

   其中，通过例子可以看出 logging 模块结合 Django 包含以下几部分：

   Module 信息
   日志记录级别
   日志分发处理器
   格式化信息
   过滤器
   Python标准库中的logging模块提供了一个灵活的机制，允许开发人员自定义筛选日志事件以及将其路由到目标位置的方法。这允许开发者采取不同的方式来处理应用程序的多个日志消息类型并按需修改配置。

2. 日志输出等级（Log Levels）

   在logging系统中，存在5种默认记录级别，列举如下：

    1. DEBUG：详细的日志记录，通常只出现在诊断问题上
    2. INFO：证明事情按预期工作
    3. WARNING：指示可能出了问题，但直接影响程序正常运行的级别
    4. ERROR：由于更严重的问题，该软件无法执行某些功能
    5. CRITICAL：指出非常严重的错误，表明程序本身可能无法继续进行
       通过 logging 模块的 setLevel 来确定记录的等级。如果您将记录器级别设置为 DEBUG，则所有可用级别的日志记录都可以被记录。如果您将其设置为
       ERROR，则所有 ERROR 和 CRITICAL 级别的日志将被记录，但其他等级的日志则不会被处理。

3. 日志输出格式

   另一个重要的logging配置是日志记录格式。Django使用string format语法的logging.basicConfig()方法，可以很容易地自定义输出的日志信息。

   格式化字符串中，可以使用以下一些占位符：

    1. %(asctime)s：日志记录的时间
    2. %(levelname)s：日志记录的级别
    3. %(name)s：日志记录器的名称
    4. %(message)s：日志记录的消息
    5. %(filename)s：产生日志记录的文件名
    6. %(module)s：产生日志记录的模块名
    7. %(funcName)s：产生日志记录的函数名
    8. %(lineno)d：产生日志记录的代码行号
    9. %(process)d：进程ID
    10. %(thread)d：线程ID
    11. %(threadName)s：线程名称
    12. %(pathname)s：产生日志记录的代码文件的完整路径名
    13. %(processName)s：进程名称
        下面是一个例子显示了如何设置自己的格式化。

    ```python
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
        },
    
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    
        # 标准输出格式
        'standard': {
            # (日志级别ID) [具体时间][线程名:线程ID][日志名字:])] [输出的模块:输出的函数]:日志内容
            'format': '%(levelname)s [%(asctime)s] %(process)d %(thread)d "%(pathname)s/%(funcName)s:%(lineno)d" %(message)s',
            'datefmt': '%d/%b/%Y:%H:%M:%S %z',
            # 23/Dec/2017:14:21:40 +0800
        },
    }
    ```
   其中， LOGGING 中定义了根记录器的配置。 它表示Django默认情况下为我们安装和配置的 logging系统。
   我们通过继承默认的配置并对其进行修改来对其进行自定义。

   除此之外，你还可以设置其他函数来更加精确地控制日志处理。例如，您可以定义过滤器来动态更改日志级别等。

   `loggerFilter.py`
    ```python
    import logging
    class UrlFilter(logging.Filter):
        def filter(self, record):
            urls_to_ignore = ['/api/template/get_software/', '/api/software/software_install_status/']
            for url in urls_to_ignore:
                if record.getMessage().startswith('"POST {}'.format(url)):
                    return False
            return True
    ```
   `settings.py`
    ```python
    from loggerFilter import *
    'filters': {
        # 只有在Django debug为True时才在屏幕打印日志
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
        # 过滤url
        'url_filter': {
            '()': UrlFilter,
        },
    }
    ```

4. 日志分发处理器

   logging模块提供了多种处理器，用于将日志消息发送到不同的目的地。Django默认配置中的处理器有：

    1. NullHandler：不做任何处理
    2. StreamHandler：将日志消息发送到输出流，如sys.stderr, sys.stdout或任何文件-like对象
    3. FileHandler：将日志消息发送到磁盘文件，默认情况下文件大小会无限增长
    4. SMTPHandler：将日志消息以电子邮件的形式发送给管理员
    5. HTTPHandler：将日志消息以GET或POST的形式发送到一个HTTP服务器
    6. BaseRotatingHandler：基类，用于将日志消息发送到磁盘文件，并支持日志文件的自动切割
    7. RotatingFileHandler：继承自BaseRotatingHandler，用于将日志消息发送到磁盘文件，并支持日志文件的自动切割
    8. TimedRotatingFileHandler：继承自BaseRotatingHandler，用于将日志消息发送到磁盘文件，并支持日志文件的按时间自动切割
    9. SocketHandler：将日志消息发送到网络套接字
    10. DatagramHandler：将日志消息发送到网络套接字，但使用UDP协议
    11. SysLogHandler：将日志消息发送到UNIX系统的syslogd守护进程
    12. NTEventLogHandler：将日志消息发送到Windows NT/2000/XP的事件日志
    13. MemoryHandler：将日志消息发送到内存中的制定buffer
    14. HTTPHandler：将日志消息以GET或POST的形式发送到一个HTTP服务器
    15. QueueHandler：将日志消息发送到multiprocessing.Queue对象
    16. QueueListener：从multiprocessing.Queue对象中读取消息，并将这些消息发送到指定的处理器

   除了上面列出的处理器，Django还提供了一些自定义的处理器，用于将日志消息发送到不同的目的地。Django默认配置中的处理器有：

    1. AdminEmailHandler：将日志消息以电子邮件的形式发送给管理员
    2. ColorizingStreamHandler：将日志消息发送到输出流，如sys.stderr, sys.stdout或任何文件-like对象，并且可以为不同的日志级别设置不同的颜色
    3. SafeExceptionReporterFilter：过滤掉敏感信息，如数据库密码等
    4. ServerFormatter：格式化日志消息，用于将日志消息发送到远程服务器
    5. SMTPFormatter：格式化日志消息，用于将日志消息以电子邮件的形式发送给管理员
    6. UnicodeStreamHandler：将日志消息发送到输出流，如sys.stderr, sys.stdout或任何文件-like对象，并且可以为不同的日志级别设置不同的颜色

   除了上面列出的处理器，Django还提供了一些自定义的处理器，用于将日志消息发送到不同的目的地。Django默认配置中的处理器有：

    1. AdminEmailHandler：将日志消息以电子邮件的形式发送给管理员
    2. ColorizingStreamHandler：将日志消息发送到输出流，如sys.stderr, sys.stdout或任何文件-like对象，并且可以为不同的日志级别设置不同的颜色
    3. SafeExceptionReporterFilter：过滤掉敏感信息，如数据库密码等


5. 配置示例

```python
LOGGING = {
    'version': 1,
    # 是否启用日志配置
    'disable_existing_loggers': False,
    # 日志文件的配置-格式化输出
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
        },

        'simple': {
            'format': '%(levelname)s %(message)s'
        },

        # 标准输出格式
        'standard': {
            # (日志级别ID) [具体时间][线程名:线程ID][日志名字:])] [输出的模块:输出的函数]:日志内容
            'format': '%(levelname)s [%(asctime)s] %(process)d %(thread)d "%(pathname)s/%(funcName)s:%(lineno)d" %(message)s',
            'datefmt': '%d/%b/%Y:%H:%M:%S %z',
            # 23/Dec/2017:14:21:40 +0800
        },
    },
    # 过滤器
    'filters': {
        # 只有在Django debug为True时才在屏幕打印日志
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
        # 过滤url
        'url_filter': {
            '()': UrlFilter,
        },
    },
    # 处理器
    'handlers': {
        # 控制台输出
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'filters': ['require_debug_true', 'url_filter'],
            'formatter': 'simple',
        },
        # 调试信息
        'info_file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filters': ['url_filter'],
            'filename': os.path.join(BASE_DIR, 'logs/info.log'),  # 输出位置(先创建文件夹)
            'formatter': 'standard',
            'maxBytes': 5 * 1024 * 1024,
            'backupCount': 10
        },
        # django日志
        'django_file': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs/django.log'),  # 输出位置(先创建文件夹)
            'formatter': 'simple',
            'maxBytes': 5 * 1024 * 1024,
            'backupCount': 100
        },
        # 重大错误日志
        'error_file': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(BASE_DIR, 'logs/error.log'),  # 输出位置(先创建文件夹)
            'formatter': 'standard',
            'maxBytes': 5 * 1024 * 1024,
            'backupCount': 10
        }
    },
    # 日志记录器
    'loggers': {
        # 自定义日志
        '': {
            'handlers': ['info_file', 'console'],
            'level': 'INFO',
            'propagate': False,
        },
        # django日志
        'django': {
            'handlers': ['django_file'],
            'level': 'DEBUG',
            'propagate': True,
        },
        # 错误日志
        'error': {
            'handlers': ['error_file'],
            'level': 'ERROR',
            'propagate': False,
        }
    },
}
```

### 四、总结

Django 内置的 logging 库提供了强大和灵活的日志记录功能，可以方便地满足不同的需求。本文介绍了 Django 中 logger
的基本配置和使用方法，希望能给读者带来一些思路。再次强调，在生产环境中日志记录很重要，因此应该专注于正确配置 logger
以及在应用程序中使用它来记录日志。

最后，提醒一下：正确地配置 logging 是必要且重要的一步，因为它直接影响实际应用程序的性能。因此，我们应该使用得当而不是滥用它。
