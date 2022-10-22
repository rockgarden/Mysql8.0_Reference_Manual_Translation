# Window函数优化

窗口函数影响优化器考虑的策略：

- 如果子查询具有窗口函数，则禁用子查询的派生表合并。子查询始终是物化的。

- 半联接不适用于窗口函数优化，因为半联接适用于WHERE和JOIN…ON中不能包含窗口函数的子查询。

- 优化器按顺序处理具有相同排序要求的多个窗口，因此可以跳过第一个窗口之后的窗口的排序。

- 优化器不会尝试合并可以在单个步骤中计算的窗口（例如，当多个OVER子句包含相同的窗口定义时）。解决方法是在window子句中定义窗口，并在OVER子句中引用窗口名称。

未用作窗口函数的聚合函数将聚合在最外层的查询中。例如，在这个查询中，MySQL发现COUNT（t1.b）在外部查询中不存在，因为它位于WHERE子句中：

`SELECT * FROM t1 WHERE t1.a = (SELECT COUNT(t1.b) FROM t2);`

因此，MySQL在子查询中聚合，将t1.b视为常量并返回t2的行数。

将WHERE替换为HAVING将导致错误：

```sql
mysql> SELECT * FROM t1 HAVING t1.a = (SELECT COUNT(t1.b) FROM t2);
ERROR 1140 (42000): In aggregated query without GROUP BY, expression #1
of SELECT list contains nonaggregated column 'test.t1.a'; this is
incompatible with sql_mode=only_full_group_by
```

出现此错误是因为COUNT(t1.b)可以存在于HAVING中，因此使外部查询聚合。

Window函数（包括用作窗口函数的聚合函数）没有上述复杂性。它们总是在写入它们的子查询中聚合，而不是在外部查询中聚合。

窗口函数的计算可能会受到[windowing_use_high_precision](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_windowing_use_high_precision)系统变量的值的影响，该变量决定是否在不损失精度的情况下计算窗口操作。默认情况下，windowing_use_high_precision处于启用状态。

对于某些移动帧聚合，可以应用反向聚合函数从聚合中删除值。这可以提高性能，但可能会降低精度。例如，将非常小的浮点值添加到非常大的值会导致非常小的值被大的值“hidden”。稍后反转大值时，小值的效果将丢失。

逆聚合导致的精度损失仅是浮点 floating-point （近似值 approximate-value）数据类型操作的一个因素。对于其他类型，反向聚合是安全的；这包括DECIMAL，它允许小数部分，但是精确值类型。

为了更快地执行，MySQL总是在安全的情况下使用反向聚合：

- 对于浮点值，逆聚合并不总是安全的，可能会导致精度损失。默认情况是避免反向聚合，这种聚合速度较慢，但保持了精度。如果允许为速度牺牲安全，可以禁用windowing_use_high_precision以允许反向聚合。

- 对于非浮动点数据类型，逆聚合始终是安全的，无论windowing_use_high_precision值如何，都会使用它。

- windowing_use_high_precision对MIN()和MAX()没有影响，它们在任何情况下都不使用反向聚合。

对于方差函数 [STDDEV_POP()](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_stddev-pop), [STDDEV_SAMP()](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_stddev-samp), [VAR_POP()](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_var-pop), [VAR_SAMP()](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_var-samp) 及其同义词的求值，可以在优化模式或默认模式下进行求值。优化模式可能会在最后有效数字中产生略微不同的结果。如果允许这种差异，可以禁用windowing_use_high_precision以允许优化模式。

对于EXPLAIN，窗口化执行计划信息太大，无法以传统的输出格式显示。要查看窗口信息，请使用 [EXPLAIN FORMAT=JSON](https://dev.mysql.com/doc/refman/8.0/en/explain.html) 并查找窗口元素。
