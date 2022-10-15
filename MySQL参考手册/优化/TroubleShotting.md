# 操作手册

## 高内存

1. 查询系统内存分配状况

    ```sql
    SELECT SUBSTRING_INDEX(event_name,'/',2) AS
        code_area, FORMAT_BYTES(SUM(current_alloc))
        AS current_alloc
        FROM sys.x$memory_global_by_current_bytes
        GROUP BY SUBSTRING_INDEX(event_name,'/',2)
        ORDER BY SUM(current_alloc) DESC;
    ```

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

2. 检查 innodb 缓存池配置
   1. 确认 `select @@innodb_buffer_pool_size` 设置：占系统内存 50%-75% 。
   2. 确认 `select @@innodb_buffer_pool_instances` 设置：小于或等于CPU核心数，确保有效提升并发性能。

3. 检查 temptable 配置
   1. 目标是尽量减少 temptable 数量；尽量避免生成磁盘临时表。
   2. 确认 `select @@internal_tmp_mem_storage_engine;` 为 TempTable，若系统内存充足可考虑设置为 MEMORY。
   3. `show variables where Variable_name like 'temptable_max_mmap' or Variable_name like 'tmp_table_size' or Variable_name like 'temptable_max_ram';`
      1. 确认 tmp_table_size 设置：若无法避免 大表+多级非直接关联+排序 查询，可视系统资源情况谨慎加大512M-1G-2G，但要防止单个查询消耗过多的全局 TempTable 资源。
         1. `SET GLOBAL tmp_table_size =  536870912`
      2. 内存中的内部临时表大于 tmp_table_size 和 max_heap_table_size ，MySQL会**自动将表从内存中转换为磁盘上格式**，在资源许可情况下谨慎加大。
         1. `SET GLOBAL temptable_max_ram =  4294967296` AND `SET GLOBAL temptable_max_mmap = 4294967296`
   4. 定期检查 `show global status like 'created_tmp%';` 确认 Created_tmp_disk_tables / Created_tmp_tables * 100% <= 25% 。
      1. 检查 慢查询日志 若是其缓存在 disk 上的都大型临时表，将对性能产生巨大影响，必须优化查询语句。
         1. 优化关联查询，尽量避免生成 temptable ，无法避免时严格控制大小（数据最小化）。
         2. 优化排序，尽量避免关联表排序，无法避免时严格控制大小（数据最小化）。

4. 监控内部临时表创建
   1. 查看监控工具是否启用 `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'memory/%';`，输出所有 EVENT_NAME ，重点关注 innodb AND temptable。
      1. `SELECT * from performance_schema.setup_instruments WHERE NAME LIKE 'memory/innodb/%';`
   2. 启用监控: `UPDATE performance_schema.setup_instruments SET ENABLED = 'YES' WHERE NAME LIKE 'memory/%';`
   3. 查看内存事件摘要信息
      1. 按 USER、HOST 和 EVENT_NAME(temptable) 查询
         1. `SELECT * FROM performance_schema.memory_summary_by_account_by_event_name WHERE EVENT_NAME = 'memory/temptable/physical_disk'`
         2. `SELECT * FROM performance_schema.memory_summary_by_account_by_event_name WHERE EVENT_NAME = 'memory/temptable/physical_ram'`
      2. 按 USER、HOST 和 EVENT_NAME(innodb) 查询
         1. ?
         2. ?
         3. ?

5. 监控用户查询
   1. `show processlist;`
