---
title: 在Centos7使用uWSGI部署Flask
tags:
  - uwsgi
abbrlink: 89e42a4b
---

Flask的部署方式有很多，常用的有Gunicorn、uWSGI、FastCGI、Gevent、Twisted和 tornado 等，这里简单介绍一下uWSGI，至于搭配 Nginx + uWSGI的部署方式，可以看另外一篇关于Nginx配置的文章。

<!-- more -->

---

#### 环境

操作系统：Centos 7
uWSGI版本：2.0.18
Python版本：3.6.8

#### 安装

```
yum install uwsgi
yum install uwsgi-plugin-python36
```
#### 启动

可以通过命令行，也可以通过配置文件来启动 uwsgi。

命令行方式启动
```shell
uwsgi --http-socket :5000 --plugins python36 --module myproject:app
```

注意 --plugins python36 参数必须在 --module myproject:app 参数前面。

通过配置文件启动，这里以 ini 格式为例，当然也支持 json, xml, yml 等格式。

```shell
### 启动命令
uwsgi --ini uwsgi.ini
或
uwsgi uwsgi.ini # 必须以 ini 作为后缀
```

#### 配置文件

在网上随便搜索 flask+uwsgi，就会出现许多的文章，可能是版本问题，没有一个是成功的。

比如这个，根本连不上：
```ini
[uwsgi]
master = true
home = venv
wsgi-file = manage.py
callable = app
socket = :5000
processes = 4
threads = 2
buffer-size = 32768
```

经过不停的试错，得到了下面的这个最简单的配置，亲测有效。
```ini
[uwsgi]
http-socket = 127.0.0.1:5000  #1
plugins = python36  #2
module = myapp:app  #3
```

1. 这里不是 socket, 也不是 http, 而是 http-socket。官网原话 (Do not use `--http` when you have a frontend webserver or you are doing some form of benchmark, use `--http-socket`. Continue reading the quickstart to understand why.)。
2. 插件是一定要安装的，要不然会报错。
3. myapp 为 ```flask``` 的主程序，里面要有 app 这个 flask 对象。

#### 记录Log

uwsgi 有很多的 log 插件，以那种方式记录log，就需要安装 对应的插件，然后在配置文件中，添加 logger 配置即可。它支持多种log记录方式，包括file、socket、syslog、redis、MongoDB、ZeroMq等等，还支持加密传输。
在配置文件中，添加 logger 即可。

安装日志插件

```shell
# file
yum install uwsgi-logger-file
# syslog
yum install uwsgi-logger-syslog
# redis
# syslog
yum install uwsgi-logger-redis
```

插件配置

```ini
[uwsgi]

# 文件
logger = file:logfile=/var/log/uwsgi.log,maxsize=2000000

# socket
logger = socket:/tmp/uwsgi.logsock
logger = socket:192.168.173.19:5050

# syslog
logger = socket:syslog:uwsgi1234
logger = socket:syslog:uwsgi1234,local6

# syslog
logger = socket:syslog:uwsgi1234
logger = socket:syslog:uwsgi1234,local6

# remote syslog
logger = rsyslog:12.34.56.78:12345,uwsgi1234

# redis
# --logger redislog[:<host>,<command>,<prefix>]
logger = redislog:127.0.0.1:6379,publish uwsgi
logger = rredislog:/tmp/redis.sock,rpush foo,example.com

# MongoDB
# --logger mongodblog[:<host>,<collection>,<node>]
logger = mongodblog
logger = mongodblog:127.0.0.1:27017,uwsgi.logs

# MongoDB
# --logger mongodblog[:<host>,<collection>,<node>]
logger = mongodblog
logger = mongodblog:127.0.0.1:27017,uwsgi.logs

# ZeroMQ
# uwsgi --logger zeromq:tcp://192.168.173.18:9191 --socket :3031 --module werkzeug.testapp:test_app

```

完整的配置文件

```ini
[uwsgi]

http-socket = 127.0.0.1:5000
module = myproject:app
plugins = python36,logfile
logger = file:/var/log/uwsgi.log,maxsize=2000000
```

#### 一些错误处理

*1*. invalid request block size: 21573 (max 4096)...skip

网上的解决方案是添加一个配置项 buffer-size = 32768 或者将 socket = :8000 改成 http = :8000，经过测试，如果直接添加 buffer-size = 32768，则访问无响应，如果改成 http = :8000，则 uwsgi 根本无法启动。

解决方案：把 socket 改成 http-socket。

*2*. no python application found

安装 uwsgi-plugin-python  插件

##### 题外话

用 uwsgi 部署我在腾讯云上的 Flask 服务后，没几个请求，服务就被系统给 Killed 掉了，看来  uwsgi 对内存还是有一定要求的，对于只有 1G 内存的腾讯云，还是安心的使用 tornado 吧。

---

*参考*

https://uwsgi-docs.readthedocs.io/en/latest/Configuration.html
https://flask.palletsprojects.com/en/1.1.x/deploying/wsgi-standalone/#uwsgi
https://uwsgi-docs.readthedocs.io/en/latest/Logging.html