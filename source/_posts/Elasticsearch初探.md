---
title: Elasticsearch初探
date: 2018-01-01 12:39:04
categories: Elasticsearch
tags:
  - Elasticsearch
  - ELK
---

[TOC]

### 前言

最近这个好火, 简单体验下

<!-- more -->


### 一: 安装和启动

####  ElasticSearch下载

> 官方地址: https://www.elastic.co/cn/downloads/elasticsearch
>
> github: https://github.com/elastic/elasticsearch

下载或clone后解压



#### 单实例节点启动

```shell
# cd elasticsearch目录下
bin/elasticsearch
bin/elasticsearch -d # 后台启动
```

默认端口9200, 启动完成后访问`http://ip:9200` 即可查看到节点信息

启动中我遇到两个错误: 

**错误一:** 

```shell
can not run elasticsearch as root  
# 不能以root用户启动
```

```shell
[root@01 bin]# groupadd xiefy
[root@01 bin]# useradd xiefy -g xiefy -p 123123
[root@01 bin]# chown -R xiefy:xiefy elasticsearch
```

**错误二:**

```shell
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

-- 错误[1]: max file descriptors过小
-- 错误[2]: max_map_count过小, max_map_count文件包含限制一个进程可以拥有的VMA(虚拟内存区域)的数量，系			  统默认是65530，修改成262144
```

```shell
# 切换到root用户
vi /etc/security/limits.conf

# 添加如下
* soft nofile 65536
* hard nofile 65536
```

```shell
# 切换到root用户
vi /etc/sysctl.conf

# 添加如下
vm.max_map_count=655360
# 重新加载配置文件或重启
sysctl -p # 从配置文件“/etc/sysctl.conf”加载内核参数设置
```



#### elasticsearch-head插件安装

>   elasticsearch-head 是一个用于浏览Elastic Search集群并与之进行交互(操作和管理)的web界面
>
>   GitHub: https://github.com/mobz/elasticsearch-head
>
>   要使用elasticsearch-head, 需要具备nodejs环境: 

**nodejs安装**

>  下载源码: https://nodejs.org/en/download/

安装方式有多种, 我用源码安装方式(包含npm)

1. 下载源码:

   ```
   https://nodejs.org/dist/v8.9.4/node-v8.9.4.tar.gz
   ```

2. 解压源码: 

   ```shell
   tar xzvf node-v* && cd node-v*
   ```

3. 安装必要的编译软件

   ```shell
   sudo yum install gcc gcc-c++
   ```

4. 编译

   ```shell
   ./configure
   make
   ```

5. 编译&安装

   ```shell
   sudo make install
   ```

6. 查看版本

   ```shell
   node --version
   npm -version
   ```


下载或克隆`elasticsearch-head`后, 进入`elasticsearch-head-master`目录:

- `npm install` 

  速度太慢可以使用淘宝镜像: `npm install -g cnpm --registry=https://registry.npm.taobao.org`

- `npm run start`

- open <http://localhost:9100/>

这时可以访问到页面, 并没有监听到集群.

解决head插件和elasticsearch之间访问跨域问题. 

修改elasticsearch目录下的`elasticsearch.yml`

```shell
# 加入以下内容
http.cors.enabled: true
http.cors.allow-origin: "*"
```

然后: http://localhost:9100/ 即可访问到管理页面. 



#### 分布式安装启动

elasticsearch的横向扩展很容易: 这里建立一个主节点(node-master), 两个随从节点(node-1, node-2)

我提前拷贝了三个es:

```shell
[xiefy@01 elk]$ ll
total 33620
-rw-r--r-- 1 root  root  33485703 Aug 17 22:42 elasticsearch-5.5.2.tar.gz
drwxr-xr-x 7 root  root      4096 Jan  8 11:17 elasticsearch-head-master
drwxr-xr-x 9 xiefy xiefy     4096 Jan  8 10:15 elasticsearch-master
drwxr-xr-x 9 xiefy xiefy     4096 Jan  8 14:13 elasticsearch-node1
drwxr-xr-x 9 xiefy xiefy     4096 Jan  8 14:16 elasticsearch-node2
-rw-r--r-- 1 root  root    921421 Jan  8 11:14 master.zip
```

分别配置三个es目录中的`config/elasticsearch.yml`

**node-master**:

```properties
cluster.name: xiefy_elastic 
node.name: node-master 
node.master: true 
network.host: 0.0.0.0 

# 除此之外, head插件需要连接到port: 9200的节点上, 还需要这个配置, 
# 用来允许 elasticsearch-head 运行时的跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
```

