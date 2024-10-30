# 一、初始ElasticSearch

## 1、了解ES

### 1）elasticsearch的作用

elasticsearch是一款非常强大的开源搜索引擎，具备非常多强大功能，可以帮助我们从海量数据中快速找到需要的内容。

例如：

- 在GitHub搜索代码
- 在电商网站搜索商品
- 在谷歌搜索答案
- 在打车软件搜索附近的车

### 2）ELK技术栈

elasticsearch结合kibana、Logstash、Beats，也就是elastic stack（ELK）。被广泛应用在日志数据分析、实时监控等领域：

![img](imgs/0ca23420640072a9fe544c0f1f26241a.png)

而elasticsearch是elastic stack的核心，负责存储、搜索、分析数据。

![img](imgs/e0a1a5a90ad7a4a2a6fb902bac7b3305.png)

### 3）Lucene

elasticsearch底层是基于**lucene**来实现的。

**Lucene**是一个Java语言的搜索引擎类库，是Apache公司的顶级项目，由DougCutting于1999年研发。官网地址：https://lucene.apache.org/ 。

1）Lucene的优势：

- 易拓展
- 高性能

2）Lucene的缺点：

- 只限于Java语言开发
- 学习曲线陡峭
- 不支持水平拓展

> 相比于Lucene，elasticSearch具备下列优势：
>
> - 支持分布式，可水平拓展
> - 提供Resultful接口，可被任意语言调用

### 4）发展历史

- 2004年Shay Banon基于Lucene开发了Compass

- 2010年Shay Banon 重写了Compass，取名为Elasticsearch。

## 2、倒排索引

倒排索引的概念是基于MySQL这样的正向索引而言的。

### 1）正向索引

那么什么是正向索引呢？例如给下表（tb_goods）中的id创建索引：

![img](imgs/41ba30930029df446ab0a6d8fc73a17d.png)

如果是根据id查询，那么直接走索引，查询速度非常快。

但如果是基于title做模糊查询，只能是逐行扫描数据，流程如下：

1）用户搜索数据，条件是title符合"%手机%"

2）逐行获取数据，比如id为1的数据

3）判断数据中的title是否符合用户搜索条件

4）如果符合则放入结果集，不符合则丢弃。回到步骤1

逐行扫描，也就是全表扫描，随着数据量增加，其查询效率也会越来越低。当数据量达到数百万时，就是一场灾难。

### 2）倒排索引

倒排索引中有两个非常重要的概念：

- 文档（Document）：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息

- 词条（Term）：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条。例如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条

**创建倒排索引**是对正向索引的一种特殊处理，流程如下：

- 将每一个文档的数据利用算法分词，得到一个个词条

- 创建表，每行数据包括词条、词条所在文档id、位置等信息

- 因为词条唯一性，可以给词条创建索引，例如hash表结构索引

如图：

![img](imgs/1f422166ae0a638b6349f82c25348bdd.png)

倒排索引的**搜索流程**如下（以搜索"华为手机"为例）：

1）用户输入条件"华为手机"进行搜索。

2）对用户输入内容**分词**，得到词条：华为、手机。

3）拿着词条在倒排索引中查找，可以得到包含词条的文档id：1、2、3。

4）拿着文档id到正向索引中查找具体文档。

如图：

![img](imgs/5c1ba083d656759d4b1d2c22d05a4e2a.png)

虽然要先查询倒排索引，再查询正向索引，但是无论是词条、还是文档id都建立了索引，查询速度非常快！无需全表扫描。

### 3）正向和倒排

那么为什么一个叫做正向索引，一个叫做倒排索引呢？

- **正向索引**是最传统的，根据id索引的方式。当根据词条查询时，必须先逐条获取每个文档，然后判断文档中是否包含所需要的词条，是**根据文档找词条的过程**。

- 而**倒排索引**则相反，是先找到用户要搜索的词条，根据词条得到保护词条的文档的id，然后根据id获取文档。是**根据词条找文档的过程**。

是不是恰好反过来了？

