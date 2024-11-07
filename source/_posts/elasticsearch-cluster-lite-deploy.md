---
title: 搭建一个简单的 ElasticSearch 集群
tags:
  - elasticsearch
  - docker
abbrlink: da97202b
date: 2020-12-28 12:06:46
---

这里以docker的方式，搭建3个节点的最小集群，未涉及到生命周期，冷热数据等，elasticsearch 版本 7.4.2。

<!-- more -->

| 机器名称 | IP地址       |
| -------- | ------------ |
| node1    | 192.168.10.1 |
| node2    | 192.168.10.2 |
| node3    | 192.168.10.3 |

#### 身份认证与鉴权

为了保护 elasticsearch 的数据安全，需要配置 elasticsearch 的用户密码和角色权限；为了保护集群内部的通信安全，需要配置集群节点的数字证书。

先启动一个 elasticsearch 节点，来创建用户密码和证书。

elasticsearch.yml 初步配置

```yaml
cluster.name: "elastic"
network.host: 0.0.0.0
network.publish_host: 192.168.10.1
node.name: "node1"
cluster.initial_master_nodes: ["node1"]
```

启动 elasticsearch，进入 elasticsearch目录。

创建用户密码

```shell
bin/elasticsearch-setup-passwords interactive
```

创建一个证书颁发机构

```
bin/elasticsearch-certutil ca
```

默认情况下会在当前目录生成一个 elastic-stack-ca.p12 文件，然后使用这个文件创建一个私有证书。

```
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

默认情况下会在当前目录生成一个 elastic-certificates.p12 的文件，将这两个文件拷贝出来，elastic-stack-ca.p12 文件妥善保管，

而 elastic-certificates.p12 文件即为鉴权证书，如果节点没有此证书，则其不会加入集群。

#### 节点1

在 elasticsearch 的 config 目录下创建 cert 目录，然后将 elastic-certificates.p12 文件拷贝到 cert 目录中。

修改 elasticsearch.yml，添加身份认证部分配置，重新启动 elasticsearch。

elasticsearch.yml 配置文件：

```yaml
# node1 config
cluster.name: "elastic"
network.host: 0.0.0.0
network.publish_host: 192.168.10.1
node.name: "node1"
cluster.initial_master_nodes: ["node1"]
discovery.seed_hosts: ["192.168.10.1", "192.168.10.2", "192.168.10.3"]

xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: cert/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: cert/elastic-certificates.p12
```

*1.* cluster.initial_master_nodes 初始化 master 节点配置只能是一个 elasticsearch master 节点有此配置。

*2.* network.publish_host 的默认值是 127.0.0.1，所以这里必须要配置 network.publish_host 为当前机器的主机IP，否则不会被其他节点发现。

因为在三台机器上，所以三个 elasticsearch 的 docker-compose.yml 可以完全相同。

```yaml
version: "3.3"
services: 
   elastic-node1:
     image: elasticsearch:7.4.2
     container_name: elastic-node1
     privileged: true
     restart: always
     environment: 
       - "ES_JAVA_OPTS=-Xms8g -Xmx8g"
       - bootstrap.memory_lock=true
     ulimits:
       memlock:
         soft: -1
         hard: -1
     volumes:
       - /etc/localtime:/etc/localtime
       - ./config/cert:/usr/share/elasticsearch/config/cert
       - ./config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
       - /home/data/elastic/:/usr/share/elasticsearch/data
       - /home/data/elastic/logs:/usr/share/elasticsearch/logs
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

#### 节点2

elasticsearch.yml

```yaml
cluster.name: "elastic"
network.host: 0.0.0.0
network.publish_host: 192.168.10.2
node.name: "node2"
node.master: false
discovery.seed_hosts: ["192.168.10.1:9300", "192.168.10.2:9300", "192.168.10.3:9300"]

xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: cert/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: cert/elastic-certificates.p12
```

#### 节点3

```yaml
cluster.name: "elastic"
network.host: 0.0.0.0
network.publish_host: 192.168.10.3
node.name: "node3"
discovery.seed_hosts: ["192.168.10.1", "192.168.10.2", "192.168.10.3"]

xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: cert/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: cert/elastic-certificates.p12
```

在三台机器的 elasticsearch 都启动之后，可以在浏览器中访问 http://192.168.10.1:9200/_cat/nodes 查询集群的节点情况，正常情况下会显示如下：

```
192.168.10.1 52 89 1 0.33 0.34 0.32 dilm * node1
192.168.10.2 49 82 0 0.37 0.35 0.35 dil  - node2
192.168.10.3 55 95 1 0.24 0.28 0.30 dilm - node3
```

如果显示的不是 3 行，那么集群就没有搭建成功。

