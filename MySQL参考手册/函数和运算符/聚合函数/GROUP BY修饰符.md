# GROUP BY 修饰符

GROUP BY 子句允许 WITH ROLLUP 修饰符导致汇总输出包含表示更高级别（即超级聚合）汇总操作的额外行。因此，ROLLUP 使您能够通过单个查询来回答多个分析级别的问题。例如，ROLLUP 可用于为 OLAP（在线分析处理）操作提供支持。

假设一个销售表有用于记录销售盈利能力的年份、国家、产品和利润列：

```sql
CREATE TABLE sales
(
    year    INT,
    country VARCHAR(20),
    product VARCHAR(32),
    profit  INT
);
```

要每年汇总表内容，请使用如下简单的 GROUP BY：

```sql
mysql> SELECT year, SUM(profit) AS profit
       FROM sales
       GROUP BY year;
+------+--------+
| year | profit |
+------+--------+
| 2000 |   4525 |
| 2001 |   3010 |
+------+--------+
```

输出显示每年的总（总）利润。要确定所有年份的总利润，您必须自己将各个值相加或运行附加查询。或者您可以使用 ROLLUP，它通过单个查询提供两个级别的分析。将 WITH ROLLUP 修饰符添加到 GROUP BY 子句会导致查询生成另一个（超级聚合）行，该行显示所有年份值的总计：

```sql
mysql> SELECT year, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP;
+------+--------+
| year | profit |
+------+--------+
| 2000 |   4525 |
| 2001 |   3010 |
| NULL |   7535 |
+------+--------+
```

年份列中的 NULL 值标识总计超聚合行。

当有多个 GROUP BY 列时，ROLLUP 具有更复杂的效果。在这种情况下，每次除最后一个分组列之外的任何值发生变化时，查询都会生成一个额外的超聚合汇总行。

例如，如果没有 ROLLUP，基于年份、国家和产品的销售表摘要可能如下所示，其中输出仅显示分析的年份/国家/产品级别(year/country/product level)的汇总值：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2000 | India   | Calculator |    150 |
| 2000 | India   | Computer   |   1200 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2001 | Finland | Phone      |     10 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   2700 |
| 2001 | USA     | TV         |    250 |
+------+---------+------------+--------+
```

添加 ROLLUP 后，查询会产生几个额外的行：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2000 | Finland | NULL       |   1600 |
| 2000 | India   | Calculator |    150 |
| 2000 | India   | Computer   |   1200 |
| 2000 | India   | NULL       |   1350 |
| 2000 | USA     | Calculator |     75 |
| 2000 | USA     | Computer   |   1500 |
| 2000 | USA     | NULL       |   1575 |
| 2000 | NULL    | NULL       |   4525 |
| 2001 | Finland | Phone      |     10 |
| 2001 | Finland | NULL       |     10 |
| 2001 | USA     | Calculator |     50 |
| 2001 | USA     | Computer   |   2700 |
| 2001 | USA     | TV         |    250 |
| 2001 | USA     | NULL       |   3000 |
| 2001 | NULL    | NULL       |   3010 |
| NULL | NULL    | NULL       |   7535 |
+------+---------+------------+--------+
```

现在输出包括四个分析级别的摘要信息，而不仅仅是一个：

- 在给定年份和国家/地区的每组产品行之后，会出现一个额外的超级汇总摘要行，显示所有产品的总数。这些行的产品列设置为 NULL。

- 在给定年份的每组行之后，会出现一个额外的超级汇总行，显示所有国家和产品的总数。这些行的国家和产品列设置为 NULL。

- 最后，在所有其他行之后，出现一个额外的超级汇总行，显示所有年份、国家和产品的总计。此行的年份、国家和产品列设置为 NULL。

每个超聚合行中的 NULL 指示符是在将行发送到客户端时生成的。服务器查看在 GROUP BY 子句中命名的列，紧随最左边的已更改值的列。对于结果集中名称与这些名称中的任何一个匹配的任何列，其值设置为 NULL。 （如果您指定按列位置对列进行分组，则服务器会按位置确定将哪些列设置为 NULL。）