那么两者方式的优缺点是什么呢？

> **正向索引：**
>
> 优点：
>
> - 可以给多个字段创建索引
>
> - 根据索引字段搜索、排序速度非常快
>
>
> 缺点：
>
> - 根据非索引字段，或者索引字段中的部分词条查找时，只能全表扫描。
>
> **倒排索引**：
>
> 优点：
>
> - 根据词条搜索、模糊搜索时，速度非常快
>
> 缺点：
>
> - 只能给词条创建索引，而不是字段
> - 无法根据字段做排序

## 3、常见概念

elasticsearch中有很多独有的概念，与mysql中略有差别，但也有相似之处。

### 1）文档和字段

elasticsearch是面向**文档（Document）**存储的，可以是数据库中的一条商品数据，一个订单信息。文档数据会被序列化为json格式后存储在elasticsearch中：

![img](imgs/2c4793adb137dd0ccb842b892f46a53d.png)

而Json文档中往往包含很多的**字段（Field）**，类似于数据库中的列。

### 2）索引和映射

**索引（Index）**，就是相同类型的文档的集合。

例如：

- 所有用户文档，就可以组织在一起，称为用户的索引；

- 所有商品的文档，可以组织在一起，称为商品的索引；

- 所有订单的文档，可以组织在一起，称为订单的索引；

![img](imgs/dc9a61e1f0f1b763f784176ae1390419.png)

因此，我们可以把索引当做是数据库中的表。

数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有**映射（mapping）**，是索引中文档的字段约束信息，类似表的结构约束。

### 3）mysql与elasticsearch

我们统一的把mysql与elasticsearch的概念做一下对比：

| **MySQL** | **Elasticsearch** | **说明**                                                     |
| --------- | ----------------- | ------------------------------------------------------------ |
| Table     | Index             | 索引(index)，就是文档的集合，类似数据库的表(table)           |
| Row       | Document          | 文档（Document），就是一条条的数据，类似数据库中的行（Row），文档都是JSON格式 |
| Column    | Field             | 字段（Field），就是JSON文档中的字段，类似数据库中的列（Column） |
| Schema    | Mapping           | Mapping（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema） |
| SQL       | DSL               | DSL是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD |

是不是说，我们学习了elasticsearch就不再需要mysql了呢？

并不是如此，两者各自有自己的擅长支出：

- Mysql：擅长事务类型操作，可以**确保数据的安全和一致性**

- Elasticsearch：擅长海量数据的**搜索、分析、计算**


因此在企业中，往往是两者结合使用：

- 对安全性要求较高的写操作，使用mysql实现

- 对查询性能要求较高的搜索需求，使用elasticsearch实现


两者再基于某种方式，实现数据的同步，保证一致性

![img](imgs/154ebee6ec11251711b8ab3fa8d2ec37.png)

## 4、安装

略

### 1）分词器的作用是什么？

- 创建倒排索引时对文档分词

- 用户搜索时，对输入的内容分词

### 2）IK分词器有几种模式？

- ik_smart：智能切分，粗粒度

- ik_max_word：最细切分，细粒度

### 3）IK分词器如何拓展词条？如何停用词条？

- 利用config目录的IkAnalyzer.cfg.xml文件添加拓展词典和停用词典

- 在词典中添加拓展词条或者停用词条

---



# 二、索引库操作

索引库就类似数据库表，mapping映射就类似表的结构。

我们要向es中存储数据，必须先创建“库”和“表”。

## 1、mapping映射属性

mapping是对索引库中文档的约束，常见的mapping属性包括：

- type：字段数据类型，常见的简单类型有：

  - 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）

  - 数值：long、integer、short、byte、double、float、

  - 布尔：boolean

  - 日期：date

  - 对象：object

- index：是否创建索引，默认为true

- analyzer：使用哪种分词器

- properties：该字段的子字段

