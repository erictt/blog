---
layout: post
title: 服务器配置及性能调优 - PHP进阶 (4)
category: 编程学习
tags: [PHP]
date: 2017-02-25
description: 服务器配置及性能调优
keywords: 性能调优, PHP, Note, 服务器
---

### PHP-FPM 配置

#### 全局配置
* `emergency_restart_threshold = 10`
    * 如果子进程在 `emergency_restart_interval` 设定的时间内收到该参数设定次数的 `SIGSEGV` 或者 `SIGBUS`退出信息号，则`FPM`会重新启动。0 表示“关闭该功能”。默认值：0（关闭）。
* `emergency_restart_interval = 1m`
    * 用于设定平滑重启的间隔时间。这么做有助于解决加速器中共享内存的使用问题。   

#### 进程池配置
* `pm.max_requests = 1000`
    * 设置每个子进程重生之前服务的请求数。对于可能存在内存泄漏的第三方模块来说是非常有用的。如果设置为 '0' 则一直接受请求，等同于 `PHP_FCGI_MAX_REQUESTS` 环境变量。默认值：0。
* `request_slowlog_timeout = 5s`
    * 当一个请求该设置的超时时间后，就会将对应的 PHP 调用堆栈信息完整写入到慢日志中。设置为 '0' 表示 'Off'。

#### 确定 PHP-FPM 的进程数
* 首先确定服务器可以分配给PHP的内存大小
* 其次确定单个PHP进程的内存消耗
* 最后确定进程数
* 上线前别忘了使用 Apache Bench 或者 Seige 来做压力测试

### Zend OPcache
* 通俗来讲，OPcache 是用来预编译PHP的代码到缓存里来加速HTTP的请求。

### Session 处理
* 由于PHP默认会将session以文件的形式存储在磁盘上，必然会加大磁盘的I/O，拖慢系统。使用内存存储的方式，比如 Memcached 或者 Redis 会是更好的选择。而且这种方式会便于以后扩展分布式系统时保证 Session 共享。

