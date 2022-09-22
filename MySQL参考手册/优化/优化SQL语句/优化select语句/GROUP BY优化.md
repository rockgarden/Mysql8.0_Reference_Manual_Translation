# GROUP BY 优化

满足 GROUP BY 子句的最通用方法是扫描整个表并创建一个新的临时表，其中每个组的所有行都是连续的，然后使用此临时表来发现组并应用聚合函数（如果有）。在某些情况下，MySQL 能够做得比这更好，并通过使用索引访问来避免创建临时表。

为 GROUP BY 使用索引的最重要的先决条件是所有 GROUP BY 列都引用来自同一索引的属性，并且索引按顺序存储其键（例如，对于 BTREE 索引是如此，但对于 HASH 则不是指数）。临时表的使用是否可以被索引访问替代还取决于查询中使用了索引的哪些部分、为这些部分指定的条件以及所选的聚合函数。

有两种方法可以通过索引访问来执行 GROUP BY 查询，如以下部分所述。第一种方法将分组操作与所有范围谓词（如果有）一起应用。第二种方法首先执行范围扫描，然后对结果元组进行分组。

- 松散索引扫描 Loose Index Scan

- 紧密索引扫描

在某些情况下，也可以在没有 GROUP BY 的情况下使用松散索引扫描。请参阅[跳过扫描范围访问方法](https://dev.mysql.com/doc/refman/8.0/en/range-optimization.html#range-access-skip-scan)。

## 松散索引扫描

处理 GROUP BY 的最有效方法是使用索引直接检索分组列。通过这种访问方法，MySQL 使用了一些索引类型的属性，即键是有序的（例如，BTREE）。此属性允许在索引中使用查找组，而不必考虑索引中满足所有 WHERE 条件的所有键。这种访问方法只考虑索引中的一小部分键，因此称为松散索引扫描。当没有 WHERE 子句时，Loose Index Scan 读取与组数一样多的键，这可能比所有键的数量小得多。如果 WHERE 子句包含范围谓词（请参阅 [使用 EXPLAIN 优化查询](../../了解查询执行计划/使用EXPLAIN优化查询.md) 中对范围连接类型的讨论），松散索引扫描会查找满足范围条件的每个组的第一个键，然后再次读取尽可能少的键数。这在以下条件下是可能的：

- 查询针对单个表。

- GROUP BY **只命名构成索引最左前缀的列**，而不命名其他列。 （如果查询有一个 DISTINCT 子句而不是 GROUP BY，则所有不同的属性都引用构成索引最左前缀的列。）例如，如果表 t1 在 (c1,c2,c3) 上有一个索引，如果查询具有 GROUP BY c1、c2，则适用松散索引扫描。如果查询有 GROUP BY c2、c3（列不是最左边的前缀）或 GROUP BY c1、c2、c4（c4 不在索引中），则不适用。

- 选择列表（如果有）中使用的唯一聚合函数是 MIN() 和 MAX()，它们都引用同一列。该列必须在索引中，并且必须紧跟在 GROUP BY 中的列之后。

- 除了 MIN() 或 MAX() 函数的参数之外，查询中引用的 GROUP BY 之外的索引的任何其他部分都必须是常量（即，它们必须以与常量相等的方式引用）。

- 对于索引中的列，必须索引完整的列值，而不仅仅是前缀。例如，对于 c1 VARCHAR(20)、INDEX (c1(10))，索引仅使用 c1 值的前缀，不能用于松散索引扫描。

如果 Loose Index Scan 适用于查询，则 EXPLAIN 输出在 Extra 列中显示 Using index for group-by。

假设表 t1(c1,c2,c3,c4) 上有一个索引 idx(c1,c2,c3)。松散索引扫描访问方法可用于以下查询：

```sql
SELECT c1, c2 FROM t1 GROUP BY c1, c2;
SELECT DISTINCT c1, c2 FROM t1;
SELECT c1, MIN(c2) FROM t1 GROUP BY c1;
SELECT c1, c2 FROM t1 WHERE c1 < const GROUP BY c1, c2;
SELECT MAX(c3), MIN(c3), c1, c2 FROM t1 WHERE c2 > const GROUP BY c1, c2;
SELECT c2 FROM t1 WHERE c1 < const GROUP BY c1, c2;
SELECT c1, c2 FROM t1 WHERE c3 = const GROUP BY c1, c2;
```

由于给出的原因，无法使用此快速选择方法执行以下查询：

- 除了 MIN() 或 MAX() 之外，还有聚合函数：
  `SELECT c1, SUM(c2) FROM t1 GROUP BY c1;`

- GROUP BY 子句中的列不构成索引的最左前缀：
  `SELECT c1, c2 FROM t1 GROUP BY c2, c3;`

- 查询引用 GROUP BY 部分之后的键的一部分，并且不存在与常量相等的部分：
  `SELECT c1, c3 FROM t1 GROUP BY c1, c2;`
  如果查询包含 WHERE c3 = const，则可以使用松散索引扫描。

除了已经支持的 MIN() 和 MAX() 引用之外，Loose Index Scan 访问方法还可以应用于选择列表中其他形式的聚合函数引用：

- 支持 AVG(DISTINCT)、SUM(DISTINCT) 和 COUNT(DISTINCT)。 AVG(DISTINCT) 和 SUM(DISTINCT) 采用单个参数。 COUNT(DISTINCT) 可以有多个列参数。参见 <https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html>

- 查询中不能有 GROUP BY 或 DISTINCT 子句。

- 前面描述的松散索引扫描限制仍然适用。

假设表 t1(c1,c2,c3,c4) 上有一个索引 idx(c1,c2,c3)。松散索引扫描访问方法可用于以下查询：

`SELECT COUNT(DISTINCT c1), SUM(DISTINCT c1) FROM t1;`

`SELECT COUNT(DISTINCT c1, c2), COUNT(DISTINCT c2, c1) FROM t1;`

## 紧密索引扫描

Tight Index Scan 可以是全索引扫描，也可以是范围索引扫描，具体取决于查询条件。

如果不满足松散索引扫描的条件，仍然可以避免为 GROUP BY 查询创建临时表。如果 WHERE 子句中有范围条件，则该方法只读取满足这些条件的键。否则，它将执行索引扫描。因为这种方法会读取 WHERE 子句定义的每个范围内的所有键，或者如果没有范围条件则扫描整个索引，因此称为紧密索引扫描。使用紧密索引扫描，只有在找到所有满足范围条件的键之后才执行分组操作。

要使此方法起作用，查询中的所有列有一个恒定的相等条件就足够了，该条件引用位于 GROUP BY 键部分之前或之间的部分键。来自相等条件的常量填充搜索键中的任何“空白”，以便可以形成索引的完整前缀。然后这些索引前缀可用于索引查找。如果 GROUP BY 结果需要排序，并且可以形成作为索引前缀的搜索键，MySQL 也避免了额外的排序操作，因为在有序索引中使用前缀搜索已经按顺序检索了所有键。

假设表 t1(c1,c2,c3,c4) 上有一个索引 idx(c1,c2,c3)。以下查询不适用于前面描述的松散索引扫描访问方法，但仍适用于紧密索引扫描访问方法。

GROUP BY 中有一个间隙，但它被条件 c2 = 'a' 覆盖：

`SELECT c1, c2, c3 FROM t1 WHERE c2 = 'a' GROUP BY c1, c3;`

GROUP BY 不是从键的第一部分开始，但有一个条件为该部分提供了一个常量：

`SELECT c1, c2, c3 FROM t1 WHERE c1 = 'a' GROUP BY c2, c3;`
