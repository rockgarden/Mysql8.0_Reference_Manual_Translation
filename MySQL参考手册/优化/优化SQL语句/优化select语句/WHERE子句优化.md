# WHERE 子句优化

本节讨论可用于处理 WHERE 子句的优化。这些示例使用 SELECT 语句，但相同的优化适用于 DELETE 和 UPDATE 语句中的 WHERE 子句。

您可能很想重写查询以使算术运算更快，同时牺牲可读性。因为 MySQL 会自动进行类似的优化，所以您通常可以避免这项工作，并将查询保留在更易于理解和维护的形式中。 MySQL执行的一些优化如下：

- 删除不必要的括号：

  `((a AND b) AND c OR (((a AND b) AND (c AND d)))) -> (a AND b AND c) OR (a AND b AND c AND d)`

- 恒定折叠 Constant folding ：

  `(a<b AND b=c) AND a=5 -> b>5 AND b=c AND a=5`

- 恒定条件去除：

   `(b>=5 AND b=5) OR (b=6 AND 5=5) OR (b=7 AND 5=6) -> b=5 或 b=6`

在 MySQL 8.0.14 及更高版本中，这发生在准备阶段而不是优化阶段，这有助于简化连接。有关更多信息和示例，请参阅第 8.2.1.9 节，“[外部连接优化](https://dev.mysql.com/doc/refman/8.0/en/outer-join-optimization.html)”。

- 索引使用的常量表达式只计算一次。

- 从 MySQL 8.0.16 开始，对数值类型的列与常量值的比较进行检查并折叠或删除无效或超出范围的值：

  ```sql
  # CREATE TABLE t (c TINYINT UNSIGNED NOT NULL);
  SELECT * FROM t WHERE c ≪ 256;
  -≫ SELECT * FROM t WHERE 1;
  ```

  有关详细信息，请参阅第 8.2.1.14 节，“[恒定折叠优化](https://dev.mysql.com/doc/refman/8.0/en/constant-folding-optimization.html)”。

- 没有 WHERE 的单个表上的 COUNT(*) 直接从 MyISAM 和 MEMORY 表的表信息中检索。当仅与一个表一起使用时，这也适用于任何 NOT NULL 表达式。

- 早期检测无效的常量表达式。 MySQL 快速检测到某些 SELECT 语句是不可能的并且不返回任何行。

- 如果您不使用 GROUP BY 或聚合函数（COUNT()、MIN() 等），HAVING 将与 WHERE 合并。

- 对于连接中的每个表，构造一个更简单的 WHERE 以获得对表的快速 WHERE 评估，并尽快跳过行。

- 在查询中的任何其他表之前首先读取所有常量表。常量表是以下任何一种：

  - 空表或只有一行的表。

  - 与 PRIMARY KEY 或 UNIQUE 索引上的 WHERE 子句一起使用的表，其中所有索引部分都与常量表达式进行比较并定义为 NOT NULL。

  以下所有表都用作常量表：
  
  ```sql
  SELECT * FROM t WHERE primary_key=1;
  SELECT * FROM t1,t2
  WHERE t1.primary_key=1 AND t2.primary_key=t1.id;
  ```

- 通过尝试所有可能性来找到用于连接表的最佳连接组合。如果 ORDER BY 和 GROUP BY 子句中的所有列都来自同一个表，则在加入时首先首选该表。

- 如果存在 ORDER BY 子句和不同的 GROUP BY 子句，或者如果 ORDER BY 或 GROUP BY 包含来自连接队列中第一个表以外的表的列，则会创建一个临时表。

- 如果使用 SQL_SMALL_RESULT 修饰符，MySQL 使用内存中的临时表。

- 查询每个表索引，并使用最佳索引，除非优化器认为使用表扫描更有效。曾经，根据最佳索引是否跨越超过 30% 的表来使用扫描，但固定百分比不再决定使用索引还是扫描之间的选择。优化器现在更加复杂，它的估计基于其他因素，例如表大小、行数和 I/O 块大小。

- 在某些情况下，MySQL 甚至可以在不查阅数据文件的情况下从索引中读取行。如果索引中使用的所有列都是数字，则仅使用索引树来解析查询。

- 在输出每一行之前，将跳过那些与 HAVING 子句不匹配的行。

一些非常快的查询示例：

```sql
SELECT COUNT(*) FROM tbl_name;

SELECT MIN(key_part1),MAX(key_part1) FROM tbl_name;

SELECT MAX(key_part2) FROM tbl_name
  WHERE key_part1=constant;

SELECT ... FROM tbl_name
  ORDER BY key_part1,key_part2,... LIMIT 10;

SELECT ... FROM tbl_name
  ORDER BY key_part1 DESC, key_part2 DESC, ... LIMIT 10;
```

MySQL 仅使用索引树解析以下查询，假设索引列是数字：

```sql
SELECT key_part1,key_part2 FROM tbl_name WHERE key_part1=val;

SELECT COUNT(*) FROM tbl_name
  WHERE key_part1=val1 AND key_part2=val2;

SELECT MAX(key_part2) FROM tbl_name GROUP BY key_part1;
```

以下查询使用索引以排序顺序检索行，而无需单独的排序传递：

```sql
SELECT ... FROM tbl_name
  ORDER BY key_part1,key_part2,... ;

SELECT ... FROM tbl_name
  ORDER BY key_part1 DESC, key_part2 DESC, ... ;
```
