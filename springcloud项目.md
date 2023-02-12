#### 分布式ID-雪花算法

由于我们的数据库在生产环境中要分片部署（MyCat），所以我们不能使用数据库本 
身的自增功能来产生主键值，只能由程序来生成唯一的主键值。我们采用的是开源的 
twitter 的 snowflake （雪花）算法。

<img src="/Users/zhaoxiang/Desktop/java系统笔记/springcloud项目.assets/image-20220606122813200.png" alt="image-20220606122813200" style="zoom:50%;" />

整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞(由数据中心ID和机器ID 
作区分)，并且效率较高，经测试，SnowFlake每秒能够产生26万ID左右。

#### openFeifn调用服务

![image-20220612152409680](/Users/zhaoxiang/Desktop/java系统笔记/springcloud项目.assets/image-20220612152409680.png)

```java
# 调用者 
  1、引入CmsClient所在的依赖	2、执行接口方法
@Service
public class IndexServiceImpl implements IndexService {
  @Autowired
  private CmsClient cmsClient;
  @Override
  public List<Content> listContentByCid(String cid) {
    List<Content> contetByCid = cmsClient.findContetByCid(cid);
    return contetByCid;
  }
}

# 接口对象 
  3、路径映射 指定对应的被调用者的路径
@Component
@FeignClient("portal-cms")
@RequestMapping("/cms/content")
public interface CmsClient {
    @RequestMapping("findContetByCid")
    public List<Content> findContetByCid(@RequestParam("cid") String cid);
}
```

## 通用配置

spring:

  datasource:

​    druid:

​      url: jdbc:mysql://localhost:3306/wfx?useUnicode=true&characterEncoding=utf8

​      username: root

​      password: zx123456

​      max-active: 40

​      driver-class-name: com.mysql.jdbc.Driver

  cloud:

​    nacos:

​      discovery:

​        ip: 127.0.0.1 

​        namespace: sit

​        server-addr: localhost:8848

<img src="/Users/zhaoxiang/Desktop/java系统笔记/springcloud项目.assets/image-20220612190442485.png" alt="image-20220612190442485" style="zoom:50%;" />

#### 网关及路由搭建

```yaml
spring:
    cloud:
        gateway:
            routes:
                - id: portal-index
                  uri: lb://portal-index
                  predicates:
                    - Path=/index/**
    application:
        name: protal-gateway
server:
    port: 8040
```

#### 跨域问题

不能在微服务加@CrossOrigin，在网关启动类添加以下代码。

```java
//设置跨域访问
@Bean
public CorsWebFilter corsFilter() {
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedMethod("*");
    config.addAllowedOrigin("*");
    config.addAllowedHeader("*");

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
    source.registerCorsConfiguration("/**", config);

    return new CorsWebFilter(source);
}
```

### Elasticsearch 非关系型数据库

存储结构： 倒排索引，通过词条（item）查询文档

| RDBS                       | ES（NOSQL）                    |
| -------------------------- | ------------------------------ |
| 数据库database             | index                          |
| 表table                    | 类型type，6.0后废弃，7完全删除 |
| 表结构schema字段和字段类型 | 映射mapping                    |
| 行row                      | 文档document以json格式存储     |
| 列column                   | 域field                        |
| 索引b+tree                 | 倒排索引                       |
| sql                        | 查询dsl                        |
| select * from              | GET http://...                 |
| Update table set           | PUT http://                    |
| DELETE                     | DELETE http://                 |

##### 启动nacos

```shell
cd /Users/zhaoxiang/devTools/nacos/bin
sh startup.sh -m standalone
```

##### 启动nginx

```shell
cd /opt/homebrew/Cellar/nginx/1.21.6_1/bin
localhost:80
```

##### docker启动ES

```shell
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -e "discovery.type=single-node" -d -p 9200:9200 -p 9300:9300  --name elasticsearch e082d8ac7e5e 
```

##### docker启动kibana

```shell
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.0.103:9200 -p 5601:5601 -d kibana:7.16.2
```

##### 安装ik分词器

