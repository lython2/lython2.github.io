---
title: logstash mongodb elastic，从死磕到放弃
tags:
  - logstash
  - elasticsearch
abbrlink: 46bbb70a
date: 2021-01-13 21:31:46
---

有一个小需求，从 mongodb 里面同步数据到 elasticsearch，尝试了 jdbc 插件和 logstash-output-mongodb，logstash-input-mongodb，最后无奈放弃，还是写个脚本定时同步吧。

<!-- more -->

---

#### 拉取 logstash 镜像

```
docker pull logstash:7.10.1
```

#### 安装两个mongodb插件

```
logstash-output-mongodb
logstash-input-mongodb
```

#### 修改两个配置文件

```
# 修改 logstash.yml 
http.host: "0.0.0.0"
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.username: username
xpack.monitoring.elasticsearch.password: password
xpack.monitoring.elasticsearch.hosts: [ "127.0.0.1:9200" ]

# 修改 logstash.conf
input {
  mongodb {
    uri => "mongodb://username:password@127.0.0.1/db_name"
    placeholder_db_dir => "/opt/logstash/tools"  #1
    placeholder_db_name => "search_mongo.db"  #2
    collection => "my_table"
    parse_method => "simple"  #3
  }
}

filter {
  mutate {
    remove_field => [ "_id", "log_entry" ]
    }
}

output {
  elasticsearch {
    hosts => ["127.0.0.1:9200"]
    index => "my-index"
    user => "username"
    password => "password"
  }
  stdout { codec => rubydebug }
}
```

注释说明：

1. placeholder_db_dir为占位目录，需要用户自己创建，因为是 docker 环境，容器内是 logstash 用户，root 用户组，所以这里把这个路径设在了已经存在的目录 /opt/logstash/tools 里；
2. placeholder_db_name同上；
3. parse_method为处理 json 数据的格式，默认是 flatten，会将镶嵌的 json 提升到顶层，也就是 {"a": {"b": "c"}} 转为 {"a_b": "c"}。还可以设置为simple，设置成simple后，{"a": {"b": "c"}} 会转为 {"a": """{"b": "c"}"""}；

#### docker-compose 文件

```yaml
version: "3.3"
services:
  logstash:
    image: logstash:7.10.1
    container_name: logstash
    privileged: true
    restart: always
    environment:
      - SERVER_NAME=logstash
    volumes:
      - /etc/localtime:/etc/localtime
      - ./logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
```

jdbc更不行

---

*参考*

https://www.cnblogs.com/zjdeblog/p/11183865.html
https://github.com/phutchins/logstash-input-mongodb
https://stackoverflow.com/questions/60864512/syncing-the-data-from-mongodb-to-elasticsearch-using-logstash

