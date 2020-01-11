---
layout:     post
title:      003-Elastic-Common-Knowledge
subtitle:    "\"003-Elastic-Common-Knowledge\""
date:       2018-11-09
author:     PZ
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - ELK
    - DevOps
---

## 003-Elastic-Common-Knowledge

[TOC]

> https://www.elastic.co/guide/cn/elasticsearch/guide/current/preface.html

>https://www.elastic.co/guide/index.html

## 1 了解Elastic


> Elasticsearch 是一个实时的分布式搜索分析引擎,提供==全文检索、结构化搜索、分析==功能
`建立在一个全文搜索引擎库 Apache Lucene™ 基础之上。 Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库--无论是开源还是私有。`
- 一个分布式的实时文档存储，每个字段 可以被索引与搜索 
- 一个分布式实时分析搜索引擎 
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据 


> 离线安装kibana插件信息<br>https://www.elastic.co/guide/cn/elasticsearch/guide/current/running-elasticsearch.html

### 1 Elasticsearch 的交互方式

- java:

```
如果你正在使用 Java，在代码中你可以使用 Elasticsearch 内置的两个客户端：

节点客户端（Node client）
    节点客户端作为一个非数据节点加入到本地集群中。换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。 
传输客户端（Transport client）
    轻量级的传输客户端可以将请求发送到远程集群。它本身不加入集群，但是它可以将请求转发到集群中的一个节点上。 

两个 Java 客户端都是通过 9300 端口并使用 Elasticsearch 的原生 传输 协议和集群交互。集群中的节点通过端口 9300 彼此通信。如果这个端口没有打开，节点将无法形成一个集群。
```

- 非java

```
所有其他语言可以使用 RESTful API 通过端口 9200 和 Elasticsearch 进行通信，你可以用你最喜爱的 web 客户端访问 Elasticsearch 。事实上，正如你所看到的，你甚至可以使用 curl 命令来和 Elasticsearch 交互。

```

> Elasticsearch 客户端插件 <br> https://www.elastic.co/guide/cn/elasticsearch/guide/current/_talking_to_elasticsearch.html#_java_api

> `curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`


```
 VERB
	适当的 HTTP 方法 或 谓词 : GET`、 `POST`、 `PUT`、 `HEAD 或者 `DELETE`。 
 PROTOCOL
	http 或者 https`（如果你在 Elasticsearch 前面有一个 `https 代理） 
PATH
    API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm
QUERY_STRING
    任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读) 
BODY
    一个 JSON 格式的请求体 (如果请求需要的话) 
```


```
# 计算集群中文档的数量
curl -i -XGET 'http://localhost:8021/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'

curl -XGET 'localhost:8021/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'


```


### 2 面向文档

> Elasticsearch 使用 JavaScript Object Notation 或者 JSON 作为文档的序列化格式



```sh
# 数据插入
curl -X PUT "localhost:8021/megacorp/employee/1" -H 'Content-Type: application/json' -d'
{
    "first_name" : "hupan",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
'

curl -X PUT "localhost:8021/megacorp/employee/2" -H 'Content-Type: application/json' -d'
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
'
curl -X PUT "localhost:8021/megacorp/employee/3" -H 'Content-Type: application/json' -d'
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
'
```

![图片.png](https://upload-images.jianshu.io/upload_images/10357485-e5651f7dd6df28bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#查询文档信息
curl -XGET 'localhost:8021/megacorp/employee/1?pretty'
#删除文档
curl -XDELETE 'localhost:8021/megacorp/employee/1?pretty'
# 一个搜索默认返回十条结果 所有雇员
curl -XGET 'localhost:8021/megacorp/employee/_search?pretty'

curl -XGET 'localhost:8021/megacorp/employee/_search?q=last_name:Smith'
# 查询表达式搜索
curl -XPOST 'localhost:8021/megacorp/employee/_search?pretty' -d '
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
'
# 复杂搜索
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}

# Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。第一个最高得分的结果很明显：John Smith 的 about 属性清楚地写着 “rock climbing” 

# 
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}

#聚合分析
> https://www.elastic.co/guide/cn/elasticsearch/guide/current/_analytics.html

