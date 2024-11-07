---
title: supervisor 简单使用
tags:
  - python
  - supervisor
abbrlink: 32dc2f8e
---

Supervisor是一个进程控制软件，它使用 Python 编写，可用来控制，守护 Linux 系统上的进程，它还提供了 http, rpc 服务接口，比 pm2 使用起来更舒服。


<!-- more -->

#### supervisor 安装

推荐使用`pip`来安装 supervisor，这样安装的就是最新版本，当然，也可以使用`yum`或`apt`来安装。

```shell
pip3 install supervisor
```

安装完成后，会在`/usr/local/bin`目录下生成三个可执行文件，分别为`supervisord`，`supervisorctl`和`echo_supervisord_conf`，执行`supervisord -v`查看是否安装成功。

```shell
supervisord -v
4.2.2
```

这 3 个文件，通过文件名，对其功能，就能猜个八九不离十了吧，这里就不再赘述了。

#### 创建配置文件

和使用 yum 安装 supervisor 不同，使用 pip 安装 supervisor  之后，并不会生成配置文件，需要自己手动生成。执行 `echo_supervisord_conf`命令，生成配置文件。

```shell
echo_supervisord_conf > /etc/supervisord.conf 
```

简单使用只需要修改几个选项即可，详细配置可以参考[官方文档](http://supervisord.org/configuration.html)。

**1. 开启 HTTP 服务**

```ini
;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; ip_address:port specifier, *:port for all iface
;username=user              ; default is no username (open server)
;password=123               ; default is no password (open server)

;修改为, 如果需要用户名密码，可以开启用户名和密码
[inet_http_server]          ; inet (TCP) server disabled by default
port=*:9001                 ; ip_address:port specifier, *:port for all iface
;username=user              ; default is no username (open server)
;password=123               ; default is no password (open server)
```

**2.指定守护进程配置文件目录**

```ini
;[include]
;files = relative/directory/*.ini

;修改为
[include]
files = supervisord.d/*.ini
```

supervisord.d 目录为相对目录，需要和 supervisord.conf 文件同级，需要手动创建。

```shell
mkdir -p /etc/supervisord.d
```

#### 启动 supervisord

配置文件设置完成后，执行一条简单的命令，即可启动 supervisord 服务。

```shell
supervisord -c /etc/supervisord.conf
```

然后通过`ps -ef | grep supervisord`或`supervisorctl status`查看 supervisord 是否启动成功。

#### 守护进行配置文件

supervisord 启动成功之后，接下来就该让他发挥作用，来守护我们的进程了。

在 /etc/supervisord.d 目录下创建对应配置文件，下面以 flask 为例，详细说明配置项。

supermonitor.ini

```ini
[program:flask-app]
directory = /home/flask-app
command = python3 /home/flask-app/app.py
autostart = true
autorestart = true
startsecs = 2
user = root
stderr_logfile = /home/flask-app/log/error.log
stdout_logfile = /home/flask-app/log/output.log
redirect_stderr = false
stdout_logfile_maxbytes = 20MB
stdout_logfile_backups = 10
```

- autostart，自动启动，执行 supervisorctl 命令后，自动启动
- command 可以是相对目录，也可以为绝对目录
- user 指定以什么用户来执行
- stderr_logfile 和 stdout_logfile 必须为绝对目录，如果不想打印 log，可以将其设为 NONE

#### supervisorctl 一些常用命令

| 命令    | 含义             |
| ------- | ---------------- |
| status  | 查看所有进程状态 |
| stop    | 停止某个进程     |
| start   | 启动某个进程     |
| restart | 重启某个进程     |
| update  | 更新配置文件     |
| reload  | 重启所有进程     |

#### HTTP服务

在配置文件 (supervisord.conf) 中开启 inet_http_server 选项之后，可以通过 http 来管理进程。

![WEB界面](/images/image-20210916182124338.png)

---

参考: http://supervisord.org/