# IS NULL 优化

MySQL 可以对 col_name IS NULL 执行与 col_name = constant_value 相同的优化。 例如，MySQL 可以使用索引和范围来搜索具有 IS NULL 的 NULL。

例子：

```sql
SELECT * FROM tbl_name WHERE key_col IS NULL;

SELECT * FROM tbl_name WHERE key_col <=> NULL;

SELECT * FROM tbl_name
  WHERE key_col=const1 OR key_col=const2 OR key_col IS NULL;
```

如果 WHERE 子句包含声明为 NOT NULL 的列的 col_name IS NULL 条件，则该表达式将被优化掉。 在列可能无论如何都会产生 NULL 的情况下（例如，如果它来自 LEFT JOIN 右侧的表），则不会发生这种优化。

MySQL 还可以优化 col_name = expr OR col_name IS NULL 的组合，这种形式在已解析的子查询中很常见。 使用此优化时，EXPLAIN 显示 [ref_or_null](/优化/了解查询执行计划/解释输出格式.md#refornull)。

这种优化可以为任何关键部分处理一个 IS NULL。

一些经过优化的查询示例，假设表 t2 的列 a 和 b 上有索引：

```sql
SELECT * FROM t1 WHERE t1.a=expr OR t1.a IS NULL;

SELECT * FROM t1, t2 WHERE t1.a=t2.a OR t2.a IS NULL;

SELECT * FROM t1, t2
  WHERE (t1.a=t2.a OR t2.a IS NULL) AND t2.b=t1.b;

SELECT * FROM t1, t2
  WHERE t1.a=t2.a AND (t2.b=t1.b OR t2.b IS NULL);

SELECT * FROM t1, t2
  WHERE (t1.a=t2.a AND t2.a IS NULL AND ...)
  OR (t1.a=t2.a AND t2.a IS NULL AND ...);
```

ref_or_null 首先读取引用键，然后单独搜索具有 NULL 键值的行。

优化只能处理一个 IS NULL 级别。 在以下查询中，MySQL 仅对表达式 (t1.a=t2.a AND t2.a IS NULL) 使用键查找，并且不能使用 b 上的键部分：

```sql
SELECT * FROM t1, t2
  WHERE (t1.a=t2.a AND t2.a IS NULL)
  OR (t1.b=t2.b AND t2.b IS NULL);
```
