---
title: Elasticsearch URI 搜索
tags:
  - elasticsearch
abbrlink: c2dd96fc
date: 2020-12-29 15:59:46
---

URI搜索将搜索词放在q参数上，通过一个URL，即可进行elasticsearch数据搜索。效果和body的query_string方法相同。不过如果搜索中有特殊字符，需要通过quote转换。

参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html


<!-- more -->


#### 格式 GET /index/_search?q=parameter

比如我们在es数据库中存储了一批ip数据，格式如下：

```
{
  "ip": "59.110.12.72",
  "port": 80,
  "title": "我的博客",
  "timestamp": "2019-12-12 01:02:03"
}
```

它的搜索语法是这样的：

#### 简单搜索

搜索开放 80 端口的数据

```
GET /my_index/_search?q=port:80
```

搜索标题为 404 Not found 的数据

```
GET /my_index/_search?q=title:"404 Not found"
```

#### OR 操作符 (||)

搜索开放 80 或 81 端口的数据

```
port: ("80", "81") 或 port: ("80" OR "81") 或 port: (80 || 81)
```

#### AND 操作符 (&&)

搜索开放 80 端口且标题为微信的数据

```
port: 80 AND title:微信 或 port: 80 && title:微信
```

#### NOT 操作符 (!)

搜索开放 5432 端口但是不包含 PostgreSQL 词的内容的数据

```
port:5432 NOT PostgreSQL 或 port:5432 !PostgreSQL 或 port:5432 -PostgreSQL
```

#### 范围搜索

开区间：搜索开放端口 8080 到 9090 之间的数据

```
port:[8080 TO 9090]
```

闭区间：搜索开放端口 8080 到 9090 之间的数据，但是不包括 8080 和 9090

```
port:{8080 TO 9090}
```

半开半闭区间：搜索开放端口 8080 到 9090 之间的数据，但是不包括 9090

```
port:[8080 TO 9090}
```

搜索开放端口不大于 8080 的数据

```
port: {8080 TO *] 或 port:>=8080
```

搜索开放端口小于 8080 的数据

```
port: [* TO 8080} 或 port:<8080
```

搜索2019/11/1日到2019/11/20之间的数据

```
timestamp:[2019-11-01 TO 2019-11-20]
```

**注意，范围搜索还有另外一种写法，这种写法不能在符号和值之间添加空格，否则会分词或者语法错误**

搜索开放端口 8080 到 9090 之间的数据

```
port:(>=8080 AND <=9090) 或 port:(+>=8080 +<=9090)
```

搜索开放端口小于50 或 大于 50000 的数据

```
port: (<50 OR >50000)
```

搜索2019/11/20日之后的数据

```
timestamp:>=2019-11-20
```

#### 特殊情况搜索

搜索包含title字段的数据

```
_exists_: title 或 title:*
```

不支持空字符串和空值搜索，以下语句不支持

```
title: ""
```

null 值会处理成 "null" 字符串

```
title: null 等同于 title: "null"
```