{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

### 3 集群内原理

- 主节点：负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。**主节点并不需要涉及到文档级别的变更和搜索等操作**

>  每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点


#### 1 集群健康检查
```
curl -X GET "localhost:8021/_cluster/health?pretty"

```
- green 所有的主分片和副本分片都正常运行
- yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行
- red 有主分片没能正常运行

#### 2 什么是索引

> ==索引==实际上是指向一个或者多个物理 分片 的 逻辑命名空间

> ==分片==是一个底层的 工作单元,保存了全部数据中的一部分,是一个 Lucene 的实例,本身就是一个完整的搜索引擎,索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

**技术上 一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档**


##### 1 建立索引，指定分片数
```
curl -X PUT "localhost:8021/blogs" -d'
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
'
```
![图片.png](https://upload-images.jianshu.io/upload_images/10357485-d186e83262b33be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2 集群中的重要配置

> https://www.elastic.co/guide/cn/elasticsearch/guide/current/important-configuration-changes.html#unicast

#### 3 水平扩容

- 在运行中的集群上是可以动态调整副本分片数目的 ，我们可以按需伸缩集群。让我们把副本数从默认的 1 增加到 2
```
curl -X PUT "localhost:8021/blogs/_settings" -d'
{
   "number_of_replicas" : 2
}
'
```

### 4 数据的输入输出

#### 1 文档的元数据

- _index ： 文档在哪存放  （==这个名字必须小写==）
- _type ：文档表示的对象类别 （==命名可以是大写或者小写==）
- _id ： 文档唯一标识 

- _version ：每个文档都有一个版本号。当每次对文档进行修改时（包括删除）， _version 的值会递增，_version 号码确保你的应用程序中的一部分修改不会覆盖另一部分所做的修改
- 


> 自动生成的 ID 是 URL-safe:
> https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-doc.html#index-doc

#### 2 取一个文档信息

```
#取文档中一部分字段信息
curl -X GET "localhost:8021/megacorp/employee/1?_source=age"
curl -X GET "localhost:8021/megacorp/employee/1?_source"
#检查文档是否存
curl -i -XHEAD "localhost:8021/megacorp/employee/1"
```

#### 3 更新文档

>  ==Elasticsearch 中文档是不可改变的==,如果要更新现有文档，需要重建索引 或者进行替换（再PUT，_version号会改变）


update:
    1 从旧文档构建 JSON 
    2 更改该 JSON 
    3 删除旧文档 
    4 索引一个新文档 


#### 4 创建新文档

- 当我们索引一个文档， 怎么确认我们正在创建一个完全新的文档，而不是覆盖现有的呢？

- POST：自动生成唯一 _id
- PUT: 

```
#方法1
PUT /website/blog/123?op_type=create
#方法2
PUT /website/blog/123/_create

{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```

#### 5 删除文档
```
curl -X DELETE "localhost:8021/website/blog/123"

```

#### 6 处理并发冲突
![image](https://www.elastic.co/guide/cn/elasticsearch/guide/current/images/elas_0301.png)

- 乐观并发控制
> Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

- 如何进行冲突控制 
> https://www.elastic.co/guide/cn/elasticsearch/guide/current/optimistic-concurrency-control.html

#### 6 文档更新

```
curl -X POST "localhost:8021/megacorp/employee/1/_update" -d'
{
"doc" : {
    "age" :        26
    }
}
'

#脚本更新
curl -X POST "localhost:8021/megacorp/employee/1/_update" -d'
{
    "script" : "ctx._source.age+=1"
}
'
```

> config/elasticsearch.yml <br>
> script.groovy.sandbox.enabled: ==false==

> https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-updates.html

#### 7 多文档取回

```
curl -X GET "localhost:9200/_mget" -d'
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
'
```
> https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Retrieving_Multiple_Documents.html

### 5 分布式文档存储

#### 1 一个文档是如何被路由到一个分片中的

> `shard = hash(routing) % number_of_primary_shards`



```
routing 是一个可变值，默认是文档的 _id,也可以设置成一个自定义的值,routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到 余数 
```
- 这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量


#### 2 主分片和副本分片如何交互

- 每个节点都有能力处理任意请求。 每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上

> 当发送请求的时候， 为了扩展负载，更好的做法是==轮询集群中所有的节点==。

#### 3  新建、索引和删除文档
![image](https://www.elastic.co/guide/cn/elasticsearch/guide/current/images/elas_0402.png)


- 客户端向 Node 1 发送新建、索引或者删除请求。 
- 节点使用文档的 _id 确定文档属于分片 0 。请求会被转发到 Node 3`，因为分片 0 的主分片目前被分配在 `Node 3 上。
- Node 3 在主分片上面执行请求。如果成功了，它将请求并行转发到 Node 1 和 Node 2 的副本分片上。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。 

> 在客户端收到成功响应时，文档变更已经在主分片和所有副本分片执行完成，变更是安全的


##### 1 数据完整性参数

- consistency：一致性，在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求 必须要有 _规定数量(quorum)_的分片副本处于活跃可用状态
- timeout

#### 4 取文档流程
![image](https://www.elastic.co/guide/cn/elasticsearch/guide/current/images/elas_0403.png)

- 客户端向 Node 1 发送获取请求。
- 节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 Node 2 。
- Node 2 将文档返回给 Node 1 ，然后将文档返回给客户端。

在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的。


#### 5 局部更新文档
![image](https://www.elastic.co/guide/cn/elasticsearch/guide/current/images/elas_0404.png)

- 客户端向 Node 1 发送更新请求。 
- 它将请求转发到主分片所在的 Node 3
- Node 3 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 retry_on_conflict 次后放弃
- 如果 Node 3 成功地更新文档，它将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， Node 3 向协调节点也返回成功，协调节点向客户端返回成功。

### 6 搜索

#### 1 多索引，多类型

```
 /_search
    在所有的索引中搜索所有的类型 
/gb/_search
    在 gb 索引中搜索所有的类型 
/gb,us/_search
    在 gb 和 us 索引中搜索所有的文档 
/g*,u*/_search
    在任何以 g 或者 u 开头的索引中搜索所有的类型 
/gb/user/_search
    在 gb 索引中搜索 user 类型 
/gb,us/user,tweet/_search
    在 gb 和 us 索引中搜索 user 和 tweet 类型 
/_all/user,tweet/_search
    在所有的索引中搜索 user 和 tweet 类型 
```

- 当在单一的索引下进行搜索的时候，Elasticsearch 转发请求到索引的每个分片中，可以是主分片也可以是副本分片，然后从每个分片中收集结果。多索引搜索恰好也是用相同的方式工作的--只是会涉及到更多的分片。

#### 2 分页

```
GET /_search?size=5             第一页
GET /_search?size=5&from=5      第二页 
GET /_search?size=5&from=10     第三页
size
    显示应该返回的结果数量，默认是 10
from    
    显示应该跳过的初始结果数量，默认是 0
```

### 7 映射和分析

在Elas中数据可以分为精确值和全文，为了促进这类在全文域中的查询，Elasticsearch 首先 ==分析 文档==，之后根据结果创建 ==倒排索引==

#### 1 倒排索引

> https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html

分词、词性、同义词

#### 2 分析器
分析 包含下面的过程：

    - 首先，将一块文本分成适合于倒排索引的独立的 词条 ， 
    - 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall

分析器 实际上是将三个功能

    - 字符过滤器：
        首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 `and`
    - 分词器
        其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。 
    - Token 过滤器
        最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a`， `and`， `the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）。    

##### 1 内置分析器

```
curl -X GET "localhost:8021/_analyze?pretty"  -d'
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
'
```

#### 3 映射

-  Elasticsearch 需要知道每个域中数据的类型。==这个信息包含在映射中==
-  索引中每个文档都有 类型 。每种类型都有它自己的 ==映射== ，或者 ==模式定义==


##### 1 简单域类型 

    字符串: string
    整数 : byte, short, integer, long
    浮点数: float, double
    布尔型: boolean
    日期: date 

##### 2 查看映射

- 查看类型映射信息 
```
curl -X GET "localhost:8021/megacorp/_mapping/employee/?pretty"
```

##### 3 自定义域映射

string 域映射的两个最重要 属性是 index 和 analyzer 。

index 属性控制怎样索引字符串。它可以是下面三个值：

     analyzed
    首先分析字符串，然后索引它。换句话说，以全文索引这个域。 
     not_analyzed
      索引这个域，所以它能够被搜索，但索引的是精确值。不会对它进行分析。 
     no
    不索引这个域。这个域不会被搜索到。  

```
{
    "tag": {
        "type":     "string",
        "index":    "not_analyzed"
    }
}

注意

其他简单类型（例如 long ， double ， date 等）也接受 index 参数，但有意义的值只有 no 和 not_analyzed ， 因为它们永远不会被分析。
```

对于 analyzed 字符串域，用 analyzer 属性指定在搜索和索引时使用的分析器。默认， Elasticsearch 使用 standard 分析器, 但你可以指定一个内置的分析器替代它，例如 whitespace 、 simple 和 `english`

```
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```

### 9 管理、监控和部署

#### 1 集群健康

```
curl 'localhost:8021/_cat/indices?v'
curl 'localhost:8021/_cluster/health?pretty'
curl 'localhost:8021/_cluster/health?level=shards&pretty'
curl 'localhost:8021/_cluster/health?level=indices&pretty'

{
  "cluster_name" : "inner_es_cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 29,
  "active_shards" : 29,   涵盖了所有索引的_所有_分片的汇总值，即包括副本分片
  "relocating_shards" : 0,  显示当前正在从一个节点迁往其他节点的分片的数量
  "initializing_shards" : 0, 是刚刚创建的分片的个数
  "unassigned_shards" : 29,  是已经在集群状态中存在的分片，但是实际在集群里又找不着。通常未分配分片的来源是未分配的副本
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}


```
