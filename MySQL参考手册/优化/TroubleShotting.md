# 操作手册

## InnoDB 监控

1. 检查监控器是否启动 `SELECT @@innodb_status_output;`
   1. 启用 `SET GLOBAL innodb_status_output=ON;`
2. 查看状态 `SHOW ENGINE INNODB STATUS;`
   1. 信号等待: SEMAPHORES
      1. 若有大量线程等待信号量，可能是磁盘I/O或InnoDB内部争用问题的结果。
      2. 进一步查看 `SHOW ENGINE INNODB MUTEX;`
         1. 检查InnoDB Mutex等待状况
            1. 是否启用 `SELECT * FROM performance_schema.setup_instruments WHERE NAME LIKE '%wait/synch/mutex/innodb%';`
            2. 启用 `UPDATE performance_schema.setup_instruments SET ENABLED = 'YES' , TIMED = 'YES' WHERE NAME LIKE '%wait/synch/mutex/innodb%';`
            3. 查看数据：事件名（EVENT_NAME）、等待次数（COUNT_STAR）、总等待时间（SUM_TIMER_wait）

               ```sql
               SELECT EVENT_NAME, COUNT_STAR, SUM_TIMER_WAIT/1000000000 SUM_TIMER_WAIT_MS
                  FROM performance_schema.events_waits_summary_global_by_event_name
                  WHERE SUM_TIMER_WAIT > 0 AND EVENT_NAME LIKE 'wait/synch/mutex/innodb/%'
                  ORDER BY COUNT_STAR DESC;
               ```

      3. 若无异常，即争用可能是由于查询的高度并行性或操作系统线程调度中引起。可将 innodb_thread_concurrency 系统变量设置为小于默认值可能会有所帮助。
   2. 缓冲池统计: BUFFER POOL AND MEMORY
      1. 进一步查看 `SELECT * FROM information_schema.INNODB_BUFFER_POOL_STATS`
      2. 关注 Buffer pool hit rate > 99%

## 内存占用过高

1. 查询系统内存分配状况

    ```sql
   SELECT SUBSTRING_INDEX(event_name,'/',2) AS
      code_area, FORMAT_BYTES(SUM(current_alloc))
      AS current_alloc, FORMAT_BYTES(SUM(high_alloc))
      AS high_alloc, FORMAT_BYTES(SUM(high_avg_alloc))
      AS high_avg_alloc
      FROM sys.x$memory_global_by_current_bytes
      GROUP BY SUBSTRING_INDEX(event_name,'/',2)
      ORDER BY SUM(current_alloc) DESC;
    ```

    结果示例：

    ```log
    +---------------------------+---------------+
    | code_area                 | current_alloc |
    +---------------------------+---------------+
    | memory/innodb             | 6.40 GiB      |
    | memory/performance_schema | 313.01 MiB    |
    | memory/sql                | 127.99 MiB    |
    | memory/mysys              | 15.20 MiB     |
    | memory/temptable          | 4.00 MiB      |
    | memory/mysqld_openssl     | 728.20 KiB    |
    | memory/myisam             | 36.28 KiB     |
    | memory/mysqlx             | 2.76 KiB      |
    | memory/csv                |  376 bytes    |
    | memory/blackhole          |  376 bytes    |
    | memory/refcache           |  336 bytes    |
    | memory/vio                |   80 bytes    |
    +---------------------------+---------------+
    ```

   1. `SHOW ENGINES;`
      1. 可考虑在配置文件中禁用不使用的ENGINE 如 MyISAM，参见 [disabled_storage_engines](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_disabled_storage_engines)。
   2. 内存消耗分类
      1. global级共享内存：`show variables where variable_name in ('innodb_buffer_pool_size','innodb_log_buffer_size');`
      2. session级私有内存：`show variables where variable_name in ('tmp_table_size','sort_buffer_size','read_buffer_size','read_rnd_buffer_size','join_buffer_size','thread_stack', 'binlog_cache_size');`
         1. tmp_table_size：是MySQL的heap（堆积）表缓冲大小，表示内存中临时表的最大值。如果内存临时表超出了限制，MySQL就会自动地把它转化为基于磁盘的MyISAM表，存储在指定的tmpdir目录下
         2. sort_buffer_size：执行ORDER BY和GROUP BY排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。如果不能，可以尝试增加sort_buffer_size变量的大小。若存储量大于 sort_buffer_size，则会在磁盘生成临时表以完成操作。在 Linux 系统中，当分配空间大于 2 M 时会使用 mmap() 而不是 malloc() 来进行内存分配，导致效率降低。
         3. read_buffer_size：顺序读缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySQL会为它分配一段内存缓冲区。
         4. read_rnd_buffer_size：随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySQL会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySQL会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
         5. join_buffer_size：每次join操作都会调用my_malloc、my_free函数申请/释放join_buffer_size的大小的内存。
         6. thread_stack：每个session连接线程被创建时，MySQL给它分配的内存大小。当MySQL创建一个新的连接线程时，需要给它分配一定大小的内存堆栈空间，以便存放客户端的请求的Query及自身的各种状态和处理信息。查看连接线程相关的系统变量的设置值 `show variables like 'thread%';`
         7. binlog_cache_size：为每个 session 分配的内存，在事务过程中用来存储二进制日志的缓存。表示的是binlog 能够使用的最大cache 内存大小。在一个事务还没有 commit 之前会先将其日志存储于 binlog_cache 中，等到事务 commit 后会将其 binlog 刷回磁盘上的 binlog 文件以持久化。当我们执行多语句事务的时候 所有session的使用的内存超过max_binlog_cache_size的值时就会报错：“Multi-statement transaction required more than 'max_binlog_cache_size' bytes ofstorage”
   3. 估算内存占用 <http://www.mysqlcalculator.com/>
      1. key_buffer_size + query_cache_size + tmp_table_size + innodb_buffer_pool_size + innodb_additional_mem_pool_size + innodb_log_buffer_size + max_connections * (sort_buffer_size + read_buffer_size + read_rnd_buffer_size + join_buffer_size + thread_stack + binlog_cache_size)

