# 使用 EXPLAIN 优化查询

EXPLAIN 语句提供有关 MySQL 如何执行语句的信息：

- EXPLAIN 适用于 SELECT、DELETE、INSERT、REPLACE 和 UPDATE 语句。
- 当 EXPLAIN 与可解释语句一起使用时，MySQL 显示来自优化器的有关语句执行计划的信息。也就是说，MySQL 解释了它将如何处理该语句，包括有关表如何连接以及以何种顺序连接的信息。有关使用 EXPLAIN 获取执行计划信息的信息，请参阅“EXPLAIN 输出格式”。
- 当 EXPLAIN 与 FOR CONNECTION connection_id 一起使用而不是可解释的语句时，它将显示在命名连接中执行的语句的执行计划。请参见“获取命名连接的执行计划信息”。
- 对于 SELECT 语句，EXPLAIN 生成可以使用 SHOW WARNINGS 显示的附加执行计划信息。请参见“扩展的 EXPLAIN 输出格式”。
- EXPLAIN 对于检查涉及分区表的查询很有用。请参见“[获取有关分区的信息](https://dev.mysql.com/doc/refman/8.0/en/partitioning-info.html)”。
- FORMAT 选项可用于选择输出格式。 TRADITIONAL 以表格格式显示输出。如果没有 FORMAT 选项，这是默认设置。 JSON 格式以 JSON 格式显示信息。

在 EXPLAIN 的帮助下，您可以看到应该在哪里为表添加索引，以便通过使用索引查找行来更快地执行语句。您还可以使用 EXPLAIN 检查优化器是否以最佳顺序连接表。要提示优化器使用与 SELECT 语句中表命名顺序相对应的连接顺序，请以 SELECT STRAIGHT_JOIN 而不仅仅是 SELECT 开始该语句。 （请参阅“[SELECT语句](https://dev.mysql.com/doc/refman/8.0/en/select.html)”。）但是，STRAIGHT_JOIN 可能会阻止使用索引，因为它禁用了半连接转换。请参阅“[使用半连接转换优化IN和EXISTS子查询谓词](https://dev.mysql.com/doc/refman/8.0/en/semijoins.html)”。

优化器跟踪有时可能会提供与 EXPLAIN 的信息互补的信息。但是，优化器跟踪格式和内容可能会因版本而异。有关详细信息，请参阅 [MySQL内部：跟踪优化器](https://dev.mysql.com/doc/internals/en/optimizer-tracing.html)。

如果在您认为应该使用索引时遇到索引未使用的问题，请运行 ANALYZE TABLE 来更新表统计信息，例如键的基数，这可能会影响优化器所做的选择。请参阅“[ANALYZE TABLE 语句](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)”。

> 笔记
EXPLAIN 还可用于获取有关表中列的信息。 EXPLAIN tbl_name 与 DESCRIBE tbl_name 和 SHOW COLUMNS FROM tbl_name 同义。 有关更多信息，请参阅“[DESCRIBE语句](https://dev.mysql.com/doc/refman/8.0/en/describe.html)”和“[SHOW COLUMNS 语句](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html)”。