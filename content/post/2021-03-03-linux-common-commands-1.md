---
layout: post
title: Linux 常用命令(1) - 检查本地端口, 查看进程, 文件
categories: [Coding]
tags: [Linux, ps, netstat, ss, lsof, less, cat, kill, tee]
date: 2021-03-03
description: Linux 常用命令, ps, netstat, ss, lsof, less, cat, kill, tee
keywords: ps, netstat, ss, lsof, less, cat, kill, tee
---

## 检查本地端口

### netstat

* 大多数linux系统已经内置了这个命令。如果没有，可以通过以下命令安装：

  * Ubuntu & Debian: `sudo apt install net-tools`
  * CentOS: `sudo yum install net-tools`

* 列出所有在使用的TCP和UDP端口，包括服务名称和 Socket：

    ```
    sudo netstat -tunlp
    ```

  * `-t` - 显示 TCP 端口
  * `-u` - 显示 UDP 端口
  * `-n` - 直接使用 IP 地址，不通过域名服务器
  * `-l` - 仅显示正在监听的端口
  * `-p` - 显示正在使用 Socket 的 PID 和程序名称

* 输出结果示例：

    ```
    $ sudo netstat -tunlp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1755/sshd
    tcp6       0      0 :::22                   :::*                    LISTEN      1755/sshd
    udp        0      0 0.0.0.0:68              0.0.0.0:*                           1075/dhclient
    udp        0      0 0.0.0.0:68              0.0.0.0:*                           1074/dhclient
    ```

  * Proto - 协议名称.
  * Local Address - 程序监听的 IP 和端口
  * PID/Program name - PID 和程序名称

* 通常情况下我们只需要查看特定的端口监听情况，可以通过 grep 管道命令来实现：

    ```
    sudo netstat -tnlp | grep :22
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1755/sshd
    tcp6       0      0 :::22                   :::*                    LISTEN      1755/sshd
    ```

  * 如果输出结果为空，说明现在没有程序在监听这个端口。
  * 你也可以通过 grep PID、协议、监听状态等来过滤结果。比如：
    * `sudo netstat -tunlp | grep 0.0.0.0`
    * `sudo netstat -tunlp | grep tcp`
    * `sudo netstat -tunlp | grep sshd`

### ss

* 使用上类似 netstat。优点是速度更快，得到更多TCP状态。
* 示例：

    ```
    $ sudo ss -tunlp
    Netid State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
    udp   UNCONN     0      0               *:68                          *:*                   users:(("dhclient",pid=1075,fd=6))
    udp   UNCONN     0      0               *:68                          *:*                   users:(("dhclient",pid=1074,fd=6))
    tcp   LISTEN     0      128             *:22                          *:*                   users:(("sshd",pid=1755,fd=3))
    tcp   LISTEN     0      128            :::22                         :::*                   users:(("sshd",pid=1755,fd=4))
    ```

### lsof

* `lsof`(list opened files) 是用来查看当前系统打开文件的工具。但是由于在类 unix 的系统中，一切皆文件，所以 lsof 也可以用来查看进程、端口等。

* 输出示例：

    ```
    COMMAND     PID   TID             USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
    systemd       1                   root  cwd       DIR                8,1     4096          2 /
    systemd       1                   root  rtd       DIR                8,1     4096          2 /
    systemd       1                   root  txt       REG                8,1  1589552      10104 /lib/systemd/systemd
    systemd       1                   root  DEL       REG                8,1                2119 /lib/x86_64-linux-gnu/libuuid.so.1.3.0
    systemd       1                   root  mem       REG                8,1   162632       2068 /lib/x86_64-linux-gnu/ld-2.23.so
    systemd       1                   root    0u      CHR                1,3      0t0          6 /dev/null
    systemd       1                   root    2u      CHR                1,3      0t0          6 /dev/null
    dnsmasq     972                dnsmasq    5u      IPv4             19118      0t0        TCP 127.0.0.1:53 (LISTEN)
    sshd       1515                   root    3u      IPv4             22999      0t0        TCP *:ssh (LISTEN)
    ...
    ```

  * Command: 进程相关的命令
  * PID: 进程 ID
  * TID: 线程 ID。为空的话说明当前是进程。
  * User: 用户名或用户ID
  * Type: 有超过70种。常见的包括
    * REG: 常规系统文件文件
    * DIR: 目录
    * FIFO: 先进先出
    * CHR: Character special file
    * BLK: Block special file
    * INET: 网络 socket
    * unix: UNIX domain socket
  * Device: 磁盘的名称
  * Size/Off: 文件大小或者文件所在的磁盘中的位置
  * Node: 文件在磁盘中的索引节点
  * Name: 文件挂载点
  * FD: 可能是以下几种:
    * 文件描述。常见的包括 cwd、rtd、txt、mem 和一些数字等等。其中，
      * cwd 表示当前的工作目录
      * rtd 表示根目录
      * txt 表示程序的可执行文件
      * mem 表示内存映射文件
      * mmap 表示内存映射设备
    * 数字描述。以数字表示的，比如标准输入输出文件。数字后面的字母表示进程对该文件的读写模式。
      * u 表示该文件被打开并处于读取/写入模式
      * r 表示只读模式
      * w 表示只写模式
      * ‘ ‘ 空字符表示未知
      * 当字母`u`, `r`, `w` 大写的情况下表示该进程拥有对文件的对应操作的锁。

#### 常见用法

* 查看所有正在监听的TCP端口: `lsof -nP -iTCP -sTCP:LISTEN`
  * `-n` - 不解析域名，直接使用IP地址
  * `-P` - 不解析端口，直接使用数字。比如默认情况下 端口22会用`ssh`来表示。带上这个参数后，会直接显示22。
  * `-iTCP -sTCP:LISTEN` - 仅显示网络文件，并且是TCP协议,以及状态为LISTEN