2. 检查监控模式是否启用
   1. 性能模式是否启用
      1. `SHOW VARIABLES LIKE 'performance_schema';`
      2. 查看监控工具启用状况 `SELECT * from performance_schema.setup_instruments;`
         1. 全部监控工具启用命令（不建议）`UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES';`
   2. 查看服务器目前正在执行的监控事务 `SELECT * FROM performance_schema.events_waits_current;`，每个线程包含一行，显示每个线程最近监视的事件。
      1. 查询历史监控事务 `SELECT EVENT_ID, EVENT_NAME, TIMER_WAIT FROM performance_schema.events_waits_history_long ORDER BY EVENT_ID;`
   3. 查看内存监控工具是否启用 `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'memory/%';`，输出所有 memory 相关 EVENT_NAME。
      1. 运行时启用命令: `UPDATE performance_schema.setup_instruments SET ENABLED = 'YES' WHERE NAME LIKE 'memory/%';`。
   4. 其它监控工具
      1. `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'wait/%';`
      2. `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'stage/%';`
      3. `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'statement/%';`
   5. 查看所有memory分配统计 `SELECT * from sys.memory_global_by_current_bytes WHERE current_count > 0;` 逐一排查。

3. 检查Innodb内存占用
   1. 查看监控工具是否启用 输出所有 innodb 相关 EVENT_NAME。
      `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'memory/innodb/%';`，
   2. 查看内存事件摘要信息
      1. 按 EVENT_NAME(innodb%) 查询
      `SELECT * FROM performance_schema.memory_summary_global_by_event_name a WHERE EVENT_NAME LIKE 'memory/innodb/%' AND CURRENT_NUMBER_OF_BYTES_USED > 0 ORDER BY SUM_NUMBER_OF_BYTES_ALLOC DESC;`。
      1. 按 USER、HOST 分组查询
      `SELECT * FROM performance_schema.memory_summary_by_account_by_event_name a WHERE EVENT_NAME LIKE 'memory/innodb/%' AND CURRENT_NUMBER_OF_BYTES_USED > 0 ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC;`
      1. 按 THREAD_ID 分组查询
      `SELECT * FROM performance_schema.memory_summary_by_thread_by_event_name a WHERE EVENT_NAME LIKE 'memory/innodb/%' AND CURRENT_NUMBER_OF_BYTES_USED > 0 ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC;`
   3. `SHOW STATUS LIKE 'Innodb_buffer%';`
   4. 检查innodb缓存池配置
      1. 确认 `select @@innodb_buffer_pool_size;` 设置：占系统内存 50%-75% ；如 32G 内存 取 8 G。
      2. 确认 `select @@innodb_buffer_pool_instances;` 设置：小于或等于CPU核心数，确保有效提升并发性能。
      3. 确认 `SELECT @@innodb_buffer_pool_chunk_size;` 设置：（innodb_buffer_pool_size / innodb_buffer_pool_chunk_size）< 1000。

