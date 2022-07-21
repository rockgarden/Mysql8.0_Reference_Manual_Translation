# InnoDB备份和恢复

## InnoDB备份

安全数据库管理的关键是定期备份。根据您的数据量、MySQL 服务器的数量和数据库工作负载，您可以单独或组合使用这些备份技术：[热备份](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_hot_backup) 和 MySQL Enterprise Backup；在 MySQL 服务器关闭时通过复制文件进行冷备份；使用 mysqldump 进行逻辑备份，用于较小的数据量或记录模式对象的结构。热备份和冷备份是复制实际数据文件的物理备份，mysqld服务器可以直接使用这些备份进行更快的恢复。

> 笔记
InnoDB 不支持使用第三方备份工具恢复的数据库。

### 热备份

mysqlbackup 命令是 MySQL Enterprise Backup 组件的一部分，可让您备份正在运行的 MySQL 实例，包括 InnoDB 表，同时生成一致的数据库快照，同时将操作中断降至最低。当 mysqlbackup 正在复制 InnoDB 表时，对 InnoDB 表的读写可以继续。 MySQL Enterprise Backup 还可以创建压缩备份文件，并备份表和数据库的子集。结合 MySQL 二进制日志，用户可以进行时间点恢复。 

### 冷备份

如果您可以关闭 MySQL 服务器，则可以进行物理备份，其中包含 InnoDB 用于管理其表的所有文件。使用以下过程：

- 执行 MySQL 服务器的[缓慢关闭](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_slow_shutdown)，并确保它没有错误地停止。
- 将所有 InnoDB 数据文件（ibdata 文件和 .ibd 文件）复制到安全的地方。
- 将所有 InnoDB 日志文件（ib_logfile 文件）复制到安全的地方。
- 将您的 my.cnf 配置文件或文件复制到安全的地方。

### 使用 mysqldump 进行逻辑备份

