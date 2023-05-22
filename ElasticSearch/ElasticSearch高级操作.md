# ElasticSearch高级操作

## 一、批量操作

### 1.1 bulk批量操作

```
POST /person1/_doc/5
{
  "name":"张三5号",
  "age":18,
  "address":"北京海淀区"
}
```

批量操作文本

```json
#批量操作
#1.删除5号
#新增8号
#更新2号 name为2号
POST _bulk
{"delete":{"_index":"person1","_id":"5"}}
{"create":{"_index":"person1","_id":"8"}}
{"name":"八号","age":18,"address":"北京"}
{"update":{"_index":"person1","_id":"2"}}
{"doc":{"name":"2号"}}
```

结果

```json
{
  "took" : 51,
  "errors" : true,
  "items" : [
    {
      "delete" : {
        "_index" : "person1",
        "_type" : "_doc",
        "_id" : "5",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 6,
        "_primary_term" : 2,
        "status" : 200
      }
    },
    {
      "create" : {
        "_index" : "person1",
        "_type" : "_doc",
        "_id" : "8",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 7,
        "_primary_term" : 2,
        "status" : 201
      }
    },
    {
      "update" : {
        "_index" : "person1",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 10,
        "_primary_term" : 2,
        "status" : 200
      }
    }
  ]
}

```


### 1.2 bulk批量操作-JavaAPI

```java
/**
 *  Bulk 批量操作
 */
@Test
public void test2() throws IOException {

    //创建bulkrequest对象，整合所有操作
    BulkRequest bulkRequest =new BulkRequest();

    /*
    # 1. 删除5号记录
    # 2. 添加6号记录
    # 3. 修改3号记录 名称为 “三号”
     */
    //添加对应操作
    //1. 删除5号记录
    DeleteRequest deleteRequest=new DeleteRequest("person1","5");
    bulkRequest.add(deleteRequest);

    //2. 添加6号记录
    Map<String, Object> map=new HashMap<>();
    map.put("name","六号");
    IndexRequest indexRequest=new IndexRequest("person1").id("6").source(map);
    bulkRequest.add(indexRequest);
    
    //3. 修改3号记录 名称为 “三号”
    Map<String, Object> mapUpdate=new HashMap<>();
    mapUpdate.put("name","三号");
    UpdateRequest updateRequest=new UpdateRequest("person1","3").doc(mapUpdate);

    bulkRequest.add(updateRequest);
    //执行批量操作


    BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
    System.out.println(response.status());

}
```


### 1.3 导入数据-分析&创建索引

```json
PUT goods
{
	"mappings": {
		"properties": {
			"title": {
				"type": "text",
				"analyzer": "ik_smart"
			},
			"price": { 
				"type": "double"
			},
			"createTime": {
				"type": "date"
			},
			"categoryName": {	
				"type": "keyword"
			},
			"brandName": {	
				"type": "keyword"
			},
	
			"spec": {		
				"type": "object"
			},
			"saleNum": {	
				"type": "integer"
			},
			
			"stock": {	
				"type": "integer"
			}
		}
	}
}
```

### 1.4 导入数据-代码实现

Goods.java

```java
@Data
public class Goods {
    private int id;
    private String title;
    private double price;
    private int stock;
    private int saleNum;
    private Date createTime;
    private String categoryName;
    private String brandName;
    private Map spec;    

    @JsonIgnore  //转json数据时忽略该字段
    private String specStr;
}
```

```java
/**
 * 从Mysql 批量导入 elasticSearch
 */
@Test
public void test3() throws IOException {
    //1.查询所有数据，mysql
    List<Goods> goodsList = goodsMapper.findAll();

    //2.bulk导入
    BulkRequest bulkRequest=new BulkRequest();

    //2.1 循环goodsList，创建IndexRequest添加数据
    for (Goods goods : goodsList) {

        //2.2 设置spec规格信息 Map的数据   specStr:{}
        String specStr = goods.getSpecStr();

        //将json格式字符串转为Map集合
        Map map = JSON.parseObject(specStr, Map.class);

        //设置spec map
        goods.setSpec(map);

        //将goods对象转换为json字符串
        String data = JSON.toJSONString(goods);

        IndexRequest indexRequest=new IndexRequest("goods").source(data,XContentType.JSON);
        bulkRequest.add(indexRequest);

    }


    BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
    System.out.println(response.status());

}
```