```shell
[root@192 /]# docker exec -it elasticsearch /bin/bash
oot@494b992d9687:/usr/share/elasticsearch/plugins/ik# wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.16.2/elasticsearch-analysis-ik-7.16.2.zip
[root@192 /]# docker restart elasticsearch
```

###### 切分粒度

```
# ik_smart 最少切分
# ik_max_world 最多切分
```

```json
# 创建索引
PUT person
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
  , "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "age":{
        "type": "integer"
      },
      "birth":{
        "type": "date"
      }
    }
  }
}
# 添加文档
PUT person/_doc/1
{
  "name":"赵翔",
  "age":23,
  "birth":"1999-02-27"
}
# 获取所有记录
GET person/_search
{
  "query": {
    "match_all": {}
  }
}
# 批量添加
PUT _bulk
{"index":{"_index":"person","_id":2}}
{"name":"zx"}
{"index":{"_index":"person","_id":3}}
{"name":"zxzx"}
{"index":{"_index":"person","_id":4}}
{"name":"jack","address":"jiangsu"}

# 修改
PUT person/_doc/1
{
  "name":"大憨猪"
}

POST person/_update/1
{
  "doc": {
      "name":"猪猪"
  }
}

# id和ids
GET person/_search
{
  "query": {
    "ids": {"values": ["1","2"]}
  }
}

# match [重要]  包含  使用的ik分词器是ik_max_world 则会分为上海市 上海 市。所以会匹配所有包含这些filed的数据   倒排索引  上海 --- 6 上海市 --- 6  市 --- 6 --- 7
GET person/_search
{
  "query": {
    "match": {
      "address": "上海市"
    }
  }
}
//结果
"hits" : [
      {
        "_index" : "person",
        "_type" : "_doc",
        "_id" : "6",
        "_score" : 3.0939224,
        "_source" : {
          "name" : "李四",
          "address" : "上海市"
        }
      },
      {
        "_index" : "person",
        "_type" : "_doc",
        "_id" : "7",
        "_score" : 1.0313075,
        "_source" : {
          "name" : "王五",
          "address" : "南京市"
        }
      }
    ]
# match 多个域匹配   and不会拆词   or会
GET person/_search
{
  "query": {
    "match": {
      "address": {
        "query": "江苏省 南京市"
        , "operator": "and"
      }
    }
  }
  
# 多域匹配
GET person/_search
{
  "query": {
    "multi_match": {
      "query": "南京",
      "fields": ["address","email"]
    }
  }
}
# term 词条匹配 不会拆词  因此文本匹配时不要使用term  还是使用match 
# 搜索字符串时 要将字段设置成 not_analyzed 无需分析的。不然es会将字符串进行分词，

# range 范围查询
GET person/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}

# 分页
GET person/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2
}

# 复合查询
must 求交集  
must not 取反
should 并集  有一个匹配即可   使用场景：搜索时，产品名包含，或者标签包含
//年龄大于十岁并且地址包含南京的
GET person/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "age": {
              "gte": 10
            }
          }
        },
        {
          "match": {
            "address": "南京"
          }
        }
      ]
    }
  }
}
# 文档高亮
GET person/_search
{
  "query": {
    "match": {
      "address": "南京"
    }
  },
  "highlight": {
    "fields": {
      "address": {}
    },
    "pre_tags": "<span style='color:red'>",
    "post_tags": "</span>"
  }
}

# boosting
出现频率越高，分数越高score
匹配词越短且越精准，则分数越高

人为干预分数

GET person/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "address": "南京"
        }
      }
      , "negative": {
        "match": {
          "address": "北"
        }
      }
      , "negative_boost": 0.2
    }
  }
}

# 过滤查询
如果搜索产品后再对品牌进行进行复合查询，会影响score，而过滤不会影响分数
filter
GET person/_search
{
  "query": {
    "match_all": {}
  }
  , "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}

# 聚合  
1、度量
求和、最大值、平均值...
GET person/_search
{
  "query": {
    "match_all": {}
  }
  , "aggs": {
      "age_avg": {
        "avg": {
          "field": "age"
        }
    }
  }
}
//案例 聚合分组品牌id
GET gulimall_product/_search
{
  "query": {
    "match_all": {}
  }
  , "aggs": {
      "age_brandId": {
        "terms": {
          "field": "brandId"
        }
    }
  }
}
//结果
"aggregations" : {
    "age_brandId" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 12,
          "doc_count" : 42
        },
        {
          "key" : 9,
          "doc_count" : 32
        }
      ]
    }
  }
//id为12的有42个，id为9的有32个
2、桶 
分组

GET person/_search
{
  "query": {
    "match_all": {}
  }
  , "aggs": {
    "name_group": {
      "terms": {
        "field": "address.keyword"
      }
    }
  }
}
```