4. 检查temptable内存占用
   1. 查看监控工具是否启用 `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'memory/temptable/%';`，输出所有 temptable 相关 EVENT_NAME。
   2. 查看内存事件摘要信息
      1. 按 USER、HOST 和 EVENT_NAME(temptable) 查询
         `SELECT * FROM performance_schema.memory_summary_by_account_by_event_name WHERE EVENT_NAME LIKE 'memory/temptable/%' AND SUM_NUMBER_OF_BYTES_ALLOC > 0 ORDER BY SUM_NUMBER_OF_BYTES_ALLOC DESC;`
      2. 按 THREAD_ID 和 EVENT_NAME(temptable) 查询，可用于定位最耗内存的事件
         `SELECT * FROM performance_schema.memory_summary_by_thread_by_event_name WHERE EVENT_NAME LIKE 'memory/temptable/%' AND SUM_NUMBER_OF_BYTES_ALLOC > 0 ORDER BY SUM_NUMBER_OF_BYTES_ALLOC DESC;`
   3. 检查 temptable 配置
      1. 目标是尽量减少 temptable 数量；尽量避免生成磁盘临时表。
      2. 确认 `select @@internal_tmp_mem_storage_engine;` 为 TempTable ，若系统内存充足可考虑设置为 MEMORY。
         MEMORY 配置参数 max_heap_table_size
      3. 查看 TempTable engine 配置
         `show variables where Variable_name like 'temptable_max_mmap' or Variable_name like 'tmp_table_size' or Variable_name like 'temptable_max_ram';`
         1. 确认 tmp_table_size 设置：
            1. 运行最复杂查询（如多个大表关联查询），通过内存事件摘要信息(HIGH_NUMBER_OF_BYTES_ALLOC)定位最耗内存的事件，确定 tmp_table_size 目标值。
            2. 若无法避免 大表+多级非直接关联+排序 查询，可视系统资源情况谨慎加大512M-1G-2G，但要防止单个查询消耗过多的全局 TempTable 资源。
            3. 全局设置命令 `SET GLOBAL tmp_table_size = 536870912;` 512M
               1. 当前SESSION：`SET SESSION tmp_table_size = 536870912;`
         2. 确认 temptable_max_ram temptable_max_mmap 设置：
            1. 内存中的内部临时表大于 tmp_table_size 和 max_heap_table_size ，MySQL会**自动将表从内存中转换为磁盘上格式**，在资源许可情况下谨慎加大。
            2. 设置命令 `SET GLOBAL temptable_max_ram = 4294967296;` AND `SET GLOBAL temptable_max_mmap = 4294967296;` 4G
   4. 定期检查 `show global status like 'created_tmp%';` 确认 Created_tmp_disk_tables / Created_tmp_tables * 100% <= 25% 。
      1. 检查 慢查询日志 若是其缓存在 disk 上的都大型临时表，将对性能产生巨大影响，必须优化查询语句。
         1. 优化关联查询，尽量避免生成 temptable ，无法避免时严格控制大小（数据最小化）。
         2. 优化排序，尽量避免关联表排序，无法避免时严格控制大小（数据最小化）。

5. 检查sql内存占用
   1. 查看监控工具是否启用 `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'memory/sql/%';`，输出所有 sql 相关 EVENT_NAME。
   2. 查看内存事件摘要信息
      1. 定位异常 EVENT_NAME
         `SELECT * from sys.memory_global_by_current_bytes WHERE EVENT_NAME LIKE 'memory/sql/%' AND current_count > 0;`
      2. 示例，若 EVENT_NAME 为 memory/sql/TABLE 的内存占用异常，则用 memory/sql/TABLE% 进行分组查询
         `SELECT * FROM performance_schema.memory_summary_global_by_event_name WHERE EVENT_NAME = 'memory/sql/TABLE';`
         1. 按 USER、HOST 分组查询，可用于定位最耗内存的用户
         `SELECT * FROM performance_schema.memory_summary_by_account_by_event_name WHERE EVENT_NAME LIKE 'memory/sql/TABLE%' AND SUM_NUMBER_OF_BYTES_ALLOC > 0 ORDER BY SUM_NUMBER_OF_BYTES_ALLOC DESC;`
         1. 按 THREAD_ID 分组查询，可用于定位最耗内存的事件
         `SELECT * FROM performance_schema.memory_summary_by_thread_by_event_name WHERE EVENT_NAME LIKE 'memory/sql/TABLE%' AND SUM_NUMBER_OF_BYTES_ALLOC > 0 ORDER BY SUM_NUMBER_OF_BYTES_ALLOC DESC;`