## 二. ElasticSearch查询

### 2.1 查询所有

####  REST方式

```json
# 默认情况下，es一次展示10条数据,通过from和size来控制分页
# 查询结果详解

GET goods/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 100
}
```


#### JavaAPI 方式

```java
/**
 * 查询所有
 *  1. matchAll
 *  2. 将查询结果封装为Goods对象，装载到List中
 *  3. 分页。默认显示10条
 */
@Test
public void matchAll() throws IOException {

    //2. 构建查询请求对象，指定查询的索引名称
    SearchRequest searchRequest=new SearchRequest("goods");

    //4. 创建查询条件构建器SearchSourceBuilder
    SearchSourceBuilder sourceBuilder=new SearchSourceBuilder();

    //6. 查询条件
    QueryBuilder queryBuilder= QueryBuilders.matchAllQuery();
    //5. 指定查询条件
    sourceBuilder.query(queryBuilder);

    //3. 添加查询条件构建器 SearchSourceBuilder
    searchRequest.source(sourceBuilder);
    // 8 . 添加分页信息  不设置 默认10条
	// sourceBuilder.from(0);
	// sourceBuilder.size(100);
    //1. 查询,获取查询结果

    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    //7. 获取命中对象 SearchHits
    SearchHits hits = searchResponse.getHits();

    //7.1 获取总记录数
  Long total= hits.getTotalHits().value;
    System.out.println("总数："+total);
    //7.2 获取Hits数据  数组
    SearchHit[] hits1 = hits.getHits();
        //获取json字符串格式的数据
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit searchHit : hits1) {
        String sourceAsString = searchHit.getSourceAsString();
        //转为java对象
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }

    for (Goods goods : goodsList) {
        System.out.println(goods);
    }

}
```


### 2.2 termQuery

term查询和字段类型有关系，首先回顾一下ElasticSearch两个数据类型

 ElasticSearch两个数据类型

```
text：会分词，不支持聚合

keyword：不会分词，将全部内容作为一个词条，支持聚合
```

term查询：不会对查询条件进行分词。

```
GET goods/_search
{
  "query": {
    "term": {
      "title": {
        "value": "华为"
      }
    }
  }
}
```

term查询，查询text类型字段时，只有其中的单词相匹配都会查到，text字段会对数据进行分词

例如：查询title 为“华为”的，title type 为text

![image-20230522105637071](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230522105637071.png)


![image-20230522105702576](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230522105702576.png)

查询categoryName 字段时，categoryName字段为keyword  ,keyword：不会分词，将全部内容作为一个词条,

即完全匹配，才能查询出结果

![image-20230522105724761](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230522105724761.png)


```json
GET goods/_search
{
  "query": {
    "term": {
      "categoryName": {
        "value": "华为手机"
      }
    }
  }
}
```

![image-20230522105744623](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230522105744623.png)




### 2.3 matchQuery

match查询：

会对查询条件进行分词。

然后将分词后的查询条件和词条进行等值匹配

默认取并集（OR）

```
# match查询
GET goods/_search
{
  "query": {
    "match": {
      "title": "华为手机"
    }
  },
  "size": 500
}
```

match 的默认搜索（or 并集）

例如：华为手机，会分词为 “华为”，“手机” 只要出现其中一个词条都会搜索到

match的 and（交集） 搜索

例如：例如：华为手机，会分词为 “华为”，“手机”  但要求“华为”，和“手机”同时出现在词条中

**总结：**

- term query会去倒排索引中寻找确切的term，它并不知道分词器的存在。这种查询适合**keyword** 、**numeric**、**date**
- match query知道分词器的存在。并且理解是如何被分词的


### 2.4 模糊查询

#### wildcard查询

wildcard查询：会对查询条件进行分词。还可以使用通配符 ?（任意单个字符） 和  * （0个或多个字符）

