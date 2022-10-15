# 为InnoDB配置Optimizer统计信息

- 15.8.10.1 [配置Persistent Optimizer统计参数](#配置persistent-optimizer统计参数)
- 15.8.10.2 配置非持久优化器统计参数
- 15.8.10.3 估算InnoDB表的分析表复杂性

本节介绍如何为InnoDB表配置持久和非持久优化器统计信息。

持久化优化器统计信息在服务器重启期间保持不变，从而实现更大的计划稳定性和更一致的查询性能。持久化优化器统计信息还提供了控制和灵活性，并具有以下附加优势：

- 您可以使用 [innodb_stats_auto_recalc](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_auto_recalc) 配置选项来控制在对表进行重大更改后是否自动更新统计信息。

- 可以将 STATS_PERSISTENT、STATS_AUTO_RECALC 和STATS_SAMPLE_PAGES 子句与CREATE TABLE和ALTER TABLE语句一起使用，为各个表配置优化器统计信息。

- 您可以在mysql中查询优化器统计数据。innodb_table_stats和mysql.innodb_index_stats表。

- 您可以查看mysql的last_update列。innodb_table_stats和mysql.innodb_index_stats表以查看上次更新统计信息的时间。

-您可以手动修改 mysql.innodb_table_stats 和 mysql.innodb_index_stats 表强制执行特定的查询优化计划或测试替代计划，而不修改数据库。

默认情况下启用持久化优化器统计信息功能（innodb_stats_persistent=ON）。

非持久性优化器统计信息在每次服务器重新启动时和其他一些操作后清除，并在下一次表访问时重新计算。因此，在重新计算统计信息时，可能会产生不同的估计值，从而导致执行计划的不同选择和查询性能的不同。

本节还提供了有关估算ANALYZE TABLE复杂性的信息，当试图在精确统计数据和ANALYZ TABLE执行时间之间取得平衡时，这些信息可能很有用。

## 配置Persistent Optimizer统计参数

持久化优化器统计信息功能通过将统计信息存储到磁盘并使其在服务器重新启动时持久化，从而提高了[计划的稳定性](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_plan_stability)，从而使优化器更有可能每次为给定的查询做出一致的选择。

当[innodb_stats_persistent=ON](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_persistent)或当单个表定义为[stats_persistent=1](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)时，优化器统计信息将持久化到磁盘。默认情况下，[innodb_stats_presistent](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_persistent)处于启用状态。

以前，优化器统计信息在重新启动服务器时和其他一些类型的操作后被清除，并在下一次表访问时重新计算。因此，在重新计算统计数据时，可能会产生不同的估计值，从而导致查询执行计划中的不同选择和查询性能的差异。

持久统计数据存储在mysql中。innodb_table_stats和mysql。innodb_index_stats表。参见 “InnoDB持久统计表”。

如果不希望将优化器统计信息持久化到磁盘，请参阅 “配置非持久化优化器统计参数”。

### 为持久优化器统计配置自动统计计算

innodb_stats_auto_recalc变量默认启用，它控制当表的行数更改超过10%时，是否自动计算统计信息。您还可以通过在创建或更改表时指定STATS_AUTO_RECALC子句，为各个表配置自动统计信息重新计算。

由于自动统计信息重新计算的异步特性（发生在后台），在运行影响表10%以上的DML操作后，可能无法立即重新计算统计信息，即使启用了innodb_stats_auto_recalc。在某些情况下，统计重新计算可能会延迟几秒钟。如果需要立即更新统计信息，请运行ANALYZE TABLE以启动统计信息的同步（前台）重新计算。

如果禁用了innodb_stats_auto_recalc，则可以通过在对索引列进行重大更改后执行ANALYZE TABLE语句来确保优化器统计信息的准确性。您还可以考虑将ANALYZE TABLE添加到加载数据后运行的设置脚本中，并在活动较少时按计划运行ANALYZ TABLE。

当向现有表中添加索引或添加或删除列时，无论innodb_stats_auto_recalc的值是多少，都会计算索引统计信息并将其添加到innodb_index_stats表中。

### 为单个表配置Optimizer统计参数

[innodb_stats_persistent](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_persistent)、innodb_stats_auto_recalc和[innodb-stats_presistent_sample_pages](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_persistent_sample_pages)是全局变量。要覆盖这些系统范围的设置并为各个表配置优化器统计参数，可以在CREATE TABLE或ALTER TABLE语句中定义STATS_PERSISTENT、STATS_AUTO_RECALC和STATS_SAMPLE_PAGES子句。

- STATS_PERSISTENT指定是否为InnoDB表启用持久统计。值DEFAULT使表的持久统计信息设置由innodb_stats_persistent设置确定。值为1将启用表的持久统计信息，而值为0将禁用该功能。在为单个表启用持久统计信息后，请在加载表数据后使用ANALYZE table计算统计信息。

- STATS_AUTO_RECALC指定是否自动重新计算持久统计信息。值DEFAULT使表的持久统计信息设置由innodb_stats_auto_recalc设置确定。如果值为1，则当10%的表数据发生更改时，将重新计算统计信息。值为0将阻止对表进行自动重新计算。使用值0时，在对表进行重大更改后，请使用ANALYZE TABLE重新计算统计信息。

- 例如，STATS_SAMPLE_PAGES指定通过ANALYZE TABLE操作计算索引列的基数和其他统计信息时要采样的索引页数。

以下CREATE TABLE示例中指定了所有三个子句：

```sql
CREATE TABLE `t1` (
`id` int(8) NOT NULL auto_increment,
`data` varchar(255),
`date` datetime,
PRIMARY KEY  (`id`),
INDEX `DATE_IX` (`date`)
) ENGINE=InnoDB,
  STATS_PERSISTENT=1,
  STATS_AUTO_RECALC=1,
  STATS_SAMPLE_PAGES=25;
```

### 配置InnoDB Optimizer统计的采样页数

优化器使用有关键分布的估计统计信息，根据索引的相对选择性为执行计划选择索引。诸如ANALYZE TABLE之类的操作会导致InnoDB从表上的每个索引中随机抽取页面，以估计索引的基数。这种采样技术称为[随机潜水](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_random_dive)。

innodb_stats_persistent_sample_pages控制采样页面的数量。您可以在运行时调整设置，以管理优化器使用的统计估计的质量。默认值为20。遇到以下问题时，请考虑修改设置：

1. 统计信息不够准确，优化器会选择次优计划，如EXPLAIN输出所示。您可以通过比较索引的实际基数（通过在索引列上运行SELECT DISTINCT确定）和mysql中的估计值来检查统计数据的准确性。innodb_index_stats表。

    如果确定统计数据不够准确，则应增加innodb_stats_persistent_sample_pages的值，直到统计数据估计值足够准确。但是，过多地增加innodb_stats_persistent_sample_pages可能会导致ANALYZE TABLE运行缓慢。

2. 分析表太慢。在这种情况下，应减少innodb_stats_persistent_sample_pages，直到ANALYZE TABLE执行时间可以接受为止。然而，过多地减少该值可能会导致统计数据不准确和查询执行计划不理想的第一个问题。

   如果无法在精确统计数据和ANALYZE TABLE执行时间之间取得平衡，请考虑减少表中索引列的数量或限制分区的数量，以降低ANALYZ TABLE的复杂性。考虑表主键中的列数也很重要，因为主键列被附加到每个非一致索引。

   有关信息，请参阅第15.8.10.3节“[InnoDB表的分析表复杂性估算](https://dev.mysql.com/doc/refman/8.0/en/innodb-analyze-table-complexity.html)”。

### 在持久性统计计算中包括删除标记的记录

默认情况下，InnoDB在计算统计数据时读取未提交的数据。对于从表中删除行的未提交事务，在计算行估计值和索引统计信息时会排除删除标记的记录，这可能会导致使用[READ UNCOMMITTED](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted)以外的事务隔离级别在表上并发操作的其他事务的执行计划不最佳。为了避免这种情况，可以启用[innodb_stats_include_delete_marked](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_include_delete_marked)，以确保在计算持久优化器统计信息时包括删除标记的记录。

启用innodb_stats_include_delete_marked时，ANALYZE TABLE会在重新计算统计信息时考虑删除标记的记录。

innodb_stats_include_delete_marked是一个影响所有innodb表的全局设置，它仅适用于持久优化器统计。

### InnoDB持久统计表

持久统计特性依赖于mysql数据库中名为innodb_table_stats和innodb_index_stats的内部托管表。这些表是在所有安装、升级和从源代码构建过程中自动设置的。

表15.6 innodb_Table_stats列

| Column name              | Description                                                       |
|--------------------------|-------------------------------------------------------------------|
| database_name            | Database name                                                     |
| table_name               | Table name, partition name, or subpartition name                  |
| last_update              | A timestamp indicating the last time that InnoDB updated this row |
| n_rows                   | The number of rows in the table                                   |
| clustered_index_size     | The size of the primary index, in pages                           |
| sum_of_other_index_sizes | The total size of other (non-primary) indexes, in pages           |

表15.7 innodb_index_stats列

| Column name      | Description                                                                    |
|------------------|--------------------------------------------------------------------------------|
| database_name    | Database name                                                                  |
| table_name       | Table name, partition name, or subpartition name                               |
| index_name       | Index name                                                                     |
| last_update      | A timestamp indicating the last time the row was updated                       |
| stat_name        | The name of the statistic, whose value is reported in the stat_value column    |
| stat_value       | The value of the statistic that is named in stat_name column                   |
| sample_size      | The number of pages sampled for the estimate provided in the stat_value column |
| stat_description | Description of the statistic that is named in the stat_name column             |

innodb_table_stats和innodb_index_stats表包含一个last_update列，用于显示上次更新索引统计信息的时间：

```log
mysql> SELECT * FROM innodb_table_stats \G
*************************** 1. row ***************************
           database_name: sakila
              table_name: actor
             last_update: 2014-05-28 16:16:44
                  n_rows: 200
    clustered_index_size: 1
sum_of_other_index_sizes: 1
...
```

```log
mysql> SELECT * FROM innodb_index_stats \G
*************************** 1. row ***************************
   database_name: sakila
      table_name: actor
      index_name: PRIMARY
     last_update: 2014-05-28 16:16:44
       stat_name: n_diff_pfx01
      stat_value: 200
     sample_size: 1
     ...
```

innodb_table_stats和innodb_index_stats表可以手动更新，这样就可以在不修改数据库的情况下强制执行特定的查询优化计划或测试替代计划。如果手动更新统计信息，请使用FLUSH TABLE tbl_name语句加载更新的统计信息。

持久统计信息被视为本地信息，因为它们与服务器实例相关。因此，在进行自动统计重新计算时，不会复制innodb_table_stats和innodb_index_stats表。如果运行ANALYZE TABLE来启动统计信息的同步重新计算，则会复制该语句（除非您禁止对其进行日志记录），并在副本上进行重新计算。

### InnoDB持久统计表示例

innodb_table_stats表为每个表包含一行。以下示例演示了收集的数据类型。

表t1包含一个主索引（a、b列）、二级索引（c、d列）和唯一索引（e、f列）：

```sql
CREATE TABLE t1 (
a INT, b INT, c INT, d INT, e INT, f INT,
PRIMARY KEY (a, b), KEY i1 (c, d), UNIQUE KEY i2uniq (e, f)
) ENGINE=INNODB;
```

插入样本数据:

```sql
mysql> SELECT * FROM t1;
+---+---+------+------+------+------+
| a | b | c    | d    | e    | f    |
+---+---+------+------+------+------+
| 1 | 1 |   10 |   11 |  100 |  101 |
| 1 | 2 |   10 |   11 |  200 |  102 |
| 1 | 3 |   10 |   11 |  100 |  103 |
| 1 | 4 |   10 |   12 |  200 |  104 |
| 1 | 5 |   10 |   12 |  100 |  105 |
+---+---+------+------+------+------+
```

要立即更新统计信息，请运行ANALYZE TABLE（如果启用了innodb_stats_auto_recalc，统计信息将在几秒钟内自动更新，前提是已达到更改表行的10%阈值）：

```sql
mysql> ANALYZE TABLE t1;
+---------+---------+----------+----------+
| Table   | Op      | Msg_type | Msg_text |
+---------+---------+----------+----------+
| test.t1 | analyze | status   | OK       |
+---------+---------+----------+----------+
```

表t1的表统计信息显示了InnoDB上次更新表统计信息的时间（2014-03-14 14:36:34）、表中的行数（5）、聚集索引大小（1页）以及其他索引的合并大小（2页）。

```sql
mysql> SELECT * FROM mysql.innodb_table_stats WHERE table_name like 't1'\G
*************************** 1. row ***************************
           database_name: test
              table_name: t1
             last_update: 2014-03-14 14:36:34
                  n_rows: 5
    clustered_index_size: 1
sum_of_other_index_sizes: 2
```

innodb_index_stats表包含每个索引的多行。innodb_index_stats表中的每一行都提供与特定索引统计相关的数据，该统计在stat_name列中命名，在stat_description列中描述。例如：

```sql
mysql> SELECT index_name, stat_name, stat_value, stat_description
       FROM mysql.innodb_index_stats WHERE table_name like 't1';
+------------+--------------+------------+-----------------------------------+
| index_name | stat_name    | stat_value | stat_description                  |
+------------+--------------+------------+-----------------------------------+
| PRIMARY    | n_diff_pfx01 |          1 | a                                 |
| PRIMARY    | n_diff_pfx02 |          5 | a,b                               |
| PRIMARY    | n_leaf_pages |          1 | Number of leaf pages in the index |
| PRIMARY    | size         |          1 | Number of pages in the index      |
| i1         | n_diff_pfx01 |          1 | c                                 |
| i1         | n_diff_pfx02 |          2 | c,d                               |
| i1         | n_diff_pfx03 |          2 | c,d,a                             |
| i1         | n_diff_pfx04 |          5 | c,d,a,b                           |
| i1         | n_leaf_pages |          1 | Number of leaf pages in the index |
| i1         | size         |          1 | Number of pages in the index      |
| i2uniq     | n_diff_pfx01 |          2 | e                                 |
| i2uniq     | n_diff_pfx02 |          5 | e,f                               |
| i2uniq     | n_leaf_pages |          1 | Number of leaf pages in the index |
| i2uniq     | size         |          1 | Number of pages in the index      |
+------------+--------------+------------+-----------------------------------+
```

stat_name列显示以下类型的统计信息：

- size：其中stat_name=size，stat_value列显示索引中的总页数。

- n_leaf_pages：其中stat_name=n_leav_pages，stat_value列显示索引中的叶页数。

- n_diff_pfxNN：其中stat_name=n_difv_pfx01，stat_value列在索引的第一列中显示不同值的数量。其中stat_name=n_diff_pfx02，stat_value列显示索引前两列中不同值的数量，依此类推。其中stat_name=n_difv_pfxNN，stat_description列显示计数的索引列的逗号分隔列表。

为了进一步说明提供基数数据的n_diff_pfxNN统计，请再次考虑前面介绍的t1表示例。如下所示，t1表是用一个主索引（列a、b）、一个辅助索引（列c、d）和一个唯一索引（列e、f）创建的：

```sql
CREATE TABLE t1 (
  a INT, b INT, c INT, d INT, e INT, f INT,
  PRIMARY KEY (a, b), KEY i1 (c, d), UNIQUE KEY i2uniq (e, f)
) ENGINE=INNODB;
```

同上插入五行样本数据。

当查询index_name、stat_name、tat_value和stat_description时，其中stat_name LIKE'n_diff%'会返回以下结果集：

```sql
mysql> SELECT index_name, stat_name, stat_value, stat_description
       FROM mysql.innodb_index_stats
       WHERE table_name like 't1' AND stat_name LIKE 'n_diff%';
+------------+--------------+------------+------------------+
| index_name | stat_name    | stat_value | stat_description |
+------------+--------------+------------+------------------+
| PRIMARY    | n_diff_pfx01 |          1 | a                |
| PRIMARY    | n_diff_pfx02 |          5 | a,b              |
| i1         | n_diff_pfx01 |          1 | c                |
| i1         | n_diff_pfx02 |          2 | c,d              |
| i1         | n_diff_pfx03 |          2 | c,d,a            |
| i1         | n_diff_pfx04 |          5 | c,d,a,b          |
| i2uniq     | n_diff_pfx01 |          2 | e                |
| i2uniq     | n_diff_pfx02 |          5 | e,f              |
+------------+--------------+------------+------------------+
```

对于PRIMARY索引，有两行n_diff%。行数等于索引中的列数。

> 笔记
对于非一致索引，InnoDB会附加主键的列。

- 其中，index_name=PRIMARY，stat_name=n_diff_pfx01，stat_value为1，表示索引的第一列（列a）中有一个不同的值。通过查看表t1中列a中的数据，确认列a中不同值的数量，其中有一个单独的不同值（1）。计数列（a）显示在结果集的stat_description列中。

- 其中，index_name=PRIMARY，stat_name=n_diff_pfx02，stat_value为5，表示索引的两列（a，b）中有五个不同的值。通过查看表t1中列a和列b中的数据，可以确定列a和b中不同值的数量，其中有五个不同值：（1,1）、（1,2）、（1.3）、（1.4）和（1,5）。计数列（a、b）显示在结果集的stat_description列中。

对于二级索引（i1），有四行n_diff%。二级索引（c，d）只定义了两列，但二级索引有四个n_diff%行，因为InnoDB用主键为所有非一致索引添加后缀。因此，有四个n_diff%行，而不是两个，用于说明二级索引列（c，d）和主键列（a，b）。

- 其中index_name=i1，stat_name=n_diff_pfx01，stat_value为1，表示索引的第一列（c列）中有一个不同的值。通过查看表t1中列c中的数据，确认列c中不同值的数量，其中有一个单独的不同值：（10）。计数列（c）显示在结果集的stat_description列中。

- 其中index_name=i1，stat_name=n_diff_pfx02，stat_value为2，表示索引的前两列（c，d）中有两个不同的值。通过查看表t1中列c和d中的数据，可以确定列c和列d中的不同值的数量，其中有两个不同的值：（10,11）和（10,12）。计数列（c，d）显示在结果集的stat_description列中。

- 其中index_name=i1，stat_name=n_diff_pfx03，stat_value为2，表示索引的前三列（c、d、a）中有两个不同的值。通过查看表t1中列c、d和a中的数据，可以确定列c、d和a中不同值的数量，其中有两个不同值：（10,11,1）和（10,12,1）。计数列（c、d、a）显示在结果集的stat_description列中。

- 其中index_name=i1，stat_name=n_diff_pfx04，stat_value为5，表示索引的四列（c、d、a、b）中有五个不同的值。通过查看表t1中列c、d、a和b中的数据，可以确定列c、da和b的不同值的数量，其中有五个不同的值：（10,11,1,1）、（10,1,1,2）、（10-11,1,3）、（10/12,1,4）和（10,12,1,5）。计数列（c、d、a、b）显示在结果集的stat_description列中。

对于唯一索引（i2uniq），有两个n_diff%行。

- 其中，index_name=i2uniq，stat_name=n_diff_pfx01，stat_value为2，表示索引的第一列（列e）中有两个不同的值。通过查看表t1中列e中的数据，确认列e中不同值的数量，其中有两个不同值：（100）和（200）。计数列（e）显示在结果集的stat_description列中。

- 其中，index_name=i2uniq，stat_name=n_diff_pfx02，stat_value为5，表示索引的两列（e，f）中有五个不同的值。通过查看表t1中e列和f列中的数据，可以确定e列和f列中不同值的数量，其中有五个不同值：（100101）、（200102）、（100103）、（200204）和（100105）。计数列（e，f）显示在结果集的stat_description列中。

### 使用innodb_Index_stats表检索索引大小

可以使用innodb_index_stats表检索表、分区或子分区的索引大小。在下面的示例中，检索表t1的索引大小。有关表t1和相应索引统计的定义，请参阅 “InnoDB持久统计表示例”。

```sql
mysql> SELECT SUM(stat_value) pages, index_name,
       SUM(stat_value)*@@innodb_page_size size
       FROM mysql.innodb_index_stats WHERE table_name='t1'
       AND stat_name = 'size' GROUP BY index_name;
+-------+------------+-------+
| pages | index_name | size  |
+-------+------------+-------+
|     1 | PRIMARY    | 16384 |
|     1 | i1         | 16384 |
|     1 | i2uniq     | 16384 |
+-------+------------+-------+
```

对于分区或子分区，可以使用同一查询和修改后的WHERE子句来检索索引大小。例如，以下查询检索表t1的分区的索引大小：

```sql
mysql> SELECT SUM(stat_value) pages, index_name,
       SUM(stat_value)*@@innodb_page_size size
       FROM mysql.innodb_index_stats WHERE table_name like 't1#P%'
       AND stat_name = 'size' GROUP BY index_name;
```
