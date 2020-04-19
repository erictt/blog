---
layout: post
title: 如何使用Linux的screen命令
category: 编程学习
date: 2014-12-22
tags: [Linux]
description: 如何使用Linux的screen命令
keywords: Linux, Command, Screen
---

摘录自: [http://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/](http://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/)

## 功能

* 多终端共享一个SSH会话
* 能在网络状况不好的状况下保持可用
* 从多个地址断开并重新连接终端会话
* 在执行耗时任务时无需维持终端会话


## 安装

* CentOS: `yum install screen`
* Ubuntu: `apt-get install screen`

## 起始命令

* `screen` 进入screen模式

## 控制命令

* `Ctrl+a` screen 使用Ctrl+a快捷键作为命令前缀来发送信号，例如`Ctrl+a+?`为打开screen帮助页面

## 创建窗口

* `Ctrl+a+c` 使用默认视窗创建新的窗口，旧视窗能然可用

## 窗口之间切换

使用screen 向前或向后切换窗口

* `Ctrl+a+n` 下一个窗口
* `Ctrl+a+p` 上一个窗口
* 两次`Ctrl+a` 可以在最近的上一个或下一个窗口中来回切换

## 挂起会话

与当前的screen分离并跳出到原有的shell会话中，并可以使用`screen -r`重新连上

对于需要在服务器执行长时间消耗的任务是个非常不错的选择

PS: 如果当前网络连接中断，screen会自动挂起会话

* `Ctrl+a+d`

## 恢复会话

如果你的会话连接由于某种原因中端，可以使用此命令恢复

* `screen -r`

## 记录screen的输出

* `Ctrl+a+H` 为当前会话创建日志进行追踪

screen 会跨多个会话进行日志追踪。对于在服务器操作较多的情况，可以有捕捉自己的操作

## 获取警告提醒

screen可以监听会话活动。在下载或者编译等耗时操作时，可用于提醒警告

当你在等待一个耗时进程的输出时，可使用`Ctrl+a+M`来监听活动。screen会等输出完成后，在页面底部闪出一条提醒。此时可切换至其他screen进行其他操作，而不需要时而切换回来查看任务是否完成

或者你在编译程序或下载文件，等待会话无内容输出的情况。可使用`Ctrl+a+_`来监听当前会话是否已无内容输出

## 锁定screen会话

当你要离开电脑一会儿，可使用`Ctrl+a+x`加入密码来锁定screen

## 停止screen

当完成任务后，推荐停止screen会话，而非挂起会话

* 输入`exit`
* `Ctrl+a+k`

