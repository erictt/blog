---
layout: post
title: Linux 常用命令(3) - 检查网络流量
category: 编程学习
tags: [Linux, tcpdump]
date: 2021-03-18
description: Linux 常用命令, tcpdump
keywords: tcpdump
---

## 检查网络流量

### tcpdump

Tcpdump 是用于捕获发出和收到的网络流量的工具。在日常工作中经常用来检查调试网络问题。它不单能捕获 TCP 流量，也能捕获 UDP、ARP 和 ICMP 流量。

首先，tcpdump 需要 root 权限，所以别忘了用 sudo 来执行。

#### 常用参数及对应的命令组合

* `-D`: 查看所有的网络接口
* `-i <interface>`: 捕获特定网络接口，输出到屏幕
* `-n`: 不反向解析域名。在捕获的记录里，默认情况tcpdump会反向解析所有的IP地址，然后显示域名地址。该参数可以避免这个行为。
* `-w <file.pcap>`: 输出到文件 file.pcap。通常大家会使用`.pcap`扩展名，表示 packet capture。
    * 在输出到文件的时候，内容是不会输出到屏幕的。如果希望查看，可以打开新的终端，然后使用 `tcpdump -r file.pcap` 命令查看。
    * 当然，你也可以使用 `tcpdump -w file.pcap &` 命令。让输出到文件的命令运行在后台。或者使用 `-l` + `tee` 命令输出到文件的同时查看。
    * 推荐使用 Wireshark 来分析捕获的流量日志。
* `-l`: 缓存输出，用于输出到屏幕的同时保存到文件。常用命令为 `tcpdump -l | tee file.out` 或者 `tcpdump -l > file.out & tail -f file.out`
* `-W 10 -C 200`: 当需要运行很长时间并且保存日志的时候，可以指定这两个参数。`-C` 指定单个文件大小， 单位为MB。`-W` 文件数量。Tcpdump会自动切分和更新日志。
* `-A | -X` 默认情况下，tcpdump **输出到屏幕**时，仅捕获数据包的headers。如果需要检查包信息，可以加上 `-A` 以ASCII格式输出，或者 `-X` 以HEX格式输出。

####  过滤流量

tcpdump使用的是[BPF](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter)语法用来作为过滤参数，很接近自然语言。比如，只希望捕获端口27017的TCP流量：`tcpdump tcp port 27017`。以下是常用的组合：

* `tcpdump [udp|tcp]`: 只捕获 UDP 或 TCP 流量。Tcpdump也支持使用[IP协议数字](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers)的格式。比如 udp 是 17， tcp 是 6，所以这个命令也可以写作 `tcpdump proto [17|6]`。 
* `tcpdump host 192.168.1.185`: 只捕获跟 host: 192.168.1.185 相关的流量。也可以指定IP范围，比如 `tcpdump net 10.10` 是捕获10.10.0.0/16 相关的所有流量。
* `tcpdump port 27017`: 只捕获跟端口27017相关的流量， 也可以指定端口范围：`tcpdump portrange 100-200`
* `tcpdump src host 192.168.1.185`: 只捕获来自 host: 192.168.1.185 的流量
* `tcpdump dst port 80`: 只捕获访问本机 80 端口的流量


复杂的过滤可以使用 `&&`， `｜｜`， `！` 或者 `and` 和 `not` 等关键词，比如：

* `tcpdump -n src 192.168.1.185 and not dst port 22`: 捕获来自 host: 192.169.1.185 并且目的端口不是22的流量
* 还可以加上 `()` 来实现更复杂的过滤： `tcpdump -n 'host 192.168.1.185 and (tcp port 80 or tcp port 443)'`

#### 几个例子

*  `tcpdump -D` 来查看所有的网络接口
    ```
    # tcpdump -D
    1.eth0
    2.nflog (Linux netfilter log (NFLOG) interface)
    3.nfqueue (Linux netfilter queue (NFQUEUE) interface)
    4.any (Pseudo-device that captures on all interfaces)
    5.lo [Loopback]
    ```
            
    * 其中 eth0 是物理网卡。any 比较特殊是包含了所有接口的流量。lo 是本地流量。其他两个我也不知道是什么。

