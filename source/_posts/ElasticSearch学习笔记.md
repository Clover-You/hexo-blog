---
title: ElasticSearch-学习笔记
date: 2021-12-21 11:13:10
tags: [学习笔记,JAVA]
categories: [JAVA]
---

## 基本概念

### Index（索引）

保存一条数据到 ElasticSearch 中叫做索引一条数据



### Type（类型）

在 **Index** 中，可以定义一个或多个类型，它类似于 MySQL 中的 Table。每一种类型的数据都放在一起。



### Document（文档）

保存在某个索引下某种类型中的一条数据（Document），在 ES 中，每一个数据都叫文档。 文档是 JSON 格式的，Document 就像是 MySQL 中的某个 Table 里面保存的内容。

![截屏2021-12-15 16.34.31](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-12-15%2016.34.31.png)

 

### 倒排索引



## Docker 安装ES、Kibana

### 下载镜像文件

```
docker pull elasticsearch:7.4.2   es搜索引擎
docker pull kibana:7.4.2   可视化检索数据
```



### 安装ES

在本地创建 config 文件夹，用于存放 ES 配置

```
mkdir ~/code/elasticsearch/config
```

创建 data 文件夹，用于存放 ES 数据

```
mkdir ~/code/elasticsearch/data
```

使用 docker 启动 ES

```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v ~/code/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v ~/code/elasticsearch/data:/usr/share/elasticsearch/data \
-v  ~/code/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2 
```

在浏览器上访问 `http://localhost:9200/` ，如果显示以下内容，则代表安装/启动成功

![截屏2021-12-19 10.32.26](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-12-19%2010.32.26.png)



### 安装Kibana

在 docker 中安装了 Kibana 之后，使用 `docker run` 启动它

```
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.21.133:9200 -p 5601:5601 \
-d kibana:7.4.2
```

稍等一会后在浏览器中输入 `http://127.0.0.1:5601` 访问 Kibana。

![截屏2021-12-19 10.53.12](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-12-19%2010.53.12.png)

## 初步检索

 

### \_cat

查看所有节点

```http
get /_cat/nodes
```

查看 es 健康状况

```http
get /_cat/health
```

查看主节点

```http
get /_cat/master
```

查看所有索引

```http
get /_cat/indices
```



### 索引一个文档

#### PUT

保存一条数据，该方式需要指定一个id。保存在那个索引那中的的哪个类型下，指定使用一个唯一标识。同样的请求若发送多次，那么被 ES 视为更新操作

在 **customer** 索引下的 **external** 类型中保存一条 “id” 为 **1** 的数据

```http
put customer/external/1
{"name": "Clover You"}
```

发送以上请求成功后得到返回结果：

```json
{
  "_index": "costomer",
  "_type": "extenal",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

在返回数据中，所有带 “**\_**” 的都称为元数据，例如 “_index”

- `_index` 当前插入的这条数据在哪个索引下
- `_type` 当前插入的数据在哪个类型下
- `_id` 这条数据保存的id
- `_version` 当前数据的版本号
- `result` 当前操作结果
  - `created` 表示新建数据
  - `updated` 表示修改数据
  - `noop` 无任何操作
- `_shards` 分片
- 



#### POST

POST 的方式和 PUT差不多，只不过 POST 可以不指定id。如果不指定id，那么就会自动生成一个id。如果指定id那么就会修改这个id的数据，并新增版本号。

```http
post customer/external/1
{"name": "Clover You"}
```





### 查询文档

使用 GET 请求查询文档

```http
get customer/external/1
```



```json
{
  "_index": "costomer",
  "_type": "extenal",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Clover You"
  }
}
```

- `_index` 查询的文档在哪个索引下
- `_type` 查询的文档在哪个类型下
- `_id` 查询的文档的唯一id
- `_version` 文档的版本号
- `_seq_no` 并发控制字段，每次更新都会 +1，用来做乐观锁
- `_primary_term`  与 `_seq_no` 一样，主分片重新分配，如重启，就会变化 

> 如果需要做乐观锁，需要带上 `_seq_no` 和 `_primary_term `参数:
>
> ```http
> put customer/external/1?if_seq_no=0&if_primary_term=1
> ```



### 更新文档

 除了使用以上描述的 POST 与 PUT 的方式更新文档外，还可以使用专门的接口进行更新。该操作会拿新旧文档元数据进行比较，如果没有任何更改，那么就不更新。

```http
post costomer/extenal/1/_update
{
	"doc": {
		"name": "new name"
	}
}
```



### 删除文档

发送 DELETE 请求指定索引、类型与唯一标识即可删除指定文档

```http
delete costomer/extenal/1
```



### 删除索引

ES 没有提供删除类型的接口，但它提供了删除索引的接口

```http
delete costomer
```

```json
{
    "acknowledged": true
}
```



### 批量操作

ES 提供了一个批量操作接口 `index/type/_bulk` ，这个接口支持新增修改删除等操作。它需要发送一个 POST 请求，并且数据格式如下：

```json
{"index": {"_id": 10}}
{"name": "Clover"}
{"index": {"_id": 11}}
{"name": "Clover You"}
```

该接口的数据需要两行为一组，第一行`{"动作，如：新增修改删除": {"_id": 文档唯一标识}}`，第二行是需要操作的数据，`{"name": "Clover You"}`

```http
POST /costomer/extenal/_bulk
{"index": {"_id": 10}}
{"name": "Clover"}
{"index": {"_id": 11}}
{"name": "Clover You"}
```



```json
{
  "took" : 5,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "costomer",
        "_type" : "extenal",
        "_id" : "10",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "costomer",
        "_type" : "extenal",
        "_id" : "11",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}

