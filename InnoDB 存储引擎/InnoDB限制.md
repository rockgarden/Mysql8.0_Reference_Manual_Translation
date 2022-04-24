# InnoDB 限制

本节介绍 InnoDB 表、索引、表空间和 InnoDB 存储引擎的其他方面的限制。

- 一个表最多可以包含 1017 列。虚拟生成的列包含在此限制中。

- 一个表最多可以包含 64 个二级索引。

- 对于使用 DYNAMIC 或 COMPRESSED 行格式的 InnoDB 表，索引键前缀长度限制为 3072 字节。

- 对于使用 REDUNDANT 或 COMPACT 行格式的 InnoDB 表，索引键前缀长度限制为 767 字节。例如，您可能会在 TEXT 或 VARCHAR 列上的列前缀索引超过 191 个字符时达到此限制，假设使用 utf8mb4 字符集并且每个字符最多 4 个字节。

- 尝试使用超过限制的索引键前缀长度会返回错误。

- 如果您在创建 MySQL 实例时通过指定 innodb_page_size 选项将 InnoDB 页面大小减小到 8KB 或 4KB，则索引键的最大长度会按比例降低，基于 16KB 页面大小的 3072 字节的限制。即页大小为8KB时最大索引键长度为1536字节，页大小为4KB时最大索引键长度为768字节。

- 适用于索引键前缀的限制也适用于全列索引键。

- 多列索引最多允许 16 列。超过限制会返回错误。
  `ERROR 1070 (42000): Too many key parts specified; max 16 parts allowed`

- 对于 4KB、8KB、16KB 和 32KB 页面大小，最大行大小（不包括任何在页外存储的可变长度列）略小于页面的一半。例如，默认 innodb_page_size 16KB 的最大行大小约为 8000 字节。但是，对于 64KB 的 InnoDB 页面大小，最大行大小约为 16000 字节。 LONGBLOB 和 LONGTEXT 列必须小于 4GB，总行大小（包括 BLOB 和 TEXT 列）必须小于 4GB。

- 如果一行的长度小于半页，则所有行都本地存储在该页内。如果超过半页，则为外部页外存储选择可变长度列，直到该行适合半页，如“[文件空间管理](https://dev.mysql.com/doc/refman/8.0/en/innodb-file-space.html)”中所述。

- 尽管 InnoDB 在内部支持大于 65,535 字节的行大小，但 MySQL 本身对所有列的组合大小施加了 65,535 的行大小限制。请参阅“[表列数和行大小的限制](https://dev.mysql.com/doc/refman/8.0/en/column-count-limit.html)”。

- 在某些较旧的操作系统上，文件必须小于 2GB。这不是 InnoDB 的限制。如果您需要一个大型系统表空间，请使用几个较小的数据文件而不是一个大型数据文件对其进行配置，或者将表数据分布在每个表的文件和通用表空间数据文件中。

- InnoDB 日志文件的最大组合大小为 512GB。

- 最小表空间大小略大于 10MB。最大表空间大小取决于 InnoDB 页面大小。最大表空间大小也是表的最大大小。
  
| InnoDB Page Size | Maximum Tablespace Size |
|------------------|-------------------------|
| 4KB              | 16TB                    |
| 8KB              | 32TB                    |
| 16KB             | 64TB                    |
| 32KB             | 128TB                   |
| 64KB             | 256TB                   |

- 一个 InnoDB 实例最多支持 2^32 (4294967296) 个表空间，其中少数表空间保留用于撤消表和临时表。

- 共享表空间最多支持 2^32 (4294967296) 个表。

- 表空间文件的路径（包括文件名）不能超过 Windows 上的 MAX_PATH 限制。 在 Windows 10 之前，MAX_PATH 限制为 260 个字符。 从 Windows 10 版本 1607 开始，MAX_PATH 限制已从常见的 Win32 文件和目录函数中删除，但您必须启用新行为。

- 有关与并发读写事务相关的限制，请参阅“[撤消日志](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-logs.html)”。