##### ES集成Spring Boot

```java
//entity
@Data
@Document(indexName = "es_index",shards = 2,replicas = 1)
public class User {
  @Field(store = true,type = FieldType.Keyword)
  private String msg;
  @Field(store = true,type = FieldType.Integer)
  private Integer age;
  @Field(store = true,type = FieldType.Date)
  private Date birth;

  public User(String name, int age, Date birth) {
    this.msg = name;
    this.age = age;
    this.birth = birth;
  }
} 
//test
@Autowired
private RestHighLevelClient restHighLevelClient;

/**
     * 判断索引是否存在
     *
     * @throws IOException
     */
@Test
public void existIndex() throws IOException {
  GetIndexRequest request = new GetIndexRequest("person");
  boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
  System.out.println("exists = " + exists);

  /**
     * 查询文档
     * GET /index/_doc/1
     * @throws IOException
     */
  @Test
  public void getDocument() throws IOException {
    // get /index/_doc/1
    GetRequest getRequest = new GetRequest("es_index", "1");
    GetResponse getResponse = restHighLevelClient.get(getRequest, RequestOptions.DEFAULT);

    System.out.println("getResponse.getSourceAsString() = " + getResponse.getSourceAsString());
    System.out.println("getResponse = " + getResponse);
  }

  //批量添加，如果没有索引会默认新建索引    
  @Test
  public void bulk() throws IOException {
    BulkRequest bulkRequest = new BulkRequest();

    List<User> userList = new ArrayList<>();
    userList.add(new User("h1", 3, new Date()));
    userList.add(new User("h2", 4, new Date()));
    userList.add(new User("h3", 5, new Date()));
    userList.add(new User("h4", 6, new Date()));
    userList.add(new User("h5", 7, new Date()));
    // 批处理请求
    int size = userList.size();
    for (int i = 0; i < size; i++) {
      // 如果是批量更新、删除改这里就可以
      bulkRequest.add(new IndexRequest("es_index")
                      .id("" + (i + 1))
                      .source(JSON.toJSONString(userList.get(i)), XContentType.JSON));
    }
    restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
  }

```

<img src="/Users/zhaoxiang/Desktop/java系统笔记/springcloud项目.assets/image-20220630174121423.png" alt="image-20220630174121423" style="zoom:67%;" />

