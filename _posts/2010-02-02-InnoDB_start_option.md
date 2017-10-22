---
layout: post
title: InnoDB 启动选项
description: ""
category: IT
analytics: true
tags: [mysql, innodb, 性能]
---

2 InnoDB 启动选项

为了在 MySQL-Max-3.23 中使用 InnoDB 表，你必须在配置文件‘my.cnf’或‘my.ini’（WINDOWS系统）中的 \[mysqld\] 区中详细指定配置参数。

作为最小设置，在 3.23 中你必须在 innodb\_data\_file\_path 上指定数据文件名能及大小。如果在‘my.cnf’中没有指定innodb\_data\_home\_dir，系统将在 MySQL 的 datadir 目录下创建数据文件。如果将 innodb\_data\_home\_dir 设为一个空串，那可以在 innodb\_data\_file\_path 中给定一个绝对路径。在 MySQL-4.0 中可以不设定 innodb\_data\_file\_path ：MySQL-4.0 将默认地在 datadir 目录下建立一个 10 MB 大小自扩充(auto-extending)的文件‘ibdata1’(在MySQL-4.0.0 与 4.0.1 中数据文件的大小为 64 MB 并且是非自扩充的(not auto-extending))。

为了得到更好的性能你必须所示的例子明确地设定 InnoDB 启动参数。

从 3.23.50 版和 4.0.2 版开始，InnoDB 允许在 innodb\_data\_file\_path 中设置的最一个数据文件描述为 auto-extending。 innodb\_data\_file\_path 语法如下所示：

pathtodatafile:sizespecification;pathtodatafile:sizespec;...
...;pathtodatafile:sizespec\[:autoextend\[:max:sizespecification\]\]

如果用 autoextend 选项描述最后一个数据文件，当 InnoDB 用尽所有表自由空间后将会自动扩充最后一个数据文件，每次增量为 8 MB。示例：

innodb\_data\_home\_dir =
innodb\_data\_file\_path = /ibdata/ibdata1:100M:autoextend

指定 InnoDB 只建立一个最初大小为 100 MB 并且当表空间被用尽时以 8MB 每块增加的数据文件。如果硬盘空间不足，可以再添加一个数据文件并将其放在其它的硬盘中。 举例来说：先检查硬盘空间的大小，设定 ibdata1 文件使它接近于硬盘空余空间大小并为 1024 \* 1024 bytes (= 1 MB)的倍数， 将 ibdata1 明确地指定在 innodb\_data\_file\_path 中。在此之后可以添加另一个数据文件：

innodb\_data\_home\_dir =
innodb\_data\_file\_path = /ibdata/ibdata1:988M;/disk2/ibdata2:50M:autoextend

注意：设定文件大小时一定要注意你的OS是否有最大文件尺寸为2GB的限制！InnoDB是不会注意你的OS文件尺寸限制的， 在一些文件系统中你可能要设定最大容量限制：

innodb\_data\_home\_dir =
innodb\_data\_file\_path = /ibdata/ibdata1:100M:autoextend:max:2000M

一个简单的 my.cnf 例子。 假设你的计算机有 128 MB RAM 和一个硬盘。下面的例子是为了使用 InnoDB 而在 my.cnf 或 my.ini 文件中可能所作的一些配置。我们假设你运行的是 MySQL-Max-3.23.50 及以上版本，或 MySQL-4.0.2 及以上版本。

这个示例适合大部分不需要将 InnoDB 数据文件和日志文件放在几个盘上的 Unix 和 Windows 用户。这个例子在 MySQL 的datadir 目录(典型的为 /mysql/data)中创建一个自扩充(auto-extending)的数据文件 ibdata1 和两个 InnoDB 运行日志文件ib\_logfile0 和 ib\_logfile1 以及 ib\_arch\_log\_0000000000 档案文件。

\[mysqld\]
\#在这里加入其它 的 MySQL 服务器配置
\#...
\# 数据文件必须
\# 能够容下数据与索引
\# 确定有足够的
\# 磁盘空间
innodb\_data\_file\_path = ibdata1:10M:autoextend
\# 设置缓冲池的大小为
\# 你的主内存大小的
\# 50 - 80 %
set-variable = innodb\_buffer\_pool\_size=70M
set-variable = innodb\_additional\_mem\_pool\_size=10M
\# 设置日志文件的大小约为
\# 缓冲池(buffer pool)
\# 大小的 25 %
set-variable = innodb\_log\_file\_size=20M
set-variable = innodb\_log\_buffer\_size=8M
\# 如果丢失最近几个事务影响
\# 不大的话可以设置
\# ..\_flush\_log\_at\_trx\_commit = 0
innodb\_flush\_log\_at\_trx\_commit=1

