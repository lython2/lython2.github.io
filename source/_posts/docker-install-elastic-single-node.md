---
title: Docker 部署单节点 ElasticSearch
tags:
  - docker
  - elasticsearch
abbrlink: f848b956
date: 2020-12-28 11:31:46
---

docker 版本19.03.2，版本 elasticsearch 版本 7.4.2，本机单节点部署。

<!-- more -->


**1. 拉取镜像**

```shell
docker pull elasticsearch:7.4.2
docker pull kibana:7.4.2
```

**2. 修改配置文件**

elasticsearch.yml

```shell
cluster.name: "single_node"
network.host: 0.0.0.0
discovery.type: "single-node"
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

**3. 创建elasticsearch存储目录**

docker elasticsearch 没有 root 用户的写权限，故此需要将存储目录更改为 1000 用户。

```shell
cd /data
mkdir -p elasticsearch/data
mkdir -p elasticsearch/logs
chown -R 1000 elasticsearch
chgrp -R 1000 elasticsearch
```

**4. 启动elasticsearch**

docker-compose.yml

```yml
version: "3.3"
services: 
   elasticsearch: 
     image: elasticsearch:7.4.2
     container_name: elasticsearch
     privileged: true
     restart: always
     environment: 
       - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
       - bootstrap.memory_lock=true
     ulimits:
       memlock:
         soft: -1
         hard: -1
     volumes:
       - /etc/localtime:/etc/localtime
       - ./config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
       - /data/elasticsearch/data:/usr/share/elasticsearch/data
       - /data/elasticsearch/logs:/usr/share/elasticsearch/logs
     logging:
       driver: "json-file"
       options:
         max-size: "10g"
     ports:
       - 9200:9200
       - 9300:9300
     networks:
       - cloud
networks:
  cloud:
    external:
      name: cloud
```

这里的 network cloud 是事先创建的，避免 docker-compose.yml 在不同目录导致网络不通。

```shell
docker network create cloud
```

**5. 使用 X-pack 创建用户名和密码**

进入 elasticsearch 容器执行

```shell
# 自动创建密码
./bin/elasticsearch-setup-passwords auto

# 手动创建密码
./bin/elasticsearch-setup-passwords interactive
```

**6. 修改 kibana 配置文件**

kibana.yml

```shell
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true

elasticsearch.username: "elastic username"
elasticsearch.password: "elastic password"
```

在 docker-compose.yml 中添加 kibana 相关配置

```yaml
kibana:
  image: kibana:7.4.2
  container_name: kibana
  privileged: true
  restart: always
  ports: 
    - 5601:5601
  environment:
    - SERVER_NAME=kibana
  volumes:
    - /etc/localtime:/etc/localtime
    - ./config/kibana.yml:/usr/share/kibana/config/kibana.yml
  logging:
    driver: "json-file"
    options:
      max-size: "2g"
  depends_on:
    - elasticsearch
  networks:
    - cloud
```

**7. 登录验证  **

浏览器访问 elasticsearch (http://127.0.0.1:9200)，访问 kibana (http://127.0.0.1:5601)，如果弹出登录框，即为验证开启成功。

注意，首次登录 kibana 的时候，需要使用 super 用户，也就是 elastic 这个账号，然后可以在 kibana 的 Management -> Security 中设置各种权限。

**8. 在http请求中进行用户验证**

和大部分数据库的 uri 访问格式一样

```shell
curl http://elastic:password@127.0.0.1:9200
```

**9. 添加 icu 分词器插件 **

进入 docker 容器执行

```shell
./bin/elasticsearch-plugin install analysis-icu

# 安装完成查看是否安装成功
./bin/elasticsearch-plugin list
```

**10. 测试分词器**

在 kibana 的 Dev Tools 中执行

```shell
GET /_analyze
{
  "analyzer": "icu_analyzer",
  "text": "北京暴雨"
}

# 返回数据
{
  "tokens" : [
    {
      "token" : "北京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "暴雨",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    }
  ]
}
```

**11. 提交新的镜像**

```shell
docker commit -a "mr.l" -m "add analysis-icu" elasticsearch elasticsearch:7.4.2-v1.0.1
```


