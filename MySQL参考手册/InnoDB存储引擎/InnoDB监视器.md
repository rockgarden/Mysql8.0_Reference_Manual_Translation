# InnoDB监视器

15.17.1 InnoDB监视器类型
15.17.2 启用InnoDB监视器
15.17.3 InnoDB标准监视器和锁定监视器输出

InnoDB监视器提供有关InnoDB内部状态的信息。此信息对于性能调整很有用。

## InnoDB监视器类型

InnoDB监视器有两种类型：

- 标准InnoDB监视器显示以下类型的信息：

  - 主后台线程完成的工作

  - 信号等待

  - 有关最近的外键和死锁错误的数据

  - 锁定等待事务

  - 活动事务持有的表和记录锁

  - 挂起的I/O操作和相关统计信息

  - 插入缓冲区和自适应哈希索引统计信息

  - 重做日志数据

  - 缓冲池统计

  - 行操作数据

- InnoDB锁定监视器打印额外的锁定信息，作为标准InnoDB监视器输出的一部分。

## 启用InnoDB监视器

当InnoDB监视器启用定期输出时，InnoDB大约每隔15秒将输出写入mysqld服务器标准错误输出（stderr）。

InnoDB将监视器输出发送到stderr，而不是stdout或固定大小的内存缓冲区，以避免潜在的缓冲区溢出。