* 查看特定的端口：`lsof -i :22`
  * 只看 TCP：`lsof -i TCP:22`
* 查看已建立的连接: `lsof  -i -s TCP:ESTABLISHED`
* 列出指定进程打开的所有文件： `lsof -p 588`
* 查看指定文件正在被哪些进程访问： `lsof -t /var/log/auth.log`

## 管理进程

### 查看进程 - ps

* `ps aux` - BSD风格，没有 `-`
  * `a` - 显示所有用户的所有的进程
  * `u` - 提供更多细节，包括进程所属用户，以及内存使用
  * `x` - 显示所有程序，包括没有 attacth 终端的

* `ps -elf` - UNIX风格，一个 `-`。（其实还有GNU风格，是两个`-`）
  * `-l` / `-f` - 类似于`u`，显示更多细节，只是侧重不同。两个参数都包含了PPID(父进程ID)
  * `-e` - 类似于 `x`，显示所有程序

* 示例：

    ```
    # ps aux | head -10
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         8  0.0  0.0      0     0 ?        S    Feb25   0:00 [rcu_bh]
    root         9  0.0  0.0      0     0 ?        R    Feb25   1:02 [rcu_sched]
    root        10  0.0  0.0      0     0 ?        S<   Feb25   0:00 [lru-add-drain]
    root        11  0.0  0.0      0     0 ?        S    Feb25   0:04 [watchdog/0]
    root        13  0.0  0.0      0     0 ?        S    Feb25   0:00 [kdevtmpfs]
    root        14  0.0  0.0      0     0 ?        S<   Feb25   0:00 [netns]
    root        15  0.0  0.0      0     0 ?        S    Feb25   0:00 [khungtaskd]
    root        16  0.0  0.0      0     0 ?        S<   Feb25   0:00 [writeback]
    root        17  0.0  0.0      0     0 ?        S<   Feb25   0:00 [kintegrityd]
    ```

  * `USER` - 用户
  * `PID` - 进程ID
  * `%CPU` - CPU占有率
  * `%MEM` - 内存占有率
  * `VSZ` - 虚拟内存占有大小(KB)
  * `RSS` - 物理内存占有大小
  * `STAT` - 进程状态 Z (僵死), S (中断), R (运行), T (停止), D (不可中断)
  * `START` - 进程开始运行时间

### 重启/杀死进程 - kill / pkill

* `kill [OPTIONS] [PID]...`
* 常用参数：
  * `1`(HUP) - 重新加载进程
  * `9`(KILL) - 直接杀死进程
  * `15`(TERM) - 逐步停止进程
  * `kill -l` 可以查看更多参数
* 用法：
    1. 使用数字 - `kill -1 <PID>` 或者 `kill -s 1 <PID>`
    2. 使用`SIG`前缀 - `kill -SIGHUP <PID>` 或者 `kill -s SIGHUP <PID>`
    3. 不使用`SIG`前缀 - `kill -HUP <PID>` 或者 `kill -s HUP <PID>`
* 复合用法：
  * `kill -9 $(pidof mongod)`
    * `pidof mongod` 会输出 mongod的PID, 作为变量传递给 kill命令
* 更多信号： <https://en.wikipedia.org/wiki/Signal_(IPC)>

## 查看文件

### less

* 对比`more`的优点是，不会一次性加载整个文件。速度会快很多。
* 常用命令：
  * `less +5 /var/log/auth.log` 从第五行开始
  * `less -N /var/log/auth.log` 显示行数
  * `less -5 /var/log/auth.log` 按空格或回车时默认翻整页，`-5` 指定每页为5行
* 在查看文件时
  * 可以使用 `/keyword` 来查找关键词，然后使用`n`或`N`来查找下一个或上一个
  * `gg`回到顶部，G至底部
  * `q` 退出阅读文件

### cat

* 默认输出所有文本到终端。可搭配less使用：
  * `cat /var/log/auth.log | less` = `less /var/log/auth.log`
* 常用：
  * `cat -n /var/log/auth.log` 显示行数
  * `cat -b /var/log/auth.log` 过滤空行后，显示行数。会覆盖 `-n`

## 写入文件

### echo

应该是最简单的输出命令了。类似java/python的print方法。

* `echo "hello world" > file.txt`

### tee

从标准输入流中读取并同时写入多个文件。

常用参数：

* `-a` - 以追加的方式写入，不覆盖文件已有文本。

常用命令：

* `command | tee file1.out file2.out file3.out`

一个特殊的使用场景：

当使用 `echo` 输入文本到 没有权限的文件时：`sudo echo "text" >> /etc/hello.conf` 是会失败的。因为重定向 `>` 不是 sudo 执行的。这个时候可以使用 `tee` 解决： `echo "text" | sudo tee -a /etc/hello.conf`

## 其他

* Mac 里的 `netstat` 跟 Linux 差异很大。比如 `-p` 在 Linux 里是指定是否查看 PID，但在 Mac 里是用来指定协议的 -- `-p tcp`。为了避免搞混，建议在 Mac 里使用 `lsof` 代替。

## 参考

* <https://linuxize.com/post/check-listening-ports-linux/>
* <https://computingforgeeks.com/netstat-vs-ss-usage-guide-linux/>
* <https://www.howtogeek.com/426031/how-to-use-the-linux-lsof-command/>
* <https://www.tecmint.com/linux-more-command-and-less-command-examples/>
* <https://www.jianshu.com/p/be0c534c6a41>
* <https://linuxize.com/post/ps-command-in-linux/>
* <https://www.runoob.com/linux/linux-comm-ps.html>
* <https://www.linuxcool.com/ps>
