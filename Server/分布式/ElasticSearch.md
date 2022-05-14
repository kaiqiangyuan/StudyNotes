[toc]

### 1、简介

Elasticsearch是一个基于[Lucene](https://baike.baidu.com/item/Lucene/6753302)的搜索服务器。它提供了一个分布式多用户能力的[全文搜索引擎](https://baike.baidu.com/item/全文搜索引擎/7847410)，基于RESTful web接口。Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。Elasticsearch用于[云计算](https://baike.baidu.com/item/云计算/9969353)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby和许多其他语言中都是可用的。根据DB-Engines的排名显示，Elasticsearch是最受欢迎的企业搜索引擎，其次是Apache Solr，也是基于Lucene。

### 2、安装ElasticSearch

[Ubuntu18.04中安装参考](https://www.myfreax.com/how-to-install-elasticsearch-on-ubuntu-18-04/) [^1]

**ubuntu使用命令行apt安装，mac下载解压使用，下面示例主要使用的是mac系统下面的es版本7.12.0**

```shell
安装jdk
sudo apt update
sudo apt install apt-transport-https
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'

sudo systemctl enable elasticsearch.service （设置开机自启动）
sudo systemctl start elasticsearch.service （启动）
sudo systemctl stop elasticsearch.service （停止）
sudo systemctl restart elasticsearch.service （重启）
sudo systemctl status elasticsearch.service （查看状态）
```

使用 ==curl -X GET "localhost:9200/"== 测试是否启动成功

```shell
{
  "name" : "Mac",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "Gz-LMjbISM-8uWPiHC3fug",
  "version" : {
    "number" : "7.12.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "78722783c38caa25a70982b5b042074cde5d3b3a",
    "build_date" : "2021-03-18T06:17:15.410153305Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

若在服务器中启动失败，可能为==内存太小==，修改配置文件**/etc/elasticsearch/jvm.options **

```shell
## -Xms4g
## -Xmx4g
-Xms258m
-Xmx258m
```

配置远程连接，则设置**elasticsearch.yml** 文件

```shell
#设置为0.0.0.0
network.host: 0.0.0.0
#取消注释
cluster.initial_master_nodes: ["node-1", "node-2"]
```

### 3、ES可视化elasticsearch-head

[elasticsearch-head Gtihub地址](https://github.com/mobz/elasticsearch-head) [^2]

```shell
#进入到项目目录
npm install 
npm run start
```

连接时遇到跨域问题

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531191421.png)

解决：修改elasticsearch.yml配置文件，增加下面跨域配置后重新启动

```shell
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 4、使用Kibana（版本需与ES一致）

[Kibana下载地址](https://www.elastic.co/cn/downloads/kibana) [^3]

解压后需要先启动es再启动kibana，默认端口为5601

==设置中文，修改kibana.yml配置文件==

```shell
i18n.locale: "zh-CN"
```

### 5、使用ik分词器（版本需与ES一致）

[iK分词器下载地址](https://github.com/medcl/elasticsearch-analysis-ik/releases) [^4]

ik提供的两个算法：

* ik_smart：最少切分
* ik_max_word：最细粒度划分

下载解压后放入到es的plugins中，并重命名文件夹名称

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192219.png)

测试查看已有的插件

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192255.png)

重新启动es与kibana，从kibana中测试ik分词器，字典中查找

==ik_smart：最少切分==

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192335.png)

==ik_max_word：最细粒度划分==

![3](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192404.png)

未自定义之前

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192653.png)

在ik配置文件中添加自己自定义的dic文件

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192725.png)

如果需要自定义的分词，多个自定义的dic文件使用 “**;”** 间隔

![asd](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531192611.png)

修改完后重新启动

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531193320.png)

变成自定义的单词

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531193401.png)

### 6、索引文档操作

> 创建索引**PUT**

**不推荐使用类型名，换成 "_doc" 使用**

```shell
PUT /索引名/类型名（后期可不写）/id
{
    请求体
}
PUT /test1/type1/1
{
  "name": "袁凯强",
  "age": 18
}
```

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531200731.png)

