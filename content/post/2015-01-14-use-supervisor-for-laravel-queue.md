---
layout: post
title: 使用Supervisor来守护Laravel Queue进程
categories: [编程学习]
tags: [Linux, PHP, Laravel]
date: 2015-01-14
description:
keywords: Linux, Laravel, Queue, Supervisor
---

## 安装Supervisor

* `sudo apt-get install supervisor`

## 编辑Queue守护进程

* `sudo vi /etc/supervisor/conf.d/myqueue.conf`

        [program:myqueue]
        command                 = php artisan queue:work  --env=production --queue=queue1,queue2
        user                    = www-data
        directory               = /path/to/app/
        process_name            = %(program_name)s_%(process_num)s
        numprocs                = 6
        autostart               = true
        autorestart             = true
        stdout_logfile          = /path/to/app/app/storage/logs/supervisor-pd-queue.log
        stdout_logfile_maxbytes = 10MB
        stderr_logfile          = /path/to/app/app/storage/logs/supervisor-pd-queue.log
        stderr_logfile_maxbytes = 10MB

## 启动进程

* `$ sudo supervisorctl`
* `> reread` # Tell supervisord to check for new items in /etc/supervisor/conf.d/
* `> add myqueue`       # Add this process to Supervisord
* `> start myqueue`     # May say "already started"

## 查看进程是否启动

* `$ ps aux | grep php`
