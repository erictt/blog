---
layout: post
title: 提升生产效率的工具集锦(APP)
categories: [生活随想]
tags: [效率工具]
date: 2016-12-11
description:
keywords: Alfred 3, Flycut, Flycut, f.lux, Spectacle, CheatSheets, Homebrew, Aria2, Sublime Text 3, iTerm2, Charles, Dash
---

## 按键说明

* ⌘ – Command Key
* ⇥ – Tab Key
* ⌃ – Control Key
* ⌥ – Option Key
* ⇧ – Shift Key
* ⏎ – Return Key
* ← Left Arrow Key
* → Right Arrow Key

## APPS

### Alfred 3

强大的启动器, 拥有各种快捷入口, 不仅仅是用来代替Spotlight。增强的workflow更让繁琐的操作变得简单起来。

* 下载: [官网](https://www.alfredapp.com/)
* Powerpack 需要付费, £19起
* 可设置启动命令, 本人使用 `⌥+⌘`, 好像是默认的。
* 本人会禁用掉Spotlight及其索引功能来提升性能:
  * 打开 系统设置 - Spotlight
  * 取消掉所有在Spotlight中的搜索结果，以及建议查找的功能
  * 左下角选择键盘快捷键, 取消两个已选中两个搜索

### Flycut

剪切板管理器

* 下载: [App Store](http://itunes.apple.com/us/app/flycut-clipboard-manager/id442160987?mt=12) / [Github](https://github.com/TermiT/Flycut)
* 快捷键：`⇧+⌘+p`, 可以不停地按`p`来切换，或者使用`← →`方向键
* 注：Alfred 3也提供了剪切板的功能，个人觉得不够便捷，在使用Flycut的时候，可以考虑给关了。
* 可设置为开机启动

### f.lux

护眼神器。根据一天的时间来调节屏幕的亮度，在晚上屏幕会变成暖色调。

* 下载： [官网](https://justgetflux.com/)
* 可设置为开机启动

### Spectacle

快捷键调节窗口大小，比如: `⌥+⌘+F`全屏, `⌃+⌘+←` 左三分之一屏

* 下载: [官网](https://www.spectacleapp.com/)
* 可设置为开机启动

### CheatSheets

按住`⌘`显示当前应用的所有快捷键

* 下载: [官网](https://cheatsheetapp.com/CheatSheet/?lang=en)

### Homebrew

快速安装软件的管理器

* 下载: [官网](http://brew.sh/)
* 安装: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

### Aria2

实现百度云, 115, 迅雷离线等满速下载。Aria2是个命令行管理工作，所以默认安装后需要做一些配置才能运行。不想折腾的可直接下载[yangshun1029](https://github.com/yangshun1029)封装好的[Aria2gui](https://github.com/yangshun1029/aria2gui).

以下是各种折腾的方案。

* 下载: [官网](https://aria2.github.io/) ／ 安装: `brew install aria2`

* 使用方法有三种。命令行、Web+命令行、Alfred Workflow。
  * 命令行方面使用可见[官方文档](https://aria2.github.io/)
  * Web和Alfred Workflow管理见下文介绍

* 嵌入式 Chrome 插件: [`115`](https://github.com/acgotaku/115), [`百度云`](https://github.com/acgotaku/BaiduExporter)。 这两个由[`雪月秋水君`](https://github.com/acgotaku/)开发的插件可以直接嵌入百度云和115网盘网页内，实现点击即可转到Aria2下载，非常方便。

* Web 管理界面:
    1. 首先命令行启动 `aria2c --enable-rpc --rpc-listen-all`
    2. <http://binux.github.io/yaaw/demo/>
        * 需要设置JSON-RPC Path: `http://localhost:6800/jsonrpc`
    3. 使用[`雪月秋水君`](https://github.com/acgotaku/)开发的Chrome插件: [YAAW](https://github.com/acgotaku/YAAW-for-Chrome)

* Workflow for Alfred: [https://github.com/Wildog/Ariafred](https://github.com/Wildog/Ariafred)
  * [官方文档](http://wil.dog/ariafred/)
  * 注: Workflow 属于powerpack付费功能
  * 常用命令:
    * 添加任务: 输入 `add` 然后追加任务地址
      * BT种子任务需要执行文件动作，然后选择`Add BT do...load to Aria2
    * 移除任务: 输入 `rm` 然后选择任务后 `回车`
    * 暂停/恢复任务: 输入 `pause`/`resume` 然后选择任务后 `回车`
    * 清除所有任务: 输入`clear` 然后 `回车`
* 配置
  * 个人觉得本地运行没必要搞太复杂就不再多写了。有兴趣的可参照[`雪月秋水君`](https://github.com/acgotaku/)提供的`aria2.conf`:

            #用户名
            #rpc-user=user
            #密码
            #rpc-passwd=passwd
            #上面的认证方式不建议使用,建议使用下面的token方式
            #设置加密的密钥
            #rpc-secret=token
            #允许rpc
            enable-rpc=true
            #允许所有来源, web界面跨域权限需要
            rpc-allow-origin-all=true
            #允许外部访问，false的话只监听本地端口
            rpc-listen-all=true
            #RPC端口, 仅当默认端口被占用时修改
            #rpc-listen-port=6800
            #最大同时下载数(任务数), 路由建议值: 3
            max-concurrent-downloads=5
            #断点续传
            continue=true
            #同服务器连接数
            max-connection-per-server=5
            #最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
            min-split-size=10M
            #单文件最大线程数, 路由建议值: 5
            split=10
            #下载速度限制
            max-overall-download-limit=0
            #单文件速度限制
            max-download-limit=0
            #上传速度限制
            max-overall-upload-limit=0
            #单文件速度限制
            max-upload-limit=0
            #断开速度过慢的连接
            #lowest-speed-limit=0
            #验证用，需要1.16.1之后的release版本
            #referer=*
            #文件保存路径, 默认为当前启动位置
            dir=/Users/xxx/Downloads
            #文件缓存, 使用内置的文件缓存, 如果你不相信Linux内核文件缓存和磁盘内置缓存时使用, 需要1.16及以上版本
            #disk-cache=0
            #另一种Linux文件缓存方式, 使用前确保您使用的内核支持此选项, 需要1.15及以上版本(?)
            #enable-mmap=true
            #文件预分配, 能有效降低文件碎片, 提高磁盘性能. 缺点是预分配时间较长
            #所需时间 none < falloc ? trunc « prealloc, falloc和trunc需要文件系统和内核支持
            file-allocation=prealloc
  * 在启动Aria2的时候别忘了加上配置的绝对路径，例: `aria2c --conf-path="/Users/username/aria2.conf"`

### 其他

* 1Password - 最好用的密码管理软件
* PDF Expert - PDF编辑阅读器
* Fantisical2 - 增强版日历，集成了提醒事项
* MWeb - markdown编辑软件，可实时预览，且支持保存到evernote，导出为pdf、docx等格式。此外还支持直接在Wordpress、Mediun等网站发布。
* Typora - 超简洁等markdown编辑软件，单一窗口，实时预览。
* OmniFocus 2 - 最强大的GTD管理工具
* Pandoc -  命令行文件格式互转工具，格式包括docx,markdown, epub, pdf等，官网：[http://pandoc.org/](http://pandoc.org/)