```
"*华*"  包含华字的
"华*"   华字后边多个字符
"华?"  华字后边多个字符
"*华"或"?华" 会引发全表（全索引）扫描 注意效率问题
```

```json
# wildcard 查询。查询条件分词，模糊查询
GET goods/_search
{
  "query": {
    "wildcard": {
      "title": {
        "value": "华*"
      }
    }
  }
}
```

#### 正则查询

```
\W：匹配包括下划线的任何单词字符，等价于 [A-Z a-z 0-9_]   开头的反斜杠是转义符

+号多次出现

(.)*为任意字符
正则查询取决于正则表达式的效率
```

```json
GET goods/_search
{
  "query": {
    "regexp": {
      "title": "\\w+(.)*"
    }
  }
}

```

#### 前缀查询

 对keyword类型支持比较好

```json
# 前缀查询 对keyword类型支持比较好
GET goods/_search
{
  "query": {
    "prefix": {
      "brandName": {
        "value": "三"
      }
    }
  }
}
```


#### JavaAPI

```java
//模糊查询
WildcardQueryBuilder query = QueryBuilders.wildcardQuery("title", "华*");//华后多个字符
//正则查询
RegexpQueryBuilder query = QueryBuilders.regexpQuery("title", "\\w+(.)*");
//前缀查询
PrefixQueryBuilder query = QueryBuilders.prefixQuery("brandName", "三");
```

### 2.5 范围&排序查询

#### REST方式

```json
# 范围查询
GET goods/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 2000,
        "lte": 3000
      }
    }
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
```

#### JavaAPI方式

```java
 //范围查询 以price 价格为条件
RangeQueryBuilder query = QueryBuilders.rangeQuery("price");

//指定下限
query.gte(2000);
//指定上限
query.lte(3000);

sourceBuilder.query(query);

//排序  价格 降序排列
sourceBuilder.sort("price",SortOrder.DESC);
```


### 2.6 queryString查询

queryString 多条件查询 , 会对查询条件进行分词。然后将分词后的查询条件和词条进行等值匹配 , 默认取并集（OR）, 可以指定多个查询字段

#### REST方式

**query_string：识别query中的连接符（or 、and）**

```
# queryString

GET goods/_search
{
  "query": {
    "query_string": {
      "fields": ["title","categoryName","brandName"], 
      "query": "华为 AND 手机"
    }
  }
}
```

**simple_query_string：不识别query中的连接符（or 、and），查询时会将 “华为”、"and"、“手机”分别进行查询**

```
GET goods/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["title","categoryName","brandName"], 
      "query": "华为 AND 手机"
    }
  }
}
```

**query_string：有default_operator连接符的脚本**

```json
GET goods/_search
{
  "query": {
    "query_string": {
      "fields": ["title","brandName","categoryName"],
      "query": "华为手机 "
      , "default_operator": "AND"
    }
  }
}

```

#### Java API

```java
QueryStringQueryBuilder query = QueryBuilders.queryStringQuery("华为手机").field("title").field("categoryName")
.field("brandName").defaultOperator(Operator.AND);
```

**simple_query_string：有default_operator连接符的脚本**

```json
GET goods/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["title","brandName","categoryName"],
      "query": "华为手机 "
      , "default_operator": "OR"
    }
  }
}
```

注意：query中的or   and 是查询时 匹配条件是否同时出现

- or 出现一个即可
- and 两个条件同时出现

default_operator的 or   and 是对结果进行 并集（or）、交集（and）


### 2.7 布尔查询

boolQuery：对多个查询条件连接 , 连接方式：

- must（and）：条件必须成立

- must_not（not）：条件必须不成立

- should（or）：条件可以成立

- filter：条件必须成立，性能比must高。不会计算得分

**得分 : **即条件匹配度 , 匹配度越高，得分越高

#### REST方式

```json 
# boolquery
#must和filter配合使用时，max_score（得分）是显示的
#must 默认数组形式
GET goods/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "brandName": {
              "value": "华为"
            }
          }
        }
      ],
      "filter":[ 
        {
        "term": {
          "title": "手机"
        }
       },
       {
         "range":{
          "price": {
            "gte": 2000,
            "lte": 3000
         }
         }
       }
      
      ]
    }
  }
}

#filter 单独使用   filter可以是单个条件，也可多个条件（数组形式）
GET goods/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "brandName": {
              "value": "华为"
            }
          }
        }
      ]
    }
  }
}
```


