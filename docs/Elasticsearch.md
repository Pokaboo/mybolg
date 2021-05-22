

##  ElasticSearch

### 分布式搜索引擎

- Lucene：java编写的类库
- Solr：基于lucene
- ElasticSearch：基于lucene
    - Elasticsearch 是一个分布式的免费开源搜索和分析引擎，适用于包括文本、数字、地理空间、结构化和非结构化数据等在内的所有类型的数据  

### 工作原理
- 原始数据会从多个来源（包括日志、系统指标和网络应用程序）输入到 Elasticsearch 中。数据采集指在 Elasticsearch 中进行索引之前解析、标准化并充实这些原始数据的过程。这些数据在 Elasticsearch 中索引完成之后，用户便可针对他们的数据运行复杂的查询，并使用聚合来检索自身数据的复杂汇总。在 Kibana 中，用户可以基于自己的数据创建强大的可视化，分享仪表板，并对 Elastic Stack 进行管理。
- 为何使用 Elasticsearch：
    -  很快：由于 Elasticsearch 是在 Lucene 基础上构建而成的，所以在全文本搜索方面表现十分出色。Elasticsearch 同时还是一个近实时的搜索平台，这意味着从文档索引操作到文档变为可搜索状态之间的延时很短，一般只有一秒。因此，Elasticsearch 非常适用于对时间有严苛要求的用例，例如安全分析和基础设施监测
    - 具有分布式的本质特征：Elasticsearch 中存储的文档分布在不同的容器中，这些容器称为分片，可以进行复制以提供数据冗余副本，以防发生硬件故障。Elasticsearch 的分布式特性使得它可以扩展至数百台（甚至数千台）服务器，并处理 PB 量级的数据
    - 功能广泛：除了速度、可扩展性和弹性等优势以外，Elasticsearch 还有大量强大的内置功能（例如数据汇总和索引生命周期管理），可以方便用户更加高效地存储和搜索数据
    - 简化了数据采集、可视化和报告过程：通过与 Beats 和 Logstash 进行集成，用户能够在向 Elasticsearch 中索引数据之前轻松地处理数据。同时，Kibana 不仅可针对 Elasticsearch 数据提供实时可视化，同时还提供 UI 以便用户快速访问应用程序性能监测 (APM)、日志和基础设施指标等数据。 

### ElasticSearch 核心术语

- 索引 index:
- 类型 type(最新版本已经移除 ):
- 文档 document:
- 字段 fields:
- 映射 mapping:
- 近实时 NRT:
- 节点 node:
- shard replica:数据分片与备份
    - 分片（shard）：把索引库拆分为多份，分别放在不同的节点上，比如有3个节点，3个节点的所有数据内容加在一起是一个完整的索引库。分别保存到三个节点上
水平扩展，提高吞吐量。
    - 备份（replica）：每个shard的备份

### ElasticSearch 集群架构原理
[![gqq1IA.png](https://z3.ax1x.com/2021/05/22/gqq1IA.png)](https://imgtu.com/i/gqq1IA)

### 倒排索引
- 正排索引
- 倒排索引：Elasticsearch 使用一种称为 倒排索引 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表

```
为什么叫倒排索引
在没有搜索引擎时，我们是直接输入一个网址，然后获取网站内容，这时我们的行为是：

document -> to -> words

通过文章，获取里面的单词，此谓「正向索引」，forward index.

后来，我们希望能够输入一个单词，找到含有这个单词，或者和这个单词有关系的文章：

word -> to -> documents
```

