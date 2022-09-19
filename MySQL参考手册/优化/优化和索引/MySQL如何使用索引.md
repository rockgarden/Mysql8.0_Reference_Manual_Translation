# MySQL 如何使用索引

索引用于快速查找具有特定列值的行。如果没有索引，MySQL 必须从第一行开始，然后读取整个表以查找相关行。表越大，成本越高。如果表有相关列的索引，MySQL 可以快速确定要在数据文件中间查找的位置，而无需查看所有数据。这比顺序读取每一行要快得多。

大多数 MySQL 索引（PRIMARY KEY、UNIQUE、INDEX 和 FULLTEXT）都存储在 [B-trees](../../词汇表.md#b-tree) 中。例外：空间数据类型的索引使用 R-trees； MEMORY 表也支持[散列索引](../../词汇表.md#哈希索引)； InnoDB 对 FULLTEXT 索引使用倒排列表。

一般来说，索引的使用如以下讨论中所述。[“B-Tree 和哈希索引的比较”](B-Tree和Hash索引的比较.md)中描述了哈希索引的特定特征（如 MEMORY 表中使用的）。

MySQL 对这些操作使用索引：

- 快速找到匹配 WHERE 子句的行。
- 从考虑中消除行。如果在多个索引之间进行选择，MySQL 通常使用找到最少行数的索引（最具[选择性](/词汇表.md#selectivity)的索引）。
- 如果表有一个多列索引，优化器可以使用索引的任何最左边的前缀来查找行。例如，如果您在 (col1, col2, col3) 上有一个三列索引，则您在 (col1)、(col1, col2) 和 (col1, col2, col3) 上有索引搜索功能。有关详细信息，请参阅[多列索引](/优化/优化和索引/多列索引.md)。
- 在执行连接时从其他表中检索行。**如果将列声明为相同的类型和大小，MySQL 可以更有效地使用列上的索引**。在此上下文中，如果将 VARCHAR 和 CHAR 声明为相同大小，则它们被视为相同。例如，VARCHAR(10) 和 CHAR(10) 的大小相同，但 VARCHAR(10) 和 CHAR(15) 不同。
- 对于非二进制字符串列之间的比较，**两列应使用相同的字符集**。例如，将 utf8 列与 latin1 列进行比较会排除使用索引。
- 如果值不能在没有转换的情况下直接比较，则比较不同的列（例如，将字符串列与时间或数字列进行比较）可能会阻止使用索引。对于数字列中的给定值（例如 1），它可能与字符串列中的任意数量的值（例如“1”、“1”、“00001”或“01.e1”）进行比较。这排除了对字符串列使用任何索引。
- 查找特定索引列 key_col 的 MIN() 或 MAX() 值。这是由预处理器优化的，该预处理器检查您是否在索引中 key_col 之前出现的所有关键部分上使用 WHERE key_part_N = constant。在这种情况下，MySQL 为每个 MIN() 或 MAX() 表达式执行单个键查找，并将其替换为常量。如果所有表达式都替换为常量，则查询立即返回。例如：
  
  ```sql
  SELECT MIN(key_part2),MAX(key_part2)
    FROM tbl_name WHERE key_part1=10;
  ```

- 如果排序或分组是在可用索引的最左前缀上完成的（例如，ORDER BY key_part1、key_part2），则对表进行排序或分组。如果所有关键部分后跟 DESC，则以相反的顺序读取密钥。 （或者，如果索引是降序索引，则按正序读取键。）请参阅[第 8.2.1.16 节，“ORDER BY 优化”](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html)，[第 8.2.1.17 节，“GROUP BY 优化”](https://dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html)和第 [8.3.13 节，“降序索引”](https://dev.mysql.com/doc/refman/8.0/en/descending-indexes.html)。
- 在某些情况下，可以优化查询以在不查阅数据行的情况下检索值。 （为查询提供所有必要结果的索引称为[覆盖索引](../../词汇表.md#covering-index)。）如果查询仅使用表中包含在某个索引中的列，则可以从索引树中检索所选值以获得更快的速度：
  
  ```sql
  SELECT key_part3 FROM tbl_name
    WHERE key_part1=1
  ```

对于小表或报表查询处理大部分或所有行的大表的查询，索引不太重要。当查询需要访问大部分行时，顺序读取比通过索引更快。即使查询不需要所有行，顺序读取也可以最大限度地减少磁盘寻道。有关详细信息，请参见[第 8.2.1.23 节，“避免全表扫描”](https://dev.mysql.com/doc/refman/8.0/en/table-scan-avoidance.html)。
