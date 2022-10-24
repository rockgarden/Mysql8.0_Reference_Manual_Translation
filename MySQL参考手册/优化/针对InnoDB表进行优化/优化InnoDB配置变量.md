# 优化InnoDB配置变量

## 优化InnoDB配置变量

不同的设置最适合负载较轻、可预测的服务器，而不是一直在接近满容量运行或活动频繁的服务器。

由于InnoDB存储引擎自动执行许多优化，因此许多性能调优任务涉及监控以确保数据库运行良好，以及在性能下降时更改配置选项。有关InnoDB性能监控的详细信息，请参阅第15.16节“[InnoDB与MySQL性能模式的集成](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-schema.html)”。

您可以执行的主要配置步骤包括：

- 控制InnoDB缓存更改数据的数据更改操作类型，以避免频繁的小磁盘写入。请参阅[配置更改缓冲](https://dev.mysql.com/doc/refman/8.0/en/innodb-change-buffer.html#innodb-change-buffer-configuration)。因为默认值是缓冲所有类型的数据更改操作，所以只有在需要减少缓冲量时才更改此设置。

- 使用innodb_adaptive_hash_index选项打开和关闭自适应散列索引功能。详见 “自适应哈希索引”。您可以在异常活动期间更改此设置，然后将其恢复为原始设置。

- 如果上下文切换是一个瓶颈，则设置InnoDB处理的并发线程数的限制。参见第15.8.4节，“[为InnoDB配置线程并发](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-thread_concurrency.html)”。

- 控制InnoDB通过其预读操作执行的预取量。当系统有未使用的I/O容量时，更多的预读可以提高查询的性能。过多的预读可能会导致重负载系统的性能周期性下降。参见第15.8.3.4节，“[配置InnoDB缓冲池预取（预读）](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-read_ahead.html)”。

- 如果您的高端I/O子系统未被默认值充分利用，请增加读取或写入操作的后台线程数。请参阅第15.8.5节，“[配置后台InnoDB I/O线程数](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-multiple_io_threads.html)”。

- 控制InnoDB在后台执行的I/O量。参见第15.8.7节，“[配置InnoDB I/O容量](https://dev.mysql.com/doc/refman/8.0/en/innodb-configuring-io-capacity.html)”。如果观察到性能周期性下降，可以缩小此设置。

- 控制确定InnoDB何时执行特定类型的后台写入的算法。参见第15.8.3.5节，“[配置缓冲池刷新](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-spin_lock_polling.html)”。该算法适用于某些类型的工作负载，但不适用于其他类型的工作负荷，因此，如果观察到性能周期性下降，您可能会禁用此功能。

- 利用多核处理器及其高速缓存配置，最大限度地减少上下文切换中的延迟。参见第15.8.8节，“[配置自旋锁定轮询](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-spin_lock_polling.html)”。

- 防止表扫描等一次性操作干扰InnoDB缓冲区缓存中存储的频繁访问数据。参见第15.8.3.3节，“[使缓冲池抗扫描](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-midpoint_insertion.html)”。

- 将日志文件调整到对可靠性和崩溃恢复有意义的大小。InnoDB日志文件通常保持较小，以避免崩溃后长时间启动。MySQL 5.5中引入的优化加快了崩溃恢复过程的某些步骤。特别是，由于内存管理算法的改进，扫描重做日志和应用重做日志的速度更快。如果您人为地将日志文件保持较小以避免长时间启动，那么现在可以考虑增加日志文件大小以减少由于重做日志记录的循环而发生的I/O。

- 为InnoDB缓冲池配置实例的大小和数量，对于具有千兆字节缓冲池的系统尤为重要。请参阅第15.8.3.2节“[配置多个缓冲池实例](https://dev.mysql.com/doc/refman/8.0/en/innodb-multiple-buffer-pools.html)”。

- 增加并发事务的最大数量，这大大提高了最繁忙数据库的可伸缩性。参见第15.6.6节“[撤销日志](https://dev.mysql.com/doc/refman/8.0/en/innodb-undo-logs.html)”。

- 将清除purge操作（一种垃圾收集）移动到后台线程中。参见第15.8.9节“[净化配置](https://dev.mysql.com/doc/refman/8.0/en/innodb-purge-configuration.html)”。要有效地测量此设置的结果，请首先调整其他I/O相关和线程相关的配置设置。

- 减少InnoDB在并发线程之间的切换量，以便繁忙服务器上的SQL操作不会排队并形成“流量堵塞(traffic jam)”。为[innodb_thread_concurrency](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_thread_concurrency)选项设置一个值，对于高性能的现代系统，该值最多为32。增加[innodb_concurrency_tickes](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_concurrency_tickets)选项的值，通常为5000左右。这些选项的组合为InnoDB在任何时间处理的线程数量设置了上限，并允许每个线程在被换出之前做大量工作，这样等待线程的数量保持在较低水平，操作可以在不过度切换上下文的情况下完成。