创建完成后

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531200751.png)

ES的类型可以有多个

- 字符串类型：[text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)、[keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)
- 数值类型：long、integer、short、byte、double、float、half float、scaled float
- 日期类型：date
- 布尔类型：boolean
- 二进制类型：binary

指定字段的类型

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531200913.png)

>获取索引或文档**GET**

通过GET请求获取具体的信息，后面跟索引名或者是id名都可以

```shell
#简单查询
GET test1
GET /test1/_doc/1
#条件查询
GET /test1/_search?q=name:袁
GET /test1/_search?q=name:袁凯强1
```

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201135.png)

通过**GET _cat/xxx** 命令可以获取系统中的其它信息

![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201233.png)

> 修改**（两种方式）**

1. 直接通过 **PUT** 方式进行修改

   ```shell
   PUT /test1/type1/1
   {
     "name": "袁凯强1",
     "age": 21
   }
   ```

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201353.png)

2. 通过 **POST** 方式进行修改（==推荐使用==）

   ```shell
   POST /test1/type1/1/_update
   {
     "doc": {
       "name": "袁凯强12",
       "age":99
     }
   }
   ```

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201442.png)

> 删除**DELETE**

```shell
DELETE test2
```

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201550.png)

> 复杂操作搜索select（**排序、分页、高亮、模糊查询、精准查询**）

```shell
GET /test1/_search
{
  "query": {
    "match": {
      "name": "袁"
    }
  }
}
```

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201737.png)

1. 查询：==query==

   ```shell
   GET /test1/_search
   {
     "query": {
       "match": {
         "name": "袁"
       }
     }
   }
   ```

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531201942.png)

2. 结果过滤 ：==_source==

   ```shell
   "_source": ["name"]
   ```

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202123.png)

3. 排序：==sort（根据数值类型进行排序）==

   ```shell
   "sort": [
       {
         "age": {
           "order": "desc"
         }
       }
     ]
   ```

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202236.png)

4. 分页：==from、size==

   ```shell
   "from": 0,
   "size": 1
   ```

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202401.png)

5. 布尔值查询：==bool==

   ```shell
       "bool": {
         "must": [
           {
             "match": {
               "name": "袁凯强"
             }
           },
           {
             "match": {
               "age": 21
             }
           }
         ]
       }
   ```

   **==must==  相当于mysql中条件中的and**

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202607.png)

   **==should==  相当于mysql中条件中的or**

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202649.png)

   **==must_not==  相当于mysql中条件中的not**

   ![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202730.png)

6. 过滤：==filter==

   ```shell
         "filter": [
           {
             "range": {
               "age": {
                 "gte": 20
               }
             }
           }
         ]
   ```

   eq相等，ne、neq不相等，gt大于，lt小于，gte、ge大于等于，lte、le 小于等于，not非，mod求模，div by是否能被某数整除，even是否为偶数，odd是否为奇

   ![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531202946.png)

7. 匹配多个条件进行查询==match==

   ```shell
     "query": {
       "match": {
         "name": "袁 张"
       }
     }
   ```

   ![3](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203040.png)

8. 精确查询：==term==，==term==查询是直接通过倒排索引指定的词条进程精确查找的

   ```shell
   PUT test2
   {
     "mappings": {
       "properties": {
         "name": {
           "type": "text"
         },
         "desc": {
           "type": "keyword"
         }
       }
     }
   }
   PUT test2/_doc/1
   {
     "name": "袁凯强hello",
     "desc": "注释"
   }
   PUT test2/_doc/2
   {
     "name": "袁凯强hello",
     "desc": "注释2"
   }
   GET _analyze
   {
     "analyzer": "keyword",
     "text": "袁凯强Hello"
   }
   GET _analyze
   {
     "analyzer": "standard",
     "text": "袁凯强Hello"
   }
   GET /test2/_search
   {
     "query": {
       "term": {
         "name": "袁"
       }
     }
   }
   GET /test2/_search
   {
     "query": {
       "term": {
         "desc": "注释"
       }
     }
   }
   ```

   **使用es默认的分词器keyword与standard**

   ==keyword不会被拆分==

   ![111](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203302.png)

   ==standard会被拆分==

   ![222](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203330.png)

   因为name与desc中的字段类型不一样，keyword不能被分词器解析

   ![444](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203442.png)

   name：text类型（trem+text分词查询=》通过standard解析）

   ![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203533.png)

   desc：keyword类型（trem+keyword精确查询=》通过keyword解析）

   ![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203605.png)

   多个值匹配精确查询

   ![2](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203647.png)