**node-1**:

```properties
cluster.name: xiefy_elastic 
node.name: node-1
network.host: 0.0.0.0
http.port: 9201 
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```

**node-2**: 参考node-1

**相关配置解释**: 

* `cluster.name`: 集群名称, 默认是elasticsearch

* `node.name`: 节点名称, 默认随机分配

* `node.master`: 是否是主节点, 默认情况下可不写, 第一个起来的就是Master, 挂掉后会重新选举Master

* `network.host`: 默认情况下只允许本机通过localhost或127.0.0.1访问, 为了测试方便, 

  我需要远程访问所以配成了`0.0.0.0`

* `http.port`: 默认为9200, 同一个服务器下启动多个es节点, 默认端口号会从9200默认递增1, 这里我手动指定了

* `discovery.zen.ping.unicast.hosts: ["host1", "host2"]`

  Elasticsearch默认使用服务发现(Zen discovery)作为集群节点间发现和通信的机制, 当启动新节点时，通过这个ip列表进行节点发现，组建集群.



分别启动三个es实例和head插件:

访问`http://ip:9100`:

![es三个节点集群](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/ELK/es%E5%B8%A6%E7%B4%A2%E5%BC%95%E7%9A%84%E9%9B%86%E7%BE%A4.png)

### 二: 基础概念

ElasticSearch与关系型数据库的一些术语比较

| 关系型数据库                 | Ela                   |
| ---------------------- | --------------------- |
| Database               | Index                 |
| Table                  | Type                  |
| Row                    | Document              |
| Column                 | Field                 |
| Schema                 | Mapping               |
| Index                  | Everything is indexed |
| SQL`                   | Query DSL             |
| select * from table... | GET http://...        |
| update tables set...   | PUT http://...        |

Server:

- Node: 一个server实例
- Cluster：多个node组成cluster
- Shard：数据分片，一个index可能会有多个shards，不同shards可能在不同nodes
- Replica：shard的备份，有一个primary shard，其余的叫做replica shards
- Gateway：管理cluster状态信息



#### Shards & Replicas

副本很重要, 主要有两个原因: 

 - 它在分片/节点发生故障时来保障高可用性, 因此, 副本分片永远不会和主/原始分片分配在同一个节点中
 - 它允许扩展搜索量和吞吐量, 因为可以在所有副本上并行执行搜索

 可以在创建索引时为索引定义分片和副本的数量。
 创建索引后，可以随时动态更改副本数，但不能再更改分片数。

 > 每个Elasticsearch分片都是Lucene索引, 在一个Lucene上的最大文档数量大约21亿(`Integer.MAX_VALUE-128` )




THE REST API:

elasticsearh提供了丰富的REST API来与集群交互, 这些API包括: 

- 检查你的集群，节点和索引的健康情况，状态和统计
- 管理你的集群，节点，索引数据和metadata
- 针对索引执行CURD（创建，读取，更新，删除）和搜索操作
- 执行高级的搜索操作，例如分页，排序，过滤，聚合，脚本等等。



通过`_cat`可以查看到很多资源:

```
curl -XGET 'localhost:9200/_cat'

=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
```

偷偷看一眼集群健康状态: `curl -XGET 'localhost:9200/_cat/health?v'`

节点的信息: ` curl -XGET 'localhost:9200/_cat/nodes?v&pretty'`

而关于和集群索引和文档的交互, 放在下一章. 



### 三: 基础用法

#### 索引创建

* 方式一: head插件可以直接新建/删除/查询索引(Index)

* 方式二: 通过rest api 

  ```shell
  # 新建
  curl -X PUT 'localhost:9200/book'

  # 删除
  curl -X DELETE 'localhost:9200/book'

  # 查看当前节点下所有Index
  curl -X GET 'http://localhost:9200/_cat/indices?v'
  ```

这样创建的Index是没有结构的. 可以看到索引信息的mappings:{}

下面来定义一个有结构映射的Index

> PUT http://ip:9200/people

```json
{
	"settings": {
		"number_of_shards": 3,
		"number_of_replicas": 1
	},
	"mappings": {
		"man": {
			"properties": {
				"name": {"type": "text"},
				"country": {"type": "keyword"},
				"age": {"type": "integer"},
				"birthday": {
					"type": "date",
					"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
				}
			}
		},
		"women": {
		}
	}
}
```

里面设置了分片数, 备份数, 一个Index和两个type的结构映射.



#### 插入数据

> PUT http://47.94.210.157:9200/people/man/1
>
> 指定ID为1 

```json
{
	"name": "伊布",
	"country": "瑞典",
	"age": 30,
	"birthday": "1988-12-12"
}
```

如果不指定ID, 会生成为随机字符串, 此时需要改为POST方式 `POST http://47.94.210.157:9200/people/man`

