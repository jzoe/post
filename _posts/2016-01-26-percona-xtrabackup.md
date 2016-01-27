---
layout: post
title: Percona XtraBackup —— 开源热备份工具
categories: [MySQL, Backup, XtraBackup]
tags: [MySQL, Backup, XtraBackup]
---

[XtraBackup][1]是Percona公司开源的mysql热备份工具，它在备份期间无需锁定我们的数据库，是一个非常强大而实用的工具。

XtraBackup直接读取主机的数据库文件，而不是通过mysql server。但我们能通过把其它机器的磁盘空间挂载在数据库所在的主机上，并指定该磁盘空间作为目标目录。

## 1.安装XtraBackup
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
{% highlight bash linenos %}
 28 # MySQL path
 29 MYSQL_HOME=$HOME/app/mysql
 30 PATH=$MYSQL_HOME/bin:$PATH
 31 export PATH
 32 
 33 # Percona XtraBackup path
 34 PERCONA_XB=$HOME/app/percona-xtrabackup-2.0.8
 35 PATH=$PERCONA_XB/bin:$PATH
 36 export PATH`
{% endhighlight bash %}

**注意查看是否已经设置了mysql环境变量，如果没有，也需要自行添加。**

添加好环境变量后，更新环境变量，使其生效：

{% highlight bash linenos %}
[gongjz@localhost bin]$ vim ~/.bash_profile
[gongjz@localhost bin]$ source ~/.bash_profile
{% endhighlight bash %}

### 注意事项
* 如果未添加mysql环境变量，会报如下错误：

{% highlight bash linenos %}
[gongjz@localhost bin]$ ./innobackupex --user=root --password=your_password /home/gongjz/backup/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

160126 16:08:07  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --unbuffered --
160126 16:08:07  innobackupex: Connected to database with mysql child process (pid=3078)
**innobackupex: Error: mysql child process has died: sh: mysql: command not found**
{% endhighlight bash %}

* 如果未添加XtraBackup的环境变量，会报如下错误：

{% highlight bash linenos %}
[gongjz@localhost bin]$ ./innobackupex --user=root --password=your_password --socket=/home/gongjz/tmp/mysql.sock /home/gongjz/backup/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

160126 16:17:32  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160126 16:17:32  innobackupex: Connected to database with mysql child process (pid=3366)
160126 16:17:38  innobackupex: Connection to database server closed
IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

innobackupex: Using mysql  Ver 14.14 Distrib 5.5.46, for linux2.6 (x86_64) using readline 5.1
innobackupex: Using mysql server version Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

sh: xtrabackup_55: command not found
innobackupex: fatal error: no 'mysqld' group in MySQL options
{% endhighlight bash %}


## 2.运行XtraBackup

在配置好环境变量后，运行报如下错误：

{% highlight bash linenos %}
[gongjz@localhost bin]$ ./innobackupex --user=root --password=your_password --socket=/home/gongjz/tmp/mysql.sock /home/gongjz/backup/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

160126 16:22:10  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160126 16:22:10  innobackupex: Connected to database with mysql child process (pid=3636)
160126 16:22:16  innobackupex: Connection to database server closed
IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

innobackupex: Using mysql  Ver 14.14 Distrib 5.5.46, for linux2.6 (x86_64) using readline 5.1
innobackupex: Using mysql server version Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

innobackupex: Created backup directory /home/gongjz/backup/2016-01-26_16-22-16
160126 16:22:16  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160126 16:22:16  innobackupex: Connected to database with mysql child process (pid=3663)
160126 16:22:18  innobackupex: Connection to database server closed

160126 16:22:18  innobackupex: Starting ibbackup with command: xtrabackup_55  --defaults-group="mysqld" --backup --suspend-at-end --target-dir=/home/gongjz/backup/2016-01-26_16-22-16 --tmpdir=/tmp
innobackupex: Waiting for ibbackup (pid=3679) to suspend
innobackupex: Suspend file '/home/gongjz/backup/2016-01-26_16-22-16/xtrabackup_suspended'

xtrabackup_55 version 2.0.8 for Percona Server 5.5.16 Linux (x86_64) (revision id: 587)
xtrabackup: uses posix_fadvise().
xtrabackup_55: Can't change dir to '/var/lib/mysql' (Errcode: 2)
xtrabackup: cannot my_setwd /var/lib/mysql
innobackupex: Error: ibbackup child process has died at ./innobackupex line 386.
[gongjz@localhost bin]$ 
{% endhighlight bash %}

查看[xtrabackup.cc/xtrabackup_backup_func()函数源码][4]，其中部分内容如下：

{% highlight cpp linenos %}
 2509 /* CAUTION(?): Don't rename file_per_table during backup */
 2510 static void
 2511 xtrabackup_backup_func(void)
 2512 {
 2513 	struct stat stat_info;
 2514 	LSN64 latest_cp;
 2515 
 2516 #ifdef USE_POSIX_FADVISE
 2517 	fprintf(stderr, "xtrabackup: uses posix_fadvise().\n");
 2518 #endif
 2519 
 2520 	/* cd to datadir */
 2521 
 2522 	if (chdir(mysql_real_data_home) != 0)
 2523 	{
 2524 		fprintf(stderr, "xtrabackup: cannot my_setwd %s\n", mysql_real_data_home);
 2525 		exit(EXIT_FAILURE);
 2526 	}
 2527 	fprintf(stderr, "xtrabackup: cd to %s\n", mysql_real_data_home);
 2528   ...
 2529 }
{% endhighlight cpp%}

发现是xtrabackup在未指定配置文件路径**`--defaults-file=/home/gongjz/etc/my.cnf`**时，会使用默认选项，故将`/var/lib/mysql`当做`datadir`。所以给它添加该配置文件参数后，可正常运行：

{% highlight bash linenos %}
[gongjz@localhost ~]$ innobackupex --defaults-file=/home/gongjz/etc/my.cnf --user=root --password=your_password -socket=/home/gongjz/tmp/mysql.sock /home/gongjz/backup/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

160127 10:56:17  innobackupex: Starting mysql with options:  --defaults-file='/home/gongjz/etc/my.cnf' --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160127 10:56:17  innobackupex: Connected to database with mysql child process (pid=2743)
160127 10:56:23  innobackupex: Connection to database server closed
IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

innobackupex: Using mysql  Ver 14.14 Distrib 5.5.46, for linux2.6 (x86_64) using readline 5.1
innobackupex: Using mysql server version Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

innobackupex: Created backup directory /home/gongjz/backup/2016-01-27_10-56-23
160127 10:56:23  innobackupex: Starting mysql with options:  --defaults-file='/home/gongjz/etc/my.cnf' --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160127 10:56:23  innobackupex: Connected to database with mysql child process (pid=2770)
160127 10:56:25  innobackupex: Connection to database server closed

160127 10:56:25  innobackupex: Starting ibbackup with command: xtrabackup_55  --defaults-file="/home/gongjz/etc/my.cnf"  --defaults-group="mysqld" --backup --suspend-at-end --target-dir=/home/gongjz/backup/2016-01-27_10-56-23 --tmpdir=/home/gongjz/tmp
innobackupex: Waiting for ibbackup (pid=2778) to suspend
innobackupex: Suspend file '/home/gongjz/backup/2016-01-27_10-56-23/xtrabackup_suspended'

xtrabackup_55 version 2.0.8 for Percona Server 5.5.16 Linux (x86_64) (revision id: 587)
xtrabackup: uses posix_fadvise().
xtrabackup: cd to /home/gongjz/data
xtrabackup: Target instance is assumed as followings.
xtrabackup:   innodb_data_home_dir = ./
xtrabackup:   innodb_data_file_path = ibdata1:10M:autoextend
xtrabackup:   innodb_log_group_home_dir = ./
xtrabackup:   innodb_log_files_in_group = 2
xtrabackup:   innodb_log_file_size = 5242880
>> log scanned up to (2574993)
[01] Copying ./ibdata1 to /home/gongjz/backup/2016-01-27_10-56-23/ibdata1
[01]        ...done
>> log scanned up to (2574993)
xtrabackup: Creating suspend file '/home/gongjz/backup/2016-01-27_10-56-23/xtrabackup_suspended' with pid '2778'

160127 10:56:27  innobackupex: Continuing after ibbackup has suspended
160127 10:56:27  innobackupex: Starting mysql with options:  --defaults-file='/home/gongjz/etc/my.cnf' --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160127 10:56:27  innobackupex: Connected to database with mysql child process (pid=2792)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
160127 10:56:29  innobackupex: Starting to lock all tables...
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
160127 10:56:39  innobackupex: All tables locked and flushed to disk

160127 10:56:39  innobackupex: Starting to backup non-InnoDB tables and files
innobackupex: in subdirectories of '/home/gongjz/data'
innobackupex: Backing up files '/home/gongjz/data/mysql/*.{frm,isl,MYD,MYI,MAD,MAI,MRG,TRG,TRN,ARM,ARZ,CSM,CSV,opt,par}' (72 files)
innobackupex: Backing up files '/home/gongjz/data/performance_schema/*.{frm,isl,MYD,MYI,MAD,MAI,MRG,TRG,TRN,ARM,ARZ,CSM,CSV,opt,par}' (18 files)
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/db.opt'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/authority.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/config.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/role_authority.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/host.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/department.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/host_prop_map.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/authority_V1@002e0.frm'
innobackupex: Backing up file '/home/gongjz/data/HostInfoMgr/sys_user.frm'
160127 10:56:39  innobackupex: Finished backing up non-InnoDB tables and files

160127 10:56:39  innobackupex: Waiting for log copying to finish

xtrabackup: The latest check point (for incremental): '2574993'
xtrabackup: Stopping log copying thread.
.>> log scanned up to (2574993)

xtrabackup: Creating suspend file '/home/gongjz/backup/2016-01-27_10-56-23/xtrabackup_suspended' with pid '2778'
xtrabackup: Transaction log of lsn (2574993) to (2574993) was copied.
160127 10:56:42  innobackupex: All tables unlocked
160127 10:56:42  innobackupex: Connection to database server closed

innobackupex: Backup created in directory '/home/gongjz/backup/2016-01-27_10-56-23'
innobackupex: MySQL binlog position: filename 'mysql-bin.000011', position 107
160127 10:56:42  innobackupex: completed OK!
[gongjz@localhost ~]$ 
{% endhighlight bash %}

## 3.XtraBackup备份结果分析
运行XtraBackup完后，查看backup文件夹，产生如下备份文件：

{% highlight bash linenos %}
[gongjz@localhost ~]$ ls backup/
2016-01-27_11-27-52
[gongjz@localhost ~]$ ls backup/2016-01-27_11-27-52/
backup-my.cnf  ibdata1  performance_schema  xtrabackup_binary       xtrabackup_checkpoints
HostInfoMgr    mysql    test                xtrabackup_binlog_info  xtrabackup_logfile
[gongjz@localhost ~]$ ls backup/2016-01-27_11-27-52/HostInfoMgr/
authority.frm           config.frm  department.frm  host_prop_map.frm   sys_user.frm
authority_V1@002e0.frm  db.opt      host.frm        role_authority.frm
[gongjz@localhost ~]$ ls data/
HostInfoMgr  localhost.localdomain.err  mysql-bin.000003  mysql-bin.000007  mysql-bin.000011  performance_schema
ibdata1      mysql                      mysql-bin.000004  mysql-bin.000008  mysql-bin.index   test
ib_logfile0  mysql-bin.000001           mysql-bin.000005  mysql-bin.000009  mysqld.log
ib_logfile1  mysql-bin.000002           mysql-bin.000006  mysql-bin.000010  mysqld.pid
[gongjz@localhost ~]$ ls data/HostInfoMgr/
authority.frm           config.frm  department.frm  host_prop_map.frm   sys_user.frm
authority_V1@002e0.frm  db.opt      host.frm        role_authority.frm
[gongjz@localhost ~]
{% endhighlight bash %} 



[1]: https://www.percona.com/doc/percona-xtrabackup/2.2/index.html "Percona XtraBackup"
[2]: https://www.percona.com/doc/percona-xtrabackup/2.2/installation.html "XtraBackup安装文档"
[3]: https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.3.3/binary/tarball/percona-xtrabackup-2.3.3-Linux-x86_64.tar.gz "Linux Generic 64位二进制安装包"
[4]: http://fossies.org/linux/drizzle/plugin/innobase/xtrabackup/xtrabackup.cc "网页搜索（CTRL + F）该函数可查看到源码"