在Windows上，除非另有配置，否则stderr将定向到默认日志文件。如果要将输出定向到控制台窗口而不是错误日志，请使用[--console](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_console)选项从控制台窗口中的命令提示符启动服务器。有关详细信息，请参阅Windows上的[默认错误日志目标](https://dev.mysql.com/doc/refman/8.0/en/error-log-destination-configuration.html#error-log-destination-configuration-windows)。

在Unix和类Unix系统上，除非另有配置，否则stderr通常定向到终端。有关更多信息，请参阅[Unix和类Unix系统上的默认错误日志目标](https://dev.mysql.com/doc/refman/8.0/en/error-log-destination-configuration.html#error-log-destination-configuration-unix)。

InnoDB监视器只应在您真正想查看监视器信息时启用，因为输出生成会导致性能下降。此外，如果监视器输出被定向到错误日志，那么如果您稍后忘记禁用监视器，日志可能会变得很大。

> 笔记
为了帮助故障排除，InnoDB在某些情况下会暂时启用标准InnoDB监视器输出。有关更多信息，请参阅第15.21节 [InnoDB故障排除](https://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting.html)。

`SELECT @@innodb_status_output;`

InnoDB监视器输出以包含时间戳和监视器名称的标题开始。例如：

```log
=====================================
2014-10-16 18:37:29 0x7fc2a95c1700 INNODB监视器输出
=====================================
```

标准InnoDB监视器（InnoDB Monitor OUTPUT）的标头也用于锁定监视器，因为后者通过添加额外的锁定信息生成相同的输出。

[innodb_status_output](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_status_output) 和 [innodb_status_output_locks](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_status_output_locks) 系统变量用于启用标准innodb监视器和innodb锁定监视器。

启用或禁用InnoDB监视器需要[PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)权限。

### 启用标准InnoDB监视器

通过将InnoDB_status_output系统变量设置为ON，启用标准InnoDB监视器。

```sql
mysql> SELECT @@innodb_status_output;
+------------------------+
| @@innodb_status_output |
+------------------------+
|                      0 |
+------------------------+
1 row in set (0.01 sec)
 
mysql> SET GLOBAL innodb_status_output=ON;
Query OK, 0 rows affected (0.01 sec)
 
mysql> SELECT @@innodb_status_output;
+------------------------+
| @@innodb_status_output |
+------------------------+
|                      1 |
+------------------------+
1 row in set (0.01 sec)
```

要禁用标准InnoDB监视器，请将InnoDB_status_output设置为OFF。

关闭服务器时，innodb_status_output变量被设置为默认的OFF值。

### 启用InnoDB锁定监视器

InnoDB Lock Monitor数据通过InnoDB Standard Monitor输出打印。必须启用InnoDB Standard Monitor和InnoDB Lock Monitor，才能定期打印InnoDB锁定监视器数据。

要启用InnoDB锁监视器，请将InnoDB_status_output_locks系统变量设置为ON。InnoDB标准监视器和InnoDB锁定监视器都必须启用，才能定期打印InnoDB Lock Monitor数据：

```sql
SET GLOBAL innodb_status_output=ON;
SET GLOBAL innodb_status_output_locks=ON;
```

要禁用InnoDB锁定监视器，请将InnoDB_status_output_locks设置为OFF。将innodb2_status_ output设置为OFF，也可以禁用Inno数据库标准监视器。

关闭服务器时，innodb_status_output和innodb_status_output_locks变量被设置为默认的OFF值。

> 笔记
要为SHOW ENGINE InnoDB STATUS输出启用InnoDB锁定监视器，只需启用innodb_status_output_locks。

### 按需获取标准InnoDB监视器输出

作为启用标准InnoDB Monitor进行定期输出的替代方法，您可以使用SHOW ENGINE InnoDB STATUS SQL语句按需获取标准InnoDB-Monitor输出，该语句将输出提取到客户端程序。如果您使用的是mysql交互式客户端，如果将常用的分号语句终止符替换为\G，则输出更具可读性：

`mysql> SHOW ENGINE INNODB STATUS\G`

如果启用INNODB锁定监视器，则SHOW ENGINE INNODB STATUS输出还包括INNODB Lock Monitor数据。

### 将标准InnoDB监视器输出定向到状态文件

通过在启动时指定 `--innodb-status-file` 选项，可以启用标准InnoDB监视器输出并将其定向到状态文件。使用此选项时，InnoDB会创建一个名为InnoDB_status的文件。pid并大约每隔15秒向其写入输出。

InnoDB会在服务器正常关闭时删除状态文件。如果发生异常关机，可能需要手动删除状态文件。

`--innodb-status-file` 选项用于临时使用，因为输出生成可能会影响性能和 innodb_status.pid 文件可能会随着时间的推移变得非常大。

## InnoDB标准监视器和锁定监视器输出

锁定监视器与标准监视器相同，只是它包含其他锁定信息。为定期输出启用任一监视器将打开相同的输出流，但如果启用了锁定监视器，则该流将包含额外信息。例如，如果启用标准监视器和锁定监视器，则会打开单个输出流。在禁用“锁定监视器”之前，流包含额外的锁定信息。

使用SHOW ENGINE INNODB STATUS语句生成时，标准监视器输出限制为1MB。此限制不适用于写入服务器标准错误输出（stderr）的输出。

标准监视器输出示例：

```log
mysql> SHOW ENGINE INNODB STATUS\G

=====================================
2022-10-16 14:11:41 0x1b90 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 13 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 11521 srv_active, 0 srv_shutdown, 148800 srv_idle
srv_master_thread log flush and writes: 0
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 15710
OS WAIT ARRAY INFO: signal count 14857
RW-shared spins 0, rounds 0, OS waits 0
RW-excl spins 0, rounds 0, OS waits 0
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 0.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
------------
TRANSACTIONS
------------
Trx id counter 17236101
Purge done for trx's n:o < 17236101 undo n:o < 0 state: running but idle
History list length 4
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 281475273534120, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273530240, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273531792, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273531016, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273533344, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273532568, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273529464, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273528688, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
--------
FILE I/O
--------
I/O thread 0 state: wait Windows aio (insert buffer thread)
I/O thread 1 state: wait Windows aio (log thread)
I/O thread 2 state: wait Windows aio (read thread)
I/O thread 3 state: wait Windows aio (read thread)
I/O thread 4 state: wait Windows aio (read thread)
I/O thread 5 state: wait Windows aio (read thread)
I/O thread 6 state: wait Windows aio (write thread)
I/O thread 7 state: wait Windows aio (write thread)
I/O thread 8 state: wait Windows aio (write thread)
I/O thread 9 state: wait Windows aio (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:
Pending flushes (fsync) log: 0; buffer pool: 18446744073709551615
274140 OS file reads, 370746 OS file writes, 194199 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 1.66 writes/s, 1.05 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 6006, seg size 6008, 1 merges
merged operations:
 insert 1, delete mark 0, delete 0
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 1106407, node heap has 8 buffer(s)
Hash table size 1106407, node heap has 5 buffer(s)
Hash table size 1106407, node heap has 1728 buffer(s)
Hash table size 1106407, node heap has 1723 buffer(s)
Hash table size 1106407, node heap has 5 buffer(s)
Hash table size 1106407, node heap has 6 buffer(s)
Hash table size 1106407, node heap has 4 buffer(s)
Hash table size 1106407, node heap has 1701 buffer(s)
1.62 hash searches/s, 0.54 non-hash searches/s
---
LOG
---
Log sequence number          85490647897
Log buffer assigned up to    85490647897
Log buffer completed up to   85490647897
Log written up to            85490647897
Log flushed up to            85490647897
Added dirty pages up to      85490647897
Pages flushed up to          85490647897
Last checkpoint at           85490647897
99958 log i/o's done, 0.54 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 0
Dictionary memory allocated 9104154
Buffer pool size   393168
Free buffers       105525
Database pages     282463
Old database pages 104105
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 41564, not young 20494584
0.00 youngs/s, 0.00 non-youngs/s
Pages read 272778, created 9685, written 139303
0.00 reads/s, 0.00 creates/s, 0.53 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 282463, unzip_LRU len: 0
I/O sum[400]:cur[0], unzip sum[0]:cur[0]
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   49146
Free buffers       12085
Database pages     36414
Old database pages 13421
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5444, not young 3227760
0.00 youngs/s, 0.00 non-youngs/s
Pages read 35214, created 1200, written 59227
0.00 reads/s, 0.00 creates/s, 0.38 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 36414, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 1
Buffer pool size   49146
Free buffers       13354
Database pages     35144
Old database pages 12953
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5427, not young 2316943
0.00 youngs/s, 0.00 non-youngs/s
Pages read 33901, created 1243, written 11656
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 35144, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 2
Buffer pool size   49146
Free buffers       13028
Database pages     35470
Old database pages 13073
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5096, not young 2269904
0.00 youngs/s, 0.00 non-youngs/s
Pages read 34230, created 1240, written 6811
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 35470, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 3
Buffer pool size   49146
Free buffers       13290
Database pages     35208
Old database pages 12976
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5088, not young 1968031
0.00 youngs/s, 0.00 non-youngs/s
Pages read 34068, created 1140, written 4701
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 35208, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 4
Buffer pool size   49146
Free buffers       12907
Database pages     35591
Old database pages 13118
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5112, not young 2210407
0.00 youngs/s, 0.00 non-youngs/s
Pages read 34426, created 1165, written 10204
0.00 reads/s, 0.00 creates/s, 0.15 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 35591, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 5
Buffer pool size   49146
Free buffers       13797
Database pages     34701
Old database pages 12789
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5213, not young 2500256
0.00 youngs/s, 0.00 non-youngs/s
Pages read 33472, created 1229, written 12221
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 34701, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 6
Buffer pool size   49146
Free buffers       13496
Database pages     35004
Old database pages 12901
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5035, not young 2712175
0.00 youngs/s, 0.00 non-youngs/s
Pages read 33694, created 1310, written 16119
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 35004, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 7
Buffer pool size   49146
Free buffers       13568
Database pages     34931
Old database pages 12874
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 5149, not young 3289108
0.00 youngs/s, 0.00 non-youngs/s
Pages read 33773, created 1158, written 18364
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 34931, unzip_LRU len: 0
I/O sum[50]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=6040, Main thread ID=6088 , state=sleeping
Number of rows inserted 787083, updated 18125, deleted 170, read 39230595
0.00 inserts/s, 0.08 updates/s, 0.00 deletes/s, 2.77 reads/s
Number of system rows inserted 25, updated 921, deleted 25, read 582959
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================
```

### 标准监视器输出部分

有关Standard Monitor报告的每个指标的描述，请参阅《Oracle Enterprise Manager For MySQL Database User's Guide》中的“指标”一章。

- Status

  此部分显示时间戳、监视器名称和每秒平均值所基于的秒数。秒数是当前时间和上次打印InnoDB监视器输出之间的经过时间。

- BACKGROUND THREAD

  srv_master_thread行显示主后台线程完成的工作。

- SEMAPHORES

  This section reports threads waiting for a semaphore and statistics on how many times threads have needed a spin or a wait on a mutex or a rw-lock semaphore. 等待信号量的大量线程可能是磁盘I/O或InnoDB内部争用问题的结果。争用可能是由于查询的高度并行性或操作系统线程调度中的问题。在这种情况下，将 [innodb_thread_concurrency](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_thread_concurrency) 系统变量设置为小于默认值可能会有所帮助。
  The Spin rounds per wait line shows the number of spinlock rounds per OS wait for a mutex.
  每个等待行的自旋循环数显示每个操作系统等待互斥锁的自旋锁循环数。

  互斥指标由 [SHOW ENGINE INNODB MUTEX](https://dev.mysql.com/doc/refman/8.0/en/show-engine.html) 报告。

  ```log
  +--------+------------------------------+-----------+
  | Type   | Name                         | Status    |
  +--------+------------------------------+-----------+
  | InnoDB | rwlock: dict0dict.cc:2537    | waits=1   |
  | InnoDB | rwlock: dict0dict.cc:323     | waits=1   |
  | InnoDB | rwlock: trx0purge.cc:227     | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1   |
  | InnoDB | rwlock: btr0sea.cc:202       | waits=3   |
  | InnoDB | rwlock: btr0sea.cc:202       | waits=1   |
  | InnoDB | rwlock: btr0sea.cc:202       | waits=6   |
  | InnoDB | rwlock: btr0sea.cc:202       | waits=1   |
  | InnoDB | sum rwlock: buf0buf.cc:786   | waits=510 |
  +--------+------------------------------+-----------+
  ```

- LATEST FOREIGN KEY ERROR

  本节提供有关最近的外键约束错误的信息。如果没有发生此类错误，则不存在。内容包括失败的语句，以及有关失败的约束以及引用和引用的表的信息。

- LATEST DETECTED DEADLOCK

  本节提供有关最近死锁的信息。如果没有发生死锁，则它不存在。内容显示了所涉及的事务、每个事务试图执行的语句、它们拥有和需要的锁，以及InnoDB决定回滚以打破死锁的事务。本节中报告的锁定模式在第15.7.1节“[InnoDB锁定](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)”中进行了解释。

- TRANSACTIONS

  如果此部分报告锁等待，则应用程序可能存在锁争用。输出还可以帮助跟踪事务死锁的原因。

- FILE I/O

  本节提供有关InnoDB用于执行各种类型I/O的线程的信息。前几项专门用于一般InnoDB处理。内容还显示挂起I/O操作的信息和I/O性能的统计信息。

  这些线程的数量由[innodb_read_io_threads](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_read_io_threads)和[innodb_ write_io_thrads](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_write_io_threads)参数控制。参见第15.14节“[InnoDB启动选项和系统变量](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html)”。

- INSERT BUFFER AND ADAPTIVE HASH INDEX

  本节显示InnoDB插入缓冲区（也称为更改缓冲区）和自适应哈希索引的状态。

  有关信息，请参阅 [更新缓冲区](InnoDB内存结构/更新缓冲.md)”和 [自适应哈希索引](InnoDB内存结构/自适应哈希索引.md)。

- LOG

  此部分显示有关InnoDB日志的信息。内容包括当前日志序列号、日志被刷新到磁盘的距离以及InnoDB最后一次执行检查点的位置。（请参阅第15.11.3节“[InnoDB检查点](https://dev.mysql.com/doc/refman/8.0/en/innodb-checkpoints.html)”。）该节还显示有关挂起写入和写入性能统计信息。

- BUFFER POOL AND MEMORY

  本节为您提供有关已读和已写页面的统计信息。您可以根据这些数字计算查询当前正在执行的数据文件I/O操作数。

  有关缓冲池统计信息的描述，请参阅[使用InnoDB Standard Monitor监视缓冲池](InnoDB内存结构/缓冲池.md#使用innodb标准监控组件来监控缓存池)。

- ROW OPERATIONS

  本节显示主线程正在做什么，包括每种类型的行操作的数量和性能。
