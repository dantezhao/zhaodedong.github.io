---
layout: post
title:  "MySQL存储引擎"
categories: Database
tags: mysql
---

* content
{:toc}

## 前言

> 除非用到某些InnoDB不具备的特性，并且没有其他方法可以替代，否则优先选择InnoDB存储引擎。




## 什么是存储引擎

关系数据库表是用于存储和组织信息的数据结构，可以将表理解为由行和列组成的表格，类似于Excel的电子表格的形式。有的表简单，有的表复杂，有的表根本不用来存储任何长期的数据，有的表读取时非常快，但是插入数据时去很差；而我们在实际开发过程中，就可能需要各种各样的表，不同的表，就意味着存储不同类型的数据，数据的处理上也会存在着差异，那么。对于MySQL来说，它提供了很多种类型的存储引擎，我们可以根据对数据处理的需求，选择不同的存储引擎，从而最大限度的利用MySQL强大的功能。

这篇博文将总结和分析各个引擎的特点，以及适用场合，并不会纠结于更深层次的东西。我的学习方法是先学会用，懂得怎么用，再去知道到底是如何能用的。下面就对MySQL支持的存储引擎进行简单的介绍。


在mysql客户端中，使用以下命令可以查看MySQL支持的引擎。

```
mysql> show engines;
+------------+---------+------------------------------------------------------------+--------------+------+------------+
| Engine     | Support | Comment                                                    | Transactions | XA   | Savepoints |
+------------+---------+------------------------------------------------------------+--------------+------+------------+
| MRG_MYISAM | YES     | Collection of identical MyISAM tables                      | NO           | NO   | NO         |
| CSV        | YES     | CSV storage engine                                         | NO           | NO   | NO         |
| MyISAM     | DEFAULT | Default engine as of MySQL 3.23 with great performance     | NO           | NO   | NO         |
| InnoDB     | YES     | Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| MEMORY     | YES     | Hash based, stored in memory, useful for temporary tables  | NO           | NO   | NO         |
+------------+---------+------------------------------------------------------------+--------------+------+------------+

```


## 常用存储引擎简介

整理关于三个存储引擎的简介：MyISAM、InnoDB、MEMORY，前两者应用比较广泛，MEMORY的原因是之前在中科院做项目的时候用过，印象最深的是就是当时在做性能测试的时候MySQL的MEMORY在大部分的时间比HBase快非常多，最后确定了使用MEMORY。

### MyISAM

MyISAM表是独立于操作系统的，这说明可以轻松地将其从Windows服务器移植到Linux服务器；每当我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表名。例如，我建立了一个MyISAM引擎的tb_Demo表，那么就会生成以下三个文件：

- tb_demo.frm，存储表定义；
- tb_demo.MYD，存储数据；
- tb_demo.MYI，存储索引。

MyISAM表无法处理事务，这就意味着有事务处理需求的表，不能使用MyISAM存储引擎。MyISAM存储引擎特别适合在以下几种情况下使用：

- 选择密集型的表。MyISAM存储引擎在筛选大量数据时非常迅速，这是它最突出的优点。
- 插入密集型的表。MyISAM的并发插入特性允许同时选择和插入数据。例如：MyISAM存储引擎很适合管理邮件或Web服务器日志数据。

**如果表主要是用于插入新记录和读出记录，那么选择MyISAM存储引擎能实现处理的高效率，比如日志系统，可以考虑使用MyISAM引擎。**

### InnoDB

InnoDB是一个健壮的事务型存储引擎，这种存储引擎已经被很多互联网公司使用，为用户操作非常大的数据存储提供了一个强大的解决方案。我的电脑上安装的MySQL 5.6.13版，InnoDB就是作为默认的存储引擎。InnoDB还引入了行级锁定和外键约束，在以下场合下，使用InnoDB是最理想的选择：

- 更新密集的表。InnoDB存储引擎特别适合处理多重并发的更新请求。
- 事务。InnoDB存储引擎是支持事务的标准MySQL存储引擎。
- 自动灾难恢复。与其它存储引擎不同，InnoDB表能够自动从灾难中恢复。
- 外键约束。MySQL支持外键的存储引擎只有InnoDB。
- 支持自动增加列AUTO_INCREMENT属性。

**如果需要对事务的完整性要求比较高，要求实现并发控制，那选择InnoDB存储引擎有很大的优势。**

如果需要频繁的进行更新，删除操作的数据库，也可以选择InnoDB存储引擎。因为该类存储引擎可以实现事务的提交和回滚。

### MEMORY

使用MySQL Memory存储引擎的出发点是速度。为得到最快的响应时间，采用的逻辑存储介质是系统内存。虽然在内存中存储表数据确实会提供很高的性能，但当mysqld守护进程崩溃时，所有的Memory数据都会丢失。获得速度的同时也带来了一些缺陷。它要求存储在Memory数据表里的数据使用的是长度不变的格式，这意味着不能使用BLOB和TEXT这样的长度可变的数据类型，VARCHAR是一种长度可变的类型，但因为它在MySQL内部当做长度固定不变的CHAR类型，所以可以使用。

一般在以下几种情况下使用Memory存储引擎：

- 目标数据较小，而且被非常频繁地访问。在内存中存放数据，所以会造成内存的使用，可以通过参数max_heap_table_size控制Memory表的大小，设置此参数，就可以限制Memory表的最大大小。
- 如果数据是临时的，而且要求必须立即可用，那么就可以存放在内存表中。
- 存储在Memory表中的数据如果突然丢失，不会对应用服务产生实质的负面影响。
- Memory同时支持散列索引和B树索引。B树索引的优于散列索引的是，可以使用部分查询和通配查询，也可以使用<、>和>=等操作符方便数据挖掘。散列索引进行“相等比较”非常快，但是对“范围比较”的速度就慢多了，因此散列索引值适合使用在=和<>的操作符中，不适合在<或>操作符中，也同样不适合用在order by子句中。

**如果需要很快的读写速度，对数据的安全性要求较低，可以选择Memory存储引擎。**


***
2016-08-08 20:14:54 hzct