```java
/*
      搜索
      searchRequest 搜索请求
      sourceBuilder 条件构造
      TermQueryBuilder 精确查询
      MatchAllQueryBuilder 查询所有
     */
    @Test
    public void search1() throws IOException {
        //指定搜索目标索引
        SearchRequest searchRequest = new SearchRequest("es_index");
        //使用搜索条件构造器，构造搜索条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
      
        //搜索条件，使用QueryBuilders工具类实现
        TermQueryBuilder queryBuilder = QueryBuilders.termQuery("name", "h1");
      
        sourceBuilder.query(queryBuilder);
        searchRequest.source(sourceBuilder);
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        System.out.println("==============================");
        for (SearchHit hits : search.getHits().getHits()) {
            System.out.println("hits.getSourceAsString() = " + hits.getSourceAsString());
        }
    }
结果：
  hits.getSourceAsString() = {"age":3,"birth":1656583417980,"name":"h1"}

 // QueryBuilders.matchAllQuery 匹配所有 其他代码不变
		MatchAllQueryBuilder allQueryBuilder = QueryBuilders.matchAllQuery();
        sourceBuilder.query(allQueryBuilder);
        searchRequest.source(sourceBuilder);
结果：
  hits.getSourceAsString() = {"age":3,"birth":1656583551573,"name":"h1"}
  hits.getSourceAsString() = {"age":4,"birth":1656583551573,"name":"h2"}
  hits.getSourceAsString() = {"age":5,"birth":1656583551573,"name":"h3"}
  hits.getSourceAsString() = {"age":6,"birth":1656583551573,"name":"h4"}
  hits.getSourceAsString() = {"age":7,"birth":1656583551573,"name":"h5"}

//分页处理数据
sourceBuilder.from(0);// 默认 0
sourceBuilder.size(2);// 默认 10

searchRequest.source(sourceBuilder);
结果：
    hits.getSourceAsString() = {"age":3,"birth":1656583898233,"name":"h1"}
    hits.getSourceAsString() = {"age":4,"birth":1656583898233,"name":"h2"}

//高亮
HighlightBuilder highlightBuilder = new HighlightBuilder(); //高亮构造器
        highlightBuilder.field("name"); //设置高亮字段
        highlightBuilder.requireFieldMatch(false); // 关闭多个高亮
        highlightBuilder.preTags("<span style='color:red'>"); //设置前缀
        highlightBuilder.postTags("</span>"); //设置后缀
        sourceBuilder.highlighter(highlightBuilder);
```

##### ES----数组扁平化处理

如果数组里的元素都是对象，那么就需要修改属性为nested，否则会检索到不该检索到的数据。

```json
PUT my_index
{
	"mappings":{
		"properties":{
			"user":{
				"type":"nested"
			}
		}
	}
}
```

##### 网关路由配置

```yaml
spring:
    cloud:
        gateway:
            routes:
                - id: portal-index
                  uri: lb://portal-index
                  predicates:
                    - Path=/index/**
                - id: portal-search
                  uri: lb://portal-search
                  predicates:
                    - Path=/search/**
    application:
        name: portal-gateway
server:
    port: 8040
```

##### 复合查询 

```java
功能拆解

1：关键字搜索

​ 关键字匹配goodsName、brandName、cat3name   【should】、分页、高亮

2：显示过滤面板

​ 思路：对第一步搜索结果进行聚合分桶【brandName，cat3name   】

3：商品过滤【brandName，cat3name   】

4：按价格进行排序
  
@Override
    public PageResultVO<ESGoods> search(Map searchMap) {
        if (searchMap == null) return new PageResultVO<>(false, "参数不合法！");

        /*
        1.复合查询
         */
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        if (!StringUtils.isEmpty(searchMap.get("keyword"))) {
            // 商品名称
            MatchQueryBuilder goodsNameQueryBuilder = QueryBuilders
                    .matchQuery("goodsName", searchMap.get("keyword"));
            //品牌名称
            MatchQueryBuilder brandNameQueryBuilder = QueryBuilders
                    .matchQuery("brandName", searchMap.get("keyword"));
            //三级分类
            MatchQueryBuilder cat3nameQueryBuilder = QueryBuilders
                    .matchQuery("cat3name", searchMap.get("keyword"));

            //should
            boolQueryBuilder.should(goodsNameQueryBuilder);
            boolQueryBuilder.should(brandNameQueryBuilder);
            boolQueryBuilder.should(cat3nameQueryBuilder);
        }

        /*
        2.分页
         */
        int page = 0;
        if (!StringUtils.isEmpty(searchMap.get("page"))) {
            page = (int) searchMap.get("page");
            //前端传1
            page--;
        }
        int size = 2;
        if (!StringUtils.isEmpty(searchMap.get("size"))) {
            size = (int) searchMap.get("size");
        }
        PageRequest pageRequest = PageRequest.of(page, size);

        NativeSearchQuery builder = new NativeSearchQueryBuilder()
                /*
                1.设置查询方式  复合查询
                2.分页
                 */

                .withQuery(boolQueryBuilder)
                .withPageable(pageRequest)
                .build();

        SearchHits<ESGoods> res = template.search(builder, ESGoods.class);
        //3.获取分页信息
        //总记录数
        long totalHits = res.getTotalHits();
        //当前页数
        List<ESGoods> esGoodsList = new ArrayList<>();
        List<SearchHit<ESGoods>> searchHits = res.getSearchHits();
        searchHits.forEach(hit -> {
            ESGoods content = hit.getContent();
            esGoodsList.add(content);
        });

        //构造分页结果
        PageResultVO<ESGoods> pageInfo = new PageResultVO<>();
        pageInfo.setTotal(totalHits);
        pageInfo.setSuccess(true);
        pageInfo.setMsg("查询成功");
        pageInfo.setData(esGoodsList);

        return pageInfo;
    }
```

