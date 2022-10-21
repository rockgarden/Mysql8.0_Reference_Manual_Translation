# 使用EXISTS策略优化子查询

某些优化适用于使用IN（或=ANY）运算符测试子查询结果的比较。本节讨论这些优化，特别是NULL值带来的挑战。讨论的最后一部分建议如何帮助优化器。

考虑以下子查询比较：

`outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)`

MySQL“从外部到内部 from outside to inside. ”评估查询。也就是说，它首先获取外部表达式outer_expr的值，然后运行子查询并捕获它生成的行。

一个非常有用的优化是“通知 inform ”子查询，唯一感兴趣的行是那些内部表达式inner_expr等于outer_expr的行。这是通过将适当的等式下推到子查询的WHERE子句中以使其更具限制性来实现的。转换后的比较如下所示：

`EXISTS (SELECT 1 FROM ... WHERE subquery_where AND outer_expr=inner_expr)`

转换后，MySQL可以使用下推等式来限制它必须检查以评估子查询的行数。

更一般地说，将N个值与返回N个值行的子查询进行比较时，会进行相同的转换。如果oe_i和ie_i表示对应的外部和内部表达式值，则此子查询比较：

```sql
(oe_1, ..., oe_N) IN
  (SELECT ie_1, ..., ie_N FROM ... WHERE subquery_where)
```

转换成：

```sql
EXISTS (SELECT 1 FROM ... WHERE subquery_where
                          AND oe_1 = ie_1
                          AND ...
                          AND oe_N = ie_N)
```

为了简单起见，下面的讨论假设一对外部和内部表达式值。

如果以下任一条件成立，则上述“下推 pushdown”策略有效：

- outer_expr和inner_expr不能为NULL。

- 无需区分NULL和FALSE子查询结果。如果子查询是WHERE子句中OR或AND表达式的一部分，则MySQL假定您不在乎。优化器注意到无需区分NULL和FALSE子查询结果的另一个实例是这种构造：

  `... WHERE outer_expr IN (subquery)`

  在这种情况下，无论 IN (subquery) 返回NULL还是FALSE，WHERE子句都会拒绝该行。

假设outer_expr已知为非NULL值，但子查询不会生成outer_xpr＝inner_expr的行。然后outer_expr IN（SELECT…）计算如下：

- NULL，如果SELECT生成inner_expr为NULL的任何行

- 如果SELECT仅生成非NULL值或不生成任何值，则返回FALSE

在这种情况下，查找outer_expr=inner_expr的行的方法不再有效。有必要查找此类行，但如果找不到任何行，也要查找inner_expr为NULL的行。大致来说，子查询可以转换为如下内容：

```sql
EXISTS (SELECT 1 FROM ... WHERE subquery_where AND
        (outer_expr=inner_expr OR inner_expr IS NULL))
```

需要评估额外的IS NULL条件是MySQL使用ref_or_NULL访问方法的原因：

```sql
mysql> EXPLAIN
       SELECT outer_expr IN (SELECT t2.maybe_null_key
                             FROM t2, t3 WHERE ...)
       FROM t1;
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
...
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t2
         type: ref_or_null
possible_keys: maybe_null_key
          key: maybe_null_key
      key_len: 5
          ref: func
         rows: 2
        Extra: Using where; Using index
...
```

