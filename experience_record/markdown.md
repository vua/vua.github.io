---
sort: 1
---

# ElasticSearch 分词

ES 5.0以后移除了string类型，将其拆分为 _text_ 和 _keyword_ 类型， _text_ 用于全文搜索， _keyword_ 用于关键词查询

初入职场接到一个需求是对用户提交内容做词云展示，当时脑海中浮现的方案就是ES，词云展示需要词和词频信息，而ES支持分词支持聚合，那么将分词结果直接聚合统计词频，问题就迎刃而解了。
可是现实给予我重创，_text_ 类型支持分词不支持聚合，_keyword_ 类型支持聚合不支持分词，what f**k?

在网上苦苦查阅到这样一个用法，mapping如下

````
{
  "foo": {
    "type": "text",
    "analyzer": "ik_max_word",
    "fields": {
      "keyword": {
        "type": "keyword",
        "ignore_above": 256
      }
    }
  }
}
````
可以利用foo.keyword字段现实聚合，满心欢喜，这不分词聚合齐活了~

人生总是充满变数。一通操作后，问题它又来了，foo.keyword聚合是以foo整个字段值做聚合，不能对分词后结果做聚合...

几经波折后，mapping改造如下
````
{
  "foo": {
    "type": "text",
    "analyzer": "ik_max_word",
    "fielddata": true
  }
}
````
Doc values 不支持 _analyzed_ 字符串字段，所以启用 _fielddata_ 将正排索引加载到JVM内存堆中，实现对 _analyzed_ 字符串字段的聚合

[docvalues-and-fielddata](https://www.elastic.co/guide/cn/elasticsearch/guide/current/docvalues-and-fielddata.html
)