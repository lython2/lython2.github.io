---
title: ElasticSearch生命周期冷热问题
tags:
  - docker
  - elasticsearch
abbrlink: 289c52a7
---

集群的 SSD 磁盘满了，elasticsearch hot 节点的数据没有自动迁移到 warm 节点，导致写入数据失败，集群变红，问题很严重。


<!-- more -->


#### 临时解决

只留最新的那个索引保持`hot`状态，其他的`hot`节点手动改成`warm`。

```shell
PUT data-flow-000001/_settings
{
  "index": {
    "routing": {
      "allocation": {
        "require": {
          "data_type" : "warm"
        }
      }
    }
  }
}
```

通过执行查看分片命令`GET _cat/shards/data-flow-000001`，可以看到数据正在节点之间迁移

```shell
data-flow-000001 8  r RELOCATING 834716   6gb 10.11.10.10 es-d01 -> 10.11.10.21 uysX9r9oRsKkaFFQMLbDPw es-d11
data-flow-000001 8  p RELOCATING 834716   6gb 10.11.10.11 es-d02 -> 10.11.10.22 GyGJx-iJS0S_VEaxlWXlLg es-d12
data-flow-000001 5  p RELOCATING 832768   6gb 10.11.10.12 es-d03 -> 10.11.10.23 uysX9r9oRsKkaFFQMLbDPw es-d13
...
```

数据迁移需要时间，在SSD释放一部分空间之后，集群很快就会变绿。

#### 问题溯源

手动解决之后，就要来看一下之前定的生命周期策略为什么没有生效了。在`kabana`上，虽然开启了`hot`配置，但是在选择节点属性来控制分片分配处，并没有做出正确的选择，导致虽然看似开启了`hot -> warm`的配置，其实没有起到作用。

![image-20210804175832490](/images/image-20210804175832490.png)

经测试，正确的选择是

![image-20210804175909528](/images/image-20210804175909528.png)

调整后的生命周期文件如下:

```shell
PUT _ilm/policy/my_lifecycle
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "1m",
        "actions": {
          "rollover": {
            "max_age": "180d",
            "max_size": "240gb",
            "max_docs": 10000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "actions": {
          "allocate": {
            "require": {
              "data_type": "warm"
            }
          }
        }
      }
    }
  }
}
```

