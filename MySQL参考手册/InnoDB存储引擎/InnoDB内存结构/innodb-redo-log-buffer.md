## 15.5.4 日志缓冲

> 原文地址： [https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log-buffer.html](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log-buffer.html)

日志缓冲在内存中缓存需要写到日志文件的内容。日志缓存大小由`innodb_log_buffer_size`变量决定，默认大小是16M。日志缓存中的内容周期性的刷到磁盘中。一个大的日志缓冲允许一个长事务在提交前不需要把redo log写到磁盘上。因此，如果你有增删改很多行数据的事务需要执行，你可以提高log buffer的值来节省磁盘I/O。

`innodb_flush_log_at_trx_commit` 变量控制日志缓冲里的数据如何刷新到磁盘。`innodb_flush_log_at_timeout` 变量控制刷新的频率。

更多相关信息请查阅内存控制和`第8.5.4节 优化InnoDB重做日志`。
