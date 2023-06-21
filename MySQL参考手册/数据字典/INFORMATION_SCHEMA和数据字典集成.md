# INFORMATION_SCHEMA和数据字典集成

<https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-information-schema.html>

随着数据字典的引入，以下[INFORMATION_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema.html)表被实现为数据字典表的视图：

- CHARACTER_SETS

- CHECK_CONSTRAINTS

- COLLATIONS

- COLLATION_CHARACTER_SET_APPLICABILITY

- COLUMNS

- COLUMN_STATISTICS

- EVENTS

- FILES

- INNODB_COLUMNS

- INNODB_DATAFILES

- INNODB_FIELDS

- INNODB_FOREIGN

- INNODB_FOREIGN_COLS

- INNODB_INDEXES

- INNODB_TABLES

- INNODB_TABLESPACES

- INNODB_TABLESPACES_BRIEF

- INNODB_TABLESTATS

- KEY_COLUMN_USAGE

- KEYWORDS

- PARAMETERS

- PARTITIONS

- REFERENTIAL_CONSTRAINTS

- RESOURCE_GROUPS

- ROUTINES

- SCHEMATA

- STATISTICS

- ST_GEOMETRY_COLUMNS

- ST_SPATIAL_REFERENCE_SYSTEMS

- TABLES

- TABLE_CONSTRAINTS

- TRIGGERS

- VIEWS

- VIEW_ROUTINE_USAGE

- VIEW_TABLE_USAGE

对这些表的查询现在更有效率了，因为他们从数据字典表获得信息，而不是通过其他更慢的方式。特别是对于每个作为数据字典表的视图的INFORMATION_SCHEMA表：

- 服务器不再必须为INFORMATION_SCHEMA表的每次查询创建一个临时表。

- 当底层数据字典表存储了以前通过目录扫描（例如，列举数据库名称或数据库内的表名称）或文件打开操作（例如，从.frm文件中读取信息）获得的值时，INFORMATION_SCHEMA查询这些值现在使用表查询。(另外，即使对于非视图的INFORMATION_SCHEMA表，数据库和表名等值也是通过数据字典的查找来获取的，不需要目录或文件扫描。)

- 在底层数据字典表上的索引允许优化器构建有效的查询执行计划，这在以前的实现中是不存在的，它是通过每个查询使用一个临时表来处理 INFORMATION_SCHEMA 表。

前面的改进也适用于显示对应于INFORMATION_SCHEMA表的信息的SHOW语句，这些表是数据字典表的视图。例如，SHOW DATABASES显示与SCHEMATA表相同的信息。

除了在数据字典表上引入视图之外，在STATISTICS和TABLES表中包含的表的统计数据现在被缓存，以提高INFORMATION_SCHEMA的查询性能。[information_schema_stats_expiry](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry)系统变量定义了缓存的表统计信息过期前的时间段。默认是86400秒（24小时）。如果没有缓存的统计数据或者统计数据已经过期，那么在查询表的统计列时，会从存储引擎中检索统计数据。要在任何时候更新某个表的缓存值，请使用[ANALYZE TABLE](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)。

information_schema_stats_expiry可以被设置为0，让INFORMATION_SCHEMA查询直接从存储引擎检索最新的统计数据，这不如检索缓存的统计数据快。

更多信息请参见第8.2.3节，"[优化INFORMATION_SCHEMA查询](https://dev.mysql.com/doc/refman/8.0/en/information-schema-optimization.html)"。

MySQL 8.0中的INFORMATION_SCHEMA表与数据字典紧密相连，导致了一些使用上的差异。见第14.7节，"[数据字典使用差异](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-usage-differences.html)"。