因为超聚合行中的 NULL 值是在查询处理的后期放入结果集中的，所以您只能在选择列表或 HAVING 子句中将它们作为 NULL 值进行测试。您不能在连接条件或 WHERE 子句中将它们测试为 NULL 值以确定要选择的行。例如，您不能将 WHERE product IS NULL 添加到查询中以从输出中消除除超聚合行之外的所有行。

NULL 值在客户端确实显示为 NULL，并且可以使用任何 MySQL 客户端编程接口进行测试。但是，此时您无法区分 NULL 是表示常规分组值还是超聚合值。要测试区别，请使用后面介绍的 [GROUPING()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_grouping) 函数。

以前，MySQL 不允许在具有 WITH ROLLUP 选项的查询中使用 DISTINCT 或 ORDER BY。在 MySQL 8.0.12 及更高版本中取消了此限制。

对于 GROUP BY ... WITH ROLLUP 查询，为了测试结果中的 NULL 值是否代表超聚合值，GROUPING() 函数可用于选择列表、HAVING 子句和（从 MySQL 8.0.12 开始） ORDER BY 子句。例如，当年列中的 NULL 出现在超聚合行中时，GROUPING(year) 返回 1，否则返回 0。同样，对于国家和产品列中的超聚合 NULL 值，GROUPING(country) 和 GROUPING(product) 分别返回 1：

```sql
mysql> SELECT
         year, country, product, SUM(profit) AS profit,
         GROUPING(year) AS grp_year,
         GROUPING(country) AS grp_country,
         GROUPING(product) AS grp_product
       FROM sales
       GROUP BY year, country, product WITH ROLLUP;
+------+---------+------------+--------+----------+-------------+-------------+
| year | country | product    | profit | grp_year | grp_country | grp_product |
+------+---------+------------+--------+----------+-------------+-------------+
| 2000 | Finland | Computer   |   1500 |        0 |           0 |           0 |
| 2000 | Finland | Phone      |    100 |        0 |           0 |           0 |
| 2000 | Finland | NULL       |   1600 |        0 |           0 |           1 |
| 2000 | India   | Calculator |    150 |        0 |           0 |           0 |
| 2000 | India   | Computer   |   1200 |        0 |           0 |           0 |
| 2000 | India   | NULL       |   1350 |        0 |           0 |           1 |
| 2000 | USA     | Calculator |     75 |        0 |           0 |           0 |
| 2000 | USA     | Computer   |   1500 |        0 |           0 |           0 |
| 2000 | USA     | NULL       |   1575 |        0 |           0 |           1 |
| 2000 | NULL    | NULL       |   4525 |        0 |           1 |           1 |
| 2001 | Finland | Phone      |     10 |        0 |           0 |           0 |
| 2001 | Finland | NULL       |     10 |        0 |           0 |           1 |
| 2001 | USA     | Calculator |     50 |        0 |           0 |           0 |
| 2001 | USA     | Computer   |   2700 |        0 |           0 |           0 |
| 2001 | USA     | TV         |    250 |        0 |           0 |           0 |
| 2001 | USA     | NULL       |   3000 |        0 |           0 |           1 |
| 2001 | NULL    | NULL       |   3010 |        0 |           1 |           1 |
| NULL | NULL    | NULL       |   7535 |        1 |           1 |           1 |
+------+---------+------------+--------+----------+-------------+-------------+
```

您可以使用 GROUPING() 将标签替换为超聚合 NULL 值，而不是直接显示 GROUPING() 结果：

```sql
mysql> SELECT
         IF(GROUPING(year), 'All years', year) AS year,
         IF(GROUPING(country), 'All countries', country) AS country,
         IF(GROUPING(product), 'All products', product) AS product,
         SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP;
+-----------+---------------+--------------+--------+
| year      | country       | product      | profit |
+-----------+---------------+--------------+--------+
| 2000      | Finland       | Computer     |   1500 |
| 2000      | Finland       | Phone        |    100 |
| 2000      | Finland       | All products |   1600 |
| 2000      | India         | Calculator   |    150 |
| 2000      | India         | Computer     |   1200 |
| 2000      | India         | All products |   1350 |
| 2000      | USA           | Calculator   |     75 |
| 2000      | USA           | Computer     |   1500 |
| 2000      | USA           | All products |   1575 |
| 2000      | All countries | All products |   4525 |
| 2001      | Finland       | Phone        |     10 |
| 2001      | Finland       | All products |     10 |
| 2001      | USA           | Calculator   |     50 |
| 2001      | USA           | Computer     |   2700 |
| 2001      | USA           | TV           |    250 |
| 2001      | USA           | All products |   3000 |
| 2001      | All countries | All products |   3010 |
| All years | All countries | All products |   7535 |
+-----------+---------------+--------------+--------+
```

