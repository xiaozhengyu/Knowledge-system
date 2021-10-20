# ElasticSearch

参考文档

-   [ElasticSearch：权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/getting-started.html)

---

## 一、简介

Elasticsearch 是一个开源的搜索引擎，建立在一个全文搜索引擎库 [Apache Lucene™](https://lucene.apache.org/core/) 基础之上。 Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库——无论是开源还是私有。

但是 Lucene 仅仅只是一个库。为了充分发挥其功能，你需要使用 Java 并将 Lucene 直接集成到应用程序中。 更糟糕的是，您可能需要获得信息检索学位才能了解其工作原理。Lucene <font color = red>非常复杂</font>。Elasticsearch 也是使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目的是使全文检索变得简单， 通过隐藏 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API。

然而，Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎。 它可以被下面这样准确的形容：

-   一个分布式的实时文档存储，*每个字段* 可以被索引与搜索
-   一个分布式实时分析搜索引擎
-   能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

Elasticsearch 将所有的功能打包成一个单独的服务，这样你可以通过程序与它提供的简单的 RESTful API 进行通信， 可以使用自己喜欢的编程语言充当 web 客户端，甚至可以使用命令行（去充当这个客户端）。

就 Elasticsearch 而言，起步很简单。对于初学者来说，它预设了一些适当的默认值，并隐藏了复杂的搜索理论知识。 它 *开箱即用* 。只需最少的理解，你很快就能具有生产力。



## 二、入门



### 2.1 ElasticSearch安装

[ElasticSearch安装（基于Docker）.md](.\实战\ElasticSearch安装（基于Docker）.md)



### 2.2 数据格式

ElasticSearch是面向文档型数据库，一条数据在这里就是一个文档。问了方便理解，将ElasticSearch里面存储文档数据和关系型数据库MySQL存储数据的概念进行一个类比：

| MySQL              | ElasticSearch     |
| ------------------ | ----------------- |
| Database（数据库） | Index（索引）     |
| Table（表）        | Types（类型）     |
| Row（行）          | Documents（文档） |
| Column（列）       | Fields（字段）    |

Types的概念已经被逐渐弱化，在ElasticSearch 6.x 中，一个Index下已经只能包含一个Type；从ElasticSearch 7.x 开始，Type的概念被彻底删除。

>   [Elasticsearch6.0以后为什么要删除Type的概念？](https://zhuanlan.zhihu.com/p/66716868)
>
>   一开始，我们我们谈到 一个 ES的索引类似于关系型数据库中的数据库，一个映射类型则相当于关系型数据库中的一张表。这是一个错误的类比，导致了错误的假设。在一个关系型数据库中，表之间是相互独立的。一个表中的列与另一个表中同名的列没有关系。然而在映射类型中却不是这样的。
>
>   在一个Elasticsearch的索引中，有相同名称字段的不同映射类型在Lucene内部是由同一个字段支持的。换言之，看下面的这个例子，user 类型中的  user_name字段和tweet类型中的user_name字段实际上是被存储在同一个字段中，而且两个user_name字段在这两种映射类型中都有相同的定义（如类型都是 text或者都是date）。
>
>   这会导致一些问题，比如，当你希望在一个索引中的两个映射类型，一个映射类型中的 deleted 字段映射为一个日期数据类型的字段，而在另一个映射类型中的deleted字段映射为一个布尔数据类型的字段，这就会失败。最重要的是，在一个索引中存储那些有很少或没有相同字段的实体会导致稀疏数据，并且干扰Lucene有效压缩文档的能力。



### 2.3 RESTful API



## 三、环境

## 四、进阶

## 五、集成

## 六、优化

## 七、面试题