```

- `took` 完成一共花费了多少毫秒
- `errors` 是否有错误
- `items` 每个子级都对应了每个操作，他们都是独立的，任何操作出错都不会影响到其它操作。



#### 对整个ES进行批量操作

除了 `index/type/_bulk` 接口外，还有一个 `_bulk` 接口可以对整个ES进行操作

它的操作与 `index/type/_bulk` 的操作没有很大区别

```http
post /_bulk
{"index", {"_index": "costomer", "_type": "extenal", "_id": "123"}}
{"name": "clover"}
```

- `_index` 指定操作的索引
- `_type` 指定操作的类型
- `_id` 文档唯一标识



## 进阶检索

[测试数据](https://github.com/elastic/elasticsearch/blob/v7.4.2/docs/src/test/resources/accounts.json)

### SearchAPI

ES 支持两种基本方式进行检索

- 一个是通过使用 REST request URI 发送搜索参数

  ```http
  GET /bank/_search?q=*&sort=account_number:asc
  ```

  

- 另一种是通过使用 REST request body 来发送它们

  ```http
  GET /bank/_search
  {
    "query": {
      "match_all": {},
    },
    "sort": [
      {"account_number": "asc"}
    ]
  }
  ```



### Query DSL

ElasticSearch 提供了一个可以执行查询的 JSON 风格的 DSL（domain-specific language领域特定语言）。它被称为Query DSL

```json
// 基本语法
{
  QUERY_NAME: {
    ARGUMENT: VALUE,
    ARGUMENT: VALUE
  }
}
// 如果针对的是某个字段，那么他的语法：
{
  QUERY_NAME: {
    FIELD_NAME: {
      ARGUMENT: VALUE,
      ARGUMENT: VALUE
    }
  }
}
```



#### match 全文检索

全文检索按照评分进行排序，会使用到排索引对检索条件进行分词匹配

```http
GET /bank/_search
{
  "query": {
    "match": {
      "address": "mill lane"
    }
  }
}
```

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 19,
      "relation" : "eq"
    },
    "max_score" : 9.507477,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 9.507477,
        "_source" : {
          ...
          "address" : "198 Mill Lane",
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 5.4032025,
        "_source" : {
          ...
          "address" : "990 Mill Road",
        }
      },
      ...
    ]
  }
}

```



#### match_phrase 短语匹配

将需要匹配的值当成一个整体单词进行检索（不进行分词）和 MySQL 中的 `like` 一致

```http
GET /bank/_search
{
  "query": {
    "match_phrase": {
      "address": "Mill Lane"
    }
  }
}
```

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 9.507477,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 9.507477,
        "_source" : {
          ...
          "address" : "198 Mill Lane",
        }
      }
    ]
  }
}

```





#### multi_match 多字段匹配

可以对多个字段使用到排索引进行检索

```http
GET /bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill urie",
      "fields": ["address", "city"]
    }
  }
}
```



```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 6.505949,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 6.505949,
        "_source" : {
          ...
          "address" : "198 Mill Lane",
          "city" : "Urie"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 5.4032025,
        "_source" : {
          ...
          "address" : "990 Mill Road",
          "city" : "Lopezo"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "345",
        "_score" : 5.4032025,
        "_source" : {
          ...
          "address" : "715 Mill Avenue",
          "city" : "Blackgum",
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "472",
        "_score" : 5.4032025,
        "_source" : {
          "address" : "288 Mill Street",
          "city" : "Movico",
        }
      }
    ]
  }
}

