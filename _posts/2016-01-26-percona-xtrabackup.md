---
layout: post
title: Percona XtraBackup —— 开源热备份工具
categories: [MySQL, Backup, XtraBackup]
tags: [MySQL, Backup, XtraBackup]
---

[XtraBackup][1]是Percona公司开源的mysql热备份工具，它在备份期间无需锁定我们的数据库，是一个非常强大而实用的工具。

XtraBackup直接读取主机的数据库文件，而不是通过mysql server。但我们能通过把其它机器的磁盘空间挂载在数据库所在的主机上，并指定该磁盘空间作为目标目录。


## 1. 安装XtraBackup
### 下载
XtraBackup提供了多种安装方式，在其文档的[安装页][2]中有详细的介绍，我采用的是[X86_64 Linux Generic][3]的压缩包
### 解压
将二进制安装包解压到指定文件夹，我路径是：`/home/gongjz/app/percona-xtrabackup-2.0.8`
<pre class="prettyprint lang-bash">
[gongjz@localhost ~]$ cd Downloads/
[gongjz@localhost Downloads]$ tar zxvf percona-xtrabackup-2.0.8-587.tar.gz 
[gongjz@localhost Downloads]$ mv percona-xtrabackup-2.0.8 ~/app/
</pre>

xtrabackup的目录结构如下：
<pre class="prettyprint lang-bash">
[gongjz@localhost ~]$ ls ~/app/percona-xtrabackup-2.0.8/
bin  share
[gongjz@localhost ~]$ ls ~/app/percona-xtrabackup-2.0.8/bin/
innobackupex  innobackupex-1.5.1  xbstream  xtrabackup  xtrabackup_51  xtrabackup_55  xtrabackup_56
[gongjz@localhost ~]$ 
</pre>

innobackupex工具是一个Perl脚本，它对基于C语言实现的xtrabackup工具进行包装。
innobackupex是与Oracle的InnoDB Hot Backup Tool 一同发布的Perl脚本innobackup的补丁包版本(patched version)。
innobackupex具有更多的功能，它集成了**xtrabacup**和其它功能，如文件拷贝和流化(file copying and streaming)，并且添加一些其它方便的功能。
它使得我们能够对InnoDB/XtraDB引擎、MyISAM的表和server的其它分区进行**point-in-time**备份。对于InnoDB/XtraDB引擎的表，在备份的同时也会将它们的**schema definitions信息**一起进行备份。

