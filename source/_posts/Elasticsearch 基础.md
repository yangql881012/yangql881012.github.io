---
title: Elasticsearch 基础
date: 2017-06-29 08:54:59
tags: Elasticsearch
toc: true
categories: 大数据技术
---
## 1.相关概念 ##
- 接近实时（NRT）
 Elasticsearch是一个接近实时的搜索平台。这意味着，从索引一个文档直到这个文档能够被搜索到有一个轻微的延迟（通常是1秒）。
- 集群（cluster）
 一个集群就是由一个或多个节点组织在一起，它们共同持有你整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。在产品环境中显式地设定这个名字是一个好习惯，但是使用默认值来进行测试/开发也是不错的。
- 节点（node）
一个节点是你集群中的一个服务器，作为集群的一部分，它存储你的数据，参与集群的索引和搜索功能。和集群类似，一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于Elasticsearch集群中的哪些节点。一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何Elasticsearch节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。
- 索引（index）
一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（必须全部是小写字母的），并且当我们要对对应于这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，如果你想，可以定义任意多的索引。
- 类型（type）
在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。比如说，我们假设你运营一个博客平台并且将你所有的数据存储到一个索引中。在这个索引中，你可以为用户数据定义一个类型，为博客数据定义另一个类型，当然，也可以为评论数据定义另一个类型。
- 文档（document）
一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式。在一个index/type里面，只要你想，你可以存储任意多的文档。注意，尽管一个文档，物理上存在于一个索引之中，文档必须被索引/赋予一个索引的type。
- 分片和复制（shards & replicas）
一个索引可以存储超出单个结点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片之所以重要，主要有两方面的原因
            - 允许你水平分割/扩展你的内容容量
            - 允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量

    至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的。在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。复制之所以重要，有两个主要原因：
            - 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。
            - 扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行

    总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。默认情况下，Elasticsearch中的每个索引被分片5个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有5个主分片和另外5个复制分片（1个完全拷贝），这样的话每个索引总共就有10个分片。