```



#### bool 复合查询

如果需要使用更复杂的查询，可以使用 `bool` 查询，它可以帮我们构造更加复杂的查询。他可以合并多个查询条件

- `must_not` 必须不是指定条件
- `must` 必须满足指定条件
- `should` 尽量满足指定条件



```http
GET /bank/_search
{
  "query": {
    "bool": {
      "must":[
        {
          "match": {
            "address": "Street"
          }
        }
      ],
      "should": [
        {
          "match": {
            "gender": "f"
          }
        }
      ]
    }
  }
}
```

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 385,
      "relation" : "eq"
    },
    "max_score" : 1.661185,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "13",
        "_score" : 1.661185,
        "_source" : {
          ...
          "gender" : "F",
          "address" : "789 Madison Street",
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "32",
        "_score" : 1.661185,
        "_source" : {
          "gender" : "F",
          "address" : "702 Quentin Street"
        }
      },
       {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "87",
        "_score" : 0.95395315,
        "_source" : {
          ...
          "gender" : "M",
          "address" : "446 Halleck Street"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "107",
        "_score" : 0.95395315,
        "_source" : {
          ...
          "gender" : "M",
          "address" : "694 Jefferson Street"
        }
      },
    ]
  }
}

```



#### term 检索

在检索精确字段时，官方推荐使用 `term` 检索，如果不是精确字段，不推荐使用 `term` 进行检索，例如字符串可能不是精确字段。因为 ES 在保存文档的时候使用了分词的问题

```json
GET /bank/_search
{
  "query": {
    "term": {
      "age": {
        "value": 30
      }
    }
  }
}
```



### aggregations（执行聚合）

聚合提供了从数据中分组和提取数据的能力。最简单的聚合方法大致等于 **SQL GROUP BY** 和 SQL 聚合函数。在 ES 中，你有执行搜索返回 **hits**（命中结果）并同时返回聚合结果。把一个响应中的所有 **hits** 分割开的能力。这是非常强大且有效的，你可以执行查询和多个聚合，并且在一次使用中得到各自的（任何一个）返回结果，使用一次简洁和简化的 **API** 来避免网络往返。

```json
"aggregations" : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```

例如搜索address中包含mill的所有人的年龄分布以及平均年龄

```http
GET /bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    }
  }
}
```

```json
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 5.4032025,
    "hits" : [...]
  },
  "aggregations" : {
    "ageAgg" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 38,
          "doc_count" : 2
        },
        {
          "key" : 28,
          "doc_count" : 1
        },
        {
          "key" : 32,
          "doc_count" : 1
        }
      ]
    },
    "ageAvg" : {
      "value" : 34.0
    }
  }
}

```