#### JavaAPI方式

布尔查询：boolQuery 

1. 查询品牌名称为:华为 
2. 查询标题包含：手机
3. 查询价格在：2000-3000 

must 、filter为连接方式

term、match为不同的查询方式

```java
       //1.构建boolQuery
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        //2.构建各个查询条件
        //2.1 查询品牌名称为:华为
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("brandName", "华为");
        boolQuery.must(termQueryBuilder);
        //2.2. 查询标题包含：手机
        MatchQueryBuilder matchQuery = QueryBuilders.matchQuery("title", "手机");
        boolQuery.filter(matchQuery);

        //2.3 查询价格在：2000-3000
        RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
        rangeQuery.gte(2000);
        rangeQuery.lte(3000);
        boolQuery.filter(rangeQuery);

        sourceBuilder.query(boolQuery);
```


### 2.8 聚合查询

指标聚合：相当于MySQL的聚合函数。max、min、avg、sum等

桶聚合：相当于MySQL的 group by 操作。不要对text类型的数据进行分组，会失败。

#### REST方式

```json
# 聚合查询

# 指标聚合 聚合函数

GET goods/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  },
  "aggs": {
    "max_price": {
      "max": {
        "field": "price"
      }
    }
  }
}

# 桶聚合  分组

GET goods/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  },
  "aggs": {
    "goods_brands": {
      "terms": {
        "field": "brandName",
        "size": 100
      }
    }
  }
}
```


####  JavaAPI方式

聚合查询：桶聚合，分组查询

**1. 查询title包含手机的数据**

**2. 查询品牌列表**

```java 
/**
 * 聚合查询：桶聚合，分组查询
 * 1. 查询title包含手机的数据
 * 2. 查询品牌列表
 */
@Test
public void testAggQuery() throws IOException {

    SearchRequest searchRequest=new SearchRequest("goods");

    SearchSourceBuilder sourceBuilder=new SearchSourceBuilder();
    //1. 查询title包含手机的数据

    MatchQueryBuilder queryBuilder = QueryBuilders.matchQuery("title", "手机");

    sourceBuilder.query(queryBuilder);
    //2. 查询品牌列表  只展示前100条
    AggregationBuilder aggregation=AggregationBuilders.terms("goods_brands").field("brandName").size(100);
    sourceBuilder.aggregation(aggregation);


    searchRequest.source(sourceBuilder);

    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    //7. 获取命中对象 SearchHits
    SearchHits hits = searchResponse.getHits();

    //7.1 获取总记录数
    Long total= hits.getTotalHits().value;
    System.out.println("总数："+total);

    // aggregations 对象
    Aggregations aggregations = searchResponse.getAggregations();
    //将aggregations 转化为map
    Map<String, Aggregation> aggregationMap = aggregations.asMap();


    //通过key获取goods_brands 对象 使用Aggregation的子类接收  buckets属性在Terms接口中体现

    //        Aggregation goods_brands1 = aggregationMap.get("goods_brands");
    Terms goods_brands =(Terms) aggregationMap.get("goods_brands");

    //获取buckets 数组集合
    List<? extends Terms.Bucket> buckets = goods_brands.getBuckets();

    Map<String,Object>map=new HashMap<>();
    //遍历buckets   key 属性名，doc_count 统计聚合数
    for (Terms.Bucket bucket : buckets) {

        System.out.println(bucket.getKey());
        map.put(bucket.getKeyAsString(),bucket.getDocCount());
    }

    System.out.println(map);

}
```

### 2.9 高亮查询

高亮查询的本质其实就是在关键词前后加上对应的html标签, 从而展示出高亮效果 , 要想实现高亮, 必须明确三要素

- 高亮字段

- 前缀

- 后缀

默认前后缀 ：em

```html
<em>手机</em>
```

#### REST方式

```json
GET goods/_search
{
  "query": {
    "match": {
      "title": "电视"
    }
  },
  "highlight": {
    "fields": {
      "title": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>"
      }
    }
  }
}
```