6. 监控进程列表
   1. 查看进程列表 `SHOW FULL PROCESSLIST;` 可估算为 session 数。
      等效查询 `SELECT * FROM performance_schema.processlist;`
      1. 通过 USER HOST 确认 当前用户 的 ID（process）
   2. 处理State
      1. State - Sending to client
         若看到State长时间显示为“Sending to client”，说明服务端发送阻塞，服务器端的网络栈写满了；业务开发同学优化查询结果，并评估这么多的返回结果是否合理。
         若要快速减少处于这个状态的线程的话，可以将net_buffer_length设置更大。
         > 说明
         若客户端使用–quick参数，会使用mysql_use_result方法：读一行处理一行。假设某业务的逻辑较复杂，每读一行数据以后要处理的逻辑若很慢，就会导致客户端要过很久才取下一行数据，可能就会出现上图结果。
         因此，对于正常的线上业务来说，若一个查询的返回结果不多，推荐使用mysql_store_result接口，直接把查询结果保存到本地内存。
      2. State - Sending data
         先确认网络是否正常；业务查询逻辑过于复杂，执行扫描全表。
         > 说明
         MySQL查询语句进入执行阶段后，就把状态设置成Sending data，发送执行结果的列相关的信息（meta data) 给客户端，再继续执行语句流程执行完成后，把状态设置成空字符串，即“Sending data”并不一定是指“正在发送数据”，而可能是处于执行器过程中的任意阶段。
      3. State - init
         这发生在ALTER TABLE、DELETE、INSERT、SELECT或UPDATE语句初始化之前。
         服务器在此状态下采取的操作包括刷新二进制日志和InnoDB日志。
   3. 删除异常PROCESS `KILL [CONNECTION | QUERY] processlist_id`

7. 查看线程表
   1. `SELECT * FROM performance_schema.threads;`
      PROCESSLIST表的HOST列不同，PROCESSLIST_HOST列不包括TCP/IP连接的端口号。
      1. 确认有否启用套接字检测工具 `SELECT NAME, ENABLED, TIMED FROM performance_schema.setup_instruments WHERE NAME LIKE 'wait/io/socket%';` 默认情况下未启用
         1. 启用命令 `UPDATE performance_schema.setup_instruments SET ENABLED='YES' WHERE NAME LIKE 'wait/io/socket%';`
         2. 检查 `SELECT * FROM performance_schema.socket_instances;`
      2. MySQL 8.0.31 提供 CONTROLLED_MEMORY、MAX_CONTROLLED_MEMORY、TOTAL_MEMORY、MAX_TOTAL_MEMORY 内存占用数据。

8. 运行一段时间后，将所有参数配置写入配置文件固化，并关闭非核心的监控功能。

## Session优化

1. 查询会话临时表空间占用情况
   `mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_SESSION_TEMP_TABLESPACES;`

   | ID进程  | SPACE      | PATH                      | SIZE       | STATE  | PURPOSE   |
   |--------|------------|---------------------------|------------|--------|-----------|
   | 238696 | 4243767283 | .\innodb_temp\temp_3.ibt  | 1153433600 | ACTIVE | INTRINSIC |

2. 查询进程（238696）
    `SHOW FULL PROCESSLIST;`

   | ID     | USER | HOST            | DB   | COMMAND | TIME  | STATE | INFO | EXECUTION_ENGINE |
   |--------|------|-----------------|------|---------|-------|-------|------|------------------|
   | 238696 | ecnt | localhost:63610 | ecnt | Sleep   | 22919 |       |      | PRIMARY          |

## View优化

### v_pom_huawei

