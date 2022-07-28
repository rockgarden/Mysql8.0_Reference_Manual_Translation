# 不符合 NDB Cluster 中的 SQL 语法

与 NDB 表一起使用时，与某些 MySQL 功能相关的一些 SQL 语句会产生错误，如下面的列表中所述：

- **Temporary tables** (临时表)。不支持临时表。尝试创建使用 NDB 存储引擎的临时表或更改现有临时表以使用 NDB 失败，并出现错误表存储引擎“ndbcluster”不支持创建选项“临时”。

- **Indexes and keys in NDB tables** (NDB 表中的索引和键)。 NDB Cluster 表上的键和索引受到以下限制：

  - **Column width**。尝试在宽度大于 3072 字节的 NDB 表列上创建索引成功，但实际上只有前 3072 字节用于索引。在这种情况下，警告Specified key is too long；发出最大密钥长度为 3072 字节，并且 SHOW CREATE TABLE 语句将索引的长度显示为 3072。

  - **TEXT and BLOB columns**。您不能在使用任何 TEXT 或 BLOB 数据类型的 NDB 表列上创建索引。

  - **FULLTEXT indexes**。 NDB 存储引擎不支持 FULLTEXT 索引，这仅适用于 MyISAM 和 InnoDB 表。
  但是，您可以在 NDB 表的 VARCHAR 列上创建索引。

  - **USING HASH keys and NULL**。在唯一键和主键中使用可为空的列意味着使用这些列的查询将作为全表扫描处理。要解决此问题，请将列设为 NOT NULL，或在不使用 USING HASH 选项的情况下重新创建索引。

  - **Prefixes**。没有前缀索引；只能索引整个列。 （如本节前面所述，NDB 列索引的大小始终与列的宽度（以字节为单位）相同，最多并包括 3072 个字节。另请参阅[第 23.2.7.6 节，“NDB Cluster 中不支持或缺少的功能”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-unsupported.html)，以获取更多信息。）

  - **BIT columns**。 BIT 列不能是主键、唯一键或索引，也不能是复合主键、唯一键或索引的一部分。

  - **AUTO_INCREMENT columns**。与其他 MySQL 存储引擎一样，NDB 存储引擎每个表最多可以处理一个 AUTO_INCREMENT 列，并且必须对该列进行索引。但是，对于没有显式主键的 NDB 表，AUTO_INCREMENT 列会自动定义并用作“隐藏”主键。因此，您不能创建具有 AUTO_INCREMENT 列且没有显式主键的 NDB 表。

以下 CREATE TABLE 语句不起作用，如下所示：

```sql
# No index on AUTO_INCREMENT column; table has no primary key
# Raises ER_WRONG_AUTO_KEY
mysql> CREATE TABLE n (
    ->     a INT,
    ->     b INT AUTO_INCREMENT
    ->     )
    -> ENGINE=NDB;
ERROR 1075 (42000): Incorrect table definition; there can be only one auto
column and it must be defined as a key 

# Index on AUTO_INCREMENT column; table has no primary key
# Raises NDB error 4335
mysql> CREATE TABLE n (
    ->     a INT,
    ->     b INT AUTO_INCREMENT,
    ->     KEY k (b)
    ->     )
    -> ENGINE=NDB;
ERROR 1296 (HY000): Got error 4335 'Only one autoincrement column allowed per
table. Having a table without primary key uses an autoincr' from NDBCLUSTER
```

以下语句创建一个表，其中包含一个主键、一个 AUTO_INCREMENT 列和该列的索引，并且成功：

```sql
# Index on AUTO_INCREMENT column; table has a primary key
mysql> CREATE TABLE n (
    ->     a INT PRIMARY KEY,
    ->     b INT AUTO_INCREMENT,
    ->     KEY k (b)
    ->     )
    -> ENGINE=NDB;
Query OK, 0 rows affected (0.38 sec)
```

