# mysqldump — 数据库备份程序

mysqldump 客户端实用程序执行逻辑备份，生成一组 SQL 语句，可以执行这些语句来重现原始数据库对象定义和表数据。 它转储一个或多个 MySQL 数据库以进行备份或传输到另一台 SQL 服务器。 mysqldump 命令还可以生成 CSV、其他分隔文本或 XML 格式的输出。

> 提示
考虑使用 [MySQL Shell 转储实用程序](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html)，它提供多线程并行转储、文件压缩和进度信息显示，以及 Oracle 云基础设施对象存储流、MySQL 数据库服务兼容性检查和修改等云功能。 使用 [MySQL Shell 负载转储实用程序](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html)可以轻松地将转储导入 MySQL 服务器实例或 MySQL 数据库服务数据库系统。 MySQL Shell 的安装说明可以在[这里](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install.html)找到。

mysqldump 至少需要转储表的 SELECT 权限，转储视图的 SHOW VIEW ，转储触发器的 TRIGGER ，如果不使用 --single-transaction 选项，则需要 LOCK TABLES ，并且（从 MySQL 8.0.21 开始）PROCESS 如果 -- 不使用 no-tablespaces 选项。 如选项描述中所述，某些选项可能需要其他权限。

要重新加载转储文件，您必须具有执行其中包含的语句所需的权限，例如由这些语句创建的对象的适当 CREATE 权限。

mysqldump 输出可以包括更改数据库排序规则的 ALTER DATABASE 语句。 这些可以在转储存储的程序以保留其字符编码时使用。 要重新加载包含此类语句的转储文件，需要受影响数据库的 ALTER 权限。

> 笔记
在 Windows 上使用带有输出重定向的 PowerShell 进行的转储会创建一个具有 UTF-16 编码的文件：`mysqldump [options] > dump.sql`
但是，UTF-16 不允许作为连接字符集（请参阅[不允许的客户端字符集](https://dev.mysql.com/doc/refman/8.0/en/charset-connection.html#charset-connection-impermissible-client-charset)），因此无法正确加载转储文件。 要解决此问题，请使用 --result-file 选项，该选项以 ASCII 格式创建输出：
`mysqldump [options] --result-file=dump.sql`

如果您的转储文件包含系统表，则不建议在服务器上启用 GTID (gtid_mode=ON) 时加载转储文件。 mysqldump 为使用非事务性 MyISAM 存储引擎的系统表发出 DML 指令，并且在启用 GTID 时不允许这种组合。

## 性能和可扩展性注意事项

mysqldump 的优势包括在恢复之前查看甚至编辑输出的便利性和灵活性。您可以克隆用于开发和 DBA 工作的数据库，或者生成现有数据库的轻微变体用于测试。它不是用于备份大量数据的快速或可扩展的解决方案。对于大数据量，即使备份步骤花费了合理的时间，恢复数据也可能非常缓慢，因为重放 SQL 语句涉及用于插入的磁盘 I/O、索引创建等。

对于大规模的备份和恢复，**物理备份**更为合适，将数据文件以原始格式复制，以便快速恢复。

如果您的表主要是 InnoDB 表，或者您有 InnoDB 和 MyISAM 表的混合，请考虑使用 mysqlbackup，它作为 MySQL Enterprise 的一部分提供。该工具以最小的中断为 InnoDB 备份提供高性能；它还可以从 MyISAM 和其他存储引擎备份表；它还提供了许多方便的选项来适应不同的备份方案。请参阅[第 30.2 节，“MySQL 企业备份概述”](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-backup.html)。

mysqldump 可以逐行检索和转储表内容，也可以从表中检索整个内容并在转储之前将其缓冲在内存中。如果要转储大型表，则在内存中缓冲可能会成为问题。要逐行转储表，请使用--quick 选项（或--opt，它启用--quick）。 --opt 选项（因此 --quick）默认启用，因此要启用内存缓冲，请使用 --skip-quick。

如果您使用最新版本的 mysqldump 生成要重新加载到非常旧的 MySQL 服务器的转储，请使用 --skip-opt 选项而不是 --opt 或 --extended-insert 选项。

有关 mysqldump 的更多信息，请参阅[第 7.4 节，“使用 mysqldump 进行备份”](https://dev.mysql.com/doc/refman/8.0/en/using-mysqldump.html)。