对于多个表达式参数，GROUPING() 返回一个表示位掩码的结果，该位掩码组合了每个表达式的结果，最低位对应于最右边表达式的结果。 例如， GROUPING(year, country, product) 的评估如下：

```sql
  result for GROUPING(product)
+ result for GROUPING(country) << 1
+ result for GROUPING(year) << 2
```

如果任何表达式表示超聚合 NULL，则此类 GROUPING() 的结果为非零，因此您可以仅返回超聚合行并过滤掉常规分组行，如下所示：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP
       HAVING GROUPING(year, country, product) <> 0;
+------+---------+---------+--------+
| year | country | product | profit |
+------+---------+---------+--------+
| 2000 | Finland | NULL    |   1600 |
| 2000 | India   | NULL    |   1350 |
| 2000 | USA     | NULL    |   1575 |
| 2000 | NULL    | NULL    |   4525 |
| 2001 | Finland | NULL    |     10 |
| 2001 | USA     | NULL    |   3000 |
| 2001 | NULL    | NULL    |   3010 |
| NULL | NULL    | NULL    |   7535 |
+------+---------+---------+--------+
```

sales 表不包含 NULL 值，因此 ROLLUP 结果中的所有 NULL 值都表示超聚合值。 当数据集包含 NULL 值时，ROLLUP 摘要可能不仅在超聚合行中包含 NULL 值，而且在常规分组行中也可能包含 NULL 值。 GROUPING() 可以区分这些。 假设表 t1 包含一个简单的数据集，其中包含一组数量值的两个分组因子，其中 NULL 表示“其他”或“未知”之类的内容：

```sql
mysql> SELECT * FROM t1;
+------+-------+----------+
| name | size  | quantity |
+------+-------+----------+
| ball | small |       10 |
| ball | large |       20 |
| ball | NULL  |        5 |
| hoop | small |       15 |
| hoop | large |        5 |
| hoop | NULL  |        3 |
+------+-------+----------+
```

一个简单的 ROLLUP 操作会产生这些结果，其中很难区分超级聚合行中的 NULL 值和常规分组行中的 NULL 值：

```sql
mysql> SELECT name, size, SUM(quantity) AS quantity
       FROM t1
       GROUP BY name, size WITH ROLLUP;
+------+-------+----------+
| name | size  | quantity |
+------+-------+----------+
| ball | NULL  |        5 |
| ball | large |       20 |
| ball | small |       10 |
| ball | NULL  |       35 |
| hoop | NULL  |        3 |
| hoop | large |        5 |
| hoop | small |       15 |
| hoop | NULL  |       23 |
| NULL | NULL  |       58 |
+------+-------+----------+
```

使用 GROUPING() 将标签替换为超聚合 NULL 值使结果更易于解释：

```sql
mysql> SELECT
         IF(GROUPING(name) = 1, 'All items', name) AS name,
         IF(GROUPING(size) = 1, 'All sizes', size) AS size,
         SUM(quantity) AS quantity
       FROM t1
       GROUP BY name, size WITH ROLLUP;
