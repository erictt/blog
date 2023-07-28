---
layout: post
title: Get Start - Elasticsearch 学习实践 (1)
categories: [编程学习]
tags: [Elasticsearch]
date: 2016-11-17
description: Elasticsearch文档学习实践之路
keywords: Elasticsearch, 文档, 全文检索, 搜索
---

## 基本概念

* 索引(indix)： 相当于关系型数据库的数据库。
* 类型(type)：相当于关系型数据库的表。
* 文档(Document)： 相当于关系型数据库中的每一行数据记录。
* 字段(field)：相当于数据类型，例如字符串、整数、日期等。
* 映射(mapping)： 用于定义文档及其内部属性字段如何被索引及搜索的。
* 节点(Node)：单台服务器称为一个节点，默认情况下最好配置2+台保证可用性。
* 分片索引(Shard)：Elasticsearch会把数据分发到多台服务器上存储。这个过程成为分片(Sharding)。
* 索引副本(Replica)：副本主要是主分片的复制。在搜索请求阻塞在单个节点时，它可以像原来的主分片一样处理用户搜索请求。

## 安装

### 使用 Docker 安装

* 文件目录结构：

        Dockerfile // Docker安装文件
        config/
            elasticsearch.yml // Elasticsearch的基本配置
            license.json // 在官网注册的Basic License
            x-pack/
                roles.yml // 定义角色

* `Dockerfile`

        FROM elasticsearch:5.2.0
        
        # Define working directory.
        WORKDIR /tmp
        
        # Define default command.
        # CMD ["/elasticsearch/bin/elasticsearch"]
        RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack
        
        # 使用IK分词器
        ENV IK_VERSION 5.2.0
        RUN wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v$IK_VERSION/elasticsearch-analysis-ik-$IK_VERSION.zip
        RUN unzip elasticsearch-analysis-ik-$IK_VERSION.zip -d /usr/share/elasticsearch/plugins/ik/
        RUN rm -f elasticsearch-analysis-ik-$IK_VERSION.zip
        
        # Mount elasticsearch.yml config
        # This is a tricky problem, x-pack configuration path is /etc/elasticsearch/ which follow the offical document,
        # but elasticsearch configuration path is /usr/share/elasticsearch/config. I've asked this issue in Github, still waiting for the answer.
        # for now, just put them under both paths.
        ADD config /usr/share/elasticsearch/config
        ADD config /etc/elasticsearch
        
        RUN set -ex \
         && for path in \
          # /data/elasticsearch \
                /etc/elasticsearch \
                /data/logs/elasticsearch \
         ; do \
          mkdir -p "$path"; \
          chown -R elasticsearch:elasticsearch "$path"; \
         done
        
        CMD ["elasticsearch"]
        
        # Expose ports.
        #   - 9200: HTTP
        #   - 9300: transport
        EXPOSE 9200
        EXPOSE 9300

* `elastcisearch.yml`

        network.host: 0.0.0.0
        path:
          data: /usr/share/elasticsearch/data
          logs: /data/logs/elasticsearch
        
        node.name:   assets
        cluster.name: assets-dev-1
        
        xpack.security.audit.enabled: true
        xpack.security.authc:
        # Gold 以上 license 才支持 realms，所以此处只能使用匿名用户
          anonymous:
            username: anonymous_user
            roles: assets_rw
            authz_exception: true
          
          # realms:
          #   file1:
          #     type: file
          #     order: 0
          #     cache.hash_algo: md5
          #   native1:
          #     type: native
          #     order: 1

* `x-pack/roles.yml`

        assets_rw:
          run_as: [ 'elastic' ]
          cluster: [ 'all' ]
          "indices":
            - names: [ '*' ]
              privileges: [ 'all' ] 

* `license.json`
  * 注册地址：`https://www.elastic.co/subscriptions`， 获得Basic License后便可使用 Kibana来监控数据
  * 安装方法：需跑起来Docker Container后，在服务器执行`curl -XPUT -u elastic 'http://<host>:<port>/_xpack/license' -d @/path/to/license.json`
