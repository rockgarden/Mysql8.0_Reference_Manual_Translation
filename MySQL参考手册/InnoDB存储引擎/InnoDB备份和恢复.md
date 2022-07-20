# InnoDB备份和恢复

## InnoDB备份

安全数据库管理的关键是定期备份。根据您的数据量、MySQL 服务器的数量和数据库工作负载，您可以单独或组合使用这些备份技术：[热备份](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_hot_backup) 和 MySQL Enterprise Backup；在 MySQL 服务器关闭时通过复制文件进行冷备份；使用 mysqldump 进行逻辑备份，用于较小的数据量或记录模式对象的结构。热备份和冷备份是复制实际数据文件的物理备份，mysqld服务器可以直接使用这些备份进行更快的恢复。

> 笔记
InnoDB 不支持使用第三方备份工具恢复的数据库。

### 热备份

mysqlbackup 命令是 MySQL Enterprise Backup 组件的一部分，可让您备份正在运行的 MySQL 实例，包括 InnoDB 表，同时生成一致的数据库快照，同时将操作中断降至最低。当 mysqlbackup 正在复制 InnoDB 表时，对 InnoDB 表的读写可以继续。 MySQL Enterprise Backup 还可以创建压缩备份文件，并备份表和数据库的子集。结合 MySQL 二进制日志，用户可以进行时间点恢复。 MySQL Enterprise Backup 是 MySQL Enterprise 订阅的一部分。

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