### 添加环境变量
在`~/.bash_profile`文件中添加下列内容:
<pre class="prettyprint lang-bash">
 28 # MySQL path
 29 MYSQL_HOME=$HOME/app/mysql
 30 PATH=$MYSQL_HOME/bin:$PATH
 31 export PATH
 32 
 33 # Percona XtraBackup path
 34 PERCONA_XB=$HOME/app/percona-xtrabackup-2.0.8
 35 PATH=$PERCONA_XB/bin:$PATH
 36 export PATH`
</pre>

**注意查看是否已经设置了mysql环境变量，如果没有，也需要自行添加。**

在添加环境变量时，一定要注意将自己安装的mysql环境变量添加$PATH之前（即应该这样配置`PATH=$MYSQL_HOME/bin:$PATH`，而不是~~`PATH=$PATH:$MYSQL_HOME/bin`~~），否则系统会根据PATH中的顺序依次查找mysql命令，这时若其它用户安装mysql后添加了环境变量会在你的之前被匹配，故不会执行自己安装的mysql。

添加好环境变量后，更新环境变量，使其生效：

<pre class="prettyprint lang-bash">
[gongjz@localhost bin]$ vim ~/.bash_profile
[gongjz@localhost bin]$ source ~/.bash_profile
</pre>

如果未正确配置mysql环境变量，会报[这个错误](#mysql-env-error)。

如果未正确配置xtrabackup环境变量，会报[这个错误](#xb-env-error)。

## 2. 运行XtraBackup
### 连接参数设置
在运行XtraBackup时，需要指定数据库的用户名和密码，这是两个基本的选项：

* 在命令行添加**`--user=dbuser`**和**`--password=XXX`**参数来指定，推荐使用**root**用户进行备份。
在未指定`--user`时，XtraBackup会认为数据库用户名就是当前执行它的系统用户。

对于不同的系统，可能需要指定其它一些可选的连接选项：

* 通过TCP/IP连接到数据库服务器时，需要指定**`--port`**和**`--host`**选项。
* 连接到本地数据库服务器时，需要通过**`--socket`**选项来指定socket文件。

这些参数都是传递给mysql子线程的，可以查看`mysql --help`获得更详细的连接参数细节。

### 数据库用户权限设置
在连接到数据库后，为了实现备份，需要有对`datadir`进行 **READ\WRITE\EXECUTE** 的权限。
db_user需要有下列的权限：

* **RELOAD** 和 **LOCK TABLES**(除非指定了 `--no-lock`选项)权限，以便在开始拷贝文件之前 FLUSH TABLES WITH READ LOCK。
* **REPLICATION CLIENT** 权限用于获取bin_log的位置。
* **CREATE TABLESPACE** 权限用于导入表。
* **SUPER** 权限用于在replication环境下，启动/停止 slave线程。

具体作用，可以查看[How innobackupex Works][5]，在后续的文章中，也会有相应的介绍，敬请期待！

要想xtrabackup进行完全的备份，需要的最少权限如下：
<pre class="prettyprint lang-sql">
mysql> CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 's3cret';
mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
mysql> FLUSH PRIVILEGES;
</pre>

如果未正确配置，会报[这个错误](#privilege-error)


### 添加`--defaults-file`参数
在未指定--defaults-file参数的情况下，innobackupex会使用my.cnf的默认配置参数，未指定的话，会导致无法正确查找到datadir，例如[这个错误](#default-file-error)。

所以给它添加该配置文件参数后，运行结果如下：
<pre class="prettyprint lang-bash">
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
</pre>

### 总结
要想顺利运行，需根据不同的环境，配置不同的参数，以下几点需特别注意：

* 添加mysql环境变量，可通过`echo $PATH`查看是否已经添加；
* 添加xtrabackup环境变量；
* 添加必要的连接参数选项`--user`, `--password`, `--socket`等，这些选项都是传递给mysql子线程的，故可以先通过将这些参数传递给mysql，看是否能够连接成功；
* 添加必要的配置文件参数选项`--defaults-file`，配置文件需确保正确无误。


## 3. XtraBackup备份结果分析

XtraBackup在运行时，会生成一个以当前时间（**yyyy-MM-dd_hh:mm:ss**)命名的目录，用于保存此次备份的数据。

运行XtraBackup完后，查看backup文件夹，产生如下备份文件：
<pre class="prettyprint lang-bash">
[gongjz@localhost ~]$ ls backup/
2016-01-27_11-27-52
</pre>

将备份其与数据库datadir目录进行比较：
<pre class="prettyprint lang-bash">
[gongjz@localhost ~]$ ll backup/2016-01-27_11-27-52/
total 18464
-rw-rw-r--. 1 gongjz gongjz      260 Jan 27 11:27 backup-my.cnf
drwxrwxr-x. 2 gongjz gongjz     4096 Jan 27 11:28 HostInfoMgr
-rw-rw----. 1 gongjz gongjz 18874368 Jan 27 11:27 ibdata1
drwxrwxr-x. 2 gongjz gongjz     4096 Jan 27 11:28 mysql
drwxrwxr-x. 2 gongjz gongjz     4096 Jan 27 11:28 performance_schema
drwxrwxr-x. 2 gongjz gongjz        6 Jan 27 11:28 test
-rw-rw-r--. 1 gongjz gongjz       13 Jan 27 11:28 xtrabackup_binary
-rw-rw-r--. 1 gongjz gongjz       23 Jan 27 11:28 xtrabackup_binlog_info
-rw-rw----. 1 gongjz gongjz       77 Jan 27 11:28 xtrabackup_checkpoints
-rw-rw----. 1 gongjz gongjz     2560 Jan 27 11:28 xtrabackup_logfile

[gongjz@localhost ~]$ ll data/
total 29096
drwx------. 2 gongjz gongjz     4096 Jan 25 17:04 HostInfoMgr
-rw-rw----. 1 gongjz gongjz 18874368 Jan 27 10:06 ibdata1
-rw-rw----. 1 gongjz gongjz  5242880 Jan 27 10:11 ib_logfile0
-rw-rw----. 1 gongjz gongjz  5242880 Dec  4 12:34 ib_logfile1
-rw-r-----. 1 gongjz gongjz     4856 Dec  4 18:27 localhost.localdomain.err
drwx------. 2 gongjz gongjz     4096 Dec  4 11:28 mysql
-rw-rw----. 1 gongjz gongjz      426 Dec  4 14:26 mysql-bin.000001
-rw-rw----. 1 gongjz gongjz      126 Dec  4 18:27 mysql-bin.000002
-rw-rw----. 1 gongjz gongjz      322 Dec  7 18:49 mysql-bin.000003
-rw-rw----. 1 gongjz gongjz      832 Dec  9 09:43 mysql-bin.000004
-rw-rw----. 1 gongjz gongjz      126 Dec  9 09:57 mysql-bin.000005
-rw-rw----. 1 gongjz gongjz      611 Dec  9 19:13 mysql-bin.000006
-rw-rw----. 1 gongjz gongjz   324358 Jan  7 14:39 mysql-bin.000007
-rw-rw----. 1 gongjz gongjz      526 Jan  7 19:11 mysql-bin.000008
-rw-rw----. 1 gongjz gongjz     4262 Jan 27 10:00 mysql-bin.000009
-rw-rw----. 1 gongjz gongjz      126 Jan 27 10:06 mysql-bin.000010
-rw-rw----. 1 gongjz gongjz      150 Jan 27 12:39 mysql-bin.000011
-rw-rw----. 1 gongjz gongjz      558 Jan 27 16:47 mysql-bin.000012
-rw-rw----. 1 gongjz gongjz      228 Jan 27 12:39 mysql-bin.index
-rw-r-----. 1 gongjz gongjz    20492 Jan 27 12:39 mysqld.log
-rw-rw----. 1 gongjz gongjz        5 Jan 27 10:11 mysqld.pid
drwx------. 2 gongjz gongjz     4096 Dec  4 11:28 performance_schema
drwx------. 2 gongjz gongjz        6 Dec  4 11:28 test
</pre>

经对比可知，xtrabackup会对数据库的**配置文件**(backup-my.cnf)、**各个数据库**(HostInfoMgr/mysql/test/performance_schema等)分别进行备份。同时也会生成以下文件：

* ibdata1:
* xtrabackup_binary：
* xtrabackup_checkpoints：记录了checkpoint的信息
* xtrabackup_binlog_info：记录binlog的信息
* xtrabackup_logfile：

xtrabackup会将每个数据库备份到单独的目录中，对其中的`HostInfoMgr`数据库进行进行比较：
<pre class="prettyprint lang-bash">
[gongjz@localhost ~]$ ls backup/2016-01-27_11-27-52/HostInfoMgr/
authority.frm           config.frm  department.frm  host_prop_map.frm   sys_user.frm
authority_V1@002e0.frm  db.opt      host.frm        role_authority.frm

[gongjz@localhost ~]$ ls data/HostInfoMgr/
authority.frm           config.frm  department.frm  host_prop_map.frm   sys_user.frm
authority_V1@002e0.frm  db.opt      host.frm        role_authority.frm
</pre>

可知xtrapbackup实际上对数据库进行了物理备份，将原数据库中的文件复制到备份文件夹中。

## 常见问题
<span id="mysql-env-error"></span>

* 如果未添加mysql环境变量，会报如下错误：

<pre class="prettyprint lang-bash">
[gongjz@localhost bin]$ ./innobackupex --user=root --password=your_password /home/gongjz/backup/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

160126 16:08:07  innobackupex: Starting mysql with options:  --password=xxxxxxxx --user='root' --unbuffered --
160126 16:08:07  innobackupex: Connected to database with mysql child process (pid=3078)
innobackupex: Error: mysql child process has died: sh: mysql: command not found
[gongjz@localhost bin]$ 
</pre>

<span id="xb-env-error"></span>

* 如果未添加XtraBackup的环境变量，会报如下错误：

<pre class="prettyprint lang-bash">
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
[gongjz@localhost bin]$ 
</pre>

<span id="privilege-error"></span>

* 为测试缺少权限时的错误，我将root用户的相关权限删除：

<pre class="prettyprint lang-sql">
mysql> revoke RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* from 'root'@'localhost';
Query OK, 0 rows affected (0.00 sec)
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
</pre>

这时，运行结果如下：
<pre class="prettyprint  lang-bash">
[gongjz@localhost ~]$ innobackupex --defaults-file=/home/gongjz/etc/my.cnf --user=root --password=Netease163 -socket=/home/gongjz/tmp/mysql.sock /home/gongjz/backup/

InnoDB Backup Utility v1.5.1-xtrabackup; Copyright 2003, 2009 Innobase Oy
and Percona LLC and/or its affiliates 2009-2013.  All Rights Reserved.

This software is published under
the GNU GENERAL PUBLIC LICENSE Version 2, June 1991.

160127 16:03:32  innobackupex: Starting mysql with options:  --defaults-file='/home/gongjz/etc/my.cnf' --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160127 16:03:32  innobackupex: Connected to database with mysql child process (pid=12111)
160127 16:03:38  innobackupex: Connection to database server closed
IMPORTANT: Please check that the backup run completes successfully.
           At the end of a successful backup run innobackupex
           prints "completed OK!".

innobackupex: Using mysql  Ver 14.14 Distrib 5.5.46, for linux2.6 (x86_64) using readline 5.1
innobackupex: Using mysql server version Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

innobackupex: Created backup directory /home/gongjz/backup/2016-01-27_16-03-38
160127 16:03:38  innobackupex: Starting mysql with options:  --defaults-file='/home/gongjz/etc/my.cnf' --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160127 16:03:38  innobackupex: Connected to database with mysql child process (pid=12141)
160127 16:03:40  innobackupex: Connection to database server closed

160127 16:03:40  innobackupex: Starting ibbackup with command: xtrabackup_55  --defaults-file="/home/gongjz/etc/my.cnf"  --defaults-group="mysqld" --backup --suspend-at-end --target-dir=/home/gongjz/backup/2016-01-27_16-03-38 --tmpdir=/home/gongjz/tmp
innobackupex: Waiting for ibbackup (pid=12150) to suspend
innobackupex: Suspend file '/home/gongjz/backup/2016-01-27_16-03-38/xtrabackup_suspended'

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
[01] Copying ./ibdata1 to /home/gongjz/backup/2016-01-27_16-03-38/ibdata1
[01]        ...done
>> log scanned up to (2574993)
xtrabackup: Creating suspend file '/home/gongjz/backup/2016-01-27_16-03-38/xtrabackup_suspended' with pid '12150'

160127 16:03:42  innobackupex: Continuing after ibbackup has suspended
160127 16:03:42  innobackupex: Starting mysql with options:  --defaults-file='/home/gongjz/etc/my.cnf' --password=xxxxxxxx --user='root' --socket='/home/gongjz/tmp/mysql.sock' --unbuffered --
160127 16:03:42  innobackupex: Connected to database with mysql child process (pid=12164)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
160127 16:03:44  innobackupex: Starting to lock all tables...
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
>> log scanned up to (2574993)
innobackupex: Error: mysql child process has died: ERROR 1227 (42000) at line 7: Access denied; you need (at least one of) the RELOAD privilege(s) for this operation
 while waiting for reply to MySQL request: 'FLUSH TABLES WITH READ LOCK;' at /home/gongjz/app/percona-xtrabackup-2.0.8/bin/innobackupex line 386.
[gongjz@localhost ~]$ 
</pre>

<span id="default-file-error"></span>

* 在配置好环境变量后，未配置`--defaults-file`参数，运行时报如下错误：

<pre class="prettyprint">
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
</pre>

查看[xtrabackup.cc/xtrabackup_backup_func()函数源码][4]，其中部分内容如下：

<?prettify lang=cpp linenums=2509?>
<pre class="prettyprint">
 /* CAUTION(?): Don't rename file_per_table during backup */
 static void
 xtrabackup_backup_func(void)
 {
 	struct stat stat_info;
 	LSN64 latest_cp;
 
 #ifdef USE_POSIX_FADVISE
 	fprintf(stderr, "xtrabackup: uses posix_fadvise().\n");
 #endif
 
 	/* cd to datadir */
 
 	if (chdir(mysql_real_data_home) != 0)
 	{
 		fprintf(stderr, "xtrabackup: cannot my_setwd %s\n", mysql_real_data_home);
 		exit(EXIT_FAILURE);
 	}
 	fprintf(stderr, "xtrabackup: cd to %s\n", mysql_real_data_home);
   ...
 }
</pre>

发现是xtrabackup在打开`datadir`时出错。在未指定配置文件路径**`--defaults-file=/home/gongjz/etc/my.cnf`**时，会使用默认选项，故将`/var/lib/mysql`当做`datadir`。

查看`innobackupex --help`可知，确实可以配置**--defaults-file**选项：
<pre class="prettyprint lang-bash">
[gongjz@localhost ~]$ innobackupex --help
Options:
    --defaults-file=[MY.CNF]
        This option specifies what file to read the default MySQL options
        from. The option accepts a string argument. It is also passed
        directly to xtrabackup's --defaults-file option. See the xtrabackup
        documentation for details.

    --defaults-extra-file=[MY.CNF]
        This option specifies what extra file to read the default MySQL
        options from before the standard defaults-file. The option accepts a
        string argument. It is also passed directly to xtrabackup's
        --defaults-extra-file option. See the xtrabackup documentation for
        details.
</pre>
所以给它添加该配置文件参数后，可正常运行（结尾处输出 `innobackupex: completed OK!`）

## 结语
在wish哥的指导下，今天才刚开始接触xtrabackup工具，对其了解非常肤浅，待后续继续学习，再修改本文中不足与错误之处。



[1]: https://www.percona.com/doc/percona-xtrabackup/2.2/index.html "Percona XtraBackup"
[2]: https://www.percona.com/doc/percona-xtrabackup/2.2/installation.html "XtraBackup安装文档"
[3]: https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.3.3/binary/tarball/percona-xtrabackup-2.3.3-Linux-x86_64.tar.gz "Linux Generic 64位二进制安装包"
[4]: http://fossies.org/linux/drizzle/plugin/innobase/xtrabackup/xtrabackup.cc "网页搜索（CTRL + F）该函数可查看到源码"
[5]: https://www.percona.com/doc/percona-xtrabackup/2.0/innobackupex/how_innobackupex_works.html#how-ibk-works "innobackup的工作原理"