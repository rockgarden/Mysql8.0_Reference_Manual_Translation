# 针对 InnoDB 表进行优化

- 8.5.1 [优化 InnoDB 表的存储布局](#优化-innodb-表的存储布局)
- 8.5.2 [优化 InnoDB 事务管理](#优化-innodb-事务管理)
- 8.5.3 [优化 InnoDB 只读事务](#优化-innodb-只读事务)
- 8.5.4 优化 InnoDB 重做日志
- 8.5.5 [InnoDB 表的批量数据加载](批量数据加载.md)
- 8.5.6 [优化 InnoDB 查询](#优化-innodb-查询)
- 8.5.7 优化 InnoDB DDL 操作
- 8.5.8 优化 InnoDB 磁盘 I/O
- 8.5.9 优化 InnoDB 配置变量
- 8.5.10 为具有许多表的系统优化 InnoDB

InnoDB 是 MySQL 客户通常在可靠性和并发性很重要的生产数据库中使用的存储引擎。 InnoDB 是 MySQL 中的默认存储引擎。 本节介绍如何优化 InnoDB 表的数据库操作。

## 优化 InnoDB 表的存储布局

一旦您的数据达到稳定大小，或者不断增长的表增加了数十或数百兆字节，请考虑使用 OPTIMIZE TABLE 语句重新组织表并压缩任何浪费的空间。重组的表需要更少的磁盘 I/O 来执行全表扫描。这是一种简单的技术，可以在其他技术（例如提高索引使用率或调整应用程序代码）不实用时提高性能。

- OPTIMIZE TABLE 复制表的数据部分并重建索引。好处来自改进的索引内数据打包，以及减少表空间和磁盘上的碎片。好处因每个表中的数据而异。您可能会发现某些收益显着而其他收益不显着，或者收益会随着时间的推移而减少，直到您下一次优化表。如果表很大或者正在重建的索引不适合缓冲池，则此操作可能会很慢。向表中添加大量数据后的第一次运行通常比以后的运行慢得多。

在 InnoDB 中，具有长 PRIMARY KEY（具有长值的单列，或形成长复合值的几列）会浪费大量磁盘空间。行的主键值在指向同一行的所有二级索引记录中重复。 （请参阅第 15.6.2.1 节，“[聚集索引和二级索引](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)”。）如果您的主键很长，则创建一个 AUTO_INCREMENT 列作为主键，或者索引一个长 VARCHAR 列的前缀而不是整个列。

使用 VARCHAR 数据类型而不是 CHAR 来存储可变长度字符串或具有许多 NULL 值的列。 [CHAR(N)](https://dev.mysql.com/doc/refman/8.0/en/char.html) 列始终使用 N 个字符来存储数据，即使字符串较短或其值为 NULL。较小的表更适合缓冲池并减少磁盘 I/O。

- 当使用 COMPACT 行格式（默认的 InnoDB 格式）和可变长度字符集（例如 utf8mb4 或 sjis）时，CHAR(N) 列占用的空间量是可变的，但仍然至少 N 字节。

对于较大的表或包含大量重复文本或数字数据的表，请考虑使用 COMPRESSED 行格式。将数据带入缓冲池或执行全表扫描所需的磁盘 I/O 较少。在做出永久性决定之前，请测量使用 COMPRESSED 与 COMPACT 行格式可以实现的压缩量。

## 优化 InnoDB 事务管理

要优化 InnoDB 事务处理，请在事务功能的性能开销和服务器的工作负载之间找到理想的平衡点。例如，如果应用程序每秒提交数千次，它可能会遇到性能问题，如果它仅每 2-3 小时提交一次，则可能会遇到不同的性能问题。

- 默认的 MySQL 设置 AUTOCOMMIT=1 可以对繁忙的数据库服务器施加性能限制。在可行的情况下，通过发出 SET AUTOCOMMIT=0 或 START TRANSACTION 语句，将几个相关的数据更改操作包装到单个事务中，然后在进行所有更改后执行 COMMIT 语句。

  如果该事务对数据库进行了修改，则 InnoDB 必须在每次事务提交时将日志刷新到磁盘。当每次更改后都有提交时（与默认的自动提交设置一样），存储设备的 I/O 吞吐量会限制每秒潜在操作的数量。

- 或者，对于仅包含单个 SELECT 语句的事务，打开 AUTOCOMMIT 有助于 InnoDB 识别只读事务并优化它们。有关要求，请参阅 [优化 InnoDB 只读事务](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html)。

- 避免在插入、更新或删除大量行后执行回滚。如果一个大事务正在降低服务器性能，回滚它会使问题变得更糟，可能需要数倍于原始数据更改操作的时间来执行。杀死数据库进程没有帮助，因为回滚在服务器启动时再次开始。

  为了尽量减少发生此问题的机会：

  - 增加[缓冲池](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_buffer_pool)的大小，使所有的数据变化变化都可以被缓存而不是立即写入磁盘。

  - 设置 [innodb_change_buffering=all](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_change_buffering) 以便除了插入之外还缓冲更新和删除操作。

  - 考虑在大数据更改操作期间定期发出 COMMIT 语句，可能将单个删除或更新分解为对较少行数进行操作的多个语句。

  为了摆脱一旦发生的失控回滚，请增加缓冲池，以便回滚成为 CPU 密集型并快速运行，或者杀死服务器并使用 innodb_force_recovery=3 重新启动，如第 [InnoDB 恢复](../../InnoDB存储引擎/InnoDB备份和恢复.md#innodb恢复) 中所述.

  这个问题预计在默认设置 [innodb_change_buffering=all](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_change_buffering) 时不常见，它允许将更新和删除操作缓存在内存中，从而使它们首先执行得更快，并且在需要时也可以更快地回滚。确保在处理具有许多插入、更新或删除的长时间运行事务的服务器上使用此参数设置。

- 如果在发生意外退出时您可以承受丢失一些最新提交的事务，您可以将 [innodb_flush_log_at_trx_commit](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit) 参数设置为 0。无论如何，InnoDB 都会尝试每秒刷新一次日志，尽管不能保证刷新。

- 修改或删除行时，不会立即物理删除行和关联的[撤消日志](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_undo_log)，甚至不会在事务提交后立即删除。保留旧数据，直到更早或同时启动的事务完成，以便这些事务可以访问已修改或已删除行的先前状态。因此，长时间运行的事务可以防止 InnoDB 清除由不同事务更改的数据。

- 当在长时间运行的事务中修改或删除行时，如果其他使用 [READ COMMITTED](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed) 和 [REPEATABLE READ](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read) 隔离级别的事务读取相同的行，则必须做更多的工作来重建旧数据。

- 当一个长时间运行的事务修改一个表时，来自其他事务的对该表的查询不会使用[覆盖索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_covering_index)技术。通常可以从二级索引中检索所有结果列的查询，而是从表数据中查找适当的值。

  如果发现二级索引页面的 PAGE_MAX_TRX_ID 太新，或者二级索引中的记录被删除标记，InnoDB 可能需要使用聚集索引查找记录。

## 优化 InnoDB 只读事务

InnoDB 可以避免与为已知为只读的事务设置[事务ID](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction_id)（TRX_ID 字段）相关的开销。只有可能执行写入操作或[锁定读取](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking_read)（例如 SELECT ... FOR UPDATE）的[事务](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction)才需要事务 ID。消除不必要的事务 ID 可以减少每次查询或数据更改语句构造[读取视图](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_read_view)时所查询的内部数据结构的大小。

InnoDB 在以下情况下检测只读事务：

- 事务使用 [START TRANSACTION READ ONLY](https://dev.mysql.com/doc/refman/8.0/en/commit.html) 语句启动。在这种情况下，尝试对数据库（对于 InnoDB、MyISAM 或其他类型的表）进行更改会导致错误，并且事务会以只读状态继续：

  `ERROR 1792 (25006): Cannot execute statement in a READ ONLY transaction.`

  您仍然可以在只读事务中更改特定于会话的临时表，或为它们发出锁定查询，因为这些更改和锁定对任何其他事务都不可见。

- 开启自动提交设置，从而保证事务是单条语句，构成事务的单条语句是“非锁定”的SELECT语句。也就是说，一个不使用 FOR UPDATE 或 LOCK IN SHARED MODE 子句的 SELECT。

- 事务在没有 READ ONLY 选项的情况下启动，但尚未执行显式锁定行的更新或语句。在需要更新或显式锁定之前，事务一直处于只读模式。

因此，对于诸如报告生成器之类的读取密集型应用程序，您可以通过在 START TRANSACTION READ ONLY 和 COMMIT 中对它们进行分组来调整 InnoDB 查询序列，或者通过在运行 SELECT 语句之前打开自动提交设置，或者简单地避免任何散布在查询中的数据更改语句。

有关 START TRANSACTION 和自动提交的信息，请参阅第 13.3.1 节，“[START TRANSACTION、COMMIT 和 ROLLBACK 语句](https://dev.mysql.com/doc/refman/8.0/en/commit.html)”。

> 笔记
符合自动提交、非锁定和只读 (AC-NL-RO) 条件的事务被排除在某些内部 InnoDB 数据结构之外，因此未在 [SHOW ENGINE INNODB STATUS](https://dev.mysql.com/doc/refman/8.0/en/show-engine.html) 输出中列出。

## 优化 InnoDB 查询

要调整 InnoDB 表的查询，请在每个表上创建一组适当的索引。有关详细信息，请参阅 [MySQL 如何使用索引](../优化和索引/MySQL如何使用索引.md)。遵循 InnoDB 索引的这些准则：

- 因为每个 InnoDB 表都有一个主键（无论您是否请求），请为每个表指定一组主键列，这些列用于最重要和时间关键的查询。

- 不要在主键中指定太多或太长的列，因为这些列值在每个二级索引中都是重复的。当索引包含不必要的数据时，读取这些数据的 I/O 和缓存它的内存会降低服务器的性能和可伸缩性。

- 不要为每一列创建单独的[二级索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_secondary_index)，因为每个查询只能使用一个索引。很少测试的列或只有几个不同值的列上的索引可能对任何查询都没有帮助。如果您对同一个表有很多查询，测试不同的列组合，请尝试创建少量的[连接索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_concatenated_index)而不是大量的单列索引。如果索引包含结果集所需的所有列（称为覆盖索引），则查询可能完全避免读取表数据。
  > 注：Mysql优化器可自行完成索引合并优化。

- 如果索引列不能包含任何 NULL 值，请在创建表时将其声明为 NOT NULL。当优化器知道每列是否包含 NULL 值时，它可以更好地确定哪个索引对查询最有效。

- 您可以使用 [优化 InnoDB 只读事务](#优化-innodb-只读事务) 中的技术来优化 InnoDB 表的单查询事务。