9. 高亮查询：==highlight==

```
"highlight":{
	"fields":{
		"name":{}
	}
}
```

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531203954.png)

自定义高亮的标签，默认为em

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204016.png)

### 7、集成SpringBoot

[API官方文档](https://www.elastic.co/guide/en/elasticsearch/client/index.html) [^5]

[API在线文档](https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client/7.12.0/org/elasticsearch/client/package-summary.html) [^6]

![222](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204349.png)

![333](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204405.png)

**maven依赖**

```shell
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

**使用的对象**

![123](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204518.png)

**导入依赖时，保证es版本与系统版本的一致**

![image](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204545.png)

精确查找时，不想被分词器解析

1. 名称上加上keyword：[keyword参考](https://blog.csdn.net/dg19971024/article/details/107103201/) [^7]

![1](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204629.png)

2. 定义的时候加上定义的类型

   [@field注解参考](https://blog.csdn.net/qq_15267341/article/details/108017505) [^8]

   [@Document注解参考](https://my.oschina.net/u/4381576/blog/3416666) [^9]

```shell
@Field(type=FieldType.Text, analyzer="ik_max_word")     表示该字段是一个文本，并作最大程度拆分，默认建立索引
@Field(type=FieldType.Text,index=false)             表示该字段是一个文本，不建立索引
@Field(type=FieldType.Date)                                表示该字段是一个文本，日期类型，默认不建立索引
@Field(type=FieldType.Long)                               表示该字段是一个长整型，默认建立索引
@Field(type=FieldType.Keyword)                         表示该字段内容是一个文本并作为一个整体不可分，默认建立索引
@Field(type=FieldType.Float)                               表示该字段内容是一个浮点类型并作为一个整体不可分，默认建立索引
date 、float、long都是不能够被拆分的
```

![222](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210531204834.png)

**使用类似于JPA的形式使用elasticsearchanalyzer和search_analyzer的区别**

[Elasticsearch中analyzer和search_analyzer的区别](https://blog.csdn.net/user2025/article/details/108691367) [^10]

```shell
analyzer和search_analyzer的区别
在创建索引，指定analyzer，ES在创建时会先检查是否设置了analyzer字段，如果没定义就用ES预设的
在查询时，指定search_analyzer，ES查询时会先检查是否设置了search_analyzer字段，如果没有设置，还会去检查创建索引时是否指定了analyzer，还是没有还设置才会去使用ES预设的
```

**使用类似于JPA的形式使用elasticsearch**

<u>*使用参考*</u> [^11] [^12]

测试代码：**关注公众号==原棵哈==回复==es==获取**

![扫码_搜索联合传播样式-标准色版](https://file-ykq.oss-cn-shanghai.aliyuncs.com/img/20210720080705.png)

### 参考网址

[^1]: https://www.myfreax.com/how-to-install-elasticsearch-on-ubuntu-18-04/
[^2]: https://github.com/mobz/elasticsearch-head
[^3]: https://www.elastic.co/cn/downloads/kibana
[^4]: https://github.com/medcl/elasticsearch-analysis-ik/releases
[^5]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[^6]: https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client/7.12.0/org/elasticsearch/client/package-summary.html
[^7]: https://blog.csdn.net/dg19971024/article/details/107103201/
[^8]: https://blog.csdn.net/qq_15267341/article/details/108017505
[^9]:https://my.oschina.net/u/4381576/blog/3416666
[^10]: https://blog.csdn.net/user2025/article/details/108691367
[^11]: https://blog.csdn.net/qq_43652509/article/details/83989257
[^12]: https://blog.csdn.net/u013089490/article/details/84323903