##### 高亮

```java
HighlightBuilder goodsNameHighLight = getHighlightBuilder("goodsName");
NativeSearchQuery builder = new NativeSearchQueryBuilder()
                /*
                1.设置查询方式  复合查询
                2.分页
                3.高亮
                 */
                .withQuery(boolQueryBuilder)
                .withPageable(pageRequest)
                .withHighlightBuilder(goodsNameHighLight)
                .build();
SearchHits<ESGoods> res = template.search(builder, ESGoods.class);
        //3.获取分页信息
        //总记录数
        long totalHits = res.getTotalHits();
        //当前页数
        List<ESGoods> esGoodsList = new ArrayList<>();
        List<SearchHit<ESGoods>> searchHits = res.getSearchHits();
        searchHits.forEach(hit -> {
            ESGoods content = hit.getContent();
            //取出高亮字段
            Map<String, List<String>> highlightFields = hit.getHighlightFields();
            Set<String> keySet = highlightFields.keySet();
            keySet.forEach(key -> {
                if ("goodsName".equals(key)) {
                    List<String> list = highlightFields.get(key);
                    content.setGoodsName(list.get(0));
                }
            });
            esGoodsList.add(content);
        });


// 设置高亮字段
private HighlightBuilder getHighlightBuilder(String... fields) {
    // 高亮条件
    HighlightBuilder highlightBuilder = new HighlightBuilder(); //生成高亮查询器
    for (String field : fields) {
        highlightBuilder.field(field);//高亮查询字段
    }
    highlightBuilder.requireFieldMatch(false);     //如果要多个字段高亮,这项要为false
    highlightBuilder.preTags("<span style=\"color:red\">");   //高亮设置
    highlightBuilder.postTags("</span>");
    //下面这两项,如果你要高亮如文字内容等有很多字的字段,必须配置,不然会导致高亮不全,文章内容缺失等
    highlightBuilder.fragmentSize(800000); //最大高亮分片数
    highlightBuilder.numOfFragments(0); //从第一个分片获取高亮片段

    return highlightBuilder;
}
```

##### 分组 桶查询

```java
/*
聚合 分组 桶 品牌和第三级标签
 */
TermsAggregationBuilder brandNameAgg = AggregationBuilders.terms("brandName_agg")
        .field("brandName.keyword");
TermsAggregationBuilder cat3nameAgg = AggregationBuilders.terms("cat3name_agg")
        .field("cat3name.keyword");
NativeSearchQuery builder = new NativeSearchQueryBuilder()
                /*
                1.设置查询方式  复合查询
                2.分页
                3.高亮
                4.分组 标签
                 */
                .withQuery(boolQueryBuilder)
                .withPageable(pageRequest)
                .withHighlightBuilder(goodsNameHighLight)
                .addAggregation(brandNameAgg)
                .addAggregation(cat3nameAgg)
                .build();
Aggregations aggregations = res.getAggregations();
        //获取聚合分组信息
        List cat3nameList = new ArrayList();
        //cat3name_agg
        Terms cat3name_agg = aggregations.get("cat3name_agg");
        List<? extends Terms.Bucket> buckets = cat3name_agg.getBuckets();
        buckets.forEach(b->{
            cat3nameList.add(b.getKey());
        });

        //读取品牌的聚合信息
        List brandNameList = new ArrayList();
        Terms brandName_agg = aggregations.get("brandName_agg");
        List<? extends Terms.Bucket> buckets1 = brandName_agg.getBuckets();
        buckets1.forEach(b1->{
            brandNameList.add(b1.getKey());
        });

        //构造分页结果
        SearchPageResult pageInfo = new SearchPageResult();
        pageInfo.setTotal(totalHits);
        pageInfo.setSuccess(true);
        pageInfo.setMsg("查询成功");
        pageInfo.setData(esGoodsList);
        pageInfo.setBrandNameList(brandNameList);
        pageInfo.setCat3NameList(cat3nameList);
        return pageInfo;
```

