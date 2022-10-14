# 配置 InnoDB 缓冲池大小

<https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-resize.html>

您可以离线或在线（服务器运行时）配置 InnoDB 缓冲池大小。

当增加或减少 innodb_buffer_pool_size 时，操作是分块执行的(the operation is performed in chunks)。块大小由 innodb_buffer_pool_chunk_size 配置选项定义，默认为 128M。

缓冲池大小必须始终等于或倍数：

PS = innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances

如果将 innodb_buffer_pool_size 配置为不等于或倍数的PS值，则缓冲池大小会自动调整为等于或倍数的PS值。

在下面的示例中，innodb_buffer_pool_size 设置为 8G，innodb_buffer_pool_instances 设置为 16。innodb_buffer_pool_chunk_size 为 128M，这是默认值。

8G 是有效的 innodb_buffer_pool_size 值，因为 8G 是 innodb_buffer_pool_instances=16 * innodb_buffer_pool_chunk_size=128M 的倍数，即 2G。

` $> mysqld --innodb-buffer-pool-size=8G --innodb-buffer-pool-instances=16 `

```sql
mysql> SELECT @@innodb_buffer_pool_size/1024/1024/1024;
+------------------------------------------+
| @@innodb_buffer_pool_size/1024/1024/1024 |
+------------------------------------------+
|                           8.000000000000 |
+------------------------------------------+
```

## 配置 InnoDB 缓冲池块大小

innodb_buffer_pool_chunk_size 可以以 1MB（1048576 字节）为单位增加或减少，但只能在启动时、命令行字符串或 MySQL 配置文件中修改。

`$> mysqld --innodb-buffer-pool-chunk-size=1342177288`

```bash
[mysqld]
innodb_buffer_pool_chunk_size=134217728
```

更改 innodb_buffer_pool_chunk_size 时适用以下条件：

* 如果缓冲池初始化时新的innodb_buffer_pool_chunk_size值乘innodb_buffer_pool_instances大于当前缓冲池大小，则innodb_buffer_pool_chunk_size被截断为innodb_buffer_pool_size/innodb_buffer_pool_instances。
  例如，如果缓冲池初始化为大小为 2GB（2147483648 字节），4 个缓冲池实例，块大小为 1GB（1073741824 字节），则块大小被截断为等于 innodb_buffer_pool_size / innodb_buffer_pool_instances 的值，如图所示 以下：

```bash
$> mysqld --innodb-buffer-pool-size=2147483648 --innodb-buffer-pool-instances=4
--innodb-buffer-pool-chunk-size=1073741824;
```

```sql
mysql> SELECT @@innodb_buffer_pool_size;
+---------------------------+
| @@innodb_buffer_pool_size |
+---------------------------+
|                2147483648 |
+---------------------------+

mysql> SELECT @@innodb_buffer_pool_instances;
+--------------------------------+
| @@innodb_buffer_pool_instances |
+--------------------------------+
|                              4 |
+--------------------------------+

# Chunk size was set to 1GB (1073741824 bytes) on startup but was
# truncated to innodb_buffer_pool_size / innodb_buffer_pool_instances

mysql> SELECT @@innodb_buffer_pool_chunk_size;
+---------------------------------+
| @@innodb_buffer_pool_chunk_size |
+---------------------------------+
|                       536870912 |
+---------------------------------+
```

* 缓冲池大小必须始终等于或倍数 PS 值。 如果更改 innodb_buffer_pool_chunk_size，innodb_buffer_pool_size 会自动调整为等于或倍数 PS 的值。 调整发生在缓冲池初始化时。 以下示例演示了此行为：

```sql
# The buffer pool has a default size of 128MB (134217728 bytes)

mysql> SELECT @@innodb_buffer_pool_size;
+---------------------------+
| @@innodb_buffer_pool_size |
+---------------------------+
|                 134217728 |
+---------------------------+

# The chunk size is also 128MB (134217728 bytes)

mysql> SELECT @@innodb_buffer_pool_chunk_size;
+---------------------------------+
| @@innodb_buffer_pool_chunk_size |
+---------------------------------+
|                       134217728 |
+---------------------------------+

# There is a single buffer pool instance

mysql> SELECT @@innodb_buffer_pool_instances;
+--------------------------------+
| @@innodb_buffer_pool_instances |
+--------------------------------+
|                              1 |
+--------------------------------+

# Chunk size is decreased by 1MB (1048576 bytes) at startup
# (134217728 - 1048576 = 133169152):

$> mysqld --innodb-buffer-pool-chunk-size=133169152

mysql> SELECT @@innodb_buffer_pool_chunk_size;
+---------------------------------+
| @@innodb_buffer_pool_chunk_size |
+---------------------------------+
|                       133169152 |
+---------------------------------+

# Buffer pool size increases from 134217728 to 266338304
# Buffer pool size is automatically adjusted to a value that is equal to
# or a multiple of innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances

mysql> SELECT @@innodb_buffer_pool_size;
+---------------------------+
| @@innodb_buffer_pool_size |
+---------------------------+
|                 266338304 |
+---------------------------+
```

