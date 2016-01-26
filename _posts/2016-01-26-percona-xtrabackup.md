---
layout: post
title: Percona XtraBackup —— 开源热备份工具
categories: [MySQL, Backup, XtraBackup]
tags: [MySQL, Backup, XtraBackup]
---

[XtraBackup][1]是Percona公司开源的mysql热备份工具，它在备份期间无需锁定我们的数据库，是一个非常强大而实用的工具。

## 安装XtraBackup
### 下载
XtraBackup提供了多种安装方式，在其文档的[安装页][2]中有详细的介绍，我采用的是[X86_64 Linux Generic][3]的压缩包
### 解压
将二进制安装包解压到指定文件夹，我路径是：`/home/gongjz/app/percona-xtrabackup-2.0.8`

{% highlight bash linenos %}
[gongjz@localhost ~]$ cd Downloads/
[gongjz@localhost Downloads]$ tar zxvf percona-xtrabackup-2.0.8-587.tar.gz 
[gongjz@localhost Downloads]$ mv percona-xtrabackup-2.0.8 ~/app/
{% endhighlight bash %}

### 添加环境变量
在`.bash_profile`文件中添加下列内容:
` 28 # MySQL path
 29 MYSQL_HOME=$HOME/app/mysql
 30 PATH=$MYSQL_HOME/bin:$PATH
 31 export PATH
 32 
 33 # Percona XtraBackup path
 34 PERCONA_XB=$HOME/app/percona-xtrabackup-2.0.8
 35 PATH=$PERCONA_XB/bin:$PATH
 36 export PATH`

**注意查看是否已经设置了mysql环境变量，如果没有，也需要自行添加。**

添加好环境变量后，更新环境变量，使其生效：

{% highlight bash linenos %}
[gongjz@localhost bin]$ vim ~/.bash_profile
[gongjz@localhost bin]$ source ~/.bash_profile
{% endhighlight bash %}

### 注意事项
* 如果未添加mysql环境变量，会报如下错误：

{% highlight bash linenos %}
>[gongjz@localhost bin]$ ./innobackupex --user=root --password=Netease163 /home/gongjz/backup/
>
>InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.
>
>This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.
>
>160126 16:08:07  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --unbuffered --
>160126 16:08:07  innobackupex: Connected to database with mysql child process (pid=3078)
**innobackupex: Error: mysql child process has died: sh: mysql: command not found**
{% endhighlight bash %}

* 如果未添加XtraBackup的环境变量，会报如下错误：
>[gongjz@localhost bin]$ ./innobackupex --user=root --password=Netease163 --socket=/home/gongjz/tmp/mysql.sock /home/gongjz/backup/
>
>InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.
>
>This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.
>
>160126 16:17:32  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160126 16:17:32  innobackupex: Connected to database with mysql child process (pid=3366)
160126 16:17:38  innobackupex: Connection to database server closed
IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".
>
>innobackupex: Using mysql  Ver 14.14 Distrib 5.5.46, for linux2.6 (x86_64) using readline 5.1
>innobackupex: Using mysql server version Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.
>
>**sh: xtrabackup_55: command not found
innobackupex: fatal error: no 'mysqld' group in MySQL options**

* 如果未
>[gongjz@localhost bin]$ ./innobackupex --user=root --password=Netease163 --socket=/home/gongjz/tmp/mysql.sock /home/gongjz/backup/
>
>InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.
>
>This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.
>
>160126 16:22:10  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160126 16:22:10  innobackupex: Connected to database with mysql child process (pid=3636)
160126 16:22:16  innobackupex: Connection to database server closed
IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".
>
>innobackupex: Using mysql  Ver 14.14 Distrib 5.5.46, for linux2.6 (x86_64) using readline 5.1
>innobackupex: Using mysql server version Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.
>
>innobackupex: Created backup directory /home/gongjz/backup/2016-01-26_16-22-16
160126 16:22:16  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160126 16:22:16  innobackupex: Connected to database with mysql child process (pid=3663)
160126 16:22:18  innobackupex: Connection to database server closed
>
>160126 16:22:18  innobackupex: Starting ibbackup with command: xtrabackup_55  --defaults-group="mysqld" --backup --suspend-at-end --target-dir=/home/gongjz/backup/2016-01-26_16-22-16 --tmpdir=/tmp
>innobackupex: Waiting for ibbackup (pid=3679) to suspend
innobackupex: Suspend file '/home/gongjz/backup/2016-01-26_16-22-16/xtrabackup_suspended'
>
>xtrabackup_55 version 2.0.8 for Percona Server 5.5.16 Linux (x86_64) (revision id: 587)
xtrabackup: uses posix_fadvise().
xtrabackup_55: Can't change dir to '/var/lib/mysql' (Errcode: 2)
xtrabackup: cannot my_setwd /var/lib/mysql
innobackupex: Error: ibbackup child process has died at ./innobackupex line 386.
[gongjz@localhost bin]$ 




[1]: https://www.percona.com/doc/percona-xtrabackup/2.2/index.html "Percona XtraBackup"
[2]: https://www.percona.com/doc/percona-xtrabackup/2.2/installation.html "XtraBackup安装文档"
[3]: https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.3.3/binary/tarball/percona-xtrabackup-2.3.3-Linux-x86_64.tar.gz "Linux Generic 64位二进制安装包"
