---
layout: post
title: 创建GIT服务器
category: 编程学习
tags: [Linux, Git]
date: 2015-01-13
description:
keywords: Linux, Git, Repository
---

## 添加git 用户

* `sudo adduser git`

## 创建authorized_keys,用于认证用户

* `su git`
* `cd`
* `mkdir .ssh && chmod 700 .ssh`
* `touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys`

## 添加认证用户公钥

* 在本地执行 `cat .ssh/id_rsa.pub | ssh git@git-server 'cat >> .ssh/authorized_keys'`

## 创建git repo

* `cd /opt/git`
* `mkdir project.git`
* `cd project.git`
* `git init --bare`
    * 输出信息: Initialized empty Git repository in /opt/git/project.git/

## 远程repo地址

* `git@git-server:/opt/git/project.git`


