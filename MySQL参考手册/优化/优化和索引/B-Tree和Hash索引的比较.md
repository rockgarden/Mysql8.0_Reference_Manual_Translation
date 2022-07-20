# B-Tree和Hash索引的比较

了解 B 树和散列数据结构有助于预测在索引中使用这些数据结构的不同存储引擎上的不同查询如何执行，特别是对于允许您选择 B 树或散列索引的 MEMORY 存储引擎。

## B-Tree 索引特征

B 树索引可用于使用 =、>、>=、<、<= 或 BETWEEN 运算符的表达式中的列比较。 如果 LIKE 的参数是 **不** 以通配符开头的常量字符串，则索引也可用于 LIKE 比较。 例如，以下 SELECT 语句使用索引：

```sql
SELECT * FROM tbl_name WHERE key_col LIKE 'Patrick%';
SELECT * FROM tbl_name WHERE key_col LIKE 'Pat%_ck%';
```

在第一条语句中，仅考虑 'Patrick' <= key_col < 'Patricl' 的行。 在第二个语句中，仅考虑 'Pat' <= key_col < 'Pau' 的行。

以下 SELECT 语句不使用索引：

```sql
SELECT * FROM tbl_name WHERE key_col LIKE '%Patrick%';
SELECT * FROM tbl_name WHERE key_col LIKE other_col;
```

在第一个语句中，LIKE 值以通配符开头。 在第二个语句中，LIKE 值不是一个常数。

如果你使用 ... LIKE '%string%' 并且字符串长度超过三个字符，MySQL 使用 Turbo Boyer-Moore 算法来初始化字符串的模式，然后使用这个模式更快地执行搜索。

如果 col_name 被索引，则使用 col_name IS NULL 的搜索会使用索引。

任何不跨越 WHERE 子句中所有 AND 级别的索引都不会用于优化查询。 (Any index that does not span all AND levels in the WHERE clause is not used to optimize the query.) 换句话说，为了能够使用索引，必须在每个 AND 组中使用索引的前缀。

以下 WHERE 子句使用索引：

```sql
... WHERE index_part1=1 AND index_part2=2 AND other_column=3

    /* index = 1 OR index = 2 */
... WHERE index=1 OR A=10 AND index=2

    /* optimized like "index_part1='hello'" */
... WHERE index_part1='hello' AND index_part3=5

    /* Can use index on index1 but not on index2 or index3 */
... WHERE index1=1 AND index2=2 OR index1=3 AND index3=3;
```

有时 MySQL 不使用索引，即使索引可用。发生这种情况的一种情况是优化器估计使用索引将需要 MySQL 访问表中很大比例的行。 （在这种情况下，表扫描可能会快得多，因为它需要更少的查找。）但是，如果这样的查询使用 LIMIT 只检索一些行，那么 MySQL 无论如何都会使用索引，因为它可以更快地找到在结果中返回的几行。

## 哈希索引特征

哈希索引与刚才讨论的有些不同：

- 它们仅用于使用 = 或 <=> 运算符的相等比较（但速度非常快）。它们不用于查找值范围的比较运算符，例如 <。依赖这种类型的单值查找的系统被称为“键值存储”；要将 MySQL 用于此类应用程序，请尽可能使用哈希索引。
- 优化器不能使用哈希索引来加速 ORDER BY 操作。 （这种类型的索引不能用于按顺序搜索下一个条目。）
- MySQL 无法确定两个值之间大约有多少行（范围优化器使用它来决定使用哪个索引）。如果您将 MyISAM 或 InnoDB 表更改为哈希索引的 MEMORY 表，这可能会影响某些查询。
- 只能使用整个键来搜索行。 （使用 B 树索引，键的任何最左边的前缀都可用于查找行。）
