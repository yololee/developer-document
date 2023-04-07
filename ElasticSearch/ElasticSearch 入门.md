# ElasticSearch 基本使用

系统环境

ElasticSearch：elasticsearch:7.14.1

kibana:7.14.1

服务器：Ubuntu 16.04.6 LTS

## 一、简介

Elasticsearch 是一个**分布式**的、开源的**搜索分析引擎**，支持各种数据类型，包括文本、数字、地理、结构化、非结构化

Elastic Search基于lucene，**封装**了许多**lucene**底层功能，提供了分布式的服务、简单易用的restful API接口和许多语言的客户端

**Elasticsearch 简称ES, 经常与Logstash 和Kibana 一起使用，江湖人称ELK.**

- 这E 自然指的就是Elasticsearch,简称ES, 具有分布式存储,搜索和分析的功能
- L 指的就是Logstash,分布式日志收集框架
- K 指的就是Kibana,可视化分析框架

## 二、参考文档

[docker部署es](https://gitee.com/huanglei1111/docker-compose/tree/master/Linux/elasticsearch)

[REST APIS 参考文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/indices-create-index.html)

[Java REST Client 参考文档](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html)

## 三、ES核心概念

### 核心概念

- 集群（Cluster）：包含一个或多个启动着es实例的机器群。通常一台机器起一个es实例。默认集群名是“elasticsearch”，同一网络，同一集群名下的es实例会自动组成集群

- 节点（Node）：一个es实例即为一个节点
- 索引（Index）：即拥有相似文档的集合
- 类型（Type）：每个索引里都可以有一个或多个type，type是index中的一个逻辑数据分类，一个type下的document，都有相同的field。7.x版本正式被去除
- 文档（Document）：es中的最小数据单元。一个document就像数据库中的一条记录。通常以json格式显示。多个document存储于一个索引（Index）中
- 映射（Mapping）：定义索引中的字段的名称，定义字段的数据类型，比如字符串、数字、布尔

### 数据类型

- 简单数据类型
  - 字符串：text：会分词，不支持聚合。keyword：不会分词，将全部内容作为一个词条，支持聚合
  - 数值
  - 布尔：boolean
  - 二进制：binary
  - 范围类型：**integer_range, float_range, long_range, double_range, date_range**
  - 日期：date
- 复杂数据类型
  - 数组：[ ] Nested: `nested` (for arrays of JSON objects 数组类型的JSON对象)
  - 对象：{ } Object: object(for single JSON objects 单个JSON对象)

## 四、REST APIS

### 索引API

```
# 创建索引
PUT /my_index_000001

# 添加映射
PUT /my_index_000001/_mapping
{
   "properties":{
     "name":{
       "type":"text"
     },
     "age":{
       "type":"integer"
     }
   }
}

# 更新映射   更新映射也就是只添加字段
PUT /my_index_000001/_mapping
{
   "properties": {
      "name":{
       "type":"text"
      },
      "picture_url": {
			  "type": "keyword",
			  "ignore_above": 500
		  }
    }
}


# 查询所有索引
GET _cat/indices

# 查看一个索引
GET /my_index_000001

# 删除索引
DELETE /my_index_000001

# 创建索引并添加映射
PUT /my_index_000002
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "birthday":{
        "type": "date",
        "format": "yyyy‐MM‐dd HH:mm:ss||yyyy‐MM‐dd"
      }
    }
  }
}

# 创建多个索引，并添加映射
PUT /my_index_000002，my_index_000003
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "birthday":{
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}
```

### 文档API

```java
# 创建索引，并添加映射
# 下边的设置允许birthday字段存储年月日时分秒、年月日及毫秒三种格式
PUT /my_index_000005
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "birthday":{
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}

# 插入一条数据 这里的1是文档id，没有指定id自动生成id
POST /my_index_000005/_doc/1
{
  "name": "test1",
  "age": 16,
  "birthday": "2015-01-01"
}
# 插入一条数据
POST /my_index_000005/_doc/2
{
  "name": "test1",
  "age": 16,
  "birthday": "1420070400001"
}
# 插入一条数据
POST /my_index_000005/_doc/3
{
  "name": "test1",
  "age": 16,
  "birthday": "2015-01-01 13:14:59"
}

# 查询这个索引中的所有文档
GET /my_index_000005/_search
# 根据id查询文档
GET /my_index_000005/_doc/1


# 删除文档
DELETE my_index_000005/_doc/1

# 修改文档
PUT my_index_000005/_doc/2
{
  "name":"李四",
  "age": 18,
  "birthday": "1420070400001"
}
```

### 查询文档API

query基本匹配查询关键字说明

| 关键字    | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| match_all | 查询简单的 匹配所有文档。在没有指定查询方式时，它是默认的查询 |
| match     | 用于全文搜索或者精确查询，如果在一个精确值的字段上使用它， 例如数字、日期、布尔或者一个 not_analyzed 字符串字段，那么它将会精确匹配给定的值 |
| range     | 查询找出那些落在指定区间内的数字或者时间 gt 大于；gte 大于等于；lt 小于；lte 小于等于 |
| term      | 被用于精确值 匹配                                            |
| terms     | terms 查询和 term 查询一样，但它允许你指定多值进行匹配       |
| exists    | 查找那些指定字段中有值的文档                                 |
| missing   | 查找那些指定字段中无值的文档                                 |
| must      | 多组合查询 必须匹配这些条件才能被包含进来                    |
| must_not  | 多组合查询 必须不匹配这些条件才能被包含进来                  |
| should    | 多组合查询 如果满足这些语句中的任意语句，将增加 _score ，否则，无任何影响。它们主要用于修正每个文档的相关性得分 |
| filter    | 多组合查询 这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档 |

环境准备

```java
# 创建索引
PUT xc_course
# 添加映射
PUT /xc_course/_mapping
{
  "properties": {
    "description": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "name": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "pic":{
        "type":"text",
        "index":false
      },
      "price": {
        "type": "float"
      },
      "studymodel": {
        "type": "keyword"
      },
      "timestamp": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }  
  }
}

POST /xc_course/_doc/1
{
"name": "Bootstrap开发",
"description": "Bootstrap是由Twitter推出的一个前台页面开发框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长页面开发的程序人员）轻松的实现一个不受浏览器限制的精美界面效果。",
"studymodel": "201002",
"price":38.6,
"timestamp":"2018-04-25 19:11:35",
"pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}

POST /xc_course/_doc/2
{
"name": "java编程基础",
"description": "java语言是世界第一编程语言，在软件开发领域使用人数最多。",
"studymodel": "201001",
"price":68.6,
"timestamp":"2018-03-25 19:11:35",
"pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}

POST /xc_course/_doc/3
{
"name": "spring开发基础",
"description": "spring 在java领域非常流行，java程序员都在用。",
"studymodel": "201001",
"price":88.6,
"timestamp":"2018-02-24 19:11:35",
"pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}
```

**只查询一条studymodel为201001的数据**

```java
POST /xc_course/_search
{
  "query": {
    "match": {
      "studymodel": "201001"
    }
  },
  "size": 1
}
```

**只查询前20条studymodel为201001的数据并以price降序排列**

```java
POST /xc_course/_search
{
  "query": {
    "match": {
      "studymodel": "201001"
    }
  },
  "size": 20,
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```

**只查询前20条price大于50小于90的数据并以price降序排列**

```java
POST /xc_course/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 50,
        "lte": 90
      }
    }
  },
  "size": 20,
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```

**只查询前20条timestamp为下面具体值的数据并以price降序排列**

```java
POST /xc_course/_search
{
  "query": {
    "terms": {
      "timestamp": [
        "2018-02-24 19:11:35",
        "2018-03-25 19:11:35"
      ]
    }
  },
  "size": 20,
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```

## 五、SQL REST API

```java
POST /_sql
{
  "query": """
  SELECT * FROM "xc_course" where studymodel = 201001 ORDER BY price DESC LIMIT 5
  """
}

POST /_sql
{
  "query": """
  SELECT * FROM "ks-logstash-log*"
  """,
  "fetch_size":5
}
```

**使用ES查询DSL进行过滤**

```java
POST /_sql
{
  "query": "SELECT * FROM xc_course ORDER BY price DESC",
  "filter":{
    "range": {
      "price": {
        "gte": 50,
        "lte": 90
      }
    }
  },
  "fetch_size":5
}


POST /_sql
{
  "query": "SELECT * FROM xc_course ORDER BY price DESC",
  "filter":{
    "terms": {
      "studymodel": ["201001"]
    }
  },
  "fetch_size":5
}
```

**异步运行sql搜索**

```java
POST /_sql
{
  "query": """
  SELECT * FROM "ks-logstash-log*"
  """,
  "fetch_size":20,
  "wait_for_completion_timeout": "1s"
}
```

- 值为`is_partial`，`true`表示搜索结果不完整
- 值`is_running`，`true`表示搜索仍在后台运行
- 一个`id`用于搜索

