# 优化MySQL Server

- 8.12.1 优化磁盘I/O
- 8.12.2 使用符号链接
- 8.12.3 [优化内存使用](#优化内存使用)

本节讨论数据库服务器的优化技术，主要处理系统配置，而不是优化SQL语句。本节中的信息适用于希望确保其管理的服务器的性能和可伸缩性的DBA；为开发人员构建安装脚本，包括设置数据库；以及为了开发、测试等目的而运行MySQL的人，他们希望最大限度地提高自己的生产力。

## 优化内存使用

- 8.12.3.1 MySQL如何使用内存
- 8.12.3.2 启用大页面支持

### MySQL如何使用内存

MySQL分配缓冲区和缓存以提高数据库操作的性能。默认配置旨在允许MySQL服务器在具有大约512MB RAM的虚拟机上启动。可以通过增加某些与缓存和缓冲区相关的系统变量的值来提高MySQL性能。您还可以修改默认配置，以便在内存有限的系统上运行MySQL。

以下列表描述了MySQL使用内存的一些方式。适用时，参考相关系统变量。有些项目是特定于存储引擎或功能的。

- InnoDB缓冲池是一个内存区域，用于保存表、索引和其他辅助缓冲区的缓存InnoDB数据。为了提高高容量读取操作的效率，缓冲池被划分为可能包含多行的页。为了提高缓存管理的效率，缓冲池被实现为页面的链接列表；使用LRU算法的变体，很少使用的数据从缓存中老化。有关更多信息，请参阅[缓冲池](../InnoDB存储引擎/InnoDB内存结构/缓冲池.md)。

  缓冲池的大小对系统性能很重要：

  - InnoDB在服务器启动时使用 malloc() 操作为整个缓冲池分配内存。 innodb_buffer_pool_size 系统变量定义缓冲池大小。通常，建议的innodb_buffer_pool_size值为系统内存的50%到75%。innodbbufferpoolsize可以在服务器运行时动态配置。有关更多信息，请参阅“配置InnoDB缓冲池大小”。

  - 在具有大量内存的系统上，可以通过将缓冲池划分为多个缓冲池实例来提高并发性。 innodb_buffer_pool_instances 系统变量定义缓冲池实例的数量。

  - 缓冲池太小可能会导致过度搅动，因为页面只在很短时间后才从缓冲池中刷新。

  - 缓冲池太大可能会由于内存竞争而导致交换。

- 存储引擎接口使优化器能够提供优化器估计可能读取多行的扫描所使用的记录缓冲区大小的信息。缓冲区大小可以根据估计值的大小而变化。InnoDB使用这种可变大小的缓冲功能来利用行预取，并减少闭锁开销(overhead of latching)和B树导航的开销。

- 所有线程共享MyISAM密钥缓冲区。[key_buffer_size](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_key_buffer_size)系统变量决定其大小。

  对于服务器打开的每个MyISAM表，索引文件打开一次；对于访问表的每个并发运行的线程，数据文件打开一次。对于每个并发线程，将为每个列分配一个表结构、列结构和大小为3*N的缓冲区（其中N是最大行长度，不包括BLOB列）。BLOB列需要五到八个字节加上BLOB数据的长度。MyISAM存储引擎维护一个额外的行缓冲区供内部使用。

- myisam_use_mmap系统变量可以设置为1，以便为所有myisam表启用内存映射。

- 如果内存中的内部临时表变得太大（由tmp_table_size和max_heap_table_size系统变量确定），MySQL会**自动将表从内存中转换为磁盘上格式**。从MySQL 8.0.16开始，磁盘上的临时表总是使用InnoDB存储引擎。您可以增加允许的临时表大小，如 [MySQL中的内部临时表使用](../优化/优化数据库结构/内部临时表使用.md) 所述。

  对于用CREATE TABLE显式创建的MEMORY表，只有max_heap_table_size系统变量决定表可以增长多少，并且不转换为磁盘格式。

- [MySQL Performance Schema](https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html)是一种低级别监视MySQL服务器执行的功能。Performance Schema以增量方式动态分配内存，将其内存使用扩展到实际的服务器负载，而不是在服务器启动期间分配所需的内存。**一旦分配了内存，只有重新启动服务器才能释放内存**。有关更多信息，请参阅第27.17节“[性能模式内存分配模型](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-model.html)”。

- 服务器用于管理客户端连接的每个线程都需要一些特定于线程的空间。下表列出了这些变量以及哪些系统变量控制它们的大小：

  - 堆栈（[thread_stack](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_thread_stack)）

  - 连接缓冲区（[net_buffer_length](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_net_buffer_length)）

  - 结果缓冲区（[net_buffer_length](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_net_buffer_length)）

  连接缓冲区和结果缓冲区都以等于 net_buffer_length 字节的大小开始，但根据需要动态扩展到 max_allowed_packet 字节。在每个SQL语句之后，结果缓冲区缩减为net_buffer_length字节。在语句运行时，也会分配当前语句字符串的副本。

  通常不应更改此变量，但如果内存很少，可以将其设置为客户端发送的预期语句长度。如果语句超过此长度，连接缓冲区将自动扩大。net_buffer_length可以设置的最大值为1MB。

  当通过更改max_allowed_packet变量的值来更改消息缓冲区大小时，如果客户端程序允许，还应更改客户端的缓冲区大小。客户端库中内置的默认max_alloved_packte值为1GB，但个别客户端程序可能会覆盖此值。

  每个连接线程都使用内存计算语句摘要。服务器为每个会话分配[max_digest_length](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_digest_length)字节。参见第27.10节“[性能模式语句摘要和采样](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-digests.html)”。

- 所有线程共享相同的基本内存。

- 当不再需要线程时，分配给它的内存将被释放并返回给系统，除非线程返回线程缓存。在这种情况下，内存将保持分配状态。

- 执行表顺序扫描的每个请求都分配一个读取缓冲区。[read_buffer_size](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_read_buffer_size)系统变量确定缓冲区大小。

- 当以任意顺序读取行时（例如，在排序之后），可以分配随机读取缓冲区以避免磁盘查找。[read_rnd_bubuffer_size](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_read_rnd_buffer_size)系统变量确定缓冲区大小。

- 所有连接都是在一次传递中执行的，大多数连接都可以在不使用临时表的情况下完成。大多数临时表都是基于内存的哈希表。具有较大行长度（计算为所有列长度之和）或包含[BLOB](https://dev.mysql.com/doc/refman/8.0/en/blob.html)列的临时表存储在磁盘上。

- 大多数执行排序的请求都会根据结果集的大小将排序缓冲区和零分配给两个临时文件。参见第B.3.3.5节，“[MySQL存储临时文件的位置](https://dev.mysql.com/doc/refman/8.0/en/temporary-files.html)”。

- 几乎所有的解析和计算都是在线程本地和可重用内存池中完成的。小项目不需要内存开销，从而避免了正常缓慢的内存分配和释放。内存仅分配给意外大的字符串。

- 对于每个具有BLOB列的表，缓冲区都会动态扩大，以读取更大的BLOB值。如果扫描表，缓冲区将增长到最大BLOB值。

- MySQL需要用于表缓存的内存和描述符。所有在用表的处理程序结构都保存在表缓存中，并作为“先进先出”（FIFO）进行管理。[table_open_cache](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_table_open_cache)系统变量定义初始表缓存大小；请参阅第8.4.3.1节“[MySQL如何打开和关闭表](https://dev.mysql.com/doc/refman/8.0/en/table-cache.html)”。

- MySQL还需要用于表定义缓存的内存。[table_definition_cache](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_table_definition_cache)系统变量定义可以存储在表定义缓存中的表定义数。如果使用大量表，可以创建大型表定义缓存以加快表的打开速度。与表缓存不同，表定义缓存占用的空间更少，并且不使用文件描述符。

- FLUSHTABLES语句或mysqladmin FLUSH-TABLES命令将关闭所有未立即使用的表，并在当前执行线程结束时将所有正在使用的表标记为关闭。这有效地释放了大部分在用内存。在关闭所有表之前，FLUSH TABLES不会返回。

- 作为GRANT, CREATE USER, CREATE SERVER 和 INSTALL PLUGIN语句的结果，服务器将信息缓存在内存中。此内存不是由相应的[REVOKE](https://dev.mysql.com/doc/refman/8.0/en/revoke.html)、[DROP USER](https://dev.mysql.com/doc/refman/8.0/en/drop-user.html)、[DROP SERVER](https://dev.mysql.com/doc/refman/8.0/en/drop-server.html)和[UNINSTALL PLUGIN](https://dev.mysql.com/doc/refman/8.0/en/uninstall-plugin.html)语句释放的，因此，对于执行许多导致缓存的语句实例的服务器，缓存内存使用量会增加，除非使用[FLUSH PRIVILEGES](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-privileges)释放它。

- 在复制拓扑中，以下设置会影响内存使用，可以根据需要进行调整：

  - 复制源上的[max_allowed_packet](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_allowed_packet)系统变量限制了源发送到其副本进行处理的最大消息大小。此设置默认为64M。

  - 多线程副本上的系统变量[replica_pending_jobs_size_max](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#sysvar_replica_pending_jobs_size_max)（来自MySQL 8.0.26）或[slave_pending_jobs_size_max](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#sysvar_slave_pending_jobs_size_max)（在MySQL 8.0.2.6之前）设置可用于保存等待处理的消息的最大内存量。此设置默认为128M。内存仅在需要时分配，但如果复制拓扑有时处理大型事务，则可能会使用内存。这是一个软限制，可以处理较大的事务。

  - 复制源或副本上的[rpl_read_size](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#sysvar_rpl_read_size)系统变量控制从二进制日志文件和中继日志文件读取的最小数据量（以字节为单位）。默认值为8192字节。为从二进制日志和中继日志文件读取的每个线程分配一个大小为该值的缓冲区，包括源上的转储线程和副本上的协调器线程。

  - [binlog_transaction_dependency_history_size](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_transaction_dependency_history_size)系统变量限制作为内存历史记录保存的行哈希数。

  - [max_binlog_cache_size](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_max_binlog_cache_size)系统变量指定单个事务的内存使用上限。

  - [max_binlog_stmt_cache_size](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_max_binlog_stmt_cache_size)系统变量指定语句缓存的内存使用上限。

ps和其他系统状态程序可能会报告mysqld使用了大量内存。这可能是由不同内存地址上的线程堆栈引起的。例如，Solaris版本的ps将堆栈之间未使用的内存计数为已用内存。要验证这一点，请使用swap-s检查可用的swap。我们用几个内存泄漏检测器（商业和开源）测试mysqld，因此应该不会有内存泄漏。

### 监视MySQL内存使用情况

下面的示例演示如何使用[性能模式](https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html)和[系统模式](https://dev.mysql.com/doc/refman/8.0/en/sys-schema.html)来监视MySQL内存使用情况。

默认情况下，大多数性能模式内存检测都处于禁用状态。可以通过更新Performance Schema [setup_Instruments](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-instruments-table.html)表的enabled列来启用仪器。内存工具的名称格式为`memory/code_area/instrument_name`，其中code_area是一个值，如sql或innodb，instrument_nam是仪器详细信息。

1. 要查看可用的MySQL内存工具，请查询Performance Schema setup_instruments表。以下查询为所有代码区域返回数百个内存工具。

   `mysql> SELECT * FROM performance_schema.setup_instruments WHERE NAME LIKE '%memory%';`

   您可以通过指定代码区域来缩小结果范围。例如，通过将InnoDB指定为代码区域，可以将结果限制为InnoDB内存工具。

   `mysql> SELECT * FROM performance_schema.setup_instruments WHERE NAME LIKE '%memory/innodb%';`

   根据您的MySQL安装，代码区域可能包括performance_schema、sql、client、innodb、myisam、csv、memory、blackhole、archive、partition等。

2. 要启用内存工具，请将性能模式工具规则添加到MySQL配置文件中。例如，要启用所有内存工具，请将此规则添加到配置文件中，然后重新启动服务器：

   `performance-schema-instrument='memory/%=COUNTED'`

   > 笔记
   在启动时启用内存工具可确保对启动时发生的内存分配进行计数。

   重新启动服务器后，对于启用的内存工具，性能模式setup_instruments表的ENABLED列应报告YES。对于内存仪器，setup_instruments表中的TIMED列被忽略，因为内存操作没有计时。

   `SELECT * FROM performance_schema.setup_instruments WHERE NAME LIKE '%memory/innodb%';`

3. 查询内存仪器数据。在本例中，在Performance Schema memory_summary_global_by_event_name表中查询内存工具数据，该表按event_name汇总数据。EVENT_NAME是仪器的名称。

   以下查询返回InnoDB缓冲池的内存数据。有关列说明，请参阅第27.12.20.10节“[内存摘要表](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-memory-summary-tables.html)”。

   `mysql> SELECT * FROM performance_schema.memory_summary_global_by_event_name WHERE EVENT_NAME LIKE 'memory/innodb/buf_buf_pool'\G`

   可以使用sys模式memory_global_by_current_bytes表查询相同的底层数据，该表全局显示服务器内的当前内存使用情况，按分配类型进行细分。

   `mysql> SELECT * FROM sys.memory_global_by_current_bytes WHERE event_name LIKE 'memory/innodb/buf_buf_pool'\G`

   此系统架构查询按代码区域聚合当前分配的内存（current_alloc）：

   ```sql
   mysql> SELECT SUBSTRING_INDEX(event_name,'/',2) AS
       code_area, FORMAT_BYTES(SUM(current_alloc))
       AS current_alloc
       FROM sys.x$memory_global_by_current_bytes
       GROUP BY SUBSTRING_INDEX(event_name,'/',2)
       ORDER BY SUM(current_alloc) DESC;
   ```

   有关sys模式的更多信息，请参阅第28章[MySQL sys模式](https://dev.mysql.com/doc/refman/8.0/en/sys-schema.html)。

## 启用大页面支持

某些硬件/操作系统体系结构支持大于默认值（通常为4KB）的内存页。此支持的实际实现取决于底层硬件和操作系统。由于减少了Translation Lookaside Buffer (TLB)未命中，执行大量内存访问的应用程序可以通过使用大型页面来提高性能。

在MySQL中，InnoDB可以使用大页面为其缓冲池和附加内存池分配内存。

MySQL中大页面的标准使用尝试使用支持的最大大小，最多4MB。在Solaris中，“超大页面”功能允许使用最多256MB的页面。此功能适用于最新的SPARC平台。可以使用--super-lige pages或--skip super-lege pages选项启用或禁用它。

MySQL还支持Linux实现大页面支持（在Linux中称为HugeTLB）。

在Linux上使用大型页面之前，必须启用内核来支持它们，并且必须配置HugeTLB内存池。作为参考，HugeTBL API记录在Linux源代码的Documentation/vm/hugetlbpage.txt文件中。

某些最新系统（如Red Hat Enterprise Linux）的内核默认启用了大页面功能。要检查内核是否是这样，请使用以下命令并查找包含“huge”的输出行：

```bash
$> cat /proc/meminfo | grep -i huge
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       4096 kB
```

非空命令输出表示存在大页面支持，但零值表示未配置任何页面以供使用。

如果您的内核需要重新配置以支持大页面，请参阅hugetlbpage。txt文件以获取说明。

假设您的Linux内核启用了大页面支持，请使用以下命令配置它以供MySQL使用。通常，您将它们放在一个rc文件或在系统引导顺序期间执行的等效启动文件中，以便每次系统启动时都执行这些命令。这些命令应该在MySQL服务器启动之前，在引导序列的早期执行。确保根据您的系统更改分配编号和组编号。

```bash
# Set the number of pages to be used.
# Each page is normally 2MB, so a value of 20 = 40MB.
# This command actually allocates memory, so this much
# memory must be available.
echo 20 > /proc/sys/vm/nr_hugepages

# Set the group number that is permitted to access this
# memory (102 in this case). The mysql user must be a
# member of this group.
echo 102 > /proc/sys/vm/hugetlb_shm_group

# Increase the amount of shmem permitted per segment
# (12G in this case).
echo 1560281088 > /proc/sys/kernel/shmmax

# Increase total amount of shared memory.  The value
# is the number of pages. At 4KB/page, 4194304 = 16GB.
echo 4194304 > /proc/sys/kernel/shmall
```

对于MySQL，您通常希望shmmax的值接近shmall的值。

要验证大页面配置，请按照前面的描述再次检查/proc/meminfo。现在您应该看到一些非零值：

```bash
$> cat /proc/meminfo | grep -i huge
HugePages_Total:      20
HugePages_Free:       20
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       4096 kB
```

使用hugetlb_shm_group的最后一步是给mysql用户一个memlock限制的“无限制”值。这可以通过编辑/etc/security/limits来完成。conf或将以下命令添加到[mysqldsafe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)脚本：

`ulimit -l unlimited`

将ulimit命令添加到mysqld_safe会导致root用户在切换到mysql用户之前将memlock限制设置为unlimited。（这假设mysqld_safe是由root启动的。）

默认情况下，MySQL中的大页面支持被禁用。要启用它，请使用--largepages选项启动服务器。例如，您可以在服务器my中使用以下行。cnf文件：

```ini
[mysqld]
large-pages
```

通过此选项，InnoDB将自动使用大页面作为其缓冲池和附加内存池。如果InnoDB无法做到这一点，它将退回到使用传统内存，并将警告写入错误日志：警告：使用传统内存池

要验证是否正在使用大页面，请再次检查/proc/meminfo：

```sql
$> cat /proc/meminfo | grep -i huge
HugePages_Total:      20
HugePages_Free:       20
HugePages_Rsvd:        2
HugePages_Surp:        0
Hugepagesize:       4096 kB
```
