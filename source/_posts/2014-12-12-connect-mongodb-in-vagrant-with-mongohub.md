---
layout: post
title: 使用mongohub在vagrant里连接mongodb
category: 编程学习
tags: [MongoDB]
date: 2014-12-12
description: 使用 Mongohub 在 Vagrant 里连接 MongoDB
keywords: MongoDB, Mongohub, Vagrant
---

重要: 注释掉mongodb配置 `# bind_ip = 127.0.0.1`

1. forword 端口27017到宿主机

2. 使用ssh tunnel方式连接
  
    SSH Host = 127.0.0.1

    SSH Port = 2222

    Username = vagrant

    Password = vagrant

    key = ~/.vagrant.d/insecure_private_key