按照年龄聚合，并且请求这些年龄段的这些人的平均工资

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "ageAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ageAgg" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 31,
          "doc_count" : 61,
          "ageAvg" : {
            "value" : 28312.918032786885
          }
        },
        {
          "key" : 39,
          "doc_count" : 60,
          "ageAvg" : {
            "value" : 25269.583333333332
          }
        },
        {
          "key" : 26,
          "doc_count" : 59,
          "ageAvg" : {
            "value" : 23194.813559322032
          }
        },
        {
          "key" : 32,
          "doc_count" : 52,
          "ageAvg" : {
            "value" : 23951.346153846152
          }
        },
        {
          "key" : 35,
          "doc_count" : 52,
          "ageAvg" : {
            "value" : 22136.69230769231
          }
        },
        {
          "key" : 36,
          "doc_count" : 52,
          "ageAvg" : {
            "value" : 22174.71153846154
          }
        },
        {
          "key" : 22,
          "doc_count" : 51,
          "ageAvg" : {
            "value" : 24731.07843137255
          }
        },
        {
          "key" : 28,
          "doc_count" : 51,
          "ageAvg" : {
            "value" : 28273.882352941175
          }
        },
        {
          "key" : 33,
          "doc_count" : 50,
          "ageAvg" : {
            "value" : 25093.94
          }
        },
        {
          "key" : 34,
          "doc_count" : 49,
          "ageAvg" : {
            "value" : 26809.95918367347
          }
        },
        {
          "key" : 30,
          "doc_count" : 47,
          "ageAvg" : {
            "value" : 22841.106382978724
          }
        },
        {
          "key" : 21,
          "doc_count" : 46,
          "ageAvg" : {
            "value" : 26981.434782608696
          }
        },
        {
          "key" : 40,
          "doc_count" : 45,
          "ageAvg" : {
            "value" : 27183.17777777778
          }
        },
        {
          "key" : 20,
          "doc_count" : 44,
          "ageAvg" : {
            "value" : 27741.227272727272
          }
        },
        {
          "key" : 23,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 27314.214285714286
          }
        },
        {
          "key" : 24,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 28519.04761904762
          }
        },
        {
          "key" : 25,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 27445.214285714286
          }
        },
        {
          "key" : 37,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 27022.261904761905
          }
        },
        {
          "key" : 27,
          "doc_count" : 39,
          "ageAvg" : {
            "value" : 21471.871794871793
          }
        },
        {
          "key" : 38,
          "doc_count" : 39,
          "ageAvg" : {
            "value" : 26187.17948717949
          }
        },
        {
          "key" : 29,
          "doc_count" : 35,
          "ageAvg" : {
            "value" : 29483.14285714286
          }
        }
      ]
    }
  }
}
```

查出所有年龄分布，并且这些年龄段中  M  的平均薪资和  F  的平均薪资以及这个年龄 段的总体平均薪资

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
	  # 年龄分布情况
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 2
      },
      "aggs": {
      	# 每个年龄的性别分布情况
        "genderAgg": {
          "terms": {
            "field": "gender.keyword",
            "size": 2
          },
          "aggs": {
          	# 每个性别的平均工资
            "balanceAvg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        },
        # 每个年龄段的平均工资
        "ageBalanceAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```



```json
{
  "aggregations" : {
    "ageAgg" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 879,
      "buckets" : [
        {
          "key" : 31,
          "doc_count" : 61,
          "genderAgg" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "M",
                "doc_count" : 35,
                "balanceAvg" : {
                  "value" : 29565.628571428573
                }
              },
              {
                "key" : "F",
                "doc_count" : 26,
                "balanceAvg" : {
                  "value" : 26626.576923076922
                }
              }
            ]
          },
          "ageBalanceAvg" : {
            "value" : 28312.918032786885
          }
        },
        {
          "key" : 39,
          "doc_count" : 60,
          "genderAgg" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "F",
                "doc_count" : 38,
                "balanceAvg" : {
                  "value" : 26348.684210526317
                }
              },
              {
                "key" : "M",
                "doc_count" : 22,
                "balanceAvg" : {
                  "value" : 23405.68181818182
                }
              }
            ]
          },
          "ageBalanceAvg" : {
            "value" : 25269.583333333332
          }
        }
      ]
    }
  }
}

```





### Mapping（映射）

映射是用来定义文档是如何被进行处理的，也就是说它是如何被检索和索引的，例如我们可以自定义：

- 哪些 string 类型的字段应该被当成全文检索的字段
- 哪个属性包含数字、日期或地理位置坐标
- 日期的格式化信息
- 可以使用自定义规则来控制动态字段的映射



> - 关系型数据库中两个数据表是独立的，即使它们里面有相同名称的列也不影响使用，但在 ES 中不是这样的。ES 是基于 Lucene 开发的搜索引擎，而 ES 中不同 type 下名称相同的 filed 最终在 Lucene 中的处理方式是一样的。
>   - 两个不同 type 下的两个 user_name，在 ES 同一个索引下其实被认为是同一个 filed，你必须在两个不同的 type 中定义相同的 filed 映射。否则，不同 type 中的相同字段名称就会在处理中出现冲突的情况，导致 Lucene 处理效率下降。
> - ElasticSearch 7.x
>   - URL 中的 type 参数为可选。比如：索引一个文档不再要求文档类型。
> - ElasticSearch 8.x
>   - 不再支持 URL 中的 type 参数
> - 解决：将索引从多类型迁移到单类型，每种类型文档一个独立索引



#### 创建映射

可以在创建索引时创建映射规则

```http
PUT /my_index
{
  "mappings": {
    "properties": {
      "age": {"type": "integer"},
      "email": {"type": "keyword"},
      "name": {"type": "text"}
    }
  }
}
```



#### 添加新字段映射

```http
PUT my_index/_mapping
{
  "prope rties": {
    "employee-id": {
      "type": "long",
      "index": false
    }
  }
}
```

- `index` 指定该字段是否参与检索，默认为 `true`



