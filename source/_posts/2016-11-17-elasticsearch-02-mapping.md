---
layout: post
title: Elasticsearch Mapping - Elasticsearch 学习实践 (2)
category: 编程学习
tags: [Elasticsearch]
date: 2016-11-17
description: Elasticsearch文档学习实践之路
keywords: Elasticsearch, Documents, 文档, 全文检索, 搜索
---

# Mapping

* Mapping是用于定义文档及其内部属性字段如何被索引及搜索的。

## 系统字段

* 首先了解下系统特别制定的字段及其用法。

### 结构定位字段

* _index: 类似数据库的库名
* _type: 类似数据库的表名
* _id: 类似数据库中每条数据的ID
* _uid: 结构是`{type}#{id}`，可用于直接准确查询某条数据

### 文档数据字段

* _source: 原始数据JSON
* _size: 原始数据JSON的大小

### 可用于索引查询的字段

* _all: 将`_source`数据用空格切分保存后作索引（不建议保存，影响性能）
* _field_names: 用于是否包含某字段查询

### 路由字段

* _parent: 用于在两种数据类型之间创建父子关系
* _routing: 用于路由到特定的分片(shard)

### 其他字段

* _meta: 用于存储特定的信息，比如有效版本号、可调用的类函数等

## 用户定义字段

* 定义用户索引的文档字段属性，系统本身是提供动态mapping，也就是说，我们可以在不定义mapping的情况下直接导入，系统会根据用户的第一倒入生成mapping。
* 但如果文档相对复杂，还是建议完整设置动态mapping来避免异常错误。

* Elasticsearch 为我们提供了常用的字段类型，包括：`text`, `keyword`, `date`, `boolean`, `long`, `integer`, `short`, `byte`, `double`, `float`, `binary`， 复杂点的`array`, `object`, `nested`, 用于定位的`geo_point`, `geo_shape`, 特殊的`ip`, `completion`, `token_count`, `murmur3`, `attachments`, `percolator`。

* 其中需要注意的是

	* `text`和`keyword`都是string， 前者会被当作文本分析并索引，可用于包含关键词搜索，后者会作为一个整体而被作为完全匹配搜索， 主要用于聚合、排序等功能使用。
	* `array` 并非数据类型，仅仅作为数据类型标识。
	* `object` 同`array` 仅作为数据类型标识，无任何意义。
	* `nested` 等同于`object`数据类型，将数据结构的KEY扁平化如： `"user.first" : [ "alice", "john" ],`

## 常用设置

* `index.mapping.total_fields.limit` 单个索引中字段数量的上限，默认`1000`。
* `index.mapping.depth.limit` 索引深度， 在数组和对象类型中的层级数量，默认`20`。
* `index.mapping.nested_fields.limit` 在对象类型中，扁平化的KEY的层级数量，默认`50`。

## 动态mapping

* 系统根据用户的第一次倒入的数据生成mapping。

* `_default_ mapping`: 默认的mapping配置，会被任何新建的mapping继承。

* 以下是JSON字段在保存中默认转换的类型对照： (可配置 `"{data_type}_detection": false` 来关闭对应的数据类型检测)

		null : 不保存
		true／false : boolean
		float : float
		integer : long
		object : object 
		array : array
		date : 须符合格式 "yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z" 的会被自动转换
		string : 如果字段不符合date, long, double, text, 会被当作keyword

### 动态模版

* 用户可自定义字段类型检测模版，在倒入数据的时候，根据条件，系统生成对应的mapping配置，示例：

		PUT my_index
		{
		  "mappings": {
		    "my_type": {
		      "dynamic_templates": [
		        {
		          "integers": {
		            "match_mapping_type": "long",
		            "mapping": {
		              "type": "integer"
		            }
		          }
		        },
		        {
		          "strings": {
		            "match_mapping_type": "string",
		            "mapping": {
		              "type": "text",
		              "fields": {
		                "raw": {
		                  "type":  "keyword",
		                  "ignore_above": 256
		                }
		              }
		            }
		          }
		        }
		      ]
		    }
		  }
		}

		PUT my_index/my_type/1
		{
		  "my_integer": 5, 
		  "my_string": "Some string" 
		}

* 示例中，`match_mapping_type` 为条件匹配判断，类型还包含`match`, `match_pattern`, `unmatch`, `path_match`, `path_unmatch`。
* `mapping` 为匹配字段相应的调整内容。
* `match`, `unmatch`, `path_match`, `path_unmatch` 可用通配符来作为匹配或排除条件
* `match_pattern` 可调整`match` 行为来支持正则匹配，如：

	"match_pattern": "regex",
  	"match": "^profit_\d+$"

* 特殊配置

	* 字段不用于排序的时候可以`"norms": false,`来关闭它。
	* 字段仅作用于排序聚合而不用于过滤查询的时候可以`"index": false`来关闭它。

* 更多示例，可参考官方文档：(dynamic-templates.html) [https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-templates.html]

## 另外

* 同一个字段名称的配置在不同的type里边必须包含同样的mapping。除了个别字段，如：`copy_to`, `dynamic`, `enabled`, `ignore_above`, `include_in_all`, `properties`。

官方mapping示例： 没啥好解释的，很简单。具体属性不明白的，查[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)比较好。

		{
		  "mappings": {
		    "user": { 
		      "_all":       { "enabled": false  }, 
		      "properties": { 
		        "title":    { "type": "text"  }, 
		        "name":     { "type": "text"  }, 
		        "age":      { "type": "integer" }  
		      }
		    },
		    "blogpost": { 
		      "_all":       { "enabled": false  }, 
		      "properties": { 
		        "title":    { "type": "text"  }, 
		        "body":     { "type": "text"  }, 
		        "user_id":  {
		          "type":   "keyword" 
		        },
		        "created":  {
		          "type":   "date", 
		          "format": "strict_date_optional_time||epoch_millis"
		        }
		      }
		    }
		  }
		}