神奇的一点, es不会限制你在创建一个文档之前索引和类型必须存在, 不存在时, es会自动创建它. 



#### 更新数据

> POST http://47.94.210.157:9200/people/man/1/_update  -- 修改ID为1的文档

```json
{
	"doc": {
		"name": "梅西梅西很像很强"
	}
}
```

还可以通过脚本方式更新: 

```json
{ "script": "ctx._source.age += 10" }
```

这里是把年龄加10, `ctx.source`引用了当前文档.



#### 索引/替换文档

> PUT http://47.94.210.157:9200/football/man/1 -- 修改ID为1的文档

```
{"name": "伊布asdasadasds3333333333333333sd"}
```

elasticsearch将会用一个新的文档取代（即重建索引）旧的文档



**删除数据**

删除文档: `DELETE http://47.94.210.157:9200/people/man/1`



#### 查看数据

* 根据ID查询

> GET http://ip:9200/people/man/1?pretty=true

```json
{
  "_index": "people",
  "_type": "man",
  "_id": "2",
  "_version": 1,
  "found": true,
  "_source": {
    "name": "伊布",
    "country": "瑞典",
    "age": 34,
    "birthday": "1972-12-12"
  }
}
```

`found`字段表示查到与否

  ​

* 查询全部

> GET http://ip:9200/Index/Type/_search

或带排序带分页的查询: 

> POST http://47.94.210.157:9200/people/_search

ES 默认 从0开始(from), 一次返回10条(size), 并按照_score字段倒排,  也可以自己指定

```json
# 带排序带分页的查询
{
	"query": { "match_all": {} },
	"sort": [{
		"birthday": {"order": "desc"}
		}
	],
	"from": 0, 
	"size": 5
}
```


```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "people",
        "_type": "man",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "伊布",
          "country": "瑞典",
          "age": 34,
          "birthday": "1972-12-12"
        }
      },
      ....
      ....
    ]
  }
}
```

```
# 返回结果解释
- took: 耗时(单位毫秒)
- timed_out: 是否超时
- hits: 命中的记录数组 
  - total: 返回的记录数
  - max_score: 最高匹配度分数
  - hits: 记录数组
    - _score: 匹配度
```



* 关键字查询

```json
{
	"query": {
		"match": {"name": "梅西"}
	}
}
```

**注意**: 这里是模糊匹配查询, 例如查询的值是"西2", 那么会查询所有记录name有"西"和name有"2"的.

关于查询多个关键字之间的逻辑运算: 

如果这样写, 两个关键字会被认为是 `or`的关系来查询

```json
{
	"query": {
		"match": {"name": "西 布"}
	}
}
```

如果是`and`关系来搜索, 需要布尔查询

```json
{
	"query": {
		"bool": {
			"must": [
				{"match": {"name": "西"}} ,
				{"match": {"name": "2"}} 
			]
		}
	}
}
```



#### 聚合查询

> POST  http://47.94.210.157:9200/people/_search

* 分组聚合

```json
{
	"aggs": {
		"group_by_age": {
			"terms": {"field": "age"}
		}
	}
}
```

返回结果中, 除了有hits数组, 还有聚合查询的结果: 

在单个请求中就可以同时查询数据和进行多次聚合运算是非常有意义的, 他可以降低网络请求的次数

```json
# 聚合查询结果
"aggregations": {
	"group_by_age": {
	  "doc_count_error_upper_bound": 0,
	  "sum_other_doc_count": 0,
	  "buckets": [
		{
		  "key": 24,
		  "doc_count": 2
		},
		{
		  "key": 32,
		  "doc_count": 1
		}
	  ]
	}
}
```

支持多个聚合, 聚合结果也会返回多个:

```json
{
	"aggs": {
		"group_by_age": {
			"terms": {"field": "age"}
		},
		"group_by_age": {
			"terms": {"field": "age"}
		}
	}
}
```

* 其他功能函数

```json
{
	"aggs": {
		"tongji_age": {
			"stats": {"field": "age"}
		}
	}
}
```

`stats`指定计算字段, 返回结果包括了总数, 最小值, 最大值, 平均值和求和