除了物理备份，建议您通过使用 [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) 转储表来定期创建逻辑备份。二进制文件可能在您不注意的情况下损坏。转储的表存储在人类可读的文本文件中，因此发现表损坏变得更容易。此外，由于格式更简单，严重数据损坏的机会更小。 mysqldump 还有一个 --single-transaction 选项，用于在不锁定其他客户端的情况下制作一致的快照。请参见“[建立备份策略](https://dev.mysql.com/doc/refman/8.0/en/backup-policy.html)”。

复制适用于 InnoDB 表，因此您可以使用 MySQL 复制功能在需要高可用性的数据库站点上保留数据库副本。请参阅“[InnoDB 和 MySQL 复制](https://dev.mysql.com/doc/refman/8.0/en/innodb-and-mysql-replication.html)”。

## InnoDB恢复

### 时间点恢复

要将 InnoDB 数据库从进行物理备份时恢复到现在，您必须在启用二进制日志记录的情况下运行 MySQL 服务器，甚至在进行备份之前。要在还原备份后实现时间点恢复，您可以应用备份后发生的二进制日志中的更改。请参见[第 7.5 节，“时间点（增量）恢复”](https://dev.mysql.com/doc/refman/8.0/en/point-in-time-recovery.html)。

### 从数据损坏或磁盘故障中恢复

如果您的数据库损坏或发生磁盘故障，您必须使用备份执行恢复。在损坏的情况下，首先找到未损坏的备份。恢复基本备份后，使用 mysqlbinlog 和 mysql 从二进制日志文件执行时间点恢复，以恢复备份后发生的更改。

在某些数据库损坏的情况下，转储、删除和重新创建一个或几个损坏的表就足够了。您可以使用 CHECK TABLE 语句来检查表是否损坏，尽管 CHECK TABLE 自然无法检测到所有可能的损坏类型。

在某些情况下，明显的数据库页面损坏实际上是由于操作系统损坏了自己的文件缓存，而磁盘上的数据可能还可以。最好先尝试重新启动计算机。这样做可以消除看起来是数据库页面损坏的错误。如果 MySQL 由于 InnoDB 一致性问题仍然无法启动，请参阅[第 15.21.3 节，“强制 InnoDB 恢复”](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html)以了解在恢复模式下启动实例的步骤，这允许您转储数据。

### InnoDB 崩溃恢复

要从 MySQL 服务器意外退出中恢复，唯一的要求是重新启动 MySQL 服务器。 InnoDB 自动检查日志并将数据库前滚到现在。 InnoDB 自动回滚崩溃时存在的未提交事务。在恢复期间，[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) 显示类似于以下的输出：

```txt
InnoDB: The log sequence number 664050266 in the system tablespace does not match
the log sequence number 685111586 in the ib_logfiles!
InnoDB: Database was not shutdown normally!
InnoDB: Starting crash recovery.
InnoDB: Using 'tablespaces.open.2' max LSN: 664075228
InnoDB: Doing recovery: scanned up to log sequence number 690354176
InnoDB: Doing recovery: scanned up to log sequence number 695597056
InnoDB: Doing recovery: scanned up to log sequence number 700839936
InnoDB: Doing recovery: scanned up to log sequence number 706082816
InnoDB: Doing recovery: scanned up to log sequence number 711325696
InnoDB: Doing recovery: scanned up to log sequence number 713458156
InnoDB: Applying a batch of 1467 redo log records ...
InnoDB: 10%
InnoDB: 20%
InnoDB: 30%
InnoDB: 40%
InnoDB: 50%
InnoDB: 60%
InnoDB: 70%
InnoDB: 80%
InnoDB: 90%
InnoDB: 100%
InnoDB: Apply batch completed!
InnoDB: 1 transaction(s) which must be rolled back or cleaned up in total 561887 row
operations to undo
InnoDB: Trx id counter is 4096
...
InnoDB: 8.0.1 started; log sequence number 713458156
InnoDB: Waiting for purge to start
InnoDB: Starting in background the rollback of uncommitted transactions
InnoDB: Rolling back trx with id 3596, 561887 rows to undo
...
./mysqld: ready for connections....
```

InnoDB 崩溃恢复包括几个步骤：

- 表空间发现

  表空间发现是 InnoDB 用来识别需要重做日志应用程序的表空间的过程。请参阅[崩溃恢复期间的表空间发现](https://dev.mysql.com/doc/refman/8.0/en/innodb-recovery.html#innodb-recovery-tablespace-discovery)。

- 重做日志应用

  重做日志应用程序在初始化期间执行，在接受任何连接之前。如果在关闭或崩溃时所有更改都从缓冲池刷新到表空间（`ibdata* 和 *.ibd 文件`），则跳过重做日志应用程序。如果重做日志文件在启动时丢失，InnoDB 也会跳过重做日志应用程序。

  - 每次值更改时，当前最大的自动增量计数器值都会写入重做日志，这使其具有崩溃安全性。在恢复期间，InnoDB 扫描重做日志以收集计数器值更改并将更改应用于内存表对象。

    有关 InnoDB 如何处理自动增量值的更多信息，请参阅[第 15.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”](https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html)和 [InnoDB AUTO_INCREMENT 计数器初始化](https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html#innodb-auto-increment-initialization)。

  - 当遇到索引树损坏时，InnoDB 将损坏标志写入重做日志，这使得损坏标志崩溃安全。 InnoDB 还在每个检查点将内存损坏标志数据写入引擎专用系统表。在恢复期间，InnoDB 从两个位置读取损坏标志并合并结果，然后将内存中的表和索引对象标记为损坏。

  - 不建议删除重做日志以加快恢复速度，即使某些数据丢失是可以接受的。只有在干净关闭后才应考虑删除重做日志，并将 [innodb_fast_shutdown](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_fast_shutdown) 设置为 0 或 1。

- 回滚不完整的事务

  不完整事务是在意外退出或快速关闭时处于活动状态的任何事务。 回滚不完整事务所需的时间可能是事务被中断之前处于活动状态的时间的三到四倍，具体取决于服务器负载。

  您不能取消正在回滚的事务。 在极端情况下，当回滚事务预计需要非常长的时间时，使用 [innodb_force_recovery](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_force_recovery) 设置为 3 或更大来启动 InnoDB 可能会更快。 请参阅[第 15.21.3 节，“强制 InnoDB 恢复”](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html)。

- 更改缓冲区合并

  将更改缓冲区（系统表空间的一部分）中的更改应用于二级索引的叶页，因为索引页被读取到缓冲池。

- 清除

  删除对活动事务不再可见的删除标记记录。

重做日志应用程序之后的步骤不依赖于重做日志（除了记录写入），并且与正常处理并行执行。其中，只有不完整事务的回滚对崩溃恢复是特殊的。插入缓冲区合并和清除在正常处理期间执行。

应用重做日志后，InnoDB 会尝试尽早接受连接，以减少停机时间。作为崩溃恢复的一部分，InnoDB 在服务器退出时回滚未提交或处于 XA PREPARE 状态的事务。回滚由后台线程执行，与来自新连接的事务并行执行。在回滚操作完成之前，新连接可能会遇到与已恢复事务的锁定冲突。

在大多数情况下，即使 MySQL 服务器在繁重的活动中意外终止，恢复过程也会自动发生，不需要 DBA 采取任何措施。如果硬件故障或严重的系统错误损坏了 InnoDB 数据，MySQL 可能会拒绝启动。在这种情况下，请参阅 “强制 InnoDB 恢复”。

有关二进制日志和 InnoDB 崩溃恢复的信息，请参阅[第 5.4.4 节，“二进制日志”](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html)。

### 崩溃恢复期间的表空间发现

如果在恢复期间，InnoDB 遇到自上次检查点以来写入的重做日志，则重做日志必须应用于受影响的表空间。在恢复期间识别受影响的表空间的过程称为表空间发现。

表空间发现依赖于 [innodb_directories](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_directories) 设置，该设置定义了在启动时扫描表空间文件的目录。 innodb_directories 默认设置为 NULL，但当 InnoDb 构建启动时要扫描的目录列表时，由 [innodb_data_home_dir](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_data_home_dir)、[innodb_undo_directory](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_undo_directory) 和 [datadir](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_datadir) 定义的目录始终附加到 innodb_directories 参数值。无论是否明确指定了 innodb_directories 设置，都会附加这些目录。使用绝对路径定义的表空间文件或位于附加到 innodb_directories 设置的目录之外的表空间文件应添加到 innodb_directories 设置中。如果之前没有发现重做日志中引用的任何表空间文件，则恢复将终止。
