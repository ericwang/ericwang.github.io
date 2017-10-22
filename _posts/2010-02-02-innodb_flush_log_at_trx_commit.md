---
layout: post
title: mysql的innodb_flush_log_at_trx_commit
description: "抱怨Innodb比MyISAM慢 100倍？那么你大概是忘了调整这个值。"
category: IT
analytics: true
tags: [mysql, 性能]
---

mysql的innodb\_flush\_log\_at\_trx\_commit

innodb\_buffer\_pool\_size
如 果用Innodb，那么这是一个重要变量。相对于MyISAM来说，Innodb对于buffer size更敏感。MySIAM可能对于大数据量使用默认的key\_buffer\_size也还好，但Innodb在大数据量时用默认值就感觉在爬了。 Innodb的缓冲池会缓存数据和索引，所以不需要给系统的缓存留空间，如果只用Innodb，可以把这个值设为内存的70%-80%。和 key\_buffer相同，如果数据量比较小也不怎么增加，那么不要把这个值设太高也可以提高内存的使用率。
innodb\_additional\_pool\_size
这个的效果不是很明显，至少是当操作系统能合理分配内存时。但你可能仍需要设成20M或更多一点以看Innodb会分配多少内存做其他用途。
innodb\_log\_file\_size
对于写很多尤其是大数据量时非常重要。要注意，大的文件提供更高的性能，但数据库恢复时会用更多的时间。我一般用64M-512M，具体取决于服务器的空间。
innodb\_log\_buffer\_size
默认值对于多数中等写操作和事务短的运用都是可以的。如 果经常做更新或者使用了很多blob数据，应该增大这个值。但太大了也是浪费内存，因为1秒钟总会 flush（这个词的中文怎么说呢？）一次，所以不需要设到超过1秒的需求。8M-16M一般应该够了。小的运用可以设更小一点。
innodb\_flush\_log\_at\_trx\_commit （这个很管用）
抱怨Innodb比MyISAM慢 100倍？那么你大概是忘了调整这个值。默认值1的意思是每一次事务提交或事务外的指令都需要把日志写入（flush）硬盘，这是很费时的。特别是使用电 池供电缓存（Battery backed up cache）时。设成2对于很多运用，特别是从MyISAM表转过来的是可以的，它的意思是不写入硬盘而是写入系统缓存。日志仍然会每秒flush到硬 盘，所以你一般不会丢失超过1-2秒的更新。设成0会更快一点，但安全方面比较差，即使MySQL挂了也可能会丢失事务的数据。而值2只会在整个操作系统 挂了时才可能丢数据。

转自:http://swachian.javaeye.com/blog/193788
