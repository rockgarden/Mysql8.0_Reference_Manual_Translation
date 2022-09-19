# InnoDB 和 MyISAM 索引统计收集

存储引擎收集有关表的统计信息以供优化器使用。表统计基于值组，其中值组是一组具有相同键前缀值的行。出于优化器的目的，一个重要的统计数据是平均值组大小。

MySQL 通过以下方式使用平均值组大小：

- 估计每个 ref 访问必须读取多少行
- 估计一个部分连接产生了多少行，也就是一个操作产生的行数
  `(...) JOIN tbl_name ON tbl_name.key = expr`

随着索引的平均值组大小的增加，该索引对于这两个目的的用处会降低，因为每次查找的平均行数会增加：为了使索引有利于优化目的，最好每个索引值都针对一个小的表中的行数。当给定的索引值产生大量行时，索引的用处不大，MySQL 不太可能使用它。

平均值组大小与表基数有关，表基数是值组的数量。 [SHOW INDEX](https://dev.mysql.com/doc/refman/8.0/en/show-index.html) 语句显示基于 N/S 的基数值，其中 N 是表中的行数，S 是平均值组大小。该比率在表中产生近似数量的值组。

对于基于 <=> 比较运算符的连接，NULL 与任何其他值没有区别：NULL <=> NULL，就像 N <=> N 对任何其他 N 一样。

但是，对于基于 = 运算符的连接，NULL 与非 NULL 值不同：当 expr1 或 expr2（或两者）为 NULL 时，expr1 = expr2 不为真。这会影响 tbl_name.key = expr 形式的比较的 [ref](/MySQL参考手册/优化/了解查询执行计划/解释输出格式.md#ref) 访问：如果 expr 的当前值为 NULL，MySQL 不会访问该表，因为比较不可能为真。

对于 = 比较，表中有多少 NULL 值并不重要。出于优化目的，相关值是非 NULL 值组的平均大小。但是，MySQL 当前不支持收集或使用该平均大小。

对于 InnoDB 和 MyISAM 表，您可以分别通过 [innodb_stats_method](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_stats_method) 和 [myisam_stats_method](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_myisam_stats_method) 系统变量来控制表统计信息的收集。这些变量具有三个可能的值，它们的区别如下：

- 当变量设置为 nulls_equal 时，所有 NULL 值都被视为相同（即，它们都形成一个值组）。

  如果 NULL 值组大小远高于平均非 NULL 值组大小，则此方法会向上倾斜平均值组大小。这使得索引对优化器来说似乎不如查找非 NULL 值的连接实际有用。因此，nulls_equal 方法可能会导致优化器在应该使用索引时不使用 **ref** 访问。

- 当变量设置为 nulls_unequal 时，NULL 值不被视为相同。相反，每个 NULL 值形成一个大小为 1 的单独值组。

  如果您有许多 NULL 值，则此方法会向下倾斜平均值组大小。如果平均非 NULL 值组大小很大，将每个 NULL 值计数为大小为 1 的组会导致优化器高估查找非 NULL 值的连接的索引值。因此，当其他方法可能更好时，nulls_unequal 方法可能会导致优化器将此索引用于 ref 查找。

- 当变量设置为 nulls_ignored 时，NULL 值将被忽略。

如果您倾向于使用许多使用 <=> 而不是 = 的连接，则 NULL 值在比较中并不特殊，并且一个 NULL 等于另一个。在这种情况下，nulls_equal 是适当的统计方法。

innodb_stats_method 系统变量具有全局值； myisam_stats_method 系统变量具有全局值和会话值。设置全局值会影响来自相应存储引擎的表的统计信息收集。设置会话值仅影响当前客户端连接的统计信息收集。这意味着您可以通过设置 myisam_stats_method 的会话值来强制使用给定方法重新生成表的统计信息，而不会影响其他客户端。

要重新生成 MyISAM 表统计信息，您可以使用以下任何一种方法：

- 执行 [myisamchk --stats_method=method_name --analyze](https://dev.mysql.com/doc/refman/8.0/en/myisamchk.html)

- 更改表使其统计信息过期（例如，插入一行然后删除它），然后设置 myisam_stats_method 并发出 [ANALYZE TABLE](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html) 语句

关于使用 innodb_stats_method 和 myisam_stats_method 的一些注意事项：

- 如前所述，您可以强制明确收集表统计信息。但是，MySQL 也可以自动收集统计信息。例如，如果在对表执行语句的过程中，其中一些语句修改了表，MySQL 可能会收集统计信息。 （例如，批量插入或删除或某些 ALTER TABLE 语句可能会发生这种情况。）如果发生这种情况，则使用当时 innodb_stats_method 或 myisam_stats_method 具有的任何值收集统计信息。因此，如果您使用一种方法收集统计信息，但稍后自动收集表的统计信息时将系统变量设置为另一种方法，则使用另一种方法。

- 无法判断使用哪种方法为给定表生成统计信息。

- 这些变量仅适用于 InnoDB 和 MyISAM 表。其他存储引擎只有一种收集表统计信息的方法。通常它更接近于 nulls_equal 方法。
