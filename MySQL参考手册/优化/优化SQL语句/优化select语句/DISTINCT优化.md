# DISTINCT 优化

DISTINCT 与 ORDER BY 结合在很多情况下需要一个临时表。

因为 DISTINCT 可能使用 GROUP BY，所以了解 MySQL 如何处理 ORDER BY 或 HAVING 子句中不属于所选列的列。请参阅第 12.20.3 节，“[MySQL 对 GROUP BY 的处理](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html)”。

在大多数情况下，可以将 DISTINCT 子句视为 GROUP BY 的特例。例如，以下两个查询是等效的：

```sql
SELECT DISTINCT c1, c2, c3 FROM t1
WHERE c1 > const;

SELECT c1, c2, c3 FROM t1
WHERE c1 > const GROUP BY c1, c2, c3;
```

由于这种等效性，适用于 GROUP BY 查询的优化也可以应用于具有 DISTINCT 子句的查询。因此，有关 DISTINCT 查询的优化可能性的更多详细信息，请参阅第 8.2.1.17 节，“[GROUP BY 优化](https://dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html)”。

将 LIMIT row_count 与 DISTINCT 结合使用时，MySQL 会在找到 row_count 唯一行时立即停止。

如果您不使用查询中命名的所有表中的列，MySQL 会在找到第一个匹配项后立即停止扫描任何未使用的表。在以下情况下，假设 t1 在 t2 之前使用（您可以使用 EXPLAIN 检查），当 MySQL 在 t2 中找到第一行时，它会停止从 t2 读取（对于 t1 中的任何特定行）：

SELECT DISTINCT t1.a FROM t1, t2 where t1.a=t2.a;
