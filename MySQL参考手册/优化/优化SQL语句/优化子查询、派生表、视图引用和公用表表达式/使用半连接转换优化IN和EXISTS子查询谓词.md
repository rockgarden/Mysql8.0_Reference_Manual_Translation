# 使用半连接转换优化 IN 和 EXISTS 子查询谓词

半连接是一种准备时转换，它支持多种执行策略，例如表拉出、重复清除、第一次匹配、松散扫描和物化。优化器使用半连接策略来改进子查询的执行，如本节所述。

对于两个表之间的内部连接，连接从一个表返回一行的次数与另一个表中的匹配次数相同。但是对于某些问题，唯一重要的信息是“是否有匹配”，而不是匹配的数量。假设有名为 class 和 roster 的表分别列出课程课程表中的班级和班级名册（每个班级注册的学生）。要列出实际有学生注册的课程，您可以使用此联接：

```sql
SELECT class.class_num, class.class_name
    FROM class
    INNER JOIN roster
    WHERE class.class_num = roster.class_num;
```

**假设class_num是类表的主键，使用 [SELECT DISTINCT](https://dev.mysql.com/doc/refman/8.0/en/select.html) 可以抑制重复，但是先生成所有匹配的行，然后再消除重复，效率低下**。

使用子查询可以获得相同的无重复结果：

```sql
SELECT class_num, class_name
    FROM class
    WHERE class_num IN
        (SELECT class_num FROM roster);
```

在这里，优化器可以识别 IN 子句要求子查询从名册表中只返回每个类号的一个实例。在这种情况下，查询可以使用半连接；也就是说，一个操作只返回与 roster 中的行匹配的类中每一行的一个实例。

以下包含 EXISTS 子查询谓词的语句等效于前面包含 IN 子查询谓词的语句：

```sql
SELECT class_num, class_name
    FROM class
    WHERE EXISTS
        (SELECT * FROM roster WHERE class.class_num = roster.class_num);
```

在 MySQL 8.0.16 及更高版本中，任何具有 EXISTS 子查询谓词的语句都受制于与具有等效 IN 子查询谓词的语句相同的半连接转换。

从 MySQL 8.0.17 开始，以下子查询被转换为反连接 antijoins ：

- NOT IN (SELECT ... FROM ...)

- NOT EXISTS (SELECT ... FROM ...).

- IN (SELECT ... FROM ...) IS NOT TRUE

- EXISTS (SELECT ... FROM ...) IS NOT TRUE.

- IN (SELECT ... FROM ...) IS FALSE

- EXISTS (SELECT ... FROM ...) IS FALSE.

简而言之，对 IN (SELECT ... FROM ...) 或 EXISTS (SELECT ... FROM ...) 形式的子查询的任何否定都将转换为反连接。

反连接是一种只返回不匹配的行的操作。考虑此处显示的查询：

```sql
SELECT class_num, class_name
    FROM class
    WHERE class_num NOT IN
        (SELECT class_num FROM roster);
```

此查询在内部被重写为 antijoin SELECT class_num, class_name FROM class ANTIJOIN roster ON class_num，它返回 class 中与 roster 中的任何行都不匹配的每一行的一个实例。这意味着，对于类中的每一行，只要在名册中找到匹配项，就可以丢弃类中的行。

如果要比较的表达式可以为空，则在大多数情况下无法应用反连接转换。此规则的一个例外是 (... NOT IN (SELECT ...)) IS NOT FALSE 并且其等效的 (... IN (SELECT ...)) IS NOT TRUE 可以转换为反连接。

外部查询规范中允许外部联接和内部联接语法，表引用可以是基表、派生表、视图引用或公用表表达式。

在 MySQL 中，子查询必须满足这些条件才能作为半连接处理（或者，在 MySQL 8.0.17 及更高版本中，如果 NOT 修改子查询，则为反连接）：

- 它必须是出现在 WHERE 或 ON 子句顶层的 IN、= ANY 或 EXISTS 谓词的一部分，可能作为 AND 表达式中的术语。例如：

  ```sql
  SELECT ...
    FROM ot1, ...
    WHERE (oe1, ...) IN
        (SELECT ie1, ... FROM it1, ... WHERE ...);
  ```

  这里，ot_i 和 it_i 表示查询的外部和内部部分中的表，oe_i 和ie_i 表示引用外部表和内部表中的列的表达式。

  在 MySQL 8.0.17 及更高版本中，子查询也可以是由 NOT、IS [NOT] TRUE 或 IS [NOT] FALSE 修改的表达式的参数。

- 它必须是没有 UNION 构造的单个 SELECT。

- 它不能包含 HAVING 子句。

- 它不能包含任何聚合函数（无论是显式还是隐式分组）。

- 它不能有 LIMIT 子句。

- 该语句不得在外部查询中使用 STRAIGHT_JOIN 连接类型。

- 不得存在 STRAIGHT_JOIN 修饰符。

- 外部表和内部表的总数必须小于连接中允许的最大表数。

- 子查询可以是相关的或不相关的。在 MySQL 8.0.16 及更高版本中，去相关查看用作 EXISTS 参数的子查询的 WHERE 子句中的平凡相关谓词，并使其可以像在 IN (SELECT b FROM ...) 中使用一样对其进行优化。术语平凡相关 trivially correlated 意味着谓词是一个相等谓词，它是 WHERE 子句中的唯一谓词（或与 AND 组合），并且一个操作数来自子查询中引用的表，另一个操作数来自外部查询块。

- DISTINCT 关键字被允许但被忽略。半连接策略自动处理重复删除。

- 允许但忽略 GROUP BY 子句，除非子查询还包含一个或多个聚合函数。

- 允许但忽略 ORDER BY 子句，因为排序与半连接策略的评估无关。

如果子查询满足上述条件，MySQL 会将其转换为半连接（或者，在 MySQL 8.0.17 或更高版本中，如果适用，则为反连接）并从这些策略中做出基于成本的选择：

- 将子查询转换为联接，或使用表拉出并将查询作为子查询表和外部表之间的内部联接运行。表拉出将表从子查询拉出到外部查询。

- Duplicate Weedout：运行 semijoin，就像它是一个连接一样，并使用临时表删除重复记录。

- FirstMatch：当扫描内部表的行组合并且给定值组有多个实例时，选择一个而不是全部返回。这种“快捷方式”扫描并消除了不必要的行的产生。

- LooseScan：使用索引扫描子查询表，该索引允许从每个子查询的值组中选择单个值。

- 将子查询具体化为用于执行连接的索引临时表，其中索引用于删除重复项。稍后在将临时表与外部表连接时，该索引也可能用于查找；如果不是，则扫描该表。有关物化的更多信息，请参阅第 8.2.2.2 节，“[使用物化优化子查询](https://dev.mysql.com/doc/refman/8.0/en/subquery-materialization.html)”。

可以使用以下 optimizer_switch 系统变量标志启用或禁用这些策略中的每一个：

- [semijoin](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_semijoin) 标志控制是否使用半连接。从 MySQL 8.0.17 开始，这也适用于反连接。

- 如果启用了半连接，则 [firstmatch](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_firstmatch) 、 [loosescan](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_loosescan) 、 [duplicateweedout](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_duplicateweedout) 和 [materialization](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_materialization) 标志可以更好地控制允许的半连接策略。

- 如果 duplicateweedout semijoin 策略被禁用，除非所有其他适用的策略也被禁用，否则不会使用它。

- 如果 duplicateweedout 被禁用，有时优化器可能会生成一个远非最佳的查询计划。这是由于贪婪搜索期间的启发式修剪而发生的，这可以通过设置 optimizer_prune_level=0 来避免。

默认情况下启用这些标志。请参见第 8.9.2 节，“[可切换的优化](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html)”。

优化器将视图和派生表的处理差异降至最低。这会影响使用 STRAIGHT_JOIN 修饰符的查询和带有可转换为半连接的 IN 子查询的视图。以下查询说明了这一点，因为处理中的变化会导致转换发生变化，从而导致不同的执行策略：

```sql
CREATE VIEW v AS
SELECT *
FROM t1
WHERE a IN (SELECT b
           FROM t2);

SELECT STRAIGHT_JOIN *
FROM t3 JOIN v ON t3.x = v.a;
```

优化器首先查看视图并将 IN 子查询转换为半连接，然后检查是否可以将视图合并到外部查询中。因为外部查询中的 STRAIGHT_JOIN 修饰符阻止半连接，优化器拒绝合并，导致派生表评估使用物化表。

EXPLAIN 输出表明使用了如下半连接策略：

- 对于扩展的 EXPLAIN 输出，以下 SHOW WARNINGS 显示的文本显示了重写的查询，该查询显示了半连接结构。 （请参阅 [扩展的 EXPLAIN 输出格式](/MySQL参考手册/优化/了解查询执行计划/扩展解释输出格式.md))。）从中您可以了解哪些表被拉出半连接。如果子查询被转换为半连接，您应该看到子查询谓词消失了，它的表和 WHERE 子句被合并到外部查询连接列表和 WHERE 子句中。

- Duplicate Weedout 的临时表使用由 Extra 列中的 Start temporary 和 End temporary 指示。未拉出且在 Start temporary 和 End temporary 覆盖的 EXPLAIN 输出行范围内的表在临时表中具有其 rowid。

- Extra 列中的 FirstMatch(tbl_name) 表示连接快捷方式。

- Extra 列中的 LooseScan(m..n) 表示使用 LooseScan 策略。 m 和 n 是关键部件号。

- 用于实现的临时表由 select_type 值为 MATERIALIZED 的行和表值为 `<subqueryN>` 的行指示。

在 MySQL 8.0.21 和更高版本中，半连接转换也可以应用于使用 [NOT] IN 或 [NOT] EXISTS 子查询谓词的单表 UPDATE 或 DELETE 语句，前提是该语句不使用 ORDER BY 或 LIMIT ，并且优化器提示或 optimizer_switch 设置允许半连接转换。
