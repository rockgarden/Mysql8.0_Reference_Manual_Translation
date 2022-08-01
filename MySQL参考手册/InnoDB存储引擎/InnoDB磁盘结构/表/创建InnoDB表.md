# 创建InnoDB表

<https://dev.mysql.com/doc/refman/8.0/en/using-innodb-tables.html>

你可以使用 `CREATE TABLE`语句来创建一张InnoDB表。

> CREATE TABLE t1 (a INT, b CHAR (20), PRIMARY KEY (a)) ENGINE=InnoDB;

当 InnoDB 被定义为默认存储引擎时，不需要 ENGINE=InnoDB 子句，这是默认的。 但是，如果要在默认存储引擎不是 InnoDB 或未知的不同 MySQL 服务器实例上重播 CREATE TABLE 语句，则 ENGINE 子句很有用。 您可以通过发出以下语句来确定 MySQL 服务器实例上的默认存储引擎：

```sql
mysql> SELECT @@default_storage_engine;
+--------------------------+
| @@default_storage_engine |
+--------------------------+
| InnoDB                   |
+--------------------------+
```

如果你打算在一个默认存储引擎不是InnoDB的服务器上使用`mysqldump`或者主从同步来又一次执行`CREATE TABLE`语句的话，你可能仍需要使用`ENGINE=InnoDB`指定存储引擎。

默认情况下，InnoDB 表是在 file-per-table 表空间中创建的。 要在 InnoDB 系统表空间中创建 InnoDB 表，请在创建表之前禁用 `innodb_file_per_table` 变量。 要在通用表空间中创建 InnoDB 表，请使用 [CREATE TABLE ... TABLESPACE](https://dev.mysql.com/doc/refman/8.0/en/create-table.html) 语法。 有关更多信息，请参阅[第 15.6.3 节，“表空间”](https://dev.mysql.com/doc/refman/8.0/en/innodb-tablespace.html)。

当你在单文件表空间创建一张表时，MySQL在数据目录中创建一个.ibd表空间文件。一般的，一张在InnoDB系统空间创建的表是存在已有的ibdata file,默认的数据在Mysql的data目录下。一张在通用表空间创建的表位已有的通用表空间文件.ibd文件。通用表空间的文件可以在MySQL数据目录之内或之外创建。更多信息请查看`第15.6.3.3节 通用表空间`。

内部实现来说，InnoDB为每一张表的实体添加进数据字典。此实体包含数据库名。例如，如果t1表是创建在test数据库上，数据库名的数句字典实体名称是'test/t1'。这意味着你可以在不同的数据库中创建相同名字的表，并且表明不会冲突。

## 行格式

InnoDB 表的行格式决定了它的行在磁盘上的物理存储方式。 InnoDB 支持四种行格式，每种都有不同的存储特性。 支持的行格式包括 REDUNDANT、COMPACT、DYNAMIC 和 COMPRESSED。 DYNAMIC 行格式是默认值。 有关行格式特征的信息，请参阅[第 15.10 节，“InnoDB 行格式”](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html)。

[innodb_default_row_format](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_default_row_format) 变量定义了默认的行格式，它的默认值是 DYNAMIC。 还可以使用 CREATE TABLE 或 ALTER TABLE 语句中的 ROW_FORMAT 表选项显式定义表的行格式。 请参阅[定义表的行格式](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html#innodb-row-format-defining)。

DYNAMIC 和 COMPRESSED 的行格式允许你充分利用InnoDB表的优势，例如表压缩和高效的跨页超长列值支持。为了能使用这个行格式，配置`innodb_file_per_table`必须是打开的(默认是打开的)。

```sql
SET GLOBAL innodb_file_per_table=1;
CREATE TABLE t3 (a INT, b CHAR (20), PRIMARY KEY (a)) ROW_FORMAT=DYNAMIC;
CREATE TABLE t4 (a INT, b CHAR (20), PRIMARY KEY (a)) ROW_FORMAT=COMPRESSED;
```

```sql
CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1 ROW_FORMAT=DYNAMIC;
```

`CREATE TABLE ... TABLESPACE` 语法也能用于在系统表空间创建动态行格式的InnoDB表，以及拥有紧凑或者冗余行格式的表。

```sql
CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE = innodb_system ROW_FORMAT=DYNAMIC;
```

## 主键

建议您为创建的每个表定义一个主键。 选择主键列时，选择具有以下特征的列：

- 最重要的查询引用的列。

- 永远不会留空的列。

- 从不具有重复值的列。

- 插入后很少更改值的列。

例如，在包含人员信息的表中，您不会在 (firstname, lastname) 上创建主键，因为可以有多个人具有相同的姓名，姓名列可能留空，有时人们会更改姓名。 由于有如此多的约束，通常没有一组明显的列可用作主键，因此您创建一个具有数字 ID 的新列作为主键的全部或部分。 您可以声明一个自动增量列，以便在插入行时自动填充升序值：

```sql
# The value of ID can act like a pointer between related items in different tables.
CREATE TABLE t5 (id INT AUTO_INCREMENT, b CHAR (20), PRIMARY KEY (id));

# The primary key can consist of more than one column. Any autoinc column must come first.
CREATE TABLE t6 (id INT AUTO_INCREMENT, a INT, b CHAR (20), PRIMARY KEY (id,a));
```

有关自动增量列的更多信息，请参阅[第 15.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”](https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html)。

尽管表可以在不定义主键的情况下正常工作，但主键涉及性能的许多方面，并且对于任何大型或经常使用的表来说都是至关重要的设计方面。 建议您始终在 CREATE TABLE 语句中指定主键。 如果创建表，加载数据，然后运行 ALTER TABLE 添加主键，则该操作比创建表时定义主键要慢得多。 有关主键的更多信息，请参阅[第 15.6.2.1 节，“聚集索引和二级索引”](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)。

## 查看InnoDB表的属性

想要查看InnoDB表的属性，可以使用`SHOW TABLE STATUS`语句：

```sql
mysql> SHOW TABLE STATUS FROM test LIKE 't%' \G;
*************************** 1. row ***************************
           Name: t1
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2021-02-18 12:18:28
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: 
        Comment:
```

更多关于`SHOW TABLE STATUS`语句的输出，请查看[第13.7.6.36节 SHOW TABLE STATUS语法](https://dev.mysql.com/doc/refman/8.0/en/show-table-status.html)。

InnoDB表属性也可以从 InnoDB Information Schema系统表查询得到：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME='test/t1' \G
*************************** 1. row ***************************
     TABLE_ID: 1144
         NAME: test/t1
         FLAG: 33
       N_COLS: 5
        SPACE: 30
   ROW_FORMAT: Dynamic
ZIP_PAGE_SIZE: 0
   SPACE_TYPE: Single
 INSTANT_COLS: 0
```

更多信息请查阅[15.14.3节 InnoDB INFORMATION_SCHEMA 元信息表](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-system-tables.html)。
