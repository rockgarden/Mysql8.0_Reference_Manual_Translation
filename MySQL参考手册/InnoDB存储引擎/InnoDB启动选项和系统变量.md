# InnoDB启动选项和系统变量

innodb_numa_interleave

为InnoDB缓冲池的分配启用NUMA交错内存策略。启用innodb_numa_interleave时，mysqld进程的numa内存策略设置为MPOL_interleave。分配InnoDB缓冲池后，NUMA内存策略被设置回MPOL_DEFAULT。要使innodb_numa_interleave选项可用，MySQL必须在支持numa的Linux系统上编译。

CMake根据当前平台是否支持NUMA设置默认的WITH_NUMA值。有关更多信息，请参阅第2.9.7节“[MySQL源配置选项](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html)”。