#### 数据迁移

> 对于已存在的映射，是不能被修改的

若需要将旧索引中的数据迁移到指定索引中，可以使用 `_reindex` API进行操作，先创建对应的索引：

```http
PUT newindex
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "email": {
        "type": "keyword"
      },
      "employer": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "firstname": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "state": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

最后发送指定请求将数据迁移

```http
POST _reindex
{
  "source": {
    "index": "bank",
    "type": "account"
  },
  "dest": {
    "index": "newindex"
  }
}
```

- `source` 源数据
  - `index` 源数据所在的索引
  - `type` 源数据所在的类型，如果没有可不填
- `dest` 目标数据
  - `index` 目标索引
  - `type` 迁移到目标索引的指定类型下



### 分词

一个 tokenizer（分词器）接收一个字符流，将之分割为独立的 tokens（词元，通常是独立的单词），然后输出 tokens 流。它遇到空白字符时就分割文本，他会将文本 **“Quick brown fox!”** 分割为  **[Quick, brown, fox!]**。该 **tokenizer** 还负责记录各个 **term（词条）**的顺序或 **position** 位置（用于 **phrase** 短语和 **word proximity** 词近邻查询），以及 **term** 所代表的原始 **word（单词）**的 **start（起始）**和 **end（结束）**的 **character offsets（字符偏移量， 用于高亮显示搜索内容）**。ES 提供了很多内置的分词器，可以用来构建 **custom analyzers（自定义分词器）**

```http
POST _analyze
{
  "analyzer": "standard",
  "text": "hello world"
}
```

```json
{
  "tokens" : [
    {
      "token" : "hello",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "world",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```



#### ik分词器

到[GitHub](https://github.com/medcl/elasticsearch-analysis-ik/releases)中安装对应版本的ik分词器并解压到 `plugins` 目录，最后重启 ES即可

```
docker restart esid
```

**ik** 提供了两种分词器，分别是`ik_smart` 、`ik_max_word`

使用 **ik** 分词器中的 `ik_smart`，可以对指定文本进行智能分词

> `ik_smart` 效果

```http
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "日出于东却落于西,相识人海却散于席"
}
```

```json
{
  "tokens" : [
    {
      "token" : "日",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "出于",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "东",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "却",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "落于",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "西",
      "start_offset" : 7,
      "end_offset" : 8,
      "type" : "CN_CHAR",
      "position" : 5
    },
    {
      "token" : "相识",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "人海",
      "start_offset" : 11,
      "end_offset" : 13,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "却",
      "start_offset" : 13,
      "end_offset" : 14,
      "type" : "CN_CHAR",
      "position" : 8
    },
    {
      "token" : "散",
      "start_offset" : 14,
      "end_offset" : 15,
      "type" : "CN_CHAR",
      "position" : 9
    },
    {
      "token" : "于",
      "start_offset" : 15,
      "end_offset" : 16,
      "type" : "CN_CHAR",
      "position" : 10
    },
    {
      "token" : "席",
      "start_offset" : 16,
      "end_offset" : 17,
      "type" : "CN_CHAR",
      "position" : 11
    }
  ]
}

```

> `ik_max_word` 效果

```json
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "日出于东却落于西,相识人海却散于席"
}
```

```json
{
  "tokens" : [
    {
      "token" : "日出",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "出于",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "东",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "却",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "落于",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "西",
      "start_offset" : 7,
      "end_offset" : 8,
      "type" : "CN_CHAR",
      "position" : 5
    },
    {
      "token" : "相识",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "人海",
      "start_offset" : 11,
      "end_offset" : 13,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "却",
      "start_offset" : 13,
      "end_offset" : 14,
      "type" : "CN_CHAR",
      "position" : 8
    },
    {
      "token" : "散",
      "start_offset" : 14,
      "end_offset" : 15,
      "type" : "CN_CHAR",
      "position" : 9
    },
    {
      "token" : "于",
      "start_offset" : 15,
      "end_offset" : 16,
      "type" : "CN_CHAR",
      "position" : 10
    },
    {
      "token" : "席",
      "start_offset" : 16,
      "end_offset" : 17,
      "type" : "CN_CHAR",
      "position" : 11
    }
  ] 
}

```



#### 自定义扩展分词库

在 **ik** 分词器配置中，修改`elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml` 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict"></entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<entry key="remote_ext_dict">http://192.168.21.133/es/fenci.txt</entry>
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

把 `http://192.168.21.133/es/fenci.txt` 改成你自己词库的位置，格式如下：

```tex
电商
乔碧萝
乔碧萝殿下
```

> 可以使用 `nginx` 来部署你的词库

修改后需要重启 **ElasticSearch** `docker restart container_id`

看效果：

> 添加词库前

![添加词库前](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-12-20%2016.21.09.png)



> 添加词库后

![修改词库后](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-12-20%2016.18.20.png)





## 整合SpringBoot

使用官方提供的高阶依赖

```xml
<dependency>
  <groupId>org.elasticsearch.client</groupId>
  <artifactId>elasticsearch-rest-high-level-client</artifactId>
  <version>7.4.2</version>
</dependency>
```

还需要把 SpringBoot 默认导入的 ES 版本修改，只需要将版本号对应你需要导入的版本就好

```xml
<properties>
  <elasticsearch.version>7.4.2</elasticsearch.version>
</properties>
```

配置 ES

```java
@Configuration
public class GuliMallElasticSearchConfig {

  public static final RequestOptions COMMON_OPTIONS;
  static {
    RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
    COMMON_OPTIONS = builder.build();
  }

  @Bean
  public RestHighLevelClient restHighLevelClient() {
    return new RestHighLevelClient(
      RestClient.builder(
        new HttpHost("localhost", 9200, "http")
      )
    );
  }

}
```



### 基本使用

#### IndexRequest

`IndexRequest` 用于索引一条文档，如果索引不存在则创建

```java
@Autowired
private RestHighLevelClient esClient;

void indexTest() throws IOException {
  IndexRequest request = new IndexRequest("users");
  request.id("1");
  User user = new User();
  user.setUserName("clover");
  user.setAge(19);
  request.source(new Gson().toJson(user), XContentType.JSON);
  IndexResponse index = esClient.index(request, GuliMallElasticSearchConfig.COMMON_OPTIONS);
  log.info("index: {}", index);
}
```

- `IndexRequest` 

  他有以下三个构造，分别是指定索引、指定索引和类型、指定索引、类型和唯一标识，在 **7.x** 中最常用的是第一个构造，可以使用 `(new IndexRequest("xxx")).id("xxx")` 来指定唯一标识。

  1. `public IndexRequest(String index)`
  2. `public IndexRequest(String index, String type)`
  3. `public IndexRequest(String index, String type, String id)`

  可通过 `source` 方法传递参数，这个方法有非常多的重载，但最常用的是这两个

  1. `public IndexRequest source(Map<String, ?> source)` 可以传递一个 **Map** ，非常方便
  2. `public IndexRequest source(String source, XContentType xContentType)` 将 `xContentType` **指定为 `JSON`** 后可传一个 `JSON` 字符串。

请求构造好后使用 `RestHighLevelClient` 中对应的api将请求发送出去。



### 复杂检索

排按照年齡聚合，并且请求这些年龄段的这些人的平均薪资

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "balanceAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

```java
@Test
@DisplayName("复杂检索")
void searchRequestTest() throws IOException {
  // 创建一个检索请求
  SearchRequest request = new SearchRequest();
  // 指定索引
  request.indices("bank");
  // 创建检索条件
  SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
  searchSourceBuilder.size(100);
  // "query": {
  //     "match_all": {}
  // }
  searchSourceBuilder.query(QueryBuilders.matchAllQuery());
  // "query": {
  //     "match": {"address", "mill"}
  // }
  //        searchSourceBuilder.query(QueryBuilders.matchQuery("address", "mill"));
  // 聚合
  //"aggs": {
  //  "ageAgg": {
  //    "terms": {
  //      "field": "age",
  //      "size": 100
  //    }
  //  }
  //}
  searchSourceBuilder.aggregation(AggregationBuilders.terms("ageAgg").field("age").size(100)
                                  //"aggs": {
                                  //  "ageAgg": {
                                  //    "terms": {
                                  //      "field": "age",
                                  //      "size": 100
                                  //    },
                                  //    "aggs": {
                                  //      "ageAvg": {
                                  //        "avg": {
                                  //          "field": "balance"
                                  //        }
                                  //      }
                                  //    }
                                  //  }
                                  //}
                                  .subAggregation(AggregationBuilders.avg("balanceAvg").field("balance"))
                                 );
  // 指定DSL
  request.source(searchSourceBuilder);
  // 执行检索
  SearchResponse search = esClient.search(request, GuliMallElasticSearchConfig.COMMON_OPTIONS);
}
```
