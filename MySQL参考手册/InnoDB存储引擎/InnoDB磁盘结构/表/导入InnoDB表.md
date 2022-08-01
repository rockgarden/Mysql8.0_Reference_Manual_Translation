# 导入 InnoDB 表

本节介绍如何使用可传输表空间功能导入表，该功能允许导入位于 file-per-table 表空间中的表、分区表或单个表分区。 您可能想要导入表的原因有很多：

- 在非生产 MySQL 服务器实例上运行reports(报告)以避免在生产服务器上增加额外负载。

- 将数据复制到新的副本服务器。

- 从备份的表空间文件中恢复表。

- 作为一种比导入转储文件更快的移动数据方式，这需要重新插入数据和重建索引。

- 将数据移动到具有更适合您的存储要求的存储介质的服务器。 例如，您可以将繁忙的表移至 SSD 设备，或将大型表移至大容量 HDD 设备。

## 先决条件

- innodb_file_per_table 变量必须启用，默认情况下是启用的。

- 表空间的页面大小必须与目标 MySQL 服务器实例的页面大小相匹配。 InnoDB 页面大小由 innodb_page_size 变量定义，该变量在初始化 MySQL 服务器实例时配置。

- 如果表有外键关系，则在执行 DISCARD TABLESPACE 之前必须禁用 foreign_key_checks。此外，您应该在同一逻辑时间点导出所有与外键相关的表，因为 ALTER TABLE ... IMPORT TABLESPACE 不会对导入的数据实施外键约束。为此，请停止更新相关表，提交所有事务，获取表上的共享锁，然后执行导出操作。

- 从另一个 MySQL 服务器实例导入表时，两个 MySQL 服务器实例必须具有通用可用性 (GA) 状态并且必须是相同的版本。否则，表必须在导入它的同一个 MySQL 服务器实例上创建。