#### JavaAPI方式

```java
/**
     *
     * 高亮查询：
     *  1. 设置高亮
     *      * 高亮字段
     *      * 前缀
     *      * 后缀
     *  2. 将高亮了的字段数据，替换原有数据
     */
@Test
public void testHighLightQuery() throws IOException {


    SearchRequest searchRequest = new SearchRequest("goods");

    SearchSourceBuilder sourceBulider = new SearchSourceBuilder();

    // 1. 查询title包含手机的数据
    MatchQueryBuilder query = QueryBuilders.matchQuery("title", "手机");

    sourceBulider.query(query);

    //设置高亮
    HighlightBuilder highlighter = new HighlightBuilder();
    //设置三要素
    highlighter.field("title");
    //设置前后缀标签
    highlighter.preTags("<font color='red'>");
    highlighter.postTags("</font>");

    //加载已经设置好的高亮配置
    sourceBulider.highlighter(highlighter);

    searchRequest.source(sourceBulider);

    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);


    SearchHits searchHits = searchResponse.getHits();
    //获取记录数
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数："+value);

    List<Goods> goodsList = new ArrayList<>();
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();

        //转为java
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);

        // 获取高亮结果，替换goods中的title
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        HighlightField HighlightField = highlightFields.get("title");
        Text[] fragments = HighlightField.fragments();
        //highlight title替换 替换goods中的title
        goods.setTitle(fragments[0].toString());
        goodsList.add(goods);
    }

    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```


## 2.10 重建索引&索引别名

```json
#查询别名 默认别名无法查看，默认别名同索引名
GET goods/_alias/
#结果
{
  "goods" : {
    "aliases" : { }
  }
}

```

**创建student_index_v1索引**

```json
# -------重建索引-----------

# 新建student_index_v1。索引名称必须全部小写

PUT student_index_v1
{
  "mappings": {
    "properties": {
      "birthday":{
        "type": "date"
      }
    }
  }
}
#查看 student_index_v1 结构
GET student_index_v1
#添加数据
PUT student_index_v1/_doc/1
{
  "birthday":"1999-11-11"
}
#查看数据
GET student_index_v1/_search

#添加数据
PUT student_index_v1/_doc/1
{
  "birthday":"1999年11月11日"
}
```

**重建索引 : 将student_index_v1 数据拷贝到 student_index_v2**

```json
# 业务变更了，需要改变birthday字段的类型为text

# 1. 创建新的索引 student_index_v2
# 2. 将student_index_v1 数据拷贝到 student_index_v2

# 创建新的索引 student_index_v2
PUT student_index_v2
{
  "mappings": {
    "properties": {
      "birthday":{
        "type": "text"
      }
    }
  }
}
# 将student_index_v1 数据拷贝到 student_index_v2
# _reindex 拷贝数据
POST _reindex
{
  "source": {
    "index": "student_index_v1"
  },
  "dest": {
    "index": "student_index_v2"
  }
}

GET student_index_v2/_search



PUT student_index_v2/_doc/2
{
  "birthday":"1999年11月11日"
}

```

**创建索引库别名：**

注意：DELETE student_index_v1 这一操作将删除student_index_v1索引库，并不是删除别名

```json
# 思考： 现在java代码中操作es，还是使用的实student_index_v1老的索引名称。
# 1. 改代码（不推荐）
# 2. 索引别名（推荐）

# 步骤：
# 0. 先删除student_index_v1
# 1. 给student_index_v2起个别名 student_index_v1



# 先删除student_index_v1
#DELETE student_index_v1 这一操作将删除student_index_v1索引库
#索引库默认的别名与索引库同名，无法删除

# 给student_index_v1起个别名 student_index_v11
POST student_index_v2/_alias/student_index_v11
#测试删除命令
POST /_aliases
{
    "actions": [
        {"remove": {"index": "student_index_v1", "alias": "student_index_v11"}}
    ]
}

# 给student_index_v2起个别名 student_index_v1
POST student_index_v2/_alias/student_index_v1

#查询别名
GET goods/_alias/


GET student_index_v1/_search
GET student_index_v2/_search

```