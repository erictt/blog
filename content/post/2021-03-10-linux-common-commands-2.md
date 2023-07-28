---
layout: post
title: Linux 常用命令(2) - 重定向输出和输入, 查看网络端口, 压缩和解压缩
categories: [Coding]
tags: [Linux, stdin, stdout, stderr, netcat, tar, gzip]
date: 2021-03-10
description: Linux 常用命令, netcat, tar, gzip
keywords: netcat, tar, gzip, stdout, stdin, stderr
---

## 重定向输出和输入

在 Bash 或者其他Linux shell里，标准的I/O流分为：

* `0` - `stdin` 标准输入
* `1` - `stdout` 标准输出
* `2` - `stderr` 标准错误

从标准输出开始。标准输出是程序正常输出的内容。默认情况下，所有信息都会输出到屏幕上。重定向输出就是捕获程序的输出然后重定向到另一个程序或者文件中。一般用 `n>` 来实现。`n` 可以是 `0`, `1`, 或者 `2`。 省略`n` 默认为 `1` 即标准输出。

举个例子， `command > file` 就是将命令 `command` 执行的结果输出到文件 `file` 里。 这个命令也等同于 `command 1> file`。 同理，`command 2> file` 就是将命令执行输出到错误重定向到文件里。 如果希望过滤输出，可以重定向到 `/dev/null` 来实现。比如， `command 2> /dev/null`，命令执行结果中的错误信息就会被过滤掉。

那么如何将标准输出和标准错误重定向到一个文件里呢？ `command &> file`，`&` 表示同时将 `1` 和 `2` 输出到文件里。 还有一种方式可以实现同样的效果：`command > file 2>&1`，追加 `2>&1` 是将命令中的输出错误重定向到标准输出中。

在日常使用中，我们也经常碰到 `>>` 。不同于 `> file` 每次执行都会覆盖整个文件， `>>` 会以追加的方式输出内容到文件。

最后来说说标准输入 `<`。用法：`command < input-file`，是将 `input-file` 中的内容作为输入提供给 `command`。这个命令等同于 `command 0< input-file`。举个例子，`grep "ERROR" < /var/log/messages`，将文件 `/var/log/messages` 作为命令 `grep` 的输入。其效果类似于 `cat /var/log/message | grep "ERROR"`。

## 查看网络端口

### netcat(nc)

netcat 是一个使用TCP或者UDP协议读写数据的工具。可以用来检测IP地址或域名端口是否开放。

* `nc -z -v 10.10.8.8 20-80`
  * 扫描 10.10.8.8 的TCP端口 20-80，查看哪些是开放的。
  * `-v` verbose，输出更多的信息
  * `-z` 仅检测开放端口
* `nc -z -v 10.10.8.8 20-80 2>&1 | grep succeeded`
  * 2>&1 重定向标准输出的错误信息到标准输出中。
  * 然后通过 grep succeeded来过滤失败的信息。

* `nc -z -v -u 10.10.8.8 80`
  * `-u` 仅检测UDP协议

## 压缩和解压缩

### tar

* 压缩
  * `tar -czf archive-name.tar.gz file1 file2 dir1...`
    * `-c` - 表明创建压缩文件
    * `-z` - 压缩方式为 gzip
    * `-f` - 指定压缩包文件名
    * `file1 file2 dir1...` - 被压缩的文件列表或文件目录
* 解压缩
  * `tar -xvf archive.tar.gz -C /home/linuxize/files`
    * `-x` - 表明解压缩
    * `-v` - 打印更多的详细信息
    * `-f archive.tar.gz` - 压缩包文件名
    * `-C /home/linuxize/files` - 解压到指定目录
  * `tar -xf archive.tar.gz file1 file2`
    * 仅解压文件 file1 和file2
  * `tar -xf archive.tar.gz --wildcards '*.js'`
    * 用通配符来需要指定解压的文件
* 列出压缩包里的文件名
  * `tar -tf archive.tar.gz`

### gzip

* `gzip -d file.gz`
  * 解压到当前目录，并删掉压缩包
  * 指定 `-k` 可以避免压缩包被删除

## 参考

* <https://linuxize.com/post/bash-redirect-stderr-stdout/>
* <https://tldp.org/LDP/abs/html/io-redirection.html>
* <https://linuxize.com/post/tcpdump-command-in-linux/>
* <https://linuxize.com/post/how-to-create-tar-gz-file/>
* <https://linuxize.com/post/netcat-nc-command-with-examples/>
