# KILL 语句

`KILL [CONNECTION | QUERY] processlist_id`

每个到mysqld的连接都在单独的线程中运行。可以使用kill processlist_id语句终止线程。

线程进程列表标识符可以从INFORMATION_SCHEMA processlist表的ID列、SHOW processlist输出的ID列和Performance SCHEMA线程表的processlist_ID列中确定。当前线程的值由CONNECTION_ID（）函数返回。

KILL允许使用可选的CONNECTION或QUERY修饰符：

- KILL CONNECTION与不带修饰符的KILL相同：它终止与给定processlist_id关联的连接，在终止连接正在执行的任何语句之后。

- KILL QUERY终止连接当前正在执行的语句，但保持连接本身不变。

查看哪些线程可以被终止的能力取决于PROCESS特权：

- 没有PROCESS，您只能看到自己的线程。

- 使用PROCESS，您可以看到所有线程。

终止线程和语句的能力取决于CONNECTION_ADMIN特权和弃用的SUPER特权：

- 如果没有CONNECTION_ADMIN或SUPER，则只能终止自己的线程和语句。

- 使用CONNECTION_ADMIN或SUPER，您可以终止所有线程和语句，但要影响以SYSTEM_USER权限执行的线程或语句，您自己的会话还必须具有SYSTEM_USER权限。

您还可以使用[mysqladmin processlist](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html)和[mysqladmin kill](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html)命令来检查和杀死线程。

使用KILL时，会为线程设置特定于线程的KILL标志。在大多数情况下，线程可能需要一些时间才能终止，因为只在特定的时间间隔检查kill标志：

- 在SELECT操作期间，对于 ORDER BY 和 GROUP BY 循环，在读取一个行块后检查标志。如果设置了kill标志，语句将中止。

- 进行表复制的 ALTER TABLE 操作会定期检查从原始表读取的每个复制行的kill标志。如果设置了kill标志，语句将中止，临时表将被删除。

  KILL语句在不等待确认的情况下返回，但KILL标志检查会在相当短的时间内中止操作。中止操作以执行任何必要的清理也需要一些时间。

- 在UPDATE或DELETE操作期间，在读取每个块之后以及在更新或删除每个行之后检查kill标志。如果设置了kill标志，语句将中止。如果不使用事务，则不会回滚更改。

- GET_LOCK()中止并返回NULL。

- 如果线程位于表锁处理程序中（状态：Locked），则表锁将快速中止。

- 如果线程正在等待写入调用中的可用磁盘空间，则会中止写入，并显示“磁盘已满”错误消息。

- EXPLAIN ANALYZE中止并打印第一行输出。这在MySQL 8.0.20及更高版本中有效。

> 警告
终止MyISAM表上的REPAIR TABLE或OPTIMIZE TABLE操作会导致表损坏且不可用。对这样一个表的任何读取或写入操作都会失败，直到您再次优化或修复它（没有中断）。