[unique_subquery](../../了解查询执行计划/解释输出格式.md#unique_subquery)和[index_subquery](../../了解查询执行计划/解释输出格式.md#index_subquery)) 子查询特定的访问方法也有“or NULL”变量。

附加的OR…IS NULL条件使查询执行稍微复杂一些（并且子查询中的一些优化变得不适用），但通常这是可以接受的。

当outer_expr可以为NULL时，情况会更糟。根据SQL将NULL解释为“未知值 unknown value,”，NULL IN（SELECT inner_expr…）应计算为：

- NULL，如果SELECT生成任何行

- 如果SELECT不生成行，则返回FALSE

为了进行正确的评估，有必要检查SELECT是否生成了任何行，因此outer_expr=inner_expr不能被下推到子查询中。这是一个问题，因为除非可以按下等式，否则许多实际子查询会变得非常慢。

本质上，根据outer_expr的值，必须有不同的方法来执行子查询。

优化器选择SQL遵从性而不是速度，因此它考虑了outer_expr可能为NULL的可能性：

- 如果outer_expr为NULL，要计算以下表达式，必须执行SELECT以确定它是否生成任何行：
  
  `NULL IN (SELECT inner_expr FROM ... WHERE subquery_where)`

  有必要在这里执行原始的SELECT，而不需要前面提到的那种pushed-down等式。

- 另一方面，当outer_expr不为NULL时，以下比较绝对必要：
  
  `outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)`

  转换为使用按下条件的表达式：

  `EXISTS (SELECT 1 FROM ... WHERE subquery_where AND outer_expr=inner_expr)`

  如果没有这种转换，子查询将很慢。

为了解决是否将条件下推到子查询中的两难问题，条件被包装在“触发器 trigger”函数中。因此，以下形式的表达式：

`outer_expr IN (SELECT inner_expr FROM ... WHERE subquery_where)`

转换为：

```sql
EXISTS (SELECT 1 FROM ... WHERE subquery_where
                          AND trigcond(outer_expr=inner_expr))
```

更一般地说，如果子查询比较基于多对外部表达式和内部表达式，则转换采用这种比较：

`(oe_1, ..., oe_N) IN (SELECT ie_1, ..., ie_N FROM ... WHERE subquery_where)`

并将其转换为以下表达式：

```sql
EXISTS (SELECT 1 FROM ... WHERE subquery_where
                          AND trigcond(oe_1=ie_1)
                          AND ...
                          AND trigcond(oe_N=ie_N)
       )
```

每个trigcond（X）是一个特殊函数，其计算结果如下：

- “链接 linked ”外部表达式oe_i不为NULL时的X

- 当“链接 linked”外部表达式oe_i为NULL时为TRUE

> 笔记
触发器函数不是使用 [CREATE TRIGGER](https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html) 创建的那种触发器。

封装在trigcond（）函数中的等式不是查询优化器的第一类谓词。大多数优化无法处理可能在查询执行时打开和关闭的谓词，因此它们假定任何trigcond（X）都是未知函数并忽略它。这些优化可以使用触发的等式：

- 引用优化：`trigcond(X=Y [OR Y IS NULL])`可用于构造ref、eq_ref或ref_OR_NULL表访问。

- 基于索引查找的子查询执行引擎：`trigcond(X=Y)` 可用于构造unique_subquery或Index_subque访问。

- 表条件生成器：如果子查询是多个表的联接，则会尽快检查触发的条件。

当优化器使用触发的条件来创建某种基于索引查找的访问（如前面列表的前两项）时，它必须有一个条件关闭时的回退策略。此回退策略始终相同：执行全表扫描。在EXPLAIN输出中，回退显示为Extra列中NULL键的完全扫描：

```sql
mysql> EXPLAIN SELECT t1.col1,
       t1.col1 IN (SELECT t2.key1 FROM t2 WHERE t2.col2=t1.col2) FROM t1\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
        ...
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t2
         type: index_subquery
possible_keys: key1
          key: key1
      key_len: 5
          ref: func
         rows: 2
        Extra: Using where; Full scan on NULL key
```

如果运行EXPLAIN，然后运行SHOW WARNINGS，您可以看到触发的条件：

```log
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: select `test`.`t1`.`col1` AS `col1`,
         <in_optimizer>(`test`.`t1`.`col1`,
         <exists>(<index_lookup>(<cache>(`test`.`t1`.`col1`) in t2
         on key1 checking NULL
         where (`test`.`t2`.`col2` = `test`.`t1`.`col2`) having
         trigcond(<is_not_null_test>(`test`.`t2`.`key1`))))) AS
         `t1.col1 IN (select t2.key1 from t2 where t2.col2=t1.col2)`
         from `test`.`t1`
```

使用触发条件会对性能产生一些影响。NULL IN（SELECT…）表达式现在可能会导致全表扫描（这很慢），而以前它不会。这是为正确的结果付出的代价（触发条件策略的目标是提高合规性，而不是速度）。

对于多个子表查询，NULL IN（SELECT…）的执行速度特别慢，因为联接优化器不会针对外部表达式为NULL的情况进行优化。它假设左侧为NULL的子查询求值非常罕见，即使有统计数据表明情况并非如此。另一方面，如果外部表达式可能为NULL，但实际上并非如此，则没有性能损失。

要帮助查询优化器更好地执行查询，请使用以下建议：

- 如果一个列确实为非空，则将其声明为NOT NULL。这还通过简化该列的条件测试来帮助优化器的其他方面。

- 如果不需要区分NULL和FALSE子查询结果，可以轻松避免缓慢的执行路径。替换如下所示的比较：

  `outer_expr [NOT] IN (SELECT inner_expr FROM ...)`
  
  用这个表达式：

  `(outer_expr IS NOT NULL) AND (outer_expr [NOT] IN (SELECT inner_expr FROM ...))`

  然后NULL IN（SELECT…）永远不会被计算，因为一旦表达式结果清晰，MySQL就会停止计算AND部分。

  另一个可能的改写：

  ```sql
  [NOT] EXISTS (SELECT inner_expr FROM ...
        WHERE inner_expr=outer_expr)
  ```

  优化器switch系统变量的[subquery_materization_cost_based](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_subquery-materialization-cost-based)标志允许控制子查询具体化和IN到EXISTS子查询转换之间的选择。