+-----------+-----------+----------+
| name      | size      | quantity |
+-----------+-----------+----------+
| ball      | NULL      |        5 |
| ball      | large     |       20 |
| ball      | small     |       10 |
| ball      | All sizes |       35 |
| hoop      | NULL      |        3 |
| hoop      | large     |        5 |
| hoop      | small     |       15 |
| hoop      | All sizes |       23 |
| All items | All sizes |       58 |
+-----------+-----------+----------+
```

## 使用 ROLLUP 时的其他注意事项

下面的讨论列出了一些特定于 MySQL 实现 ROLLUP 的行为。

在 MySQL 8.0.12 之前，当您使用 ROLLUP 时，您不能同时使用 ORDER BY 子句对结果进行排序。 换句话说，ROLLUP 和 ORDER BY 在 MySQL 中是互斥的。 但是，您仍然可以控制排序顺序。 要解决阻止将 ROLLUP 与 ORDER BY 一起使用的限制并实现分组结果的特定排序顺序，请将分组结果集生成为派生表并将 ORDER BY 应用于它。 例如：

```sql
mysql> SELECT * FROM
         (SELECT year, SUM(profit) AS profit
         FROM sales GROUP BY year WITH ROLLUP) AS dt
       ORDER BY year DESC;
+------+--------+
| year | profit |
+------+--------+
| 2001 |   3010 |
| 2000 |   4525 |
| NULL |   7535 |
+------+--------+
```

从 MySQL 8.0.12 开始，ORDER BY 和 ROLLUP 可以一起使用，这使得使用 ORDER BY 和 GROUPING() 可以实现分组结果的特定排序顺序。 例如：

```sql
mysql> SELECT year, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP
       ORDER BY GROUPING(year) DESC;
+------+--------+
| year | profit |
+------+--------+
| NULL |   7535 |
| 2000 |   4525 |
| 2001 |   3010 |
+------+--------+
```

在这两种情况下，超聚合汇总行都会根据计算它们的行进行排序，并且它们的位置取决于排序顺序（升序排序在末尾，降序排序在开头）。

LIMIT 可用于限制返回给客户端的行数。 LIMIT 在 ROLLUP 之后应用，因此该限制适用于 ROLLUP 添加的额外行。 例如：

```sql
mysql> SELECT year, country, product, SUM(profit) AS profit
       FROM sales
       GROUP BY year, country, product WITH ROLLUP
       LIMIT 5;
+------+---------+------------+--------+
| year | country | product    | profit |
+------+---------+------------+--------+
| 2000 | Finland | Computer   |   1500 |
| 2000 | Finland | Phone      |    100 |
| 2000 | Finland | NULL       |   1600 |
| 2000 | India   | Calculator |    150 |
| 2000 | India   | Computer   |   1200 |
+------+---------+------------+--------+
```

将 LIMIT 与 ROLLUP 一起使用可能会产生更难以解释的结果，因为用于理解超聚合行的上下文较少。

MySQL 扩展允许在选择列表中命名未出现在 GROUP BY 列表中的列。 （有关非聚合列和 GROUP BY 的信息，请参阅第 12.20.3 节，“[MySQL 对 GROUP BY 的处理](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html)”。）在这种情况下，服务器可以从摘要行中的非聚合列中自由选择任何值，这包括额外的 WITH ROLLUP 添加的行。 例如，在以下查询中，country 是未出现在 GROUP BY 列表中的非聚合列，并且为该列选择的值是不确定的：

```sql
mysql> SELECT year, country, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP;
+------+---------+--------+
| year | country | profit |
+------+---------+--------+
| 2000 | India   |   4525 |
| 2001 | USA     |   3010 |
| NULL | USA     |   7535 |
+------+---------+--------+
```

当未启用 [ONLY_FULL_GROUP_BY](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_only_full_group_by) SQL 模式时，允许此行为。 如果启用了该模式，服务器将拒绝查询为非法，因为 country 未在 GROUP BY 子句中列出。 启用 ONLY_FULL_GROUP_BY 后，您仍然可以使用 [ANY_VALUE()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value) 函数对非确定性值列执行查询：

```sql
mysql> SELECT year, ANY_VALUE(country) AS country, SUM(profit) AS profit
       FROM sales
       GROUP BY year WITH ROLLUP;
+------+---------+--------+
| year | country | profit |
+------+---------+--------+
| 2000 | India   |   4525 |
| 2001 | USA     |   3010 |
| NULL | USA     |   7535 |
+------+---------+--------+
```

在 MySQL 8.0.28 及更高版本中，除非在 WHERE 子句中调用，否则汇总列不能用作 [MATCH()](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html#function_match) 的参数（并且会因错误而被拒绝）。 有关详细信息，请参阅第 12.10 节，“[全文搜索功能](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html)”。
