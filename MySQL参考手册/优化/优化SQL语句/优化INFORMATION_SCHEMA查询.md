# 优化INFORMATION_SCHEMA查询

监视数据库的应用程序可能经常使用INFORMATION_SCHEMA表。要最有效地编写这些表的查询，请使用以下一般准则：

尝试只查询INFORMATION_SCHEMA表，这些表是数据字典表的视图。

尝试仅查询静态元数据。选择列或使用动态元数据和静态元数据的检索条件会增加处理动态元数据的开销。

> 笔记
INFORMATION_SCHEMA查询中数据库和表名的比较行为可能与您预期的不同。有关详细信息，请参见第10.8.7节 [在INFORMATION_SCHEMA搜索中使用排序规则](https://dev.mysql.com/doc/refman/8.0/en/charset-collation-information-schema.html)。

这些INFORMATION_SCHEMA表实现为数据字典表上的视图，因此查询它们可以从数据字典中检索信息：

```txt
CHARACTER_SETS
CHECK_CONSTRAINTS
COLLATIONS
COLLATION_CHARACTER_SET_APPLICABILITY
COLUMNS
EVENTS
FILES
INNODB_COLUMNS
INNODB_DATAFILES
INNODB_FIELDS
INNODB_FOREIGN
INNODB_FOREIGN_COLS
INNODB_INDEXES
INNODB_TABLES
INNODB_TABLESPACES
INNODB_TABLESPACES_BRIEF
INNODB_TABLESTATS
KEY_COLUMN_USAGE
PARAMETERS
PARTITIONS
REFERENTIAL_CONSTRAINTS
RESOURCE_GROUPS
ROUTINES
SCHEMATA
STATISTICS
TABLES
TABLE_CONSTRAINTS
TRIGGERS
VIEWS
VIEW_ROUTINE_USAGE
VIEW_TABLE_USAGE
```

某些类型的值，甚至对于非视图INFORMATION_SCHEMA表，也是通过从数据字典中查找来检索的。这包括数据库和表名称、表类型和存储引擎等值。

某些INFORMATION_SCHEMA表格包含提供表格统计信息的列：

```txt
STATISTICS.CARDINALITY
TABLES.AUTO_INCREMENT
TABLES.AVG_ROW_LENGTH
TABLES.CHECKSUM
TABLES.CHECK_TIME
TABLES.CREATE_TIME
TABLES.DATA_FREE
TABLES.DATA_LENGTH
TABLES.INDEX_LENGTH
TABLES.MAX_DATA_LENGTH
TABLES.TABLE_ROWS
TABLES.UPDATE_TIME
```

这些列表示动态表元数据；即随着表内容的改变而改变的信息。

默认情况下，MySQL从MySQL中检索这些列的缓存值。index_stats和mysql。tablestats在查询列时对字典表进行统计，这比直接从存储引擎检索统计信息更有效。如果缓存的统计信息不可用或已过期，MySQL将从存储引擎中检索最新的统计信息并将其缓存在MySQL中。index_stats和mysql。table_stats字典表。后续查询将检索缓存的统计信息，直到缓存的统计数据过期。服务器重新启动或首次打开mysql。index_stats和mysql。tablestats表不会自动更新缓存的统计信息。

[information_schema_stats_expiry](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry)会话变量定义缓存统计信息过期之前的时间段。默认值为86400秒（24小时），但时间段可以延长至一年。

要随时更新给定表的缓存值，请使用[ANALYZE table](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)。

查询统计信息列不会在mysql中存储或更新统计信息。index_stats和mysql.table_stats字典表在以下情况下：

- 当缓存的统计信息尚未过期时。

- 当information_schema_stats_expiry设置为0时。

- 当服务器处于只读、超级只读、事务只读或innodb_read_only模式时。

- 当查询还提取性能架构数据时。

informationschemastats_expiry是一个会话变量，每个客户端会话都可以定义自己的到期值。从存储引擎检索并由一个会话缓存的统计信息可用于其他会话。

> 笔记
如果启用了innodb_read_only系统变量，ANALYZE TABLE可能会失败，因为它无法更新使用innodb的数据字典中的统计表。对于更新密钥分布的ANALYZE TABLE操作，即使该操作更新了表本身（例如，如果它是MyISAM表），也可能会失败。要获取更新的分发统计信息，请将information_schema_stats_expiry设置为0。

对于作为数据字典表视图实现的INFORMATION_SCHEMA表，底层数据字典表上的索引允许优化器构造高效的查询执行计划。要查看优化器所做的选择，请使用EXPLAIN。要查看服务器用于执行INFORMATION_SCHEMA查询的查询，请在EXPLAIN之后立即使用SHOW WARNINGS。

考虑以下语句，它标识utf8mb4字符集的排序规则：

```sql
mysql> SELECT COLLATION_NAME
       FROM INFORMATION_SCHEMA.COLLATION_CHARACTER_SET_APPLICABILITY
       WHERE CHARACTER_SET_NAME = 'utf8mb4';
+----------------------------+
| COLLATION_NAME             |
+----------------------------+
| utf8mb4_general_ci         |
| utf8mb4_bin                |
| utf8mb4_unicode_ci         |
...
```

服务器如何处理该语句？要了解详情，请使用解释：

```sql
mysql> EXPLAIN SELECT COLLATION_NAME
       FROM INFORMATION_SCHEMA.COLLATION_CHARACTER_SET_APPLICABILITY
       WHERE CHARACTER_SET_NAME = 'utf8mb4'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: cs
   partitions: NULL
         type: const
possible_keys: PRIMARY,name
          key: name
      key_len: 194
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using index
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: col
   partitions: NULL
         type: ref
possible_keys: character_set_id
          key: character_set_id
      key_len: 8
          ref: const
         rows: 68
     filtered: 100.00
        Extra: NULL
2 rows in set, 1 warning (0.01 sec)
```

要查看用于满足该语句的查询，请使用SHOW WARNINGS：

```sql
mysql> SHOW WARNINGS\G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `mysql`.`col`.`name` AS `COLLATION_NAME`
         from `mysql`.`character_sets` `cs`
         join `mysql`.`collations` `col`
         where ((`mysql`.`col`.`character_set_id` = '45')
         and ('utf8mb4' = 'utf8mb4'))
```

如SHOW WARNINGS所示，服务器将对COLLATION_CHARACTER_SET_APPLICABILITY的查询处理为对mysql系统数据库中CHARACTER_SET和排序规则数据字典表的查询。