1. 运行目标视图

   ```sql
   TRUNCATE TABLE performance_schema.memory_summary_global_by_event_name;
   FLUSH STATUS;
   SELECT count(*) FROM ecnt.v_pom_huawei;
   SHOW STATUS LIKE 'Created_tmp%';
   SHOW STATUS LIKE 'Handler%';
   SHOW FULL PROCESSLIST;
   SHOW SESSION VARIABLES;
   ```

   > 注意：TRUNCATE memory_summary_global_by_event_name 会重置所有内存统计基线

2. 分析运行状况
   1. 临时表创建
      `SHOW STATUS LIKE 'Created_tmp%';`
      1. Created_tmp_disk_tables = 1
         1. 查看参数
            `show variables where Variable_name like 'temptable_max_mmap' or Variable_name like 'tmp_table_size' or Variable_name like 'temptable_max_ram';`
         2. 将内存临时表空间限制加到系统允许的最大值 `SET SESSION tmp_table_size = 2147483648;`
            由于 physical_disk 分配值无法获取，只能获得 physical_ram 分配值，所以重新运行目标视图，获取临时表总分配空间，可测量的上限值即 tmp_table_size (MAX)
         3. 重新 *运行目标视图* ，查询 temptable 内存占用情况（根据 THREAD_ID 判断）
            `SELECT * FROM performance_schema.memory_summary_by_thread_by_event_name WHERE EVENT_NAME LIKE 'memory/temptable/%' AND SUM_NUMBER_OF_BYTES_ALLOC > 0 ORDER BY SUM_NUMBER_OF_BYTES_ALLOC DESC;`
         4. HIGH_NUMBER_OF_BYTES_USED = 1610613120，大于 GLOBAL tmp_table_size = 512M，说明生成的临时表过大。
            1. 加大内存临时表上限值：tmp_table_size = 1073741824， temptable_max_ram = 4294967296
               1. 在现在硬件系统的条件下，且负荷较低，对SSD磁盘无争用的情况下，性能提升基本忽略（10%以内）
            2. 减小 temptable 大小
   2. 读写统计
      `SHOW STATUS LIKE 'Handler%';`
      1. Handler_read_rnd_next = 2524392
         1. 读取数据文件中下一行的请求数巨大。进行大量的表扫描，表没有被正确索引，或查询排除使用索引。
      2. Handler_write = 1262195
         1. 在临时表表中插入行的请求数，即创建大型临时表。
      3. Handler_read_next = 0
         1. 按键顺序读取下一行的请求数。如果正在使用范围约束查询索引列或正在进行索引扫描，则此值将递增。

3. 分析语句

   ```sql
   EXPLAIN FORMAT=JSON SELECT count(*) FROM ecnt.v_pom_huawei;
   SHOW WARNINGS;
   ```

   WARNINGS 输出：

   ```sql
   /* select#2 */ 
   SELECT `ecnt`.`a`.`engInfoEngineeringName` FROM `ecnt`.`polist_huawei` `a` WHERE ( `ecnt`.`a`.`poNumber` = `ecnt`.`pocreateexp`.`poUnionRelease` ) LIMIT 1
   ```

   1. `/* select#2 */` Select_type = DERIVED, type = ALL
      查询生成了DERIVED表，且优化器无法将基表的基础索引应用到DERIVED表（多对多关系），导致性能低下
      1. polist_huawei 与 pocreateexp 建立外键关系
      2. 基于 polist_huawei 建立 pocreateexp 要关联查询信息的中间表
      3. 临时处理： 对 DERIVED表 强制分组或去重 优化器将创建 index
         `SELECT department_id, poNumber FROM polist_huawei GROUP BY poNumber`

> 派生表的具体化适用于公用表表达式 (CTE)。此外，以下注意事项特别适用于 CTE。
如果 CTE 由查询具体化，则它会为查询具体化一次，即使查询多次引用它也是如此。
递归 CTE 总是物化的。
如果 CTE 被具体化，如果优化器估计索引可以加快顶层语句对 CTE 的访问，则它会自动添加相关索引。这类似于派生表的自动索引，只是如果 CTE 被多次引用，优化器可能会创建多个索引，以便以最合适的方式加速每个引用的访问。
TODO：这个自动添加的索引是否是由关键词 DISTINCT 触发？如何控制？

### v_pom_order_huawei_bind_use

1. Select_type = SIMPLE, type = ALL
   tpye为ALL，添加 contractid + projectid 组合索引，但对性能无提升，这是由于基表 pom_order_huawei_bind 是一张关联用的中间表，索引 与 全表 扫描差异小。
