# Utility 语句

## EXPLAIN 语句

```txt
{EXPLAIN | DESCRIBE | DESC}
    tbl_name [col_name | wild]

{EXPLAIN | DESCRIBE | DESC}
    [explain_type]
    {explainable_stmt | FOR CONNECTION connection_id}

{EXPLAIN | DESCRIBE | DESC} ANALYZE [FORMAT = TREE] select_statement

explain_type: {
    FORMAT = format_name
}

format_name: {
    TRADITIONAL
  | JSON
  | TREE
}

explainable_stmt: {
    SELECT statement
  | TABLE statement
  | DELETE statement
  | INSERT statement
  | REPLACE statement
  | UPDATE statement
}
```

DESCRIBE 和 EXPLAIN 语句是同义词。在实践中，DESCRIBE 关键字更常用于获取有关表结构的信息，而 EXPLAIN 则用于获取查询执行计划（即 MySQL 如何执行查询的解释）。

下面的讨论根据这些用法使用 DESCRIBE 和 EXPLAIN 关键字，但 MySQL 解析器将它们视为完全同义词。

### 获取表结构信息

[DESCRIBE](https://dev.mysql.com/doc/refman/8.0/en/describe.html)提供有关表中列的信息：

```log
mysql> DESCRIBE City;
+------------+----------+------+-----+---------+----------------+
| Field      | Type     | Null | Key | Default | Extra          |
+------------+----------+------+-----+---------+----------------+
| Id         | int(11)  | NO   | PRI | NULL    | auto_increment |
| Name       | char(35) | NO   |     |         |                |
| Country    | char(3)  | NO   | UNI |         |                |
| District   | char(20) | YES  | MUL |         |                |
| Population | int(11)  | NO   |     | 0       |                |
+------------+----------+------+-----+---------+----------------+
```

描述是显示列的快捷方式。这些语句还显示视图的信息。[SHOW COLUMNS](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html)的说明提供了有关输出列的更多信息。参见第 13.7.7.5 节，“[显示列报表](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html)”。

默认情况下，DESCRIBE 显示表中所有列的信息。col_name（如果给定）是表中列的名称。在这种情况下，语句仅显示指定列的信息。wild（如果给定）是一个模式字符串。它可以包含 SQL%和\_wildcard(通配符)。在这种情况下，语句仅显示名称与字符串匹配的列的输出。除非字符串包含空格或其他特殊字符，否则无需将字符串括在引号内。

提供 DESCRIBE 语句是为了与 Oracle 兼容。

SHOW CREATE TABLE、SHOW TABLE STATUS 和 SHOW INDEX 语句还提供有关表的信息。参见第 13.7.7 节“[SHOW 语句](https://dev.mysql.com/doc/refman/8.0/en/show.html)”。

### 获取执行计划信息

EXPLAIN语句提供有关MySQL如何执行语句的信息：

- EXPLAIN适用于SELECT、DELETE、INSERT、REPLACE和UPDATE语句。在MySQL 8.0.19及更高版本中，它也可以使用TABLE语句。

- 当EXPLAIN与可解释语句一起使用时，MySQL会显示来自优化器的关于语句执行计划的信息。也就是说，MySQL解释了它将如何处理该语句，包括有关表如何连接以及以何种顺序连接的信息。有关使用EXPLAIN获取执行计划信息的信息。

- 当EXPLAIN与FOR CONNECTION_id而不是可解释语句一起使用时，它会显示在指定连接中执行的语句的执行计划。参见第8.8.4节“[获取命名连接的执行计划信息](https://dev.mysql.com/doc/refman/8.0/en/explain-for-connection.html)”。

- 对于可解释的语句，EXPLAIN生成额外的执行计划信息，可以使用SHOW WARNINGS显示这些信息。

- EXPLAIN对于检查涉及分区表的查询非常有用。参见第24.3.5节，“[获取分区信息](https://dev.mysql.com/doc/refman/8.0/en/partitioning-info.html)”。

- FORMAT选项可用于选择输出格式。TRADITIONAL以表格格式显示输出。如果没有FORMAT选项，这是默认值。JSON格式以JSON格式显示信息。在MySQL 8.0.16及更高版本中，与传统格式相比，TREE提供了更精确的查询处理描述的树状输出；它是唯一显示哈希连接用法的格式，并且总是用于EXPLAIN ANALYZE。

EXPLAIN需要执行解释语句所需的相同权限。此外，EXPLAIN还要求任何解释视图都具有SHOW VIEW权限。如果指定的连接属于其他用户，则EXPLAIN…FOR CONNECTION还需要PROCESS特权。

在EXPLAIN的帮助下，您可以看到应该在何处向表中添加索引，以便通过使用索引查找行来加快语句的执行速度。您还可以使用EXPLAIN检查优化器是否以最佳顺序连接表。要提示优化器使用与SELECT语句中表的命名顺序相对应的连接顺序，请以 SELECT STRAIGHT_JOIN 而不是SELECT开始语句。

优化器跟踪有时可能提供与EXPLAIN互补的信息。但是，优化器跟踪格式和内容会在不同版本之间发生变化。有关详细信息，请参阅 [跟踪优化器](../../MySQL内部手册/跟踪优化器.md)。

如果在您认为应该使用索引的情况下没有使用索引，则运行ANALYZE TABLE来更新表统计信息，例如键的基数，这可能会影响优化器的选择。参见第13.7.3.1节，“[分析表声明](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)”。

> 笔记
MySQL Workbench具有可视化解释功能，可提供Explain输出的可视化表示。请参阅教程：[使用解释提高查询性能](https://dev.mysql.com/doc/workbench/en/wb-tutorial-visual-explain-dbt3.html)。

### 使用EXPLAIN ANALYZE获取信息

MySQL 8.0.18引入了EXPLAIN ANALYZE，它运行一条语句并生成EXPLAIN输出，以及基于迭代器的定时和其他信息，这些信息是关于优化器的期望如何与实际执行匹配的。对于每个迭代器，提供以下信息：

- 预计执行成本

  （一些迭代器未计入成本模型，因此未包含在估算中。）

- 估计返回的行数

- 返回第一行的时间

- 执行此迭代器（包括子迭代器，但不包括父迭代器）所花费的时间（毫秒）。

  （当有多个循环时，此图显示每个循环的平均时间。）

- 迭代器返回的行数

- 循环数

查询执行信息使用TREE输出格式显示，其中节点表示迭代器。EXPLAIN ANALYZE始终使用TREE输出格式。在MySQL 8.0.21及更高版本中，可以选择使用FORMAT=TREE显式指定该值；TREE以外的格式仍然不受支持。

EXPLAIN ANALYZE可以与SELECT语句以及多表UPDATE和DELETE语句一起使用。从MySQL 8.0.19开始，它也可以与TABLE语句一起使用。

从MySQL 8.0.20开始，可以使用[KILL QUERY](https://dev.mysql.com/doc/refman/8.0/en/kill.html)或CTRL-C终止此语句。

EXPLAIN ANALYZE不能与FOR CONNECTION一起使用。

输出示例：

```log
mysql> EXPLAIN ANALYZE SELECT * FROM t1 JOIN t2 ON (t1.c1 = t2.c2)\G
*************************** 1. row ***************************
EXPLAIN: -> Inner hash join (t2.c2 = t1.c1)  (cost=4.70 rows=6)
(actual time=0.032..0.035 rows=6 loops=1)
    -> Table scan on t2  (cost=0.06 rows=6)
(actual time=0.003..0.005 rows=6 loops=1)
    -> Hash
        -> Table scan on t1  (cost=0.85 rows=6)
(actual time=0.018..0.022 rows=6 loops=1)

mysql> EXPLAIN ANALYZE SELECT * FROM t3 WHERE i > 8\G
*************************** 1. row ***************************
EXPLAIN: -> Filter: (t3.i > 8)  (cost=1.75 rows=5)
(actual time=0.019..0.021 rows=6 loops=1)
    -> Table scan on t3  (cost=1.75 rows=15)
(actual time=0.017..0.019 rows=15 loops=1)

mysql> EXPLAIN ANALYZE SELECT * FROM t3 WHERE pk > 17\G
*************************** 1. row ***************************
EXPLAIN: -> Filter: (t3.pk > 17)  (cost=1.26 rows=5)
(actual time=0.013..0.016 rows=5 loops=1)
    -> Index range scan on t3 using PRIMARY  (cost=1.26 rows=5)
(actual time=0.012..0.014 rows=5 loops=1)
```

示例输出中使用的表由以下语句创建：

```sql
CREATE TABLE t1 (
    c1 INTEGER DEFAULT NULL,
    c2 INTEGER DEFAULT NULL
);

CREATE TABLE t2 (
    c1 INTEGER DEFAULT NULL,
    c2 INTEGER DEFAULT NULL
);

CREATE TABLE t3 (
    pk INTEGER NOT NULL PRIMARY KEY,
    i INTEGER DEFAULT NULL
);
```

此语句输出中显示的实际时间值以毫秒为单位。
