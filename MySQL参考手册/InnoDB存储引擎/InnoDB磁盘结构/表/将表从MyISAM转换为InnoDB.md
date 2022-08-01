# 将表从 MyISAM 转换为 InnoDB

<https://dev.mysql.com/doc/refman/8.0/en/converting-tables-to-innodb.html>

如果你想把MyISAM表转换为InnoDB表从而获得更高的可靠性和可扩展性，在转换之前你需要回顾一下这里的指导和提示。

> 笔记
在以前的 MySQL 版本中创建的分区 MyISAM 表与 MySQL 8.0 不兼容。 此类表必须在升级之前准备好，通过删除分区或将它们转换为 InnoDB。 有关详细信息，请参阅[第 24.6.2 节，“与存储引擎相关的分区限制”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-storage-engines.html)。

## 调整 MyISAM 和 InnoDB 的内存使用情况

当您从 MyISAM 表过渡时，降低 key_buffer_size 配置选项的值以释放缓存结果不再需要的内存。增加 innodb_buffer_pool_size 配置选项的值，该选项执行为 InnoDB 表分配缓存内存的类似作用。 InnoDB 缓冲池缓存表数据和索引数据，加快查询查找速度，并将查询结果保存在内存中以供重用。有关缓冲池大小配置的指导，请参阅[第 8.12.3.1 节，“MySQL 如何使用内存”](https://dev.mysql.com/doc/refman/8.0/en/memory-use.html)。

## 处理过长或过短的事务

由于 MyISAM 表不支持事务，您可能没有过多关注自动提交配置选项以及 COMMIT 和 ROLLBACK 语句。这些关键字对于允许多个会话同时读取和写入 InnoDB 表非常重要，从而在写入繁重的工作负载中提供显着的可扩展性优势。

当事务打开时，系统会保留事务开始时所见数据的快照，如果系统在杂散事务继续运行时插入、更新和删除数百万行，这可能会导致大量开销。因此，请注意避免运行时间过长的事务：

- 如果您使用 mysql 会话进行交互式实验，请务必在完成后 COMMIT（完成更改）或 ROLLBACK（撤消更改）。关闭交互式会话，而不是让它们长时间打开，以避免意外地让交易长时间打开。

- 确保应用程序中的任何错误处理程序也 ROLLBACK 未完成的更改或 COMMIT 已完成的更改。

- ROLLBACK 是一个相对昂贵的操作，因为 INSERT、UPDATE 和 DELETE 操作是在 COMMIT 之前写入 InnoDB 表的，期望大多数更改都成功提交并且很少发生回滚。在试验大量数据时，避免对大量行进行更改然后回滚这些更改。

- 当使用一系列 INSERT 语句加载大量数据时，请定期提交结果以避免事务持续数小时。在数据仓库的典型加载操作中，如果出现问题，您会截断表（使用 TRUNCATE TABLE）并从头开始，而不是执行回滚。

前面的提示可以节省在太长的事务期间可能浪费的内存和磁盘空间。当事务比应有的短时，问题是 I/O 过多。对于每个 COMMIT，MySQL 确保每个更改都安全地记录到磁盘，这涉及一些 I/O。

- 对于 InnoDB 表上的大多数操作，您应该使用设置 autocommit=0。从效率的角度来看，当您发出大量连续的 INSERT、UPDATE 或 DELETE 语句时，这可以避免不必要的 I/O。从安全角度来看，如果您在 mysql 命令行或应用程序的异常处理程序中出错，这允许您发出 ROLLBACK 语句来恢复丢失或乱码的数据。

- 在运行一系列查询以生成报告或分析统计信息时，autocommit=1 适用于 InnoDB 表。在这种情况下，没有与 COMMIT 或 ROLLBACK 相关的 I/O 损失，并且 InnoDB 可以[自动优化只读工作负载](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html)。

- 如果您进行了一系列相关更改，请在最后使用一个 COMMIT 一次完成所有更改。例如，如果您将相关的信息片段插入到多个表中，则在进行所有更改后执行一次 COMMIT。或者，如果您运行许多连续的 INSERT 语句，则在加载所有数据后执行一次 COMMIT；如果您正在执行数百万条 INSERT 语句，则可能通过每一万或十万条记录发出一次 COMMIT 来拆分庞大的事务，这样事务就不会变得太大。

- 请记住，即使是 SELECT 语句也会打开事务，因此在交互式 mysql 会话中运行一些报告或调试查询后，发出 COMMIT 或关闭 mysql 会话。

有关相关信息，请参阅[第 15.7.2.2 节，“自动提交、提交和回滚”](https://dev.mysql.com/doc/refman/8.0/en/innodb-autocommit-commit-rollback.html)。

## 处理死锁

你也许能在MySQL的error log或者`SHOW ENGINE InnoDB STATUS`的输出中看到`deadlocks`相关的信息。不要管这可怕的名字，死锁对于InnoDB表来说并不是严重的问题，并且通常不需要修正。当两个事务开始修改多张表的时候，用不同的顺序访问表，它们可以达到一种状态，每个事务都在等待对方的锁且任务一方都拿不到锁。当`deadlock detection`是开启（默认是开启的）的时候，MySQL 立即会检测到这种情况并取消（rools back）“较小的”的事务，让另一个事务执行。如果使用`innodb_deadlock_detect`配置选项禁用死锁检测，遇到死锁的情况下InnoDB会根据`innodb_lock_wait_timeout`设置来回滚事务。

另一方面，你的应用需要错误处理来重启因死锁被强制取消的事务。当你重新发起和之前相同的SQL语句，原来的计时问题不再有用。要么其他的事务已经结束，然后你的事务可以继续处理，要么其他的事务仍在继续执行，然后你的事务一直等待到它结束。

如果死锁警告重复不断的出现，你可能需要检测你的应用代码来重新排列SQL查询的顺序以保持事务之间的执行顺序保持一致，或者缩短你的事务。你可以开启`innodb_print_all_deadlocks`选项来测试从而可以在MySQL的error log中看到所有的死锁警告，而不是在`SHOW ENGINE INNODB STATUS`输出中仅仅看到的最近的死锁警告。

更多的信息，请查看[第15.7.5节 InnoDB中的死锁](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html).

## 规划存储布局

为了获得InnoDB表的最好性能，你可以调整一系列存储布局相关的参数。

当你转换的MyISAM表是非常庞大、访问频繁时且存储重要数据时，考察和考虑`innodb_file_per_table`和`innodb_page_size`配置选项，和`CREATE TABLE`语句的`ROW_FORMAT`和`KEY_BLOCK_SIZE`的子句。

在你初始测试时，最重要的设置是`innodb_file_per_table`。当这个设置被打开时，新的InnoDB表是隐式使用file-per-table表空间来创建的。与InnoDB系统表空间对比，当表被删除或清空时file-per-table表空间允许操作系统重新使用。file-per-table表空间同时支持DYNAMIC和COMPRESSED行格式，相关的特性包括表压缩，对长的可变长度的列的高效的跨页存储，以及大的索引前缀。更多信息，请查看[第15.6.3.2节 file-per-table 表空间](https://dev.mysql.com/doc/refman/8.0/en/innodb-file-per-table-tablespaces.html)。

你也可以存储InnoDB表在通用共享表空间，此空间支持多种表和所有的行格式。更多信息请查看[第15.6.3.3节 通用表空间](https://dev.mysql.com/doc/refman/8.0/en/general-tablespaces.html)。

## 转换现有表

使用`ALTER TABLE`语句来转换一张非InnoDB表到InnoDB：

`ALTER TABLE table_name ENGINE=InnoDB;`

## 克隆表结构

你可能会通过克隆一张MyISAM表创建一张InnoDB表，而不是通过`ALTER TABLE`来执行转换，然后在切换前可以并行测试新老表之间的兼容性。

使用相同的列和索引定义来创建一张空的InnoDB表，使用`SHOW CREATE TABLE table_name\G`来查看需要用到的完整的`CREATE TABLE`语句。通过`ENGINE=INNODB`子句来改变存储引擎。

## 传输数据

要将大量数据传输到如上一节所示创建的空 InnoDB 表中，请使用 `INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns` 插入行。

您还可以在插入数据后为 InnoDB 表创建索引。 从历史上看，创建新的二级索引对于 InnoDB 来说是一项缓慢的操作，但现在您可以在加载数据后创建索引，而索引创建步骤的开销相对较小。

如果您对辅助键有 UNIQUE 约束，您可以通过在导入操作期间暂时关闭唯一性检查来加快表导入：

```sql
SET unique_checks=0;
... import operation ...
SET unique_checks=1;
```

对于大型的表，因为InnoDB能够使用它的change buffer能够批量的写入二级索引从而能够节约磁盘I/O。当然要确保数据没有重复的key。`unique_checks`允许但不要求存储引擎忽略重复的key。
为了更高效的控制插入处理，你可以分批的插入数据到大型表。

```sql
INSERT INTO newtable SELECT * FROM oldtable
   WHERE yourkey > something AND yourkey <= somethingelse;
```

所有的记录都已被插入后，你就可以重命名这张表了。

在大表转换过程中，增加 InnoDB 缓冲池的大小以减少磁盘 I/O。 通常，推荐的缓冲池大小为系统内存的 50% 到 75%。 您还可以增加 InnoDB 日志文件的大小。

## 存储需求

如果你打算在转换期间创建好几个数据表的InnoDB引擎拷贝，建议你在file-per-table表空间创建表以便你能够在删除这些表时这些空间能被重复利用。当`innodb_file_per_table`配置选项是打开的时候（默认就是打开的），新创建的InnoDB表隐式使用file-per-table表空间。

不管你是把MyISAM表直接转换为InnoDB表还是创建一个InnoDB表的副本，要确保程序处理期间你有足够的磁盘空间来存储新老数据表的数据。**InnoDB表需要比MyISAM表更多的磁盘空间**。如果在运行`ALTER TABLE` 时缺少磁盘空间，数据库可能会花好几个小时来回滚操作。对于插入来说，InnoDB使用插入缓冲来批量合并二级索引到索引中。这可以节省大量的磁盘I/O,然而在回滚时却无法使用这套机制，并且回滚操作所需要的时间可能是插入操作的30倍。

在一个异常回滚的情况下，如果你的数据库中没有特别重要的数据，建议你直接kill数据库进程而不是等待数百万的磁盘I/O操作完成。想了解完整的处理流程，请查看[第15.20.2节 强制InnoDB恢复](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html)。

## 为每一张表定义一个主键

PRIMARY KEY 子句是影响 MySQL 查询性能以及表和索引空间使用的关键因素。主键唯一标识表中的一行。表中的每一行都应该有一个主键值，并且没有两行可以有相同的主键值。

这些是主键的指导方针，然后是更详细的解释。

- 为每个表声明一个 PRIMARY KEY。通常，它是您在查找单行时在 WHERE 子句中引用的最重要的列。

- 在原始 CREATE TABLE 语句中声明 PRIMARY KEY 子句，而不是稍后通过 ALTER TABLE 语句添加它。

- 仔细选择列及其数据类型。首选数字列而不是字符或字符串列。

- 如果没有另一个稳定的、唯一的、非空的数字列可供使用，请考虑使用自动增量列。

- 如果不确定主键列的值是否会改变，自动增量列也是一个不错的选择。更改主键列的值是一项昂贵的操作，可能涉及重新排列表内和每个二级索引内的数据。

考虑向任何还没有主键的表添加主键。根据表格的最大投影大小使用最小的实用数值类型。这可以使每一行稍微更紧凑，这可以为大型表节省大量空间。如果表有任何二级索引，则空间节省会成倍增加，因为主键值在每个二级索引条目中重复。除了减少磁盘上的数据大小外，小的主键还可以让更多的数据放入缓冲池，加快各种操作并提高并发性。

如果表在某个较长的列（例如 VARCHAR）上已有主键，请考虑添加一个新的无符号 AUTO_INCREMENT 列并将主键切换到该列，即使查询中未引用该列。这种设计更改可以在二级索引中节省大量空间。您可以将以前的主键列指定为 UNIQUE NOT NULL 以强制执行与 PRIMARY KEY 子句相同的约束，即防止所有这些列中出现重复值或空值。

如果您将相关信息分布在多个表中，通常每个表都使用相同的列作为其主键。例如，一个人事数据库可能有几个表，每个表都有一个员工编号的主键。一个销售数据库可能有一些表的主键是客户编号，而其他表的主键是订单号。由于使用主键的查找速度非常快，因此您可以为此类表构建高效的连接查询。

如果您完全不使用 PRIMARY KEY 子句，MySQL 会为您创建一个不可见的子句。它是一个 6 字节的值，可能比您需要的要长，因此会浪费空间。因为它是隐藏的，所以您不能在查询中引用它。

## 应用程序性能注意事项

InnoDB 的可靠性和可扩展性特性需要比等效的 MyISAM 表更多的磁盘存储。您可以稍微更改列和索引定义，以获得更好的空间利用率，减少处理结果集时的 I/O 和内存消耗，以及更好的查询优化计划以有效利用索引查找。

如果您为主键设置数字 ID 列，请使用该值与任何其他表中的相关值交叉引用，尤其是对于连接查询。例如，与其接受国家名称作为输入并执行查询以搜索相同的名称，不如执行一次查找以确定国家 ID，然后执行其他查询（或单个连接查询）以跨多个表查找相关信息。与其将客户或目录项目编号存储为可能会占用多个字节的数字字符串，不如将其转换为数字 ID 以进行存储和查询。一个 4 字节的无符号 INT 列可以索引超过 40 亿个项目（美国的含义是十亿：10 亿）。有关不同整数类型的范围，请参阅[第 11.1.2 节，“Integer Types (Exact Value) - INTEGER、INT、SMALLINT、TINYINT、MEDIUMINT、BIGINT”](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)。

## 了解与 InnoDB 表关联的文件

InnoDB 文件比 MyISAM 文件需要更多的关注和计划。

- 您不得删除代表 InnoDB [系统表空间](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_system_tablespace)的 [ibdata 文件](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_ibdata_file)。

- [第 15.6.1.4 节“移动或复制 InnoDB 表”](https://dev.mysql.com/doc/refman/8.0/en/innodb-migration.html)中描述了将 InnoDB 表移动或复制到不同服务器的方法。