<!-- more -->
## 2.基本操作 ##
- elasticsearch 命令方式：
```
curl -<REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>
```
- 要检查集群健康
```
[yangql@hadoop01 bin]$ curl 'hadoop01:9200/_cat/health?v'
epoch      timestamp cluster    status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1498747113 10:38:33  es-cluster green
```
当我们询问集群状态的时候，我们要么得到绿色、黄色或红色。绿色代表一切正常（集群功能齐全），黄色意味着所有的数据都是可用的，但是某些复制没有被分配（集群功能齐全），红色则代表因为某些原因，某些数据不可用。注意，即使是集群状态是红色的，集群仍然是部分可用的（它仍然会利用可用的分片来响应搜索请求），但是可能你需要尽快修复它，因为你有丢失的数据。
- 集群中的节点列表
```
[yangql@hadoop01 bin]$ curl 'hadoop01:9200/_cat/nodes?v'
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.1.231            3          94   0    0.00    0.00     0.00 mdi       *      es01
192.168.1.232            3          94   0    0.01    0.01     0.00 mdi       -      es02
192.168.1.233            3          92   0    0.00    0.00     0.00 mdi       -      es03
```
- 列出所有的索引
```
[yangql@hadoop01 bin]$ curl 'hadoop01:9200/_cat/indices?v'
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   customer 2J6OutOFS9OpLlTFISH7Tw   5   1          0            0      1.2kb           650b
```
- 创建一个索引现在让我们创建一个叫做“customer”的索引，然后再列出所有的索引：
```
curl -XPUT 'hadoop01:9200/customer
[yangql@hadoop01 bin]$ curl 'hadoop01:9200/_cat/indices?v'
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   customer 2J6OutOFS9OpLlTFISH7Tw   5   1          0            0      1.2kb           650b
```
我们现在有一个叫做customer的索引，并且它有5个主分片和1份复制（都是默认值），其中包含0个文档。
- 创建一个文档
```
[yangql@hadoop01 bin]$  curl -XPUT 'hadoop01:9200/customer/external/1?pretty' -d '
>         {
>           "name": "John Doe"
>         }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "created" : true
}
[yangql@hadoop01 bin]$
```
external”类型,文档的ID是1,<font color="red">Elasticsearch在你想将文档索引到某个索引的时候，并不强制要求这个索引被显式地创建。在前面这个例子中，如果customer索引不存在，Elasticsearch将会自动地创建这个索引。</font>
- 查询一个文档
```
[yangql@hadoop01 bin]$ curl -XGET 'hadoop01:9200/customer/external/1?pretty'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```
- 删除
```
[yangql@hadoop01 bin]$ curl -XDELETE 'hadoop01:9200/customer?pretty'
{
  "acknowledged" : true
}
```
- 替换文档
```
[yangql@hadoop01 bin]$ curl -XPUT 'hadoop01:9200/customer/external/1?pretty' -d '
>             {
>               "name": "John Doe"
>             }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "created" : true
}
[yangql@hadoop01 bin]$ curl -XPUT 'hadoop01:9200/customer/external/1?pretty' -d '
>             {
>               "name": "Jane Doe"
>             }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "created" : false
}
[yangql@hadoop01 bin]$
```
以上的命令将ID为1的文档的name字段的值从“John Doe”改成了“Jane Doe”。如果我们使用一个不同的ID，一个新的文档将会被索引，当前已经在索引中的文档不会受到影响。
- 无ID创建document  
在创建document的时候，ID部分是可选的。如果不指定，Elasticsearch将产生一个随机的ID来索引这个文档。Elasticsearch生成的ID会作为索引API调用的一部分被返回。
```
[yangql@hadoop01 bin]$ curl -XPOST 'hadoop01:9200/customer/external?pretty' -d '
>             {
>               "name": "Jane Doe"
>             }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "AVz0W-I9v5C-xxEQKDDx",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "created" : true
}
[yangql@hadoop01 bin]$
```
- 更新文档  
除了可以索引、替换文档之外，我们也可以更新一个文档。但要注意，Elasticsearch底层并不支持原地更新。在我们想要做一次更新的时候，Elasticsearch先删除旧文档，然后在索引一个更新过的新文档。
```
[yangql@hadoop01 bin]$  curl -XPOST 'hadoop01:9200/customer/external/1/_update?pretty' -d '
>         {
>           "doc": { "name": "Jane Doe" }
>         }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 2,
  "result" : "noop",
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "failed" : 0
  }
}
下面的例子展示了怎样将我们ID为1的文档的name字段改成“Jane Doe”的同时，给它加上age字段：
[yangql@hadoop01 bin]$ curl -XPOST 'hadoop01:9200/customer/external/1/_update?pretty' -d '
>         {
>           "doc": { "name": "Jane Doe", "age": 20 }
>         }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  }
}
更新也可以通过使用简单的脚本来进行。这个例子使用一个脚本将age加5
[yangql@hadoop01 bin]$ curl -XPOST 'hadoop01:9200/customer/external/1/_update?pretty' -d '
>         {
>           "script" : "ctx._source.age += 5"
>         }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  }
}
```
- 删除文档
```
 curl -XDELETE 'hadoop01:9200/customer/external/2?pretty'
 一次删除符合某个查询条件的多个文档
 [yangql@hadoop01 bin]$ curl -XDELETE 'hadoop01:9200/customer/external/_query?pretty' -d '
>         {
>           "query": { "match": { "name": "John" } }
>         }'
{
  "found" : false,
  "_index" : "customer",
  "_type" : "external",
  "_id" : "_query",
  "_version" : 1,
  "result" : "not_found",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  }
}
```

## 3.批处理 ##
 bulk API按顺序执行这些动作。如果其中一个动作因为某些原因失败了，将会继续处理它后面的动作。当bulk API返回时，它将提供每个动作的状态（按照同样的顺序），所以你能够看到某个动作成功与否。