##### 过滤查询 品牌/三级标签

```java
/*
过滤品牌 第三级分类
 */
if(!StringUtils.isEmpty(searchMap.get("brandNameFilter"))) {
    TermQueryBuilder brandNameFilterBuilder = QueryBuilders
            .termQuery("brandName.keyword",searchMap.get("brandNameFilter"));
    boolQueryBuilder.filter(brandNameFilterBuilder);
}

if(!StringUtils.isEmpty(searchMap.get("cat3NameFilter"))) {
    TermQueryBuilder cat3NameFilterBuilder = QueryBuilders
            .termQuery("cat3name.keyword",searchMap.get("cat3NameFilter"));
    boolQueryBuilder.filter(cat3NameFilterBuilder);
}
```

##### 排序

```java

 NativeSearchQueryBuilder nativeSearchQueryBuilder = new NativeSearchQueryBuilder()
                /*
                1.设置查询方式  复合查询
                2.分页
                3.高亮
                4.分组 三级标签
                5.排序
                 */
                .withQuery(boolQueryBuilder)
                .withPageable(pageRequest)
                .withHighlightBuilder(goodsNameHighLight)
                .addAggregation(brandNameAgg)
                .addAggregation(cat3nameAgg);
//6.排序
SortBuilder priceSortBuilder = null;
if (!StringUtils.isEmpty(searchMap.get("priceSort"))) {
    if ("desc".equals(searchMap.get("priceSort"))) {
        priceSortBuilder = SortBuilders.fieldSort("price")
                .order(SortOrder.DESC);
    } else {
        priceSortBuilder = SortBuilders.fieldSort("price")
                .order(SortOrder.ASC);
    }
    nativeSearchQueryBuilder.withSort(priceSortBuilder);
}

NativeSearchQuery builder = nativeSearchQueryBuilder
        .build();
```

##### 同步索引库

```java
search中开发service 功能为将商品同步到索引库中
然后作为接口ESClient放入api模块中
Goods模块通过openFeign进行通信将商品信息同步到索引库中
  
@Autowired
private ESClient esClient; openFeign调用ESClient下的goods2e接口
  
 @Override
    public Result auditGoods(String spuId) {

        //1:根据spuId获取商品信息
        WxbGoods goods = this.baseMapper.findGoodsById(spuId);
        if (goods == null)
            return new Result(false, "该商品不存在");

        //2：修改商品的审核状态
        goods.setAuditStatus("1");
        int i = this.baseMapper.updateById(goods);

        //3：同步索引库
        esClient.goods2es(goods);
    return new Result(true,"同步成功！");
}

 /*
    商品同步索引库
     */
    @Override
    public Result goods2es(WxbGoods goods) {
        //spu->esGoods
        ESGoods esGoods = new ESGoods();
        esGoods.setSpuId(goods.getId());

        esGoods.setBrandId(goods.getBrand().getId());
        esGoods.setBrandName(goods.getBrand().getName());
        esGoods.setCid1id(goods.getCat1().getId());
        esGoods.setCat1name(goods.getCat1().getName());
        esGoods.setCid2id(goods.getCat2().getId());
        esGoods.setCat2name(goods.getCat2().getName());
        esGoods.setCid3id(goods.getCat3().getId());
        esGoods.setCat3name(goods.getCat3().getName());
        esGoods.setCreateTime(new Date());
        esGoods.setGoodsName(goods.getGoodsName());
        esGoods.setPrice(goods.getPrice().doubleValue());
        esGoods.setSmallPic(goods.getSmallPic());
        template.save(esGoods);
        return new Result(true,"同步索引库成功！");
    }
```

