# File-Per-Table 表空间

一个单文件表的表空间保存了单独一张InnoDB表的数据以及索引，这些数据都保存在文件系统上一个单独的数据文件中。

本节中的单文件表的表空间将按照如下几个主题来介绍它相关的特征：

- [File-Per-Table 表空间](#file-per-table-表空间)
  - [单文件表的表空间配置](#单文件表的表空间配置)
  - [File-Per-Table表空间数据文件](#file-per-table表空间数据文件)
  - [单文件表表空间具备的优势](#单文件表表空间具备的优势)
  - [File-Per-Table表空间的缺点](#file-per-table表空间的缺点)

## 单文件表的表空间配置

InnoDB默认在单文件类型得表空间创建表，这个行为受`innodb_file_per_table`配置变量控制。禁用`innodb_file_per_table`会导致InnoDB在系统空间创建表。

`innodb_file_per_table`配置项可以在配置文件中设定，也可以在服务器运行的时候使用`SET GLOBAL`语句来进行设定。运行时修改配置需要足够的权限来设置全局系统变量，具体请查看[5.1.9.1 系统变量的权限](https://dev.mysql.com/doc/refman/8.0/en/system-variable-privileges.html)。

使用配置文件：

```ini
[mysqld]
innodb_file_per_table=ON
```

服务器运行时使用`SET GLOBAL`:

`mysql> SET GLOBAL innodb_file_per_table=ON;`

## File-Per-Table表空间数据文件

单文件表的表空间构建在一个.ibd数据文件中，这个文件存储在MySQL数据文件夹下某个元数据目录里。.ibd文件用表名来命名（table_name.ibd）。例如表test.t1的数据文件位于MySQL数据目录下的test目录中：

```sql
mysql> USE test;  
  
mysql> CREATE TABLE t1 (
   id INT PRIMARY KEY AUTO_INCREMENT,
   name VARCHAR(100) 
 ) ENGINE = InnoDB; 
 
shell> cd /path/to/mysql/data/test
shell> ls 
t1.ibd
```

你可以使用 `CREATE TABLE`的`DATA DIRECTORY`子句来显示得在MySQL数据目录之外创建单文件表空间的数据文件。

## 单文件表表空间具备的优势

File-per-table 表空间与共享表空间（如系统表空间或通用表空间）相比具有以下优点。

- 在截断或删除在 file-per-table 表空间中创建的表后，磁盘空间将返回给操作系统。截断或删除存储在共享表空间中的表会在共享表空间数据文件中创建可用空间，该空间只能用于 InnoDB 数据。换句话说，共享表空间数据文件在表被截断或删除后不会缩小。

- 对驻留在共享表空间中的表执行表复制 ALTER TABLE 操作会增加表空间占用的磁盘空间量。此类操作可能需要与表中的数据加上索引一样多的额外空间。该空间不会像 file-per-table 表空间那样释放回操作系统。

- 在驻留在每表文件表空间中的表上执行时，TRUNCATE TABLE 性能更好。

- File-per-table 表空间数据文件可以在单独的存储设备上创建，用于 I/O 优化、空间管理或备份目的。

- 您可以从另一个 MySQL 实例导入驻留在 file-per-table 表空间中的表。

- 在 file-per-table 表空间中创建的表支持系统表空间不支持的 DYNAMIC 和 COMPRESSED 行格式相关的功能。请参阅[第 15.10 节，“InnoDB 行格式”](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html)。

- 当发生数据损坏、备份或二进制日志不可用或 MySQL 服务器实例无法重新启动时，存储在单个表空间数据文件中的表可以节省时间并提高成功恢复的机会。

- 在 file-per-table 表空间中创建的表可以使用 MySQL Enterprise Backup 快速备份或恢复，而不会中断其他 InnoDB 表的使用。这对于备份计划不同或需要较少备份的表很有用。有关详细信息，请参阅[进行部分备份](https://dev.mysql.com/doc/mysql-enterprise-backup/8.0/en/partial.html)。

- File-per-table 表空间允许通过监视表空间数据文件的大小来监视文件系统上的表大小。

- 当 innodb_flush_method 设置为 O_DIRECT 时，常见的 Linux 文件系统不允许并发写入单个文件，例如共享表空间数据文件。因此，将 file-per-table 表空间与此设置结合使用时，可能会提高性能。

- 共享表空间中的表的大小受 64TB 表空间大小限制。相比之下，每个 file-per-table 表空间的大小限制为 64TB，这为各个表的大小增长提供了充足的空间。

## File-Per-Table表空间的缺点

与系统表空间或通用表空间等共享表空间相比，File-per-table 表空间具有以下缺点。

- 使用 file-per-table 表空间，每个表可能都有未使用的空间，只能由同一个表的行使用，如果管理不当，可能会导致空间浪费。

- fsync 操作在多个 file-per-table 数据文件而不是单个共享表空间数据文件上执行。由于 fsync 操作是针对每个文件的，因此无法组合多个表的写入操作，这可能会导致 fsync 操作的总数增加。

- mysqld 必须为每个 file-per-table 表空间保留一个打开的文件句柄，如果您在 file-per-table 表空间中有许多表，这可能会影响性能。

- 当每个表都有自己的数据文件时，需要更多的文件描述符。

- 可能会出现更多碎片，这会阻碍 DROP TABLE 和表扫描性能。然而，如果碎片是被管理的，file-per-table 表空间可以提高这些操作的性能。

- 删除驻留在每表文件表空间中的表时会扫描缓冲池，这对于大型缓冲池可能需要几秒钟。使用广泛的内部锁执行扫描，这可能会延迟其他操作。

- innodb_autoextend_increment 变量定义了在自动扩展共享表空间文件变满时扩展其大小的增量大小，不适用于 file-per-table 表空间文件，无论 innodb_autoextend_increment 设置如何，这些文件都会自动扩展。最初的 file-per-table 表空间扩展是少量的，之后扩展以 4MB 为增量发生。