InnoDB 不会自己建立目录，必须自己使用操作系统命令建立相应的目录。检查你的 MySQL 服务程序在 datadir 目录里 有足够的权限建立文件。

注意：在某些文件系统中 数据文件大小必须小于2G！ 所有运行日志文件的大小总和必须小于 2G 或 4G,这依赖于具体的 MySQL 系统版本。 数据文件的总和必须大于等于 10 MB.

当第一次建立 InnoDB 数据库时，建议最好以命令行方式启动 MySQL 服务。这样 InnoDB 数据库建立时的提示信息将在屏幕上显示，从而可以看到建立过程。 下面第 3 节所示就是 InnoDB 数据库建立时的屏幕显示。例如，在 Windows 下使用下列指令启动 mysqld-max.exe ：

your-path-to-mysqld&gt;mysqld-max --console

在 Windows 系统下 my.cnf 或 my.ini 放在哪里？规则如下 ：

只能存在一个 my.cnf 或 my.ini 文件
my.cnf 文件必须放在 C: 的根目录下
my.ini 文件必须放在 WINDIR 目录下，例：C:\\WINDOWS 或 C:\\WINNT。可以使用 MS-DOS 的 SET 命令查看 WINDIR 目录值
如果你的 PC 使用启动引导程序引导系统而 C: 不是启动磁盘，那只能唯一地使用 my.ini 作为设置文件

Unix 下在哪里指定配置文件？在 Unix 下 mysqld 按下列顺序搜索配置文件：

/etc/my.cnf 全局选项
COMPILATION\_DATADIR/my.cnf 服务器范围的选项
defaults-extra-file 采用 --defaults-extra-file=.... 设置的默认文件
~/.my.cnf 用户指定文件
COMPILATION\_DATADIR 是 MySQL 的数据文件目录，它是在 mysqld 被编译时以 ./configure 设置指定 (典型的是 /usr/local/mysql/data 二进制安装或 /usr/local/var 以源安装)。

如果不有确定 mysqld 从哪里读取 my.cnf 或 my.ini，可以在第一命令行上详细指定它的目录：mysqld --defaults-file=your\_path\_to\_my\_cnf。

InnoDB 的数据文件目录是对 innodb\_data\_home\_dir 与 innodb\_data\_file\_path 的数据文件名或目录联合 ，如果需要将在它们之间增加一个“/”或“\\”。如果关键字innodb\_data\_home\_dir 没有在 my.cnf 中明确指定，它的默认值为“.”，即目录“./”，这意味着 MySQL 的 datadir of MySQL.

一个高级的 my.cnf 示例。假设你有一台 2 GB RAM 和3个 60 GB 硬盘(路径分别为 "/", "/dr2" 和 “/dr3”)装有 Linux。下面的例子是为了使用 InnoDB 而在 my.cnf 文件中可能所作的一些配置。

注意：InnoDB 不会自己创建文件目录：你必须自己创建它们。使用 Unix 或 MS-DOS mkdir 命令建立相应的数据与日志文件目录。

\[mysqld\]
\#在这里加入其它 的 MySQL 服务器配置
\#...
\# 如果不使用InnoDB表将一列一行注释去除
\# skip-innodb
\#
\# 数据文件必须
\# 能够容下数据与索引
\# 确定有足够的
\# 磁盘空间
innodb\_data\_file\_path = /ibdata/ibdata1:2000M;/dr2/ibdata/ibdata2:2000M:autoextend
\# 设置缓冲池的大小为
\# 你的主内存大小的
\# 50 - 80 %，但是
\# 在 Linux x86 总内存
\# 使用必须小于 2 GB
set-variable = innodb\_buffer\_pool\_size=1G
set-variable = innodb\_additional\_mem\_pool\_size=20M
innodb\_log\_group\_home\_dir = /dr3/iblogs
\# ..\_log\_arch\_dir 必须和
\# ..\_log\_group\_home\_dir一样；
\# 从 4.0.6开始，可以省略它
innodb\_log\_arch\_dir = /dr3/iblogs
set-variable = innodb\_log\_files\_in\_group=3
\# 设置日志文件的大小约为
\# 缓冲池(buffer pool)
\# 大小的 15 %
set-variable = innodb\_log\_file\_size=150M
set-variable = innodb\_log\_buffer\_size=8M
\# 如果丢失最近几个事务影响
\# 不大的话可以设置
\# ..\_flush\_log\_at\_trx\_commit = 0
innodb\_flush\_log\_at\_trx\_commit=1
set-variable = innodb\_lock\_wait\_timeout=50
\#innodb\_flush\_method=fdatasync
\#set-variable = innodb\_thread\_concurrency=5