- 批量插入
```
[yangql@hadoop01 bin]$ curl -XPOST 'hadoop01:9200/customer/external/_bulk?pretty' -d '
>         {"index":{"_id":"1"}}
>         {"name": "John Doe" }
>         {"index":{"_id":"2"}}
>         {"name": "Jane Doe" }
>         '
{
  "took" : 636,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "1",
        "_version" : 5,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "created" : false,
        "status" : 200
      }
    },
    {
      "index" : {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "created" : true,
        "status" : 201
      }
    }
  ]
}
```
- 更新删除
```
curl -XPOST 'hadoop01:9200/customer/external/_bulk?pretty' -d '
			 {"update":{"_id":"1"}}
			 {"doc": { "name": "John Doe becomes Jane Doe" } }
			 {"delete":{"_id":"2"}}
			 '
```
## 4.查询 ##
Elasticsearch提供一种JSON风格的特定领域语言，利用它你可以执行查询。这杯称为查询DSL  
我们在customer索引中搜索（\_search端点），并且q=\*参数指示Elasticsearch去匹配这个索引中所有的文档。pretty参数，和以前一样，仅仅是告诉Elasticsearch返回美观的JSON结果
          - took —— Elasticsearch执行这个搜索的耗时，以毫秒为单位
          - timed_out —— 指明这个搜索是否超时
          - \_shards —— 指出多少个分片被搜索了，同时也指出了成功/失败的被搜索的shards的数量
          - hits —— 搜索结果
          - hits.total —— 能够匹配我们查询标准的文档的总数目
          - hits.hits —— 真正的搜索结果数据（默认只显示前10个文档）
          - \_score和max_score —— 现在先忽略这些字段
```
[yangql@hadoop01 bin]$ curl 'hadoop01:9200/customer/_search?q=*&pretty'
{
  "took" : 2370,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "AVz0W-I9v5C-xxEQKDDx",
        "_score" : 1.0,
        "_source" : {
          "name" : "Jane Doe"
        }
      },
      {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Jane Doe"
        }
      },
      {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "John Doe"
        }
      }
    ]
  }
}
请求方法体
[yangql@hadoop01 bin]$  curl -XPOST 'hadoop01:9200/customer/_search?pretty' -d '
>             {
>               "query": { "match_all": {} }
>             }'
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "AVz0W-I9v5C-xxEQKDDx",
        "_score" : 1.0,
        "_source" : {
          "name" : "Jane Doe"
        }
      },
      {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Jane Doe"
        }
      },
      {
        "_index" : "customer",
        "_type" : "external",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "John Doe"
        }
      }
    ]
  }
}
```
- 查询所有
```
{
          "query": { "match_all": {} }
}
```
分解以上的这个查询，其中的query部分告诉我查询的定义，match_all部分就是我们想要运行的查询的类型。match_all查询，就是简单地查询一个指定索引下的所有的文档。

除了这个query参数之外，我们也可以通过传递其它的参数来影响搜索结果。比如，下面做了一次match_all并只返回第一个文档：
```
        curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
        {
          "query": { "match_all": {} },
          "size": 1
        }'
```

注意，如果没有指定size的值，那么它默认就是10。

下面的例子，做了一次match_all并且返回第11到第20个文档：
```
        curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
        {
          "query": { "match_all": {} },
          "from": 10,
          "size": 10
        }'
```
其中的from参数（0-based）从哪个文档开始，size参数指明从from参数开始，要返回多少个文档。这个特性对于搜索结果分页来说非常有帮助。注意，如果不指定from的值，它默认就是0。   
下面这个例子做了一次match_all并且以账户余额降序排序，最后返前十个文档：
```
        curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
        {
          "query": { "match_all": {} },
          "sort": { "balance": { "order": "desc" } }
        }'
```
- 指定字段查询
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
       {
         "query": { "match_all": {} },
         "_source": ["account_number", "balance"]
       }'
```
注意到上面的例子仅仅是简化了_source字段。它仍将会返回一个叫做_source的字段，但是仅仅包含account_number和balance来年改革字段。