- **Restrictions on foreign keys**。 NDB 8.0 中对外键约束的支持与 InnoDB 提供的支持相当，但受到以下限制：

  - 作为外键引用的每一列都需要一个明确的唯一键，如果它不是表的主键。

  - 当引用是父表的主键时，不支持 ON UPDATE CASCADE。
  这是因为主键的更新是通过删除旧行（包含旧主键）加上插入新行（使用新主键）来实现的。这对 NDB 内核是不可见的，它认为这两行是相同的，因此无法知道应该级联此更新。

  - 如果子表包含任何 TEXT 或 BLOB 类型的一个或多个列，则也不支持 ON DELETE CASCADE。 （错误 #89511、错误 #27484882）

  - 不支持设置默认值。 （InnoDB 也不支持。）

  - 接受 NO ACTION 关键字，但将其视为 RESTRICT。 NO ACTION 是标准 SQL 关键字，是 MySQL 8.0 中的默认值。 （也与 InnoDB 相同。）

  - 在早期版本的 NDB Cluster 中，当使用外键引用另一个表中的索引创建表时，即使索引中列的顺序不匹配，有时似乎也可以创建外键，因为并不总是在内部返回适当的错误。此问题的部分修复改进了在大多数情况下内部使用的错误；但是，如果父索引是唯一索引，则仍有可能发生这种情况。 （错误号 18094360）

  有关详细信息，请参阅[第 13.1.20.5 节，“外键约束”](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)和[第 1.7.3.2 节，“外键约束”](https://dev.mysql.com/doc/refman/8.0/en/constraint-foreign-key.html)。

- **NDB Cluster and geometry 数据类型**。 NDB 表支持几何数据类型（WKT 和 WKB）。但是，不支持空间索引。

- **字符集和二进制日志文件**。目前，ndb_apply_status 和 ndb_binlog_index 表是使用 latin1 (ASCII) 字符集创建的。由于此表中记录了二进制日志的名称，因此这些表中未正确引用使用非拉丁字符命名的二进制日志文件。这是一个已知问题，我们正在努力解决。 （错误 #50226）

  要解决此问题，请在命名二进制日志文件或设置任何 --basedir、--log-bin 或 --log-bin-index 选项时仅使用 Latin-1 字符。

- **Creating NDB tables with user-defined partitioning**。 使用用户定义的分区创建 NDB 表。 NDB Cluster 中对用户定义的分区的支持仅限于 [LINEAR] KEY 分区。在 CREATE TABLE 语句中使用具有 ENGINE=NDB 或 ENGINE=NDBCLUSTER 的任何其他分区类型会导致错误。

  可以覆盖此限制，但在生产设置中不支持这样做。有关详细信息，请参阅[用户定义的分区和 NDB 存储引擎（NDB Cluster）](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-storage-engines.html#partitioning-limitations-ndb)。

  **Default partitioning scheme**。默认情况下，所有 NDB Cluster 表都按 KEY 分区，使用表的主键作为分区键。如果没有为表显式设置主键，则使用 NDB 存储引擎自动创建的“隐藏”主键。有关这些和相关问题的更多讨论，请参阅[第 24.2.5 节，“KEY Partitioning”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-key.html)。

  不允许使用 CREATE TABLE 和 ALTER TABLE 语句，这些语句会导致用户分区的 NDBCLUSTER 表不满足以下两个要求中的一个或两个，并失败并出现错误：

  a. 该表必须具有显式主键。

  b. 表的分区表达式中列出的所有列都必须是主键的一部分。

  **例外**。如果使用空列列表（即使用 PARTITION BY [LINEAR] KEY()）创建用户分区的 NDBCLUSTER 表，则不需要显式主键。

  **NDBCLUSTER表最大分区数**。 使用用户定义的分区时，可以为 NDBCLUSTER 表定义的最大分区数是每个节点组 8 个。 （有关 NDB Cluster 节点组的更多信息，请参阅 [NDB Cluster 节点、节点组、片段副本和分区](../NDB节点和分区.md) 。

  **不支持删除分区**。无法使用 ALTER TABLE ... DROP PARTITION 从 NDB 表中删除分区。 NDB 表支持 ALTER TABLE 的其他分区扩展 - ADD PARTITION、REORGANIZE PARTITION 和 COALESCE PARTITION，但使用复制等未优化。请参阅[第 24.3.1 节，“RANGE 和 LIST 分区的管理”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management-range-list.html)和[第 13.1.9 节，“ALTER TABLE 语句”](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html)。

  **分区选择**。 NDB 表不支持分区选择。有关详细信息，请参阅[第 24.5 节，“分区选择”](https://dev.mysql.com/doc/refman/8.0/en/partitioning-selection.html)。

- **JSON 数据类型**。 NDB 8.0 提供的 mysqld 中的 NDB 表支持 MySQL JSON 数据类型。

  NDB 表最多可以有 3 个 JSON 列。

  NDB API 没有用于处理 JSON 数据的特殊规定，它仅将其视为 BLOB 数据。将数据作为 JSON 处理必须由应用程序执行。
