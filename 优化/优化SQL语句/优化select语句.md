# Optimizing SELECT Statements

查询以 SELECT 语句的形式执行数据库中的所有查找操作。调整这些语句是重中之重，无论是实现动态网页的亚秒级响应时间，还是缩短生成大量隔夜报告的时间。

除了 SELECT 语句，查询的调优技术也适用于 CREATE TABLE...AS SELECT、INSERT INTO...SELECT 和 DELETE 语句中的 WHERE 子句等结构。这些语句有额外的性能考虑，因为它们结合了写操作和面向读的查询操作。

NDB Cluster 支持连接下推优化，其中合格的连接被完整地发送到 NDB Cluster 数据节点，在那里它可以分布在它们之间并并行执行。有关此优化的更多信息，请参阅 [NDB 下推连接的条件](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-options-variables.html#ndb_join_pushdown-conditions)。

优化查询的主要考虑因素是：

- 要使慢速 SELECT ... WHERE 查询更快，首先要检查的是是否可以添加[索引](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_index)。在 WHERE 子句中使用的列上设置索引，以加快评估、过滤和最终检索结果的速度。为避免浪费磁盘空间，请构建一小组索引来加速应用程序中使用的许多相关查询。
- 对于使用连接和外键等功能引用不同表的查询，索引尤其重要。您可以使用 EXPLAIN 语句来确定哪些索引用于 SELECT。请参阅第 [8.3.1 节，“MySQL 如何使用索引”](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html)和第 [8.8.1 节，“使用 EXPLAIN 优化查询”](https://dev.mysql.com/doc/refman/8.0/en/using-explain.html)。
- 隔离和调整查询的任何部分，例如函数调用，这会花费过多时间。根据查询的结构，可以为结果集中的每一行调用一次函数，甚至为表中的每一行调用一次函数，这极大地放大了任何低效率。Isolate and tune any part of the query, such as a function call, that takes excessive time. Depending on how the query is structured, a function could be called once for every row in the result set, or even once for every row in the table, greatly magnifying any inefficiency.
- 尽量减少查询中的全表扫描次数，尤其是对于大表。
- 通过定期使用 ANALYZE TABLE 语句使表统计信息保持最新，因此优化器拥有构建有效执行计划所需的信息。
- 了解特定于每个表的存储引擎的调优技术、索引技术和配置参数。 InnoDB 和 MyISAM 都有一套指导方针来启用和维持查询的高性能。有关详细信息，请参阅第 [8.5.6 节，“优化 InnoDB 查询”](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-queries.html)和第 [8.6.1 节，“优化 MyISAM 查询”](https://dev.mysql.com/doc/refman/8.0/en/optimizing-queries-myisam.html)。
- 您可以使用第 [8.5.3 节“优化 InnoDB 只读事务”](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-ro-txn.html)中的技术来优化 InnoDB 表的单查询事务。
- 避免以难以理解的方式转换查询，尤其是在优化器自动执行某些相同的转换时。Avoid transforming the query in ways that make it hard to understand, especially if the optimizer does some of the same transformations automatically.
- 如果性能问题不能通过基本准则之一轻松解决，请通过阅读 EXPLAIN 计划并调整索引、WHERE 子句、连接子句等来调查特定查询的内部细节。（当您达到一定的专业水平时，阅读 EXPLAIN 计划可能是您每次查询的第一步。）
- 调整 MySQL 用于缓存的内存区域的大小和属性。通过有效使用 InnoDB 缓冲池、MyISAM 键缓存和 MySQL 查询缓存，重复查询运行得更快，因为结果是从内存中检索的第二次和后续时间(the results are retrieved from memory the second and subsequent times.)。
- 即使对于使用缓存内存区域快速运行的查询，您仍然可以进一步优化，以便它们需要更少的缓存内存，从而使您的应用程序更具可扩展性。可扩展性意味着您的应用程序可以处理更多的并发用户、更大的请求等，而不会出现性能大幅下降。
- 处理锁定问题，您的查询速度可能会受到同时访问表的其他会话的影响。