* `tcpdump -i <interface>` 来查看特定的接口，比如 `tcpdump -i any`  可以捕获所有接口的所有流量，输出到屏幕。 可以追加 `-c <number>` 来指定捕获包的数量。
    
    ```
    # tcpdump -i any -c 10 -n
    tcpdump: data link type PKTAP
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on any, link-type PKTAP (Apple DLT_PKTAP), capture size 262144 bytes
    23:23:45.199840 IP 127.0.0.1.57437 > 127.0.0.1.64582: Flags [F.], seq 3146809726, ack 228856637, win 6375, options [nop,nop,TS val 1294890263 ecr 1294763197], length 0
    23:23:45.199884 IP 127.0.0.1.57437 > 127.0.0.1.64582: Flags [F.], seq 0, ack 1, win 6375, options [nop,nop,TS val 1294890263 ecr 1294763197], length 0
    23:23:45.199903 IP 127.0.0.1.64582 > 127.0.0.1.57437: Flags [.], ack 1, win 6376, options [nop,nop,TS val 1294890263 ecr 1294890263], length 0
    23:23:45.199908 IP 127.0.0.1.64582 > 127.0.0.1.57437: Flags [.], ack 1, win 6376, options [nop,nop,TS val 1294890263 ecr 1294890263], length 0
    23:23:45.432277 IP 192.168.10.123.63284 > 70.103.220.153.4501: UDP, length 132
    23:23:45.511805 IP 70.103.220.153.4501 > 192.168.10.123.63284: UDP, length 132
    23:23:45.716903 ARP, Announcement 192.168.10.101, length 28
    23:23:46.639929 IP 192.168.10.124.49154 > 255.255.255.255.6667: UDP, length 188
    23:23:46.837854 IP 192.168.10.123.64333 > 172.217.6.69.443: Flags [.], seq 1059163531:1059164949, ack 51371873, win 2048, options [nop,nop,TS val 1294891899 ecr 1148464805], length 1418
    23:23:46.837860 IP 192.168.10.123.64333 > 172.217.6.69.443: Flags [.], seq 1418:2836, ack 1, win 2048, options [nop,nop,TS val 1294891899 ecr 1148464805], length 1418
    10 packets captured
    13 packets received by filter
    0 packets dropped by kernel
    ```

#### 输出日志解析

典型的结构：

```
[Timestamp] [Protocol] [Src IP].[Src Port] > [Dst IP].[Dst Port]: [Flags], [Seq], [Ack], [Win Size], [Options], [Data Length]
```

例子：

```
23:25:46.253052 IP 192.168.10.123.64581 > 192.168.10.107.8009: Flags [P.], seq 139835987:139836097, ack 801463949, win 2048, options [nop,nop,TS val 1295011047 ecr 5804669], length 110
```

其中：

* `23:25:46.253052` 时间戳
* `IP` 包协议。IP意思是 IPv4
* `192.168.10.123.64581` 来源IP和端口，最后一个 `.` 后边是端口
* `>` 数据流从源IP到目的IP
* `192.168.10.107.8009` 目的IP和端口，最后一个 `.` 后边是端口
* `Flags [P.]` TCP 标识。 `[P.]` 是 发送告知以接收的数据包，用于告知上一个数据包和发送数据。其他的标识包括：
    * [S] - SYN (数据包发送者发起TCP第一次握手)
    * [S.] - SYN-ACK (接送者发起第二次握手，并告知上一个数据包接受不了成功)
    * [.] - ACK (发送者发起第三次握手，告知接收成功)
    * [P] - PSH (发送数据)
    * [F] - FIN (发送者告知数据传输完成)
    * [R] - RST (接收者发起，表示接收到了意外数据包)
    * 还有 URG、CWR等
* `seq 139835987:139836097`
    * 当前数据流中包含了完整数据包的第 139835987 到 139836097 byte
* `ack 801463949`
    * 最后的期望的 bytes 数量
* `win 2048`
    * 接收者可以缓存的 bytes
* `options [nop,nop,TS val 1295011047 ecr 5804669]`
    * `nop`: no operation
    * `TS val` TCP 时间戳
    * `ecr`: echo reply.
* `length 110`
    * 本次数据流传输的大小 当前是 1024 bytes

## 参考

* https://linuxize.com/post/tcpdump-command-in-linux/
* https://www.tcpdump.org/manpages/tcpdump.1.html
* https://en.wikipedia.org/wiki/Berkeley_Packet_Filter
* https://www.keycdn.com/support/tcp-flags