注意：我们已在不同的硬盘上放置了两个数据文件， InnoDB 将从数据文件的底部填充表空间。在某些情况下所有的数据被分配到不同的物理硬盘中会提高数据库的性能。 将日志文件与数据文件分别放在不同的物理硬盘中对提高性能通常是很有益的。你同样可以使用一个 RAW 磁盘分区( raw disk partitions(raw devices)) 作为数据文件， 在一些 Unixe 系统中这将提高 I/O 能力。 如何在 my.cnf 中详细指定它们请查看第 12.1 节。

警告：在 Linux x86 上必须小心不能将内存使用设置太高， glibc 会把进程堆增长到线程堆栈之上，这将会使服务器崩溃。下面的接近或超过于 2G 将会很危险：

innodb\_buffer\_pool\_size + key\_buffer +
max\_connections \* (sort\_buffer + record\_buffer) + max\_connections \* 2 MB

每个线程将使用 2MB(MySQL AB 二进制版本为 256 KB)的堆栈，在最坏的环境下还会使用 sort\_buffer + record\_buffer 的附加内存。

如何调整其它的 mysqld 服务器参数？查看 MySQL 用户手册可以得到更详细的信息。适合大多数用户的典型参数如下所示：

skip-locking
set-variable = max\_connections=200
set-variable = record\_buffer=1M
set-variable = sort\_buffer=1M
\# 设置索引缓冲(key\_buffer)大小为
\# 你的 RAM 的 5 - 50% ，这主要依赖于
\# 系统中 MyISAM 表使用量。
\# 但是必须保证索引缓冲(key\_buffer)与 InnoDB
\# 的缓冲池(buffer pool)大小总和
\# 小于 RAM 的 80%。
set-variable = key\_buffer=...

注意：在 my.cnf 文件中有些参数是为了设置数字的，它们的设置格式为：set-variable = innodb... = 123，而其它(字符串和逻辑型)的采用另一设置格式：innodb\_... = ... .

各设置参数的含义如下：

innodb\_data\_home\_dir
这是InnoDB表的目录共用设置。如果没有在 my.cnf 进行设置，InnoDB 将使用MySQL的 datadir 目录为缺省目录。如果设定一个空字串,可以在innodb\_data\_file\_path 中设定绝对路径。

