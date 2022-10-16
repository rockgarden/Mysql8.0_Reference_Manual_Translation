# 操作手册

## InnoDB 监控

1. 检查监控器是否启动 `SELECT @@innodb_status_output;`
   1. 启用 `SET GLOBAL innodb_status_output=ON;`
2. 查看状态 `SHOW ENGINE INNODB STATUS`
   1. 信号等待: SEMAPHORES
      1. 若有大量线程等待信号量，可能是磁盘I/O或InnoDB内部争用问题的结果。
      2. 进一步查看 `SHOW ENGINE INNODB MUTEX`
         1. 检查InnoDB Mutex等待状况
            1. 是否启用 `SELECT * FROM performance_schema.setup_instruments WHERE NAME LIKE '%wait/synch/mutex/innodb%';`
            2. 查看数据：事件名（EVENT_NAME）、等待次数（COUNT_STAR）、总等待时间（SUM_TIMER_wait）

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
        AS current_alloc
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

2. 检查监控模式是否启用
   1. 性能模式是否启用
      1. `SHOW VARIABLES LIKE 'performance_schema';`
         1. 全部监控工具启用命令（不建议）`UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES';`
   2. 查看服务器目前正在执行的监控事务 `SELECT * FROM performance_schema.events_waits_current`，每个线程包含一行，显示每个线程最近监视的事件。
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
      2. 按 USER、HOST 分组查询
      `SELECT * FROM performance_schema.memory_summary_by_account_by_event_name a WHERE EVENT_NAME LIKE 'memory/innodb/%' AND CURRENT_NUMBER_OF_BYTES_USED > 0 ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC;`
      3. 按 THREAD_ID 分组查询
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
            3. 设置命令 `SET GLOBAL tmp_table_size =  536870912`。
         2. 确认 temptable_max_ram temptable_max_mmap 设置：
            1. 内存中的内部临时表大于 tmp_table_size 和 max_heap_table_size ，MySQL会**自动将表从内存中转换为磁盘上格式**，在资源许可情况下谨慎加大。
            2. 设置命令 `SET GLOBAL temptable_max_ram =  4294967296` AND `SET GLOBAL temptable_max_mmap = 4294967296`
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
   1. 查看进程列表 `SHOW FULL PROCESSLIST;`
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

7. 运行一段时间后，将所有参数配置写入配置文件固化，并关闭非核心的监控功能。
