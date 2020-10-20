---
title: SQL分析与优化一
date: 2019-12-12 11:16:42
tags: [sql]
categories: [Sql]
---
随着业务量的增大，广告系统出现响应慢、挂死。
分析日志和代码后初步定位问题在sql。

## 生成Oracle AWR报告
AWR全称Automatic Workload Repository，自动负载信息库，是Oracle 10g版本后推出的一种性能收集和分析工具，提供了一个时间段内整个系统的报表数据。通过AWR报告，可以分析指定的时间段内数据库系统的性能。

可以使用sqlplus登录创建快照
```
#登录oracle服务器，sqlplus登录
[root@dszy-cs-app-ad-check-db1 ec2-user]# su - oracle
[oracle@dszy-cs-app-ad-check-db1 ~]$ cd $ORACLE_HOME
[oracle@dszy-cs-app-ad-check-db1 dbhome_1]$ cd rdbms/admin
[oracle@dszy-cs-app-ad-check-db1 admin]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Thu Dec 12 15:56:35 2019

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> 

```
<!--more-->
执行@awrrpt.sql
```
SQL> @awrrpt.sql

Current Instance
~~~~~~~~~~~~~~~~

   DB Id    DB Name	 Inst Num Instance
----------- ------------ -------- ------------
 3757755618 ADDB		1 addb


Specify the Report Type
~~~~~~~~~~~~~~~~~~~~~~~
Would you like an HTML report, or a plain text report?
Enter 'html' for an HTML report, or 'text' for plain text
Defaults to 'html'
Enter value for report_type: 

```
选择html类型

```
Type Specified: html
Instances in this Workload Repository schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   DB Id     Inst Num DB Name	   Instance	Host
------------ -------- ------------ ------------ ------------
* 3757755618	    1 ADDB	   addb 	dszy-cs-app-
						ad-check-db1

Using 3757755618 for database Id
Using	       1 for instance number


Specify the number of days of snapshots to choose from
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Entering the number of days (n) will result in the most recent
(n) days of snapshots being listed.  Pressing <return> without
specifying a number lists all completed snapshots.

```
导出1天
```
Enter value for num_days: 1

Listing the last day's Completed Snapshots

							Snap
Instance     DB Name	    Snap Id    Snap Started    Level
------------ ------------ --------- ------------------ -----
addb	     ADDB	      13874 12 Dec 2019 00:00	   1
			      13875 12 Dec 2019 01:00	   1
			      13876 12 Dec 2019 02:00	   1
			      13877 12 Dec 2019 03:00	   1
			      13878 12 Dec 2019 04:00	   1
			      13879 12 Dec 2019 05:00	   1
			      13880 12 Dec 2019 06:00	   1
			      13881 12 Dec 2019 07:00	   1
			      13882 12 Dec 2019 08:00	   1
			      13883 12 Dec 2019 09:00	   1
			      13884 12 Dec 2019 10:00	   1
			      13885 12 Dec 2019 11:00	   1
			      13886 12 Dec 2019 12:00	   1
			      13887 12 Dec 2019 13:00	   1
			      13888 12 Dec 2019 14:00	   1
			      13889 12 Dec 2019 15:00	   1
			      13890 12 Dec 2019 16:00	   1



Specify the Begin and End Snapshot Ids
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 

```
可以看到oracle每小时统计一次
根据时间选择开始结束节点。这里选择10：00 - 11：00，即13884 - 13885
生成报告到执行文件：/tmp/awr-20191212.html
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enter value for begin_snap: 13884 
Begin Snapshot Id specified: 13884

Enter value for end_snap: 13885
End   Snapshot Id specified: 13885



Specify the Report Name
~~~~~~~~~~~~~~~~~~~~~~~
The default report file name is awrrpt_1_13884_13885.html.  To use this name,
press <return> to continue, otherwise enter an alternative.

Enter value for report_name: /tmp/awr-20191212.html

```

## 分析AWR报告

### DB Time
![head](/images/awr-head.jpg)

**Sessions**
表示采集实例连接的会话数。这个数可以帮助我们了解数据库的并发用户数大概的情况。
**Cursors/session**
每个会话平均打开的游标数。
**Elapsed**
两个快照的间隔时间
**DB Time**
表示用户操作花费的时间，包括CPU时间和等待事件。通常通过这个数值判读数据库的负载情况。
一般来说，Elapsed时间乘以CPU个数如果大于DB Time，就是正常的，系统压力不大，反之就说明压力较大
>_上图中Elapsed*cup<DB Time，系统压力较大_

### Load Profile
![loadProfile](/images/awr-loadProfile.jpg)
load_profile指标主要用来显示当前系统的一些指示性能的总体参数
**Redo_size**
用来显示平均每秒的日志尺寸和平均每个事务的日志尺寸，有时候可以结合Transactions这个每秒事务数，分析当前事务的繁忙程度
**Transactions**
每秒的事务数。表示一个系统的事务繁忙程度。
一般来说Transactions不超过200都是正常的，或者200左右都是正常的，超过1000就是非常繁忙了.

看看平均每秒的日志尺寸是5位数的，平均每个事务的日志尺寸是4位数的，说明了系统访问比较频繁，而单个业务不复杂，如果反过来，平均每秒日志尺寸比平均每事务日志尺寸小很多，说明系统访问不频繁，而业务比较比较复杂，响应时间很久

### SQL Statistics
SQL Statistics从几个维度列举了系统执行比较慢的SQL，可以点击查看SQL，进行调优
>SQL Statistics包含：
_SQL ordered by Elapsed Time
SQL ordered by CPU Time
SQL ordered by User I/O Wait Time
SQL ordered by Gets
SQL ordered by Reads
SQL ordered by Physical Reads (UnOptimized)
SQL ordered by Executions
SQL ordered by Parse Calls
SQL ordered by Sharable Memory
SQL ordered by Version Count
Complete List of SQL Text_


![statistics](/images/awr-statistics.png)

上图是SQL ordered by Elapsed Time和SQL ordered by CPU Time的示例
可以看到SQL id为fnxksbrgqbk9c单次执行时间为2.81s,占用cpu0.53s，c4fab3d63qh3m单次执行时间为1.01s,占用cpu0.56s。
找到问题sql，下一步就可以对sql进行调优。

AWR的性能指标还有很多，这里只是简单说明了一些重要指标。
感兴趣的同学可以深入学习一下。