```json
 "aggregations": {
    "tongji_age": {
      "count": 3,
      "min": 24,
      "max": 32,
      "avg": 26.666666666666668,
      "sum": 80
    }
  }
```

也可指定某种类型的计算

```json
{
	"aggs": {
		"tongji_age": {
			"sum": {"field": "age"}
		}
	}
}
```

返回结果

```json
"aggregations": {
    "tongji_age": {
      "value": 80
    }
 }
```





### 四: 高级查询

分为**子条件查询**和**复合条件**查询: 

类型: 

* **全文本查询**: 针对文本类型数据
* **字段级别查询**: 针对结构化数据, 如日期, 数字



#### 文本查询

* 模糊匹配

```json
{
	"query": {
		"match": {"name": "西2"}
	}
}
```

* 短语匹配

```json
{
	"query": {
		"match_phrase": {"name": "西2"}
	}
}
```

* 多个字段匹配

```json
{
	"query": {
		"multi_match": {
			"query": "瑞典",
			"fields": ["name", "country"]
		}
	}
}
```



**语法查询**: 根据语法规则查询:

* 带有布尔逻辑的查询

```json
{
	"query": {
		"query_string": {
			"query": "(西 AND 梅) OR 布"
		}
	}
}
```

* query_string 查询多个字段

```json
{
	"query": {
		"query_string": {
			"query": "西梅 OR  瑞典",
			"fields": ["country", "name"]
		}
	}
}
```



#### 结构化数据查询

```json
{
	"query": {"term": { "age": 24}}
}	
```

* 带范围的查询

```json
{
	"query": {
		"range": {
			"birthday": {
				"gte": "1980-01-01",
				"lte": "now"
			}
		}
	}
}	
```





#### 子条件查询

Filter Context: 用来做数据过滤, 在查询过程中, 只判断该文档是否满足条件(y or not)

Filter和Query的区别? 

Filter要结合bool使用, 查询结果会放入缓存中, 速度较Query更快

```json
{
	"query": {
		"bool": {
			"filter": {
				"term": { "age": 24 }
			}
		}
	}
}
```



#### 复合查询

* 固定分数查询

```json
{
	"query": {
		"match": {"name": "梅西"}
	}
}
```

可以看到查询的结果, 每条数据的`_score`不同, 代表了与查询值的匹配程度的分数.

但查询不一定都需要产生文档得分，特别在过滤文档集合的时候。为了避免不必要的文档得分计算，Elasticsearch会检查这种情况并自动的优化这种查询。

例如在`bool`查询中返回年龄范围在20-50的文档, 对我来说这个范围内的文档都是等价的, 即他们的相关度都是一样的(filter子句查询，不会改变得分).

```json
{
	"query": {
		"constant_score": {
			"filter": {
				"match": {"name": "梅西"}
			},
			"boost": 2
		}
	}
}
```

可以看到查询结果, 每条数据的`_score`都为2, 如果不指定`boost`则默认为1

* 布尔查询

```json
{
	"query": {
		"bool": {
			"should": [
				{ "match": {"name": "梅西"}},
				{ "match": {"country": "阿"}}
			]
		}
	}	
}
```

这里两个match之间是或的逻辑关系. `should` 如果改为 `must` 代表与逻辑.

我们也可以把`must`，`should`，`must_not`同时组合到`bool`子句。此外，我们也可以组合`bool` 到任何一个`bool`子句中，实现复杂的多层`bool`子句嵌套逻辑。

再加一层Filter, 只有age=32的能返回

```json
{
	"query": {
		"bool": {
			"must": [
				{ "match": {"name": "梅西"} },
				{ "match": {"country": "阿根廷"}}
			],
			"filter": [{
				"term": {
					"age": 32
				}
			}]
		}
	}	
}
```

country=阿根廷的不返回:

```json
{
	"query": {
		"bool": {
			"must_not": {
				"term": {"country": "阿根廷"}
			}
		}
	}
}
```



### 五: 关于中文分词

为什么需要中文分词?

首先看一下默认的分词规则.

```json
# 英文
GET http://47.94.210.157:9200/_analyze?analyzer=standard&pretty=true&text=hello world, elasticsearch

{
  "tokens": [
    {
      "token": "hello",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "world",
      "start_offset": 6,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "elasticsearch",
      "start_offset": 13,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

可以看到, 英文的默认分词是根据标点符号和空格默认来分的.

再看看中文的:

```json
GET http://47.94.210.157:9200/_analyze?analyzer=standard&pretty=true&text=你好,啊

