# LIMIT查询优化

如果只需要结果集中指定数量的行，请在查询中使用LIMIT子句，而不是获取整个结果集并丢弃多余的数据。

MySQL有时会优化具有LIMIT row_count子句而没有HAVING子句的查询：

- 如果使用LIMIT只选择了几行，MySQL在某些情况下会使用索引，而通常情况下它更愿意进行全表扫描。

- 如果将LIMIT *row_count* 与ORDER BY组合，MySQL将在找到排序结果的第一行row_cunt后立即停止排序，而不是对整个结果进行排序。如果使用索引进行排序，这将非常快。如果必须进行filesort，则在找到第一个row_count之前，将选择所有与查询匹配但没有LIMIT子句的行，并对其中的大部分或全部进行排序。在找到初始行之后，MySQL不会对结果集的任何剩余部分进行排序。

  这种行为的一种表现是，带有和不带LIMIT的ORDER BY查询可能会以不同的顺序返回行，如本节稍后所述。

- 如果将LIMIT *row_count* 与DISTINCT结合使用，MySQL会在找到row_cunt唯一行后立即停止。

- 在某些情况下，GROUPBY可以通过按顺序读取索引（或对索引进行排序），然后计算摘要直到索引值改变来解决。在这种情况下，LIMIT row_count不会计算任何不必要的GROUP BY值。

- 一旦MySQL向客户端发送了所需的行数，它就会中止查询，除非您使用的是 SQL_CALC_FOUND_ROWS。在这种情况下，可以使用 SELECT FOUND_ROWS() 检索行数。参见第12.16节“[信息功能](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html)”。

- LIMIT 0快速返回空集。这对于检查查询的有效性非常有用。它还可以用于在使用MySQL API的应用程序中获取结果列的类型，该API使结果集元数据可用。使用mysql客户端程序，可以使用--columntypeinfo选项显示结果列类型。

- 如果服务器使用临时表来解析查询，它将使用LIMIT row_count子句来计算需要多少空间。

- 如果索引未用于ORDER BY，但也存在LIMIT子句，则优化器可以避免使用合并文件，并使用内存中的文件排序操作对内存中的行进行排序。

如果ORDER BY列中有多行具有相同的值，则服务器可以自由地以任何顺序返回这些行，并且根据总体执行计划，返回这些行的方式可能会有所不同。换句话说，这些行的排序顺序相对于无序列是不确定的。

影响执行计划的一个因素是LIMIT，因此带有和不带LIMIT的ORDER BY查询可能会返回不同顺序的行。考虑这个查询，它按类别列排序，但与id和评级列无关：

```sql
mysql> SELECT * FROM ratings ORDER BY category;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
|  2 |        3 |    5.0 |
|  7 |        3 |    2.7 |
+----+----------+--------+
```

包括LIMIT可能会影响每个类别值中的行顺序。例如，这是一个有效的查询结果：

```sql
mysql> SELECT * FROM ratings ORDER BY category LIMIT 5;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  4 |        2 |    3.5 |
|  3 |        2 |    3.7 |
|  6 |        2 |    3.5 |
+----+----------+--------+
```

在每种情况下，行都按ORDERBY列排序，这是SQL标准所要求的全部内容。

如果重要的是确保具有和不具有LIMIT的相同行顺序，请在order BY子句中包含其他列，以使顺序具有确定性。例如，如果id值是唯一的，您可以通过如下排序使给定类别值的行按id顺序显示：

```sql
mysql> SELECT * FROM ratings ORDER BY category, id;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
|  2 |        3 |    5.0 |
|  7 |        3 |    2.7 |
+----+----------+--------+

mysql> SELECT * FROM ratings ORDER BY category, id LIMIT 5;
+----+----------+--------+
| id | category | rating |
+----+----------+--------+
|  1 |        1 |    4.5 |
|  5 |        1 |    3.2 |
|  3 |        2 |    3.7 |
|  4 |        2 |    3.5 |
|  6 |        2 |    3.5 |
+----+----------+--------+
```

对于带有ORDER BY或GROUP BY和LIMIT子句的查询，优化器会在默认情况下选择有序索引，这样做会加快查询执行速度。在MySQL 8.0.21之前，无法覆盖此行为，即使在使用其他优化可能更快的情况下也是如此。从MySQL 8.0.21开始，可以通过将optimizer_switch系统变量的[prefer_ordering_index](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_prefer-ordering-index)标志设置为关闭来关闭此优化。

示例：首先，我们创建并填充一个表t，如下所示：

```sql
# Create and populate a table t:

mysql> CREATE TABLE t (
    ->     id1 BIGINT NOT NULL,
    ->     id2 BIGINT NOT NULL,
    ->     c1 VARCHAR(50) NOT NULL,
    ->     c2 VARCHAR(50) NOT NULL,
    ->  PRIMARY KEY (id1),
    ->  INDEX i (id2, c1)
    -> );

# [Insert some rows into table t - not shown]
```

验证prefer_ordering_index标志是否已启用：

```sql
mysql> SELECT @@optimizer_switch LIKE '%prefer_ordering_index=on%';
+------------------------------------------------------+
| @@optimizer_switch LIKE '%prefer_ordering_index=on%' |
+------------------------------------------------------+
|                                                    1 |
+------------------------------------------------------+
```

由于下面的查询有一个LIMIT子句，如果可能的话，我们希望它使用有序索引。在这种情况下，正如我们从EXPLAIN输出中看到的，它使用表的主键。

```sql
mysql> EXPLAIN SELECT c2 FROM t
    ->     WHERE id2 > 3
    ->     ORDER BY id1 ASC LIMIT 2\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: index
possible_keys: i
          key: PRIMARY
      key_len: 8
          ref: NULL
         rows: 2
     filtered: 70.00
        Extra: Using where
```

现在我们禁用prefer_ordering_index标志，并重新运行相同的查询；这次它使用索引i（包括WHERE子句中使用的id2列）和filesort：

```sql
mysql> SET optimizer_switch = "prefer_ordering_index=off";

mysql> EXPLAIN SELECT c2 FROM t
    ->     WHERE id2 > 3
    ->     ORDER BY id1 ASC LIMIT 2\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: range
possible_keys: i
          key: i
      key_len: 8
          ref: NULL
         rows: 14
     filtered: 100.00
        Extra: Using index condition; Using filesort
```