- 如果表是通过在 CREATE TABLE 语句中指定 DATA DIRECTORY 子句在外部目录中创建的，那么您在目标实例上替换的表必须使用相同的 DATA DIRECTORY 子句定义。如果子句不匹配，则会报告架构不匹配错误。要确定源表是否使用 DATA DIRECTORY 子句定义，请使用 SHOW CREATE TABLE 查看表定义。有关使用 DATA DIRECTORY 子句的信息，请参阅[第 15.6.1.2 节，“在外部创建表”](https://dev.mysql.com/doc/refman/8.0/en/innodb-create-table-external.html)。

- 如果未在表定义中显式定义 ROW_FORMAT 选项或使用 ROW_FORMAT=DEFAULT，则源实例和目标实例上的 innodb_default_row_format 设置必须相同。否则，当您尝试导入操作时会报告架构不匹配错误。使用 SHOW CREATE TABLE 检查表定义。使用 SHOW VARIABLES 检查 innodb_default_row_format 设置。有关相关信息，请参阅[定义表的行格式](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html#innodb-row-format-defining)。

## 导入表

此示例演示如何导入驻留在 file-per-table 表空间中的常规非分区表。

1. 在目标实例上，创建一个与您要导入的表具有相同定义的表。 （您可以使用 SHOW CREATE TABLE 语法获取表定义。）如果表定义不匹配，则在尝试导入操作时会报告架构不匹配错误。

    ```sql
    mysql> USE test;
    mysql> CREATE TABLE t1 (c1 INT) ENGINE=INNODB;
    ```

2. 在目标实例上，丢弃刚刚创建的表的表空间。 （导入前必须丢弃接收表的表空间。）
   `mysql> ALTER TABLE t1 DISCARD TABLESPACE;`

3. 在源实例上，运行 FLUSH TABLES ... FOR EXPORT 以静默您要导入的表。当表被停顿时，表上只允许只读事务。

   ```sql
   mysql> USE test;
   mysql> FLUSH TABLES t1 FOR EXPORT;
   ```

   FLUSH TABLES ... FOR EXPORT 确保对命名表的更改刷新到磁盘，以便在服务器运行时可以制作二进制表副本。当 FLUSH TABLES ... FOR EXPORT 运行时，InnoDB 在表的模式目录中生成一个 .cfg 元数据文件。 .cfg 文件包含在导入操作期间用于架构验证的元数据。

   > 笔记
   执行 FLUSH TABLES ... FOR EXPORT 的连接必须在操作运行时保持打开状态；否则，.cfg 文件将被删除，因为在连接关闭时会释放锁。

4. 将 .ibd 文件和 .cfg 元数据文件从源实例复制到目标实例。例如：
   `$> scp /path/to/datadir/test/t1.{ibd,cfg} destination-server:/path/to/datadir/test`
   必须在释放共享锁之前复制 .ibd 文件和 .cfg 文件，如下一步所述。

   > 笔记
   如果您从加密的表空间导入表，InnoDB 会生成一个 .cfp 文件以及一个 .cfg 元数据文件。 .cfp 文件必须与 .cfg 文件一起复制到目标实例。 .cfp 文件包含一个传输密钥和一个加密的表空间密钥。导入时，InnoDB 使用传输密钥来解密表空间密钥。有关相关信息，请参阅[第 15.13 节，“InnoDB 静态数据加密”](https://dev.mysql.com/doc/refman/8.0/en/innodb-data-encryption.html)。

5. 在源实例上，使用 UNLOCK TABLES 释放 FLUSH TABLES ... FOR EXPORT 语句获取的锁：

   ```sql
   mysql> USE test;
   mysql> FLUSH TABLES t1 FOR EXPORT;
   ```

   UNLOCK TABLES 操作还会删除 .cfg 文件。

6. 在目标实例上，导入表空间：

   ```sql
   mysql> USE test;
   mysql> ALTER TABLE t1 IMPORT TABLESPACE;
   ```

## 导入表分区

此示例演示如何导入单个表分区，其中每个分区驻留在一个 file-per-table 表空间文件中。

在以下示例中，导入了一个四分区表的两个分区（p2 和 p3）。

1. 在目标实例上，创建一个与要从中导入分区的分区表具有相同定义的分区表。 （您可以使用 SHOW CREATE TABLE 语法获取表定义。）如果表定义不匹配，则在尝试导入操作时会报告架构不匹配错误。

   ```sql
   mysql> USE test;
   mysql> CREATE TABLE t1 (i int) ENGINE = InnoDB PARTITION BY KEY (i) PARTITIONS 4;
   ```

   在 /datadir/test 目录中，四个分区中的每一个都有一个表空间 .ibd 文件。

   ```sql
   mysql> \! ls /path/to/datadir/test/
   t1#p#p0.ibd  t1#p#p1.ibd  t1#p#p2.ibd t1#p#p3.ibd
   ```

2. 在目标实例上，丢弃您打算从源实例导入的分区。 （在导入分区之前，您必须从接收分区表中丢弃相应的分区。）
   `mysql> ALTER TABLE t1 DISCARD PARTITION p2, p3 TABLESPACE;`

   两个废弃分区的表空间 .ibd 文件将从目标实例的 /datadir/test 目录中删除，留下以下文件：

   ```sql
   mysql> \! ls /path/to/datadir/test/
   t1#p#p0.ibd  t1#p#p1.ibd
   ```

   > 笔记
   当 ALTER TABLE ... DISCARD PARTITION ... TABLESPACE 在子分区表上运行时，允许分区和子分区表名称。 指定分区名称时，该分区的子分区将包含在操作中。

3. 在源实例上，运行 FLUSH TABLES ... FOR EXPORT 以静默分区表。 当表被停顿时，表上只允许只读事务。
  
   ```sql
   mysql> USE test;
   mysql> FLUSH TABLES t1 FOR EXPORT;
   ```

   FLUSH TABLES ... FOR EXPORT 确保对命名表的更改刷新到磁盘，以便在实例运行时可以进行二进制表副本。 当 FLUSH TABLES ... FOR EXPORT 运行时，InnoDB 为表的模式目录中的每个表的表空间文件生成一个 .cfg 元数据文件。

   ```sql
   mysql> \! ls /path/to/datadir/test/
   t1#p#p0.ibd  t1#p#p1.ibd  t1#p#p2.ibd t1#p#p3.ibd
   t1#p#p0.cfg  t1#p#p1.cfg  t1#p#p2.cfg t1#p#p3.cfg
   ```

   .cfg 文件包含在导入操作期间用于架构验证的元数据。 FLUSH TABLES ... FOR EXPORT 只能在表上运行，不能在单个表分区上运行。

4. 将分区 p2 和分区 p3 的 .ibd 和 .cfg 文件从源实例模式目录复制到目标实例模式目录。
   `$> scp t1#p#p2.ibd t1#p#p2.cfg t1#p#p3.ibd t1#p#p3.cfg destination-server:/path/to/datadir/test`

5. 在源实例上，使用 UNLOCK TABLES 释放 FLUSH TABLES ... FOR EXPORT 获取的锁：

   ```sql
   mysql> USE test;
   mysql> UNLOCK TABLES;
   ```

6. 在目标实例上，导入表分区 p2 和 p3：

   ```sql
   mysql> USE test;
   mysql> ALTER TABLE t1 IMPORT PARTITION p2, p3 TABLESPACE;
   ```

   >笔记
   当 ALTER TABLE ... IMPORT PARTITION ... TABLESPACE 在子分区表上运行时，允许分区和子分区表名称。 指定分区名称时，该分区的子分区将包含在操作中。

## 限制

- 可传输表空间功能仅支持驻留在 file-per-table 表空间中的表。驻留在系统表空间或通用表空间中的表不支持它。共享表空间中的表不能被停顿。

- FLUSH TABLES ... FOR EXPORT 在具有 FULLTEXT 索引的表上不受支持，因为无法刷新全文搜索辅助表。导入具有 FULLTEXT 索引的表后，运行 OPTIMIZE TABLE 以重建 FULLTEXT 索引。或者，在导出操作之前删除 FULLTEXT 索引，并在目标实例上导入表后重新创建索引。

- 由于 .cfg 元数据文件的限制，在导入分区表时不会报告分区类型或分区定义差异的架构不匹配。报告列差异。

- 在 MySQL 8.0.19 之前，索引键部分排序顺序信息不会存储到表空间导入操作期间使用的 .cfg 元数据文件中。因此，索引键部分的排序顺序被假定为升序，这是默认值。因此，如果导入操作中涉及的一个表是使用 DESC 索引键部分排序顺序定义的，而另一个表没有定义，则记录可能会以意外的顺序排序。解决方法是删除并重新创建受影响的索引。有关索引键部分排序顺序的信息，请参阅第 13.1.15 节，“CREATE INDEX 语句”。

  .cfg 文件格式在 MySQL 8.0.19 中更新，包括索引键部分排序信息。上述问题不影响 MySQL 8.0.19 服务器实例或更高版本之间的导入操作。

## 使用说明

- ALTER TABLE ... IMPORT TABLESPACE 不需要 .cfg 元数据文件来导入表。但是，在没有 .cfg 文件的情况下导入时不会执行元数据检查，并且会发出类似于以下内容的警告：

  ```txt
  Message: InnoDB: IO Read error: (2, No such file or directory) Error opening '.\
  test\t.cfg', will attempt to import without schema verification
  1 row in set (0.00 sec)
  ```

  只有在没有预期架构不匹配的情况下才应考虑导入没有 .cfg 元数据文件的表。 在无法访问元数据的崩溃恢复场景中，无需 .cfg 文件即可导入的功能可能很有用。

- 在 Windows 上，InnoDB 在内部以小写形式存储数据库、表空间和表名。 为避免在 Linux 和 Unix 等区分大小写的操作系统上出现导入问题，请使用小写名称创建所有数据库、表空间和表。 确保名称以小写形式创建的一种简便方法是在初始化服务器之前将 lower_case_table_names 设置为 1。 （禁止使用与初始化服务器时使用的设置不同的 lower_case_table_names 设置来启动服务器。）

  ```ini
  [mysqld]
  lower_case_table_names=1
  ```

- 在子分区表上运行 ALTER TABLE ... DISCARD PARTITION ... TABLESPACE 和 ALTER TABLE ... IMPORT PARTITION ... TABLESPACE 时，允许使用分区和子分区表名称。 指定分区名称时，该分区的子分区将包含在操作中。

## 内部

以下信息描述了在表导入过程中写入错误日志的内部信息和消息。

当 ALTER TABLE ... DISCARD TABLESPACE 在目标实例上运行时：

- 表被锁定在 X 模式。

- 表空间与表分离。

当 FLUSH TABLES ... FOR EXPORT 在源实例上运行时：

- 为导出而刷新的表被锁定在共享模式。

- 清除协调器线程已停止。

- 脏页同步到磁盘。

- 表元数据被写入二进制 .cfg 文件。

此操作的预期错误日志消息：

```txt
[Note] InnoDB: Sync to disk of '"test"."t1"' started.
[Note] InnoDB: Stopping purge
[Note] InnoDB: Writing table metadata to './test/t1.cfg'
[Note] InnoDB: Table '"test"."t1"' flushed to disk
```

当 UNLOCK TABLES 在源实例上运行时：

- 二进制 .cfg 文件被删除。

- 正在导入的一个或多个表上的共享锁被释放并重新启动清除协调器线程。

此操作的预期错误日志消息：

```txt
[Note] InnoDB: Deleting the meta-data file './test/t1.cfg'
[Note] InnoDB: Resuming purge
```

当 ALTER TABLE ... IMPORT TABLESPACE 在目标实例上运行时，导入算法对正在导入的每个表空间执行以下操作：

- 检查每个表空间页面是否损坏。

- 每个页面上的空间 ID 和日志序列号 (LSN) 都会更新。

- 为标题页验证标志并更新 LSN。

- Btree 页面已更新。

- 页面状态设置为脏，以便将其写入磁盘。

此操作的预期错误日志消息：

```txt
[Note] InnoDB: Importing tablespace for table 'test/t1' that was exported
from host 'host_name'
[Note] InnoDB: Phase I - Update all pages
[Note] InnoDB: Sync to disk
[Note] InnoDB: Sync to disk - done!
[Note] InnoDB: Phase III - Flush changes to disk
[Note] InnoDB: Phase IV - Flush complete
```

> 笔记
您可能还会收到一个表空间被丢弃的警告（如果您丢弃了目标表的表空间）和一条消息，指出由于缺少 .ibd 文件而无法计算统计信息：

  ```txt
  [Warning] InnoDB: Table "test"."t1" tablespace is set as discarded.
  7f34d9a37700 InnoDB: cannot calculate statistics for table
  "test"."t1" because the .ibd file is missing. For help, please refer to
  http://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting.html
  ```