```json
{
    "age": 21,
    "weight": 52.1,
    "isMarried": false,
    "info": "黑马程序员Java讲师",
    "email": "zy@itcast.cn",
    "score": [99.1, 99.5, 98.9],
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

对应的每个字段映射（mapping）：

- age：类型为 integer；参与搜索，因此需要index为true；无需分词器

- weight：类型为float；参与搜索，因此需要index为true；无需分词器

- isMarried：类型为boolean；参与搜索，因此需要index为true；无需分词器

- info：类型为字符串，需要分词，因此是text；参与搜索，因此需要index为true；分词器可以用ik_smart

- email：类型为字符串，但是不需要分词，因此是keyword；不参与搜索，因此需要index为false；无需分词器

- score：虽然是数组，但是我们只看元素的类型，类型为float；参与搜索，因此需要index为true；无需分词器

- name：类型为object，需要定义多个子属性

- name.firstName；类型为字符串，但是不需要分词，因此是keyword；参与搜索，因此需要index为true；无需分词器

- name.lastName；类型为字符串，但是不需要分词，因此是keyword；参与搜索，因此需要index为true；无需分词器

## 2、索引库的CRUD

这里我们统一使用Kibana编写DSL的方式来演示。

### 1）创建索引库和映射

**基本语法：**

- 请求方式：PUT

- 请求路径：/索引库名，可以自定义

- 请求参数：mapping映射

格式：

```json
PUT /索引库名称
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}
```

**示例：**

```json
PUT /heima
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email":{
        "type": "keyword",
        "index": "falsae"
      },
      "name":{
        "properties": {
          "firstName": {
            "type": "keyword"
          }
        }
      },
      // ... 略
    }
  }
}
```

### 2）查询索引库

**基本语法**：

- 请求方式：GET

- 请求路径：/索引库名

- 请求参数：无

格式：

`GET /索引库名`

示例：

```json
GET /heima
```

### 3）修改索引库

倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改mapping**。

虽然无法修改mapping中已有的字段，但是却允许添加新的字段到mapping中，因为不会对倒排索引产生影响。

**语法说明**

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
    }
  }
}
```

**示例**

![img](imgs/ff8b3d6b973b7750ef6d9535c4ab3227.png)

### 4）删除索引库

**语法：**

- 请求方式：DELETE

- 请求路径：/索引库名

- 请求参数：无

**格式：**

`DELETE /索引库名`

---



# 三、文档操作

## 1、新增文档

**语法**

```json
POST /索引库名/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
    // ...
}
```

**示例**

```json
POST /heima/_doc/1
{
    "info": "黑马程序员Java讲师",
    "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

**响应**

![img](imgs/2427510d831b09a20cbcf1905b407ada.png)



## 2、查询文档

**语法**

`GET /{索引库名称}/_doc/{id}`

**示例**

```json
GET /heima/_doc/1
```

**响应**

![img](imgs/89fdbf0a4016c0065e5b2457205fbb88.png)

## 3、删除文档

**语法**

```json
DELETE /{索引库名}/_doc/id值
```

**示例**

```json
# 根据id删除数据
DELETE /heima/_doc/1
```

**响应**

![img](imgs/4ea7dab94defe6971d9dbbcdbb46ac5a.png)

## 4、修改文档

修改有两种方式：

- 全量修改：直接覆盖原来的文档

- 增量修改：修改文档中的部分字段

### 1）全量修改

全量修改是覆盖原来的文档，其本质是：

- 根据指定的id删除文档

- 新增一个相同id的文档

**注意**：如果根据id删除时，id不存在，第二步的新增也会执行，也就从修改变成了新增操作了。

**语法**

```json
PUT /{索引库名}/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    // ... 略
}
```

**示例**

```json
PUT /heima/_doc/1
{
    "info": "黑马程序员高级Java讲师",
    "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```



### 2）增量修改

增量修改是只修改指定id匹配的文档中的部分字段。

**语法：**

```json
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```

**示例**

```json
POST /heima/_update/1
{
  "doc": {
    "email": "ZhaoYun@itcast.cn"
  }
}
```

---



# 四、RestAPI