{
  "tokens": [
    {
      "token": "你",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "好",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "啊",
      "start_offset": 3,
      "end_offset": 4,
      "type": "<IDEOGRAPHIC>",
      "position": 2
    }
  ]
}
```

可以看到ES对中文的分词并不智能, 是将汉字全部分开了, 所以引入中文分词.



#### IK

> IK: https://github.com/medcl/elasticsearch-analysis-ik
>
> The IK Analysis plugin integrates Lucene IK analyzer into elasticsearch, support customized dictionary.

**安装**

1.download or compile

- optional 1 - download pre-build package from here: <https://github.com/medcl/elasticsearch-analysis-ik/releases>

  unzip plugin to folder `your-es-root/plugins/`

- optional 2 - use elasticsearch-plugin to install ( version > v5.5.1 ):

  `./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.1/elasticsearch-analysis-ik-6.2.1.zip`

2.restart elasticsearch

两种安装方式, 任选其一, 注意版本就好



Github里有`Quick Example` 可以看下怎么使用

需要在建立索引时指定ik分词器, 建立索引和搜索索引字段都需要指定, 例如:

`"analyzer": "ik_max_word"`和`"search_analyzer": "ik_max_word"`

IK提供两种分词规则: 

* ik_max_word: 会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合；
* ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”。

除此之外, IK也支持扩展自定义词典, 以及热更新.

```json
# Test
GET http://47.94.210.157:9200/_analyze?analyzer=ik_max_word&pretty=true&text=你好,啊

{
  "tokens": [
    {
      "token": "你好",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "啊",
      "start_offset": 3,
      "end_offset": 4,
      "type": "CN_CHAR",
      "position": 1
    }
  ]
}
```






### 六: Spring Boot 集成 Elastic Search



#### 版本参考

| Spring Boot Version (x) | Spring Data Elasticsearch Version (y) | Elasticsearch Version (z) |
| ----------------------- | ------------------------------------- | ------------------------- |
| x <= 1.3.5              | y <= 1.3.4                            | z <= 1.7.2*               |
| x >= 1.4.x              | 2.0.0 <=y < 5.0.0**                   | 2.0.0 <= z < 5.0.0**      |



| 服务器集群ES版本      | 5.5.2         |
| -------------- | ------------- |
| Spring boot    | 1.5.9.RELEASE |
| Elastic Search | 5.5.2         |
| log4j-core     | 2.7           |

#### 集成步骤

1. 引入Maven依赖:

   ```xml
   <properties>
   	<log4j-core.version>2.7</log4j-core.version>
   	<elasticsearch-version>5.5.2</elasticsearch-version>
   </properties>

   <dependency>
   	<groupId>org.elasticsearch.client</groupId>
   	<artifactId>transport</artifactId>
   	<version>${elasticsearch.version}</version>
   </dependency>

   <!--
   <dependency>
   	<groupId>org.elasticsearch</groupId>
   	<artifactId>elasticsearch</artifactId>
   	<version>${elasticsearch-version}</version>
   </dependency>
   -->
   ```

   **注意**: transport中依赖了elasticsearch, 但默认是`2.4.6`版本, 需要指定下elasticsearch的版本`5.5.2`



   2. 也可以直接引入:

      ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
      </dependency>
      ```

      但是`spring-boot-starter-data-elasticsearch`只支持到`2.4.x`版本的es. 

      如果使用`5.x.x`版本ES, 就用上面那种方式单独引入ES依赖. 

   ​

2. 添加配置类

   ```java
   @Configuration
   public class ElasticSearchConfig {

       /** 集群host */
       @Value("${spring.data.elasticsearch.cluster-nodes}")
       private String clusterNodes;

       /** 集群名称 */
       @Value("${spring.data.elasticsearch.cluster-name}")
       private String clusterName;

       @Bean
       public TransportClient client() throws UnknownHostException{

           InetSocketTransportAddress node = new InetSocketTransportAddress(
                   InetAddress.getByName(clusterNodes), 9300
           );

           Settings settings = Settings.builder().put("cluster.name", clusterName).build();

           TransportClient client = new PreBuiltTransportClient(settings);
           client.addTransportAddress(node);
           return client;
       }
   }
   ```

   `application.properties`中配置: 

   * `spring.data.elasticsearch.cluster-nodes=xxx`
   * `spring.data.elasticsearch.cluster-name=xxx`



#### 测试用例

简单的CRUL操作: 

> github: https://github.com/thank037/elasticsearch_demo.git

`@Link com.thank.elasticsearch.TestElasticSearchCRUD.java`



