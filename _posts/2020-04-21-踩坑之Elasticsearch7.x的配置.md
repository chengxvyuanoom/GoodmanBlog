---
layout: post
title:  "关于Elasticsearch7.x的配置上的坑"
date:   2020-04-21 12:03:12
tags: Elasticsearch Java Config
date: 2020-04-21 12:51:12
categories: Elasticsearch7.x

---

* TOC
{:toc}

####  关于Elasticsearch7.x的配置上的坑

#### 一、TooManyBucketsException

> ​	Caused by: org.elasticsearch.search.aggregations.MultiBucketConsumerService$TooManyBucketsException: Trying to **create** too many buckets. Must be **less** **than** **or** equal **to**: [10000] but was [10314]. This **limit** can be **set** **by** changing the [search.max_buckets] cluster **level** setting.

​	Es7.x防止OOM的保护机制，最大桶聚合数默认10000，实际中如果按照时间分区，并且有多层嵌套聚合，是很容易发生这个错误的，在之后版本的Es中已经将该参数设置为65535，并且官方文档并未说明该值最大可以设置多少，理论上可以无限大，但是如果bucket一直在超，考虑自己的聚合逻辑如何优化，或者采用MapReduce的思想来解决

​	解决办法:

​		docker-compose :

> ​        search.max_buckets：65535

​       curl 命令 :

> ​       curl -X PUT -u 用户名:密码 "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d '{"persistent": { "search.max_buckets": 65535 }}'

#### 二、默认分片数太少

> ​		**this** action would add [20] total shards, but **this** cluster currently has [1986]/[2000] maximum shards

​		Es7.x中的分片默认值是1000，所以在设计之初，最好将分片数先预算好，生命周期不长的索引，分片尽量小些。

解决办法:

​		docker-compose :

> ​        cluster.max_shards_per_node：10000

​       curl 命令 :

> ​       curl -X PUT -u 用户名:密码 "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d '{"persistent": { "cluster.max_shards_per_node":10000}}'

#### 三、Data Too Large（Too-many-requests-429）

> ​	CircuitBreakingException[[FIELDDATA] Data too **large**, data **for** [proccessDate] would be larger than **limit** **of** [xxxgb]

​	这个属于

keytool -import -v -trustcacerts -storepass 123456 -alias edison -file /Users/edison/alice.p12 -keystore

keytool -genkey -alias dp -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore  /Users/edison/dp.keystore -storepass 123456 

