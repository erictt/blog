---
layout: post
title: Linux 常用命令笔记
categories: [编程学习]
date: 2014-12-18
tags: [Linux]
description: 收录Linux中常用命令
keywords: Linux
---

## [Manage User](#manage-user)

* /etc/shadow : Secure user account information
* /etc/passwd : User account information
* /etc/gshadow : Contains the shadowed information for group accounts
* /etc/group : Defines the groups to which users belong
* /etc/sudoers : List of who can run what by sudo
* /home/* : Home directories

### Add new Group

* `groupadd [group]`

### Add new User

* `useradd -m -g [initial_group] -G [additional_groups] -s [login_shell] [username]`
* `-m` creates the user home directory as /home/username. Within their home directory, a non-root user can write files, delete them, install programs, and so on.
* `-g` defines the group name or number of the user's initial login group. If specified, the group name must exist; if a group number is provided, it must refer to an already existing group. If not specified, the behaviour of useradd will depend on the USERGROUPS_ENAB variable contained in /etc/login.defs. The default behaviour (USERGROUPS_ENAB yes) is to create a group with the same name as the username, with GID equal to UID.
* `-G` introduces a list of supplementary groups which the user is also a member of. Each group is separated from the next by a comma, with no intervening spaces. The default is for the user to belong only to the initial group.
* `-s` defines the path and file name of the user's default login shell. After the boot process is complete, the default login shell is the one specified here. Ensure the chosen shell package is installed if choosing something other than Bash.

### Set user to sudoers

* Add user to sudo group: `sudo usermod -aG sudo <username>`
* Remove user from sudo group: `gpasswd -d user sudo`

### SSH login without password

* `ssh-copy-id user@server`
* `cat .ssh/id_rsa.pub | ssh user@server 'cat >> .ssh/authorized_keys'`
  * Avoid entering passphrase (just once):
    * `eval "$(ssh-agent -s)"`
    * `ssh-add ~/.ssh/id_rsa`
  * [Auto-launching ssh-agent on msysgit] (<https://help.github.com/articles/working-with-ssh-key-passphrases/#auto-launching-ssh-agent-on-msysgit>)

### Generating SSH keys

* `ssh-keygen -t rsa -C "your_email@example.com"`

### Change hostname

* `vi /etc/hostname`
* don't forget to add `127.0.0.1 <hostname>` to /etc/hosts

### Accept ssh host keys automatically on Linux

* `ssh -oStrictHostKeyChecking=no user@remote_host`
* Disable strict host key checking permanently in ssh

  * Add the following to either ~/.ssh/config or /etc/ssh/ssh_config.

    * Sample 1(disable host key checking for a particular host):

                Host remote_host.com
                StrictHostKeyChecking no

    * Sample 2(turn off host key checking for all hosts you connect to):

                Host *
                StrictHostKeyChecking no

    * Sample 3(avoid host key verification, and not use known_hosts file for 192.168.1.* subnet):

                Host 192.168.0.*
                StrictHostKeyChecking no
                UserKnownHostsFile=/dev/null

* Add the fingerprint for a server to your known_hosts manually.

    `ssh-keyscan -H <ip-address> >> ~/.ssh/known_hosts`

    `ssh-keyscan -H <hostname> >> ~/.ssh/known_hosts`

    Remove them:

    `ssh-keygen -R <ip-address>`

    `ssh-keygen -R <hostname>`

## Netstat Command

* Find out on which port a program is running

  * `netstat -ap | grep ssh`
  * `netstat -an | grep ':80'`

## Service Management

* Check service automatically start or not.
  * `ls /etc/rc2.d` // Just need to check the runlevel 2 on Ubuntu
* Add service `apache2`
  * `update-rc.d apache2 defaults`
* Remove service `apache2`
  * `update-rc.d -f  apache2 remove`
* Check the status of service `apache2`
  * `service apache2 status`
* Check Server `hhvm` autoload level
  * `sudo initctl show-config hhvm`

## Remove executable bit recursively from files (not directories)

* `chmod -R -x+X *`

  * `-R` - operate recursively
  * `-x` - remove executable flags for all users
  * `+X` - set executable flags for all users if it is a directory
  * Note: On OS X, have to do it separately in two commands: `sudo chmod -R -x * && sudo chmod -R +X *`

* `find . -type f -exec chmod -x {} \;`
* `chmod -x $(find . -type f)`

## Batch replace text inside text file

* `find . -type f -name \*.txt -exec sed -i.bak 's|http://|rtmp://|g' {} +`

  * This will create backups of each file. I suggest you check a few to make sure it did what you want, then you can delete them using `find . -name \*.bak -delete`

## Find and remove files

* `find . -type f -name "FILE-TO-FIND" -exec rm -f {} \;`

  * `-name "FILE-TO-FIND"` : File pattern.
  * `-exec rm -rf {} \;` : Delete all files matched by file pattern.
  * `-type f` : Only match files and do not include directory names.

## Bug Collection

* [Ubuntu 14.04] Nodejs ERROR : /usr/bin/env: node: No such file or directory
  * `ln -s /usr/bin/nodejs /usr/bin/node`

## Checking Hardware

* Mainboard: `dmidecode | grep -i 'serial number'`
* CPU: `cat /proc/cpuinfo`
* Hard Drive:
  * partition `fdisk -l`
  * size `df -h`
  * usage `du -h`
* Memory: `cat /proc/meminfo`

## Kill all the process

* `ps aux | grep procname | awk '{print $2}' | xargs kill -9`

## Git Related

* Delete remote tags.
  * `git tag -l | xargs -n 1 git push --delete origin`
* Delete local tasg.
  * `git tag -l | xargs git tag -d`