```sql
# The buffer pool has a default size of 2GB (2147483648 bytes)

mysql> SELECT @@innodb_buffer_pool_size;
+---------------------------+
| @@innodb_buffer_pool_size |
+---------------------------+
|                2147483648 |
+---------------------------+

# The chunk size is .5 GB (536870912 bytes)

mysql> SELECT @@innodb_buffer_pool_chunk_size;
+---------------------------------+
| @@innodb_buffer_pool_chunk_size |
+---------------------------------+
|                       536870912 |
+---------------------------------+

# There are 4 buffer pool instances

mysql> SELECT @@innodb_buffer_pool_instances;
+--------------------------------+
| @@innodb_buffer_pool_instances |
+--------------------------------+
|                              4 |
+--------------------------------+

# Chunk size is decreased by 1MB (1048576 bytes) at startup
# (536870912 - 1048576 = 535822336):

$> mysqld --innodb-buffer-pool-chunk-size=535822336

mysql> SELECT @@innodb_buffer_pool_chunk_size;
+---------------------------------+
| @@innodb_buffer_pool_chunk_size |
+---------------------------------+
|                       535822336 |
+---------------------------------+

# Buffer pool size increases from 2147483648 to 4286578688
# Buffer pool size is automatically adjusted to a value that is equal to
# or a multiple of innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances

mysql> SELECT @@innodb_buffer_pool_size;
+---------------------------+
| @@innodb_buffer_pool_size |
+---------------------------+
|                4286578688 |
+---------------------------+
```

更改 innodb_buffer_pool_chunk_size 时应小心，因为更改此值会增加缓冲池的大小，如上面的示例所示。 在更改 innodb_buffer_pool_chunk_size 之前，计算对 innodb_buffer_pool_size 的影响，以确保生成的缓冲池大小是可以接受的。

> 笔记
为避免潜在的性能问题，块的数量（innodb_buffer_pool_size / innodb_buffer_pool_chunk_size）不应超过 1000。

## 在线配置 InnoDB 缓冲池大小

innodb_buffer_pool_size 配置选项可以使用 SET 语句动态设置，允许您在不重新启动服务器的情况下调整缓冲池的大小，此方法取值必须用字节数表示。

`mysql> SET GLOBAL innodb_buffer_pool_size=603979776;`

>笔记
缓冲池大小必须等于或倍数 innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances。更改这些变量设置需要重新启动服务器。

通过 InnoDB API 执行的活动事务和操作(Active transactions and operations)应在调整缓冲池大小之前完成。启动调整大小操作时，直到所有活动事务都完成后操作才会开始。一旦调整大小操作正在进行，需要访问缓冲池的新事务和操作必须等到调整大小操作完成。该规则的例外是允许对缓冲池进行并发(concurrent access)访问，同时对缓冲池进行碎片整理，并在缓冲池大小减小时撤回页面。允许并发访问的一个缺点是它可能导致在页面被撤回(withdrawn)时可用页面暂时短缺。

>笔记
Nested transactions could fail if initiated after the buffer pool resizing operation begins.
如果在缓冲池大小调整操作开始后启动嵌套事务，则可能会失败。

### 监控在线缓冲池大小调整进度

Innodb_buffer_pool_resize_status 报告缓冲池大小调整进度。

```sql
mysql> SHOW STATUS WHERE Variable_name='InnoDB_buffer_pool_resize_status';
+----------------------------------+----------------------------------+
| Variable_name                    | Value                            |
+----------------------------------+----------------------------------+
| Innodb_buffer_pool_resize_status | Resizing also other hash tables. |
+----------------------------------+----------------------------------+
```

缓冲池大小调整进度也记录在服务器错误日志中。 此示例显示了增加缓冲池大小时记录的注释：

```log
[Note] InnoDB: Resizing buffer pool from 134217728 to 4294967296. (unit=134217728)
[Note] InnoDB: disabled adaptive hash index.
[Note] InnoDB: buffer pool 0 : 31 chunks (253952 blocks) was added.
[Note] InnoDB: buffer pool 0 : hash tables were resized.
[Note] InnoDB: Resized hash tables at lock_sys, adaptive hash index, dictionary.
[Note] InnoDB: completed to resize buffer pool from 134217728 to 4294967296.
[Note] InnoDB: re-enabled adaptive hash index.
```

此示例显示减小缓冲池大小时记录的注释：

```log
[Note] InnoDB: Resizing buffer pool from 4294967296 to 134217728. (unit=134217728)
[Note] InnoDB: disabled adaptive hash index.
[Note] InnoDB: buffer pool 0 : start to withdraw the last 253952 blocks.
[Note] InnoDB: buffer pool 0 : withdrew 253952 blocks from free list. tried to relocate 0 pages.
(253952/253952)
[Note] InnoDB: buffer pool 0 : withdrawn target 253952 blocks.
[Note] InnoDB: buffer pool 0 : 31 chunks (253952 blocks) was freed.
[Note] InnoDB: buffer pool 0 : hash tables were resized.
[Note] InnoDB: Resized hash tables at lock_sys, adaptive hash index, dictionary.
[Note] InnoDB: completed to resize buffer pool from 4294967296 to 134217728.
[Note] InnoDB: re-enabled adaptive hash index.
```

### 在线缓冲池大小调整内部结构

The resizing operation is performed by a background thread.
调整大小操作由后台线程执行。 增加缓冲池大小时，调整大小操作如下：

* 以块(chunks)的形式添加页面 (块大小由innodb_buffer_pool_chunk_size定义)
* Converts hash tables, lists, and pointers to use new addresses in memory 转换哈希表、列表和指针以使用内存中的新地址
* Adds new pages to the free list 将新页面添加到空闲列表

当这些操作正在进行时，其他线程被阻止访问缓冲池。

减小缓冲池大小时，调整大小操作:

* Defragments the buffer pool and withdraws (frees) pages 对缓冲池进行碎片整理并撤回（释放）页面
* 删除块中的页面(块大小由 innodb_buffer_pool_chunk_size 定义)
* 转换哈希表、列表和指针以使用内存中的新地址

Of these operations, only defragmenting the buffer pool and withdrawing pages allow other threads to access to the buffer pool concurrently.
在这些操作中，只有对缓冲池进行碎片整理和撤回页面允许其他线程同时访问缓冲池。
