# 针对 InnoDB 表进行优化

<https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb.html>

- 8.5.1 [优化InnoDB表的存储布局](#优化-innodb-表的存储布局)
- 8.5.2 [优化InnoDB事务管理](#优化-innodb-事务管理)
- 8.5.3 [优化InnoDB只读事务](#优化innodb只读事务)
- 8.5.4 [优化InnoDB重做日志](#优化innodb重做日志)
- 8.5.5 [InnoDB表的批量数据加载](#innodb表的批量数据加载)
- 8.5.6 [优化InnoDB查询](#优化innodb查询)
- 8.5.7 [优化InnoDB DDL操作](#优化innodb-ddl操作)
- 8.5.8 [优化InnoDB磁盘I/O](优化InnoDB磁盘IO.md)
- 8.5.9 [优化InnoDB配置变量](优化InnoDB配置变量.md)
- 8.5.10 [为具有许多表的系统优化InnoDB](#针对多表系统优化innodb)

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

## 优化InnoDB只读事务

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

## 优化InnoDB重做日志

考虑以下优化重做日志记录的指导原则：

- 增加重做日志文件的大小。当InnoDB已将重做日志文件写入满时，它必须将缓冲池的修改内容写入检查点中的磁盘。小的重做日志文件会导致许多不必要的磁盘写入。

  在MySQL 8.0.30中，重做日志文件大小由[innodb_redo_log_capacity](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_redo_log_capacity)设置决定。InnoDB尝试维护32个相同大小的重做日志文件，每个文件等于1/32*InnoDB_redo_log_capacity。因此，更改innodb_redo_log_capacity设置会更改重做日志文件的大小。

  在MySQL 8.0.30之前，使用[innodb_log_file_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_file_size)和[innodb_log_files_in_group](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_files_in_group)变量配置重做日志文件的大小和数量。

- 有关修改重做日志文件配置的信息，请参阅第15.6.5节“[重做日志](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html)”。

- 考虑增加[日志缓冲区](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_log_buffer)的大小。大型日志缓冲区使大型事务能够运行，而无需在事务提交之前将日志写入磁盘。因此，如果您有更新、插入或删除许多行的事务，那么增大日志缓冲区可以节省磁盘I/O。日志缓冲区大小是使用innodb_Log_buffer_size配置选项配置的，该选项可以在MySQL 8.0中动态配置。

- 配置[innodb_log_write_ahead_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_write_ahead_size)配置选项以避免“写入时读取”。此选项定义重做日志的预写块大小。将innodb_log_write_ahead_size设置为与操作系统或文件系统缓存块大小匹配。由于重做日志的预写块大小与操作系统或文件系统缓存块大小不匹配，重做日志块未完全缓存到操作系统或系统时，会发生读写操作。

  innodb_log_write_ahead_size的有效值是innodb日志文件块大小（2^n^）的倍数。最小值是InnoDB日志文件块大小（512）。指定最小值时，不会发生提前写入。最大值等于[innodb_page_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_page_size)值。如果为innodb_log_write_ahead_size指定的值大于innodb_page_size值，则innodb_log_write_ahead_ssize设置将被截断为innodb2_page_ssize值。

  相对于操作系统或文件系统缓存块大小，将innodb_log_write_ahead_size值设置得太低会导致写入时读取。设置过高的值可能会对日志文件写入的fsync性能产生轻微影响，因为一次写入多个块。

- MySQL 8.0.11引入了专用的日志写入线程，用于将重做日志记录从日志缓冲区写入系统缓冲区，并将系统缓冲区刷新到重做日志文件。以前，单个用户线程负责这些任务。从MySQL 8.0.22开始，可以使用innodb_log_writer_threads变量启用或禁用日志编写器线程。专用日志编写器线程可以提高高并发系统的性能，但对于低并发系统，禁用专用日志编写线程可以提供更好的性能。

- 优化等待刷新重做的用户线程对旋转延迟的使用。旋转延迟有助于减少延迟。在低并发期间，减少延迟可能不是一个优先事项，在这些期间避免使用旋转延迟可能会减少能耗。在高并发期间，您可能希望避免在旋转延迟上消耗处理能力，以便将其用于其他工作。以下系统变量允许设置高和低水印值，这些值定义了使用旋转延迟的边界。

  - [innodb_log_wait_for_flush_spin_hwm](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_wait_for_flush_spin_hwm)：定义用户线程在等待刷新重做时不再旋转的最大平均日志刷新时间。默认值为400微秒。

  - [innodb_log_spin_cpu_abs_lwm](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_spin_cpu_abs_lwm)：定义用户线程在等待刷新重做时不再旋转的最小cpu使用量。该值表示为CPU核心使用量的总和。例如，默认值80是单个CPU核心的80%。在具有多核处理器的系统上，值150表示一个CPU核的100%使用率加上另一个CPU核心的50%使用率。

  - [innodb_log_spin_cpu_pct_hwm](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_spin_cpu_pct_hwm)：定义用户线程在等待刷新重做时不再旋转的最大cpu使用量。该值表示为所有CPU核心的总处理能力的百分比。默认值为50%。例如，两个CPU核的100%使用率是四个CPU核服务器上CPU处理能力的50%。

    innodb_log_spin_cpu_pct_hwm配置选项尊重处理器关联性。例如，如果一个服务器有48个内核，但mysqld进程仅固定在4个CPU内核上，则忽略其他44个CPU内核。

## InnoDB表的批量数据加载

这些性能提示是对第 8.2.5.1 节“[优化 INSERT 语句](https://dev.mysql.com/doc/refman/8.0/en/insert-optimization.html)”中快速插入的一般准则的补充。

- 将数据导入 InnoDB 时，请关闭自动提交模式，因为它会为每次插入执行日志刷新到磁盘。要在导入操作期间禁用自动提交，请使用 SET autocommit 和 COMMIT 语句将其括起来：

  ```sql
  SET autocommit=0;
  ... SQL import statements ...
  COMMIT;
  ```

  [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) 选项 --opt 创建可以快速导入 InnoDB 表的转储文件，即使不使用 SET autocommit 和 COMMIT 语句包装它们。

- 如果您对辅助键有 UNIQUE 约束，您可以通过在导入会话期间暂时关闭唯一性检查来加快表导入：

  ```sql
  SET unique_checks=0;
  ... SQL import statements ...
  SET unique_checks=1;
  ```
  
  对于大表，这节省了大量的磁盘 I/O，因为 InnoDB 可以使用它的更改缓冲区来批量写入二级索引记录。确保数据不包含重复键。

- 如果您的表中有 FOREIGN KEY 约束，您可以通过在导入会话期间关闭外键检查来加快表导入：

  ```sql
  SET foreign_key_checks=0;
  ... SQL import statements ...
  SET foreign_key_checks=1;
  ```

  对于大表，这可以节省大量的磁盘 I/O。

- 如果需要插入多行，请使用多行 INSERT 语法来减少客户端和服务器之间的通信开销：

  `INSERT INTO yourtable VALUES (1,2), (5,5), ...;`

  此技巧适用于插入任何表，而不仅仅是 InnoDB 表。

- 当对具有自动增量列的表进行批量插入时，将 [innodb_autoinc_lock_mode](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) 设置为 2（交错）而不是 1（连续）。有关详细信息，请参阅第 15.6.1.6 节，“[InnoDB 中的 AUTO_INCREMENT 处理](https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html)”。

- 执行批量插入时，以 PRIMARY KEY 顺序插入行会更快。 InnoDB 表使用[聚集索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_clustered_index)，这使得按 PRIMARY KEY 顺序使用数据相对较快。以 PRIMARY KEY 顺序执行批量插入对于不完全适合缓冲池的表尤其重要。

- 为了在将数据加载到 InnoDB FULLTEXT 索引时获得最佳性能，请遵循以下步骤：

  a. 在表创建时定义列 FTS_DOC_ID，类型为 BIGINT UNSIGNED NOT NULL，具有名为 FTS_DOC_ID_INDEX 的唯一索引。例如：

  ```sql
  CREATE TABLE t1 (
  FTS_DOC_ID BIGINT unsigned NOT NULL AUTO_INCREMENT,
  title varchar(255) NOT NULL DEFAULT '',
  text mediumtext NOT NULL,
  PRIMARY KEY (`FTS_DOC_ID`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
  CREATE UNIQUE INDEX FTS_DOC_ID_INDEX on t1(FTS_DOC_ID);
  ```

  b. 将数据加载到表中。

  c. 加载数据后创建 FULLTEXT 索引。

  > 笔记
  在创建表时添加 FTS_DOC_ID 列时，请确保在更新 FULLTEXT 索引列时更新 FTS_DOC_ID 列，因为 FTS_DOC_ID 必须随着每次 INSERT 或 UPDATE 单调增加。如果您选择在创建表时不添加 FTS_DOC_ID 并让 InnoDB 为您管理 DOC ID，InnoDB 会将 FTS_DOC_ID 作为隐藏列添加到下一个 CREATE FULLTEXT INDEX 调用中。但是，这种方法需要重新构建表，这会影响性能。

- 如果将数据加载到新的 MySQL 实例中，请考虑使用 ALTER INSTANCE {ENABLE|DISABLE} INNODB REDO_LOG 语法禁用重做日志记录。禁用重做日志有助于通过避免重做日志写入来加速数据加载。有关详细信息，请参阅[禁用重做日志记录](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html#innodb-disable-redo-logging)。

  > 警告
  此功能仅用于将数据加载到新的 MySQL 实例中。不要在生产系统上禁用重做日志记录。在禁用重做日志记录时允许关闭和重新启动服务器，但在禁用重做日志记录时服务器意外停止可能会导致数据丢失和实例损坏。

使用 MySQL Shell 导入数据。 MySQL Shell 的并行表导入实用程序 util.importTable() 为大型数据文件提供到 MySQL 关系表的快速数据导入。 MySQL Shell 的转储加载实用程序 util.loadDump() 还提供并行加载功能。请参阅 [MySQL Shell 实用程序](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities.html)。

## 优化InnoDB查询

要调整 InnoDB 表的查询，请在每个表上创建一组适当的索引。有关详细信息，请参阅 [MySQL 如何使用索引](../优化和索引/MySQL如何使用索引.md)。遵循 InnoDB 索引的这些准则：

- 因为每个 InnoDB 表都有一个主键（无论您是否请求），请为每个表指定一组主键列，这些列用于最重要和时间关键的查询。

- 不要在主键中指定太多或太长的列，因为这些列值在每个二级索引中都是重复的。当索引包含不必要的数据时，读取这些数据的 I/O 和缓存它的内存会降低服务器的性能和可伸缩性。

- 不要为每一列创建单独的[二级索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_secondary_index)，因为每个查询只能使用一个索引。很少测试的列或只有几个不同值的列上的索引可能对任何查询都没有帮助。如果您对同一个表有很多查询，测试不同的列组合，请尝试创建少量的[连接索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_concatenated_index)而不是大量的单列索引。如果索引包含结果集所需的所有列（称为覆盖索引），则查询可能完全避免读取表数据。
  > 注：Mysql优化器可自行完成索引合并优化。

- 如果索引列不能包含任何 NULL 值，请在创建表时将其声明为 NOT NULL。当优化器知道每列是否包含 NULL 值时，它可以更好地确定哪个索引对查询最有效。

- 您可以使用 优化InnoDB只读事务 中的技术来优化 InnoDB 表的单查询事务。

## 优化InnoDB DDL操作

对表和索引（CREATE、ALTER和DROP语句）的许多DDL操作都可以在线执行。详见第15.12节“[InnoDB和在线DDL](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl.html)”。

对添加辅助索引的在线DDL支持意味着您通常可以通过创建没有辅助索引的表，然后在加载数据后添加辅助索引，来加快创建和加载表及相关索引的过程。

使用TRUNCATE TABLE清空表，而不是DELETE FROM tbl_name。外键约束可以使TRUNCATE语句像常规DELETE语句一样工作，在这种情况下，DROP TABLE和CREATE TABLE等命令序列可能最快。

因为主键对于每个InnoDB表的存储布局是不可或缺的，并且更改主键的定义涉及到重新组织整个表，所以始终将主键设置为CREATE TABLE语句的一部分，并提前计划，这样以后就不需要ALTER或DROP主键。

## 针对多表系统优化InnoDB

如果您配置了[非持久性优化器统计信息](https://dev.mysql.com/doc/refman/8.0/en/innodb-statistics-estimation.html)（非默认配置），InnoDB将在启动后第一次访问表时计算该表的索引[基数值](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_cardinality)，而不是将这些值存储在表中。在将数据划分为多个表的系统上，这一步骤可能会花费大量时间。由于此开销仅适用于初始的表打开操作，为了“预热”表以备以后使用，请在启动后立即通过发出SELECT 1 FROM tbl_name LIMIT 1语句来访问它。

默认情况下，优化器统计信息将持久化到磁盘，由innodb_stats_persistent配置选项启用。有关持久优化器统计信息的信息，请参阅第15.8.10.1节“[配置持久优化程序统计参数](https://dev.mysql.com/doc/refman/8.0/en/innodb-persistent-stats.html)”。
