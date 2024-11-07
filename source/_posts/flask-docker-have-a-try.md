---
title: Flask docker初探
tags:
  - python
  - flask
  - gevent
  - docker
abbrlink: '92640653'
date: 2020-12-28 02:06:58
description:
---


Docker: 18.06.1-ce, build e68fc7a
Flask: 1.0.2
Tornado: 5.1.1
Tepository: python:2.7.15-alpine3.7


<!-- more -->


#### 安装基础镜像

docker pull python:2.7.15-alpine3.7

#### Dockerfile

```
FROM python:2.7.15-alpine3.7
LABEL author="limu"

WORKDIR /home/app/liteweb

ADD ./ ./

RUN pip install flask
RUN pip install flask_script
RUN pip install tornado

CMD ["python", "liteweb.py"]
```


注意：ADD这一句，直接将目录中的内容添加到容器中，ADD后面直接跟目录，如果采用*ADD ./* ./会打乱目录，COPY命令也是一样。

#### 创建镜像

docker build -t liteweb:flask ./

此命令会执行Dockerfile文件，根据Dockerfile内容创建镜像，后面的路径为Dockerfile文件所在目录。

命令成功后执行 docker images可以查看镜像。

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
liteweb             flask               faf4f55bd3d5        2 hours ago         78.3MB
python              2.7.15-alpine3.7    20a59ebdfcae        6 days ago          58.5MB
```

#### 运行镜像

1. 打开终端进入容器查看

   docker run -it faf4f55bd3d5 /bin/sh

2. 添加映射端口，将容器内的80端口映射到宿主机的81端口。
   docker run -it -p 81:80 faf4f55bd3d5 /bin/sh

3. 后台方式运行

   docker run -d liteweb:flask

4. 自定义容器名称
    docker run -d --name tornado liteweb:flask

5. 挂载目录，可将宿主机的目录挂载到容器中，方便查看日志。

   docker run -d -v /home/app/logs:/home/app/liteweb/app/log liteweb:flask

6. 完整的命令
    docker run -d --name tornado -p 80:80 -v /home/app/logs:/home/app/liteweb/app/log liteweb:flask
#### 修改镜像

1. 有时候想要修改镜像，但是又不想重新build，可以进入容器修改内容，然后根据容器提交镜像。

    docker commit -a "limu" -m "add gevent" a404c6c174a2  python:flask-web

2. 修改镜像标签

    docker tag faf4f55bd3d5 liteweb:flask1.0
    
#### 保存镜像

1. 直接保存为tar包

   docker save liteweb:flask -o liteweb-flask.tar 或

   docker save liteweb:flask > liteweb-flask.tar

2. 压缩后保存
    docker save liteweb:flask | gzip > liteweb-flask.tgz

#### 加载镜像

1. 加载tar包
    docker load -i liteweb-flask.tar

2. 加载tgz包
    gunzip -c liteweb-flask.tgz | docker load

#### gcc问题
   如果需要pycrypto，gevent这样的python包，需要在alpine中安装gcc编译器，可以在Dockerfile文件中添加一条命令

RUN apk add gcc g++ make libffi-dev openssl-dev

但是这样依赖，镜像就变大了很多。