innodb\_data\_file\_path 单独指定数据文件的路径与大小。数据文件的完整路径由 innodb\_data\_home\_dir 与这里所设定值的组合。 文件大小以 MB 单位指定。因此在文件大小指定后必有“M”。 InnoDB 也支持缩写“G”， 1G = 1024M。从 3.23.44 开始，在那些支持大文件的操作系统上可以设置数据文件大小大于 4 GB。而在另一些操作系统上数据文件必须小于 2 GB。数据文件大小总和至少要达到 10 MB。在 MySQL-3.23 中这个参数必须在 my.cnf 中明确指定。在 MySQL-4.0.2 以及更新版本中则不需如此，系统会默认在 MySQL 的 datadir 目录下创建一个 16 MB 自扩充(auto-extending)的数据文件 ibdata1。你同样可以使用一个 原生磁盘分区(RAW raw disk partitions(raw devices)) 作为数据文件， 如何在 my.cnf 中详细指定它们请查看第 12.1 节。
innodb\_mirrored\_log\_groups 为了保护数据而设置的日志文件组的拷贝数目，默认设置为 1。在 my.cnf 中以数字格式设置。
innodb\_log\_group\_home\_dir InnoDB 日志文件的路径。必须与 innodb\_log\_arch\_dir 设置相同值。 如果没有明确指定将默认在 MySQL 的 datadir 目录下建立两个 5 MB 大小的 ib\_logfile... 文件。
innodb\_log\_files\_in\_group 日志组中的日志文件数目。InnoDB 以环型方式(circular fashion)写入文件。数值 3 被推荐使用。在 my.cnf 中以数字格式设置。
innodb\_log\_file\_size 日志组中的每个日志文件的大小(单位 MB)。如果 n 是日志组中日志文件的数目，那么理想的数值为 1M 至下面设置的缓冲池(buffer pool)大小的 1/n。较大的值，可以减少刷新缓冲池的次数，从而减少磁盘 I/O。但是大的日志文件意味着在崩溃时需要更长的时间来恢复数据。 日志文件总和必须小于 2 GB，3.23.55 和 4.0.9 以上为小于 4 GB。在 my.cnf 中以数字格式设置。
innodb\_log\_buffer\_size InnoDB 将日志写入日志磁盘文件前的缓冲大小。理想值为 1M 至 8M。大的日志缓冲允许事务运行时不需要将日志保存入磁盘而只到事务被提交(commit)。 因此，如果有大的事务处理，设置大的日志缓冲可以减少磁盘I/O。 在 my.cnf 中以数字格式设置。
innodb\_flush\_log\_at\_trx\_commit 通常设置为 1，意味着在事务提交前日志已被写入磁盘， 事务可以运行更长以及服务崩溃后的修复能力。如果你愿意减弱这个安全，或你运行的是比较小的事务处理，可以将它设置为 0 ，以减少写日志文件的磁盘 I/O。这个选项默认设置为 0。
innodb\_log\_arch\_dir The directory where fully written log files would be archived if we used log archiving. 这里设置的参数必须与innodb\_log\_group\_home\_dir 相同。 从 4.0.6 开始，可以忽略这个参数。
innodb\_log\_archive 这个值通常设为 0。 既然从备份中恢复(recovery)适合于 MySQL 使用它自己的 log files，因而通常不再需要 archive InnoDB log files。这个选项默认设置为 0。
innodb\_buffer\_pool\_size InnoDB 用来高速缓冲数据和索引内存缓冲大小。 更大的设置可以使访问数据时减少磁盘 I/O。在一个专用的数据库服务器上可以将它设置为物理内存的 80 %。 不要将它设置太大，因为物理内存的使用竞争可能会影响操作系统的页面调用。在 my.cnf 中以数字格式设置。
innodb\_additional\_mem\_pool\_size InnoDB 用来存储数据字典(data dictionary)信息和其它内部数据结构(internal data structures)的存储器组合(memory pool)大小。理想的值为 2M，如果有更多的表你就需要在这里重新分配。如果 InnoDB 用尽这个池中的所有内存，它将从操作系统中分配内存，并将错误信息写入 MySQL 的错误日志中。在 my.cnf 中以数字格式设置。
innodb\_file\_io\_threads InnoDB 中的文件 I/O 线程。 通常设置为 4，但是在 Windows 下可以设定一个更大的值以提高磁盘 I/O。在 my.cnf 中以数字格式设置。
innodb\_lock\_wait\_timeout 在回滚(rooled back)之前，InnoDB 事务将等待超时的时间(单位 秒)。InnoDB 会自动检查自身在锁定表与事务回滚时的事务死锁。如果使用LOCK TABLES 命令，或在同一个事务中使用其它事务安全型表处理器(transaction safe table handlers than InnoDB)，那么可能会发生一个 InnoDB 无法注意到的死锁。在这种情况下超时将用来解决这个问题。这个参数的默认值为 50 秒。在 my.cnf 中以数字格式设置。
innodb\_flush\_method 这个参数仅仅与 Unix 相关。这个参数默认值为 fdatasync。 另一个设置项为 O\_DSYNC。这仅仅影响日志文件的转储，在 Unix 下以 fsync 转储数据。InnoDB 版本从 3.23.40b 开始，在 Unix 下指定 fdatasync 为使用 fsync 方式、指定 O\_DSYNC 为使用 O\_SYNC 方式。由于这在某些 Unix 环境下还有些问题所以在 'data' versions 并没有被使用。
innodb\_force\_recovery 警告：此参数只能在你希望从一个被损坏的数据库中转储(dump)数据的紧急情况下使用！ 可能设置的值范围为 1 - 6。查看下面的章节 'Forcing recovery' 以了解这个参数的具体含义。参数设置大于 0 的值代表着 InnoDB 防止用户修改数据的安全度。从 3.23.44 开始，这个参数可用。在my.cnf 中以数字格式设置。
innodb\_fast\_shutdown InnoDB 缺少在关闭之前清空插入缓冲。这个操作可能需要几分钟，在极端的情况下可以需要几个小时。如果这个参数据设置为 1 ，InnoDB 将跳过这个过程而直接关闭。从 3.23.44 和 4.0.1 开始，此参数可用。从 3.23.50 开始，此参数的默认值为 1。
innodb\_thread\_concurrency InnoDB 会试图将 InnoDB 服务的使用的操作系统进程小于或等于这里所设定的数值。此参数默认值为 8。如果计算机系统性能较低或innodb\_monitor 显示有很多线程等侍信号，应该将这个值设小一点。如果你的计算机系统有很我的处理器与磁盘系统，则可以将这个值设高一点以充分利用你的系统资源。建议设值为处理器数目+ 磁盘数目。 从 3.23.44 和 4.0.1 开始，此参数可用。在 my.cnf 中以数字格式设置。