##### rocketMQ  审核不再同步到索引库，而是同步到消息队列中

```java
@Autowired
private RocketMQTemplate rocketMQTemplate;
//需要在nacos配置中心进行goods核心配置文件的配置
@Value("${topic}")
private String ESTOPIC;
@Value("${tag}")
private String ESTAG;

//发送消息【同步索引库】
SendResult sendResult = rocketMQTemplate.syncSend(ESTOPIC + ":" + ESTAG, goods);
//获取消息发送状态
SendStatus sendStatus = sendResult.getSendStatus();
if (sendStatus == SendStatus.SEND_OK) {
  return new Result(true, "审核成功");
} else {
  //todo:哪个消息  什么时间  发送状态【补偿】
  return new Result(true, "同步失败");
}


rocketmq:
    name-server: 127.0.0.1:9876
    producer:
        group: wfx-goods-producer
        send-message-timeout: 3000
        compress-message-body-threshold: 4096
        max-message-size: 4194304
        retry-next-server: false
        retry-times-when-send-async-failed: 2
        retry-times-when-send-failed: 2

topic: goods-2-es
tag: goods-2-es-tag
```

#### 消息监听 search

```java
消费者添加以下配置即可
rocketmq:
    name-server: 127.0.0.1:9876
      
@Component
@RocketMQMessageListener(consumerGroup = "portal-search-consumer",  自定义一个消费组
    topic = "goods-2-es",selectorExpression = "goods-2-es-tag",  topic和tag  和生产者一一对应
    consumeMode = ConsumeMode.CONCURRENTLY,
    messageModel = MessageModel.CLUSTERING)
public class ConsumerListener implements RocketMQListener<WxbGoods> {

    @Autowired
    ISearchService iSearchService;

    public void onMessage(WxbGoods goods) {
        //同步至索引库
        iSearchService.goods2es(goods);
    }
}    

将数据库的记录改为0未审核，然后删除es中的对应的文档，然后重新初始化，消费掉数据，数据库会改为1，es也会同步更新数据。
```

## 面试题

#### 面试题：消息发送怎么解决0丢失的问题？

1：Rocketmq发送消息默认是有重试机制，发送失败默认重试2次

2：同步消息和异步消息都会返回消息状态，如果失败，通过补偿逻辑记录消息【mysql】

#### 面试题：怎么保证消息消费的时候0丢失？

答：Rocketmq默认就有重试机制，如果第一次消费的时候，broker收到的回应是（ConsumeConcurrentlyStatus.RECONSUME_LATER），那么这条消息不会丢失，会进入重试队列，重试的次数与重试的时间可以配置，如果重试的次数超过配置的次数，那么这条消息没有丢失，但是放入死信队列

#### 面试题：RocketMQ是全局有序的吗？如果不是怎么做到全局有序？

答：默认一个topic有4个queue，只能做到每个queue的局部有序，不能做到全局有序，如果要做到全局有序，可以将消息发送到一个指定的queue里面。

#### 面试题：怎么解决重试幂等性问题？

答：如果重试很有可能出现幂等性问题，需要业务逻辑做配合，比如可以先判断数据库的状态，然后根据数据库的状态，做对应的处理



### SSO单点登录

