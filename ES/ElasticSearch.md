[TOC]

# ES概述

> es是一个开源的==高扩展==的==分布式全文检索引擎==，它近乎可以==实时存储、检索数据==；本身扩展性好。
>
> 通过==Restful Api==来隐藏lucene的复杂性。

# 安装

## elasticSearch的安装

1.准备 安装node.js插件

2.下载Es安装包 [百度网盘地址](https://pan.baidu.com/s/1KWA6Uqw6r06D3WRApyUkxA) 提取地址:zksm

3.解压之后找到bin目录启动

![image-20200608112820445](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200608112820445.png)

## elasticsearch-head的安装

4.安装图形化界面(elasticsearch-head-master)

前提:安装node.js环境

​         1.[安装教程](https://www.cnblogs.com/aizai846/p/11441693.html)

5.解压elasticsearch-head-master

6.以管理员的身份启动在指定路径下启动

![image-20200608113232949](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200608113232949.png)

### 排坑

> 当通过访问localhost:9100访问不到9200,涉及到一个跨域问题

![image-20200608113401943](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200608113401943.png)2.解决,在es的elasticsearch.yml文件中添加配置允许跨域

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## kibana的安装

1.解压文件kibana-7.6.1-windows-x86_64(上面百度云盘中有)

2.启动bin目录下的kibana.bat

3.如果要设置国际化在kibana.yml文件中添加 ***i18n.locale: "zh-CN"***

# ES的核心概念

与数据库的对应关系(注意:es一切皆json)

|   Relation DB    |           ElasticSearch            |
| :--------------: | :--------------------------------: |
| 数据库(database) |            索引(indexs)            |
|   表（tables）   | 类型(types)==相当于sql中数据类型== |
|    行（rows）    |          文档(documents)           |
|  字段(columns)   |           （字段）field            |

es的概念:集群，节点，索引，类型，文档，分片，映射


> 分片:简单来讲就是在ES中所有数据的文件块，也是**数据的最小单元块**
>
> 实列场景：
>             假设 IndexA 有2个分片，我们向 IndexA 中插入10条数据 (10个文档)，那么这10条数据会尽可能平均的分为5条存储在第一个    	        分片，剩下的5条会存储在另一个分片中

> 节点:一个运行中的 Elasticsearch 实例称为一个节点

一个人就是一个集群:默认的集群名字就是elasticsearch

![image-20200609071357075](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609071357075.png)



> 倒排索引：通俗来讲正向索引是通过key找value，反向索引则是通过value找key。

​    假设我们有  every day to Study

​                         forever study every day

| term    | doc_1 | doc_2 |
| ------- | ----- | ----- |
| Study   | ✔     | ❌     |
| to      | ✔     | ❌     |
| every   | ✔     | ✔     |
| day     | ✔     | ✔     |
| forever | ❌     | ✔     |
| study   | ❌     | ✔     |

# ik分词器

> 什么是ik分词器:即把一段中文或者别的词划分成一个个关键字,会把数据库中或者索引库中的数据进行分词，然后进行匹配操作。默认的中文分词是将每个字看成一个词，比如说:“我爱你”，会被划分为“我”，“爱”，“你”
>
> 这是不符合要求的,所以我们需要安装中文的分词器来解决这个问题。

ik分词器提供了两种分词的算法:ik_smart和ik_max_word（下面截图演示）

ik_smart为最少切分:我只切分一次

ik_max_word为最细粒度划分

## 安装

1.解压elasticsearch-analysis-ik-7.6.1.zip（上面百度云盘中）

2.将解压的内容放入es中pluging文件夹下

![image-20200609201610952](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609201610952.png)

3.启动es 

![image-20200609201724916](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609201724916.png)

4.启动kibina

## 界面显示

### 1.使用ik_smart

![image-20200609201924276](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609201924276.png)

### 2.使用ik_max_word

![image-20200609202057122](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609202057122.png)

### 遇到的问题

> 当我们需要自定义自己的搜索内容不想让它拆分了,比如“中国小黄人”我想让“小黄人”是一个整体的词
>
> 这时候我们就需要自定义字典了；

配置目录为

![image-20200609202420901](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609202420901.png)

自定义字典

![image-20200609202559658](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609202559658.png)

添加自定义词典,重启es服务

![image-20200609202642950](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609202642950.png)

加载自定义词典显示

![image-20200609202803566](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609202803566.png)

结果展示

![image-20200609202901748](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609202901748.png)

# Rest风格操作

> 一种软件风格,而不是标准,提供了一组设计原则和约束条件。它主要用于客户端和服务器交互的软件。

| **method** | url地址                                         | **描述**               |
| ---------- | ----------------------------------------------- | ---------------------- |
| PUT        | localhost:9200/索引名称/类型名称/文档id         | 创建文档（指定文档id） |
| POST       | localhost:9200/索引名称/类型名称                | 创建文档（随机文档id） |
| POST       | localhost:9200/索引名称/类型名称/文档id/_update | 修改文档               |
| DELETE     | localhost:9200/索引名称/类型名称/文档id         | 删除文档               |
| GET        | localhost:9200/索引名称/类型名称/文档id         | 查询文档通过文档id     |
| POST       | localhost:9200/索引名称/类型名称/_search        | 查询所有数据           |

## 关于索引的基本操作

1.创建一个索引

```
put /索引名称/~类型名称~(可以有可以没有)/文档id 
{请求体}
```

![image-20200609211212478](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609211212478.png)

![image-20200609211244497](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609211244497.png)

### 创建关系型数据库

> 为每个字段指定类型

![image-20200609211912599](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609211912599.png)

![image-20200609212152962](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609212152962.png)

扩展命令

![image-20200609212609819](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609212609819.png)

### 修改数据

> 方式1：

![image-20200609213057009](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609213057009.png)

> 方式二(推荐)：

![image-20200609214740576](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609214740576.png)

### 删除索引

![image-20200609215020781](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609215020781.png)

### 简单查询

![image-20200609221625802](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609221625802.png)

### 复杂查询

![image-20200609222049760](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609222049760.png)

> 复杂操作搜索 select ( 排序，分页，高亮，模糊查询，精准查询！)

#### match

![image-20200609230514745](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609230514745.png)

#### source

> 指定输出哪些字段 相当于sql中的 select name,age from table (假设table中有好多字段)

![image-20200609234152236](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609234152236.png)

#### sort

![image-20200609234706422](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609234706422.png)

#### 分页查询

![image-20200609234843015](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609234843015.png)

#### boolean

> must相当于and ，所有符合条件的 where name=“张三” and age=3

![image-20200609235505643](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200609235505643.png)

> should相当于sql中的or

![image-20200610000020291](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610000020291.png)	

> must_not 相当于sql中的not

![image-20200610000306924](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610000306924.png)

#### filter

> 设置如>= <=等等

![image-20200610000952295](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610000952295.png)

#### 匹配多个条件

> 多个条件使用空格隔开,只要满足其中一个结果既可以被查出来 这个时候就可以通过max_score基本判断占比权重

![image-20200610001242554](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610001242554.png)

#### 精确查询

> term 查询是直接通过倒排索引指定的词条进程精确查找的！

![image-20200610002616448](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610002616448.png)

#### 高亮显示

![image-20200610070939193](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610070939193.png)



#### 关于分词

##### keyword

> 如果类型是keyword的字段不会拆分

![image-20200610001841792](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610001841792.png)

##### **text**

> 如果类型是text的字段会被拆分

![image-20200610001923926](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610001923926.png)

# 整合SpringBoot

```xml
<properties>
        <java.version>1.8</java.version>
        <!--因为我们这边使用到的es版本是7.6.1所以我们指定版本-->
        <elasticsearch.version>7.6.1</elasticsearch.version>
</properties>

<dependencies>
		 <!--es整合SpringBoot的pom-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
            <version>2.3.0.RELEASE</version>
        </dependency>
        <!--使用高版本的pom-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.6.2</version>
        </dependency>
 <dependencies>
```

版本号验证

![image-20200610110330689](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200610110330689.png)

## 创建索引

```java
@SpringBootTest
class BootApplicationTests {


    @Autowired
    @Qualifier("restHighLevelClient")
    private RestHighLevelClient client;


    //创建索引
    @Test
    void testCreateIndex() throws IOException {
        CreateIndexRequest indexRequest = new CreateIndexRequest("xuehy");
        CreateIndexResponse response = client.indices().create(indexRequest, RequestOptions.DEFAULT);
        System.out.println(response);
    }

    //测试索引是否存在
    @Test
    void testIndexExist() throws IOException {
        GetIndexRequest getIndexRequest = new GetIndexRequest("xuehy");
        boolean exists = client.indices().exists(getIndexRequest, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    //删除指定的索引
    @Test
    void deleteIndex() throws IOException{
        DeleteIndexRequest deleteRequest = new DeleteIndexRequest("xuehy");
        AcknowledgedResponse acknowledgedResponse = client.indices().delete(deleteRequest, 				           RequestOptions.DEFAULT);
        System.out.println(acknowledgedResponse.isAcknowledged());
    }
}
```

## 文档的具体操作



```java
 /****************关于文档的操作******************/

    //添加测试文档
    @Test
    public void testAddDocument() throws IOException {
        User user = new User("xhy",26);
        //创建请求
        IndexRequest request = new IndexRequest("xuehy");
        //相当于put xuehy/_doc/1
        request.id("1");
        request.timeout("1s");
        // 将我们的数据放入请求 json
        request.source(JSON.toJSONString(user), XContentType.JSON);
        //客户端发送请求,获取响应结果
        IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);
        System.out.println(indexResponse.toString());
        //获取返回的状态
        System.out.println(indexResponse.status());
    }

    //获取文档是否存在
    @Test
    public void testDocumentExists() throws IOException {
        GetRequest getRequest= new GetRequest("xuehy","1");
        // 不获取返回的 _source 的上下文了
        getRequest.fetchSourceContext((new FetchSourceContext(false)));
        getRequest.storedFields("_none_");
        boolean exists = client.exists(getRequest, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    //获取文档的信息
    @Test
    public void testGetDocument() throws IOException{
        GetRequest getRequest = new GetRequest("xuehy","1");
        //显示指定的字段 按照age排序
        /*getRequest.fetchSourceContext((new FetchSourceContext(true,new String[]{"age"},new String[]{})));
        getRequest.storedFields("age");*/
        GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
        System.out.println(getResponse.getSourceAsString());//打印文档的内容 已json的格式输出
        System.out.println(getResponse); //返回的内容格式和命令是一样的
    }

    //更新文档的信息
    @Test
    public void testUpdateDocument() throws IOException{
        UpdateRequest updateRequest = new UpdateRequest("xuehy","1");
        updateRequest.timeout("1s");

        //POST xuehy/_doc/1/_update
        User user = new User("小薛");
        updateRequest.doc(JSON.toJSONString(user),XContentType.JSON);
        UpdateResponse update = client.update(updateRequest, RequestOptions.DEFAULT);
        System.out.println(update.status());
    }

    //删除文档信息
    @Test
    public void testDeleteDocument() throws IOException {
        DeleteRequest deleteRequest = new DeleteRequest("xuehy","1");
        DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);
        System.out.println(deleteResponse.status());
    }

    //批量操作
    @Test
    public void testBulkRequest() throws IOException{
        BulkRequest bulkRequest = new BulkRequest();
        ArrayList<User> userList = new ArrayList<>();
        userList.add(new User("A",3));
        userList.add(new User("B",3));
        userList.add(new User("C",3));
        userList.add(new User("D",3));
        userList.add(new User("E",3));
        for (int i=0;i<userList.size();i++){
            bulkRequest.add(new IndexRequest("xhy")
                    .id(""+(i+1)) //如果不写id 这边的话就是自动递增
                    .source(JSON.toJSONString(userList.get(i)),XContentType.JSON));
        }
        BulkResponse bulk = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(bulk.hasFailures());
    }

    //特殊查询
    @Test
    public void testSearch() throws IOException{
        SearchRequest searchRequest = new SearchRequest("xhy1");
        //构建搜索条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("title");
        sourceBuilder.highlighter(highlightBuilder);
        //特别注意:使用精确查询的类型需要设置为 keyword格式 否则会进行分词拆分
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "这是一个小标题");
        sourceBuilder.query(termQueryBuilder);
        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        System.out.println(JSON.toJSONString(search.getHits()));
        System.out.println("==============");
        for (SearchHit documentFields : search.getHits().getHits()) {
            System.out.println(documentFields.getSourceAsMap());
        }
    }
```

