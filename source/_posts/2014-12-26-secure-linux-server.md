---
layout: post
title: Linux服务器安全设置
category: 编程学习
tags: [Linux]
date: 2014-12-26
description: Linux服务器安全设置，常用配置包括iptable，使用SSH Key认证登录，禁用root密码登录，防火墙设置等
keywords: Linux,Note
---

翻译自[Linode: Securing Your Server](https://www.linode.com/docs/security/securing-your-server)

以下配置环境为:

* 本地: Mac OS X
* 服务器: Ubuntu 14.04

## 添加用户，并将其加入sudo组

例如增加webapps用户，并将其加入webapps用户组：

* `ssh root@<your-service-ip>` // 使用root登录服务器
* `groupadd webapps` // 创建webapps用户组
* `useradd -m -g webapps -s /bin/bash webapps`
    * `-m` 创建home根目录
    * `-g` 指定用户组
    * `-s` 指定使用的bash
* `sudo usermod -aG sudo webapps`
    * `-G` 附加用户组

## 在本地创建并使用SSH key认证登录服务器

### 生成密钥

* `ssh-keygen -t rsa -C "your_email@example.com"`
* 避免在每次登录的时候每次都需要输入passphrase
    * `eval "$(ssh-agent -s)"`
    * `ssh-add ~/.ssh/id_rsa`

### 上传至服务器

* 使用webapps登录服务器，并执行`mkdir ~/.ssh && touch ~/.ssh/authorized_keys`创建空文件authorized_keys用于存放公钥
* 在本地执行`cat ~/.ssh/id_rsa.pub | ssh webapps@<your-server-ip> 'cat >> .ssh/authorized_keys'`

### 禁止root用户密码登录

* `sudo vi /etc/ssh/sshd_config`
    * 修改 `PermitRootLogin yes` 为 `PermitRootLogin without-password`
* `sudo service ssh restart` 重启ssh服务

## 创建防火墙

### 创建文件 `/etc/iptables.firewall.rules` 用于存放防火墙规则，内容如下：

* 以下规则仅开放HTTP (80), HTTPS (443), SSH (22)三个端口

        *filter

        #  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
        -A INPUT -i lo -j ACCEPT
        -A INPUT -d 127.0.0.0/8 -j REJECT

        #  Accept all established inbound connections
        -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

        #  Allow all outbound traffic - you can modify this to only allow certain traffic
        -A OUTPUT -j ACCEPT

        #  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
        -A INPUT -p tcp --dport 80 -j ACCEPT
        -A INPUT -p tcp --dport 443 -j ACCEPT

        #  Allow SSH connections
        #
        #  The -dport number should be the same port number you set in sshd_config
        #
        -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

        #  Allow ping
        -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

        #  Log iptables denied calls
        -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

        #  Drop all other inbound - default deny unless explicitly allowed policy
        -A INPUT -j DROP
        -A FORWARD -j DROP

        COMMIT

### 激活防火墙配置

* `sudo iptables-restore < /etc/iptables.firewall.rules`

### 查看防火墙配置信息

* `sudo iptables -L`

### 配置启动执行脚本，保证防火墙配置在服务器每次重启后即时生效

* `sudo vi /etc/network/if-pre-up.d/firewall` 编辑文件，内容如下：

        #!/bin/sh
        /sbin/iptables-restore < /etc/iptables.firewall.rules

* `sudo chmod +x /etc/network/if-pre-up.d/firewall` 修改文件权限为可执行

## 安装配置Fail2Ban

Fail2Ban可以有效防止同一IP多次失败登录。它会临时创建防火墙规则来阻止攻击。

### 安装

* `sudo apt-get install fail2ban`

### 设置

* `bantime` 指定阻止保持时长
* `maxretry` 设置IP可以尝试连接的次数

### 重启fail2ban服务

* `sudo service fail2ban restart`