```java
依赖
<dependency>
  <groupId>com.agapsys.libs</groupId>
  <artifactId>security-framework</artifactId>
  <version>1.0.0</version>
  </dependency>
控制器
  @RestController
  @RequestMapping("/user/")
  public class WxbMemeberController {

    @Autowired
    private IWxbMemeberService iWxbMemeberService;
    @RequestMapping("login")
    public Result login(@RequestBody WxbMemeber wxbMemeber){
      return iWxbMemeberService.login(wxbMemeber);
    }
  }
接口
public interface IWxbMemeberService extends IService<WxbMemeber> {
  Result login(WxbMemeber wxbMemeber);
}
服务层
@Service
public class WxbMemeberServiceImpl extends ServiceImpl<WxbMemeberMapper, WxbMemeber> implements IWxbMemeberService {

  @Override
  public Result login(WxbMemeber memeber) {
    //根据username获取user信息
    QueryWrapper<WxbMemeber> queryWrapper = new QueryWrapper<>();
    queryWrapper.eq("account", memeber.getAccount());
    WxbMemeber wxbMemeber = this.baseMapper.selectOne(queryWrapper);
    if (wxbMemeber == null) {
      return new Result(false, "用户名或者密码错误");
    }
    BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
    if (!encoder.matches(memeber.getPassword(), wxbMemeber.getPassword())) {
      return new Result(false, "用户名或者密码错误");
    }
    //获取私钥
    PrivateKey privateKey = null;
    try {
      privateKey = RsaUtils.getPrivateKey(ResourceUtils.getFile("classpath:rsa").getPath());
    } catch (Exception e) {
      e.printStackTrace();
    }
    //颁发令牌
    wxbMemeber.setPassword(null);
    String token = JwtUtils.generateTokenExpireInMinutes(wxbMemeber, privateKey, 30);
    return new Result(true, "success", token);
  }
}
```

```java
全局过滤器

package com.portal.filter;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.portal.JsonUtils;
import com.portal.JwtUtils;
import com.portal.RsaUtils;
import com.portal.entity.user.WxbMemeber;
import com.portal.vo.Result;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.util.ResourceUtils;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.net.URI;
import java.security.PublicKey;

@Component
public class GlobalAuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        ServerHttpResponse response = exchange.getResponse();
        ServerHttpRequest request = exchange.getRequest();

        //放行资源
        URI uri = request.getURI();
        String[] whiteList = {"/user/login", "/search/search", "/search/init",
                "/index/findContetByCid", "/index/hot/goodsList", "/pay/notify",
                "/sec/findDateMenues", "/sec/findSecByDateBetw", "/sec/order"};

        for (String w : whiteList) {

            if (uri.getPath().equals(w)) {
                return chain.filter(exchange);//放行
            }
        }

        //判断请求头里面是否有token
        String token = request.getHeaders().getFirst("token");
        if (token == null) {
            //不能放行，直接响应客户端
            return response(response, new Result(false, "请登录", "invalid token"));
        }

        //校验令牌
        //加载公钥
        PublicKey publicKey = null;
        try {
            publicKey = RsaUtils.getPublicKey(ResourceUtils.getFile("classpath:rsa.pub").getPath());

        } catch (Exception e) {
            e.printStackTrace();

        }

        //校验令牌
        try {
            WxbMemeber infoFromToken = (WxbMemeber) JwtUtils.getInfoFromToken(token, publicKey, WxbMemeber.class);
            //传递用户信息【请求头】
            ServerHttpRequest newRequest = request.mutate().header("token", JsonUtils.toString(infoFromToken)).build();
            ServerWebExchange newExchange = exchange.mutate().request(newRequest).build();
            //放行
            return chain.filter(newExchange);

        } catch (MalformedJwtException e) {
            e.printStackTrace();
            return response(response, new Result(false, "非法令牌", "invalid token"));
        } catch (ExpiredJwtException e) {
            return response(response, new Result(false, "令牌过期", "invalid token"));
        } catch (Exception e) {
            return response(response, new Result(false, "其他异常", "invalid token"));
        }

    }

    @Override
    public int getOrder() {
        return 0;
    }

    private Mono<Void> response(ServerHttpResponse response, Result res) {
        //不能放行，直接返回，返回json信息
        response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");

        ObjectMapper objectMapper = new ObjectMapper();
        String jsonStr = null;
        try {
            jsonStr = objectMapper.writeValueAsString(res);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        DataBuffer dataBuffer = response.bufferFactory().wrap(jsonStr.getBytes());

        return response.writeWith(Flux.just(dataBuffer));//响应json数据
    }
}
```
