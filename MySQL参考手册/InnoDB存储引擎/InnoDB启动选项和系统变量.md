# InnoDB启动选项和系统变量

<https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_file_per_table>

* [innodb_numa_interleave](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_numa_interleave)

  为InnoDB缓冲池的分配启用NUMA交错内存策略。启用innodb_numa_interleave时，mysqld进程的numa内存策略设置为MPOL_interleave。分配InnoDB缓冲池后，NUMA内存策略被设置回MPOL_DEFAULT。要使innodb_numa_interleave选项可用，MySQL必须在支持numa的Linux系统上编译。

  CMake根据当前平台是否支持NUMA设置默认的WITH_NUMA值。有关更多信息，请参阅第2.9.7节“[MySQL源配置选项](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html)”。

* [innodb_file_per_table](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_file_per_table)

  当启用innodb_file_per_table时，默认情况下，表是在按文件表空间中创建的。当禁用时，默认情况下表是在系统表空间中创建的。

  innodb_file_per_table变量可以在运行时使用SET GLOBAL语句进行配置，也可以在启动时在命令行中指定，或者在选项文件中指定。在运行时配置需要足够的权限来设置全局系统变量，并立即影响到所有连接的操作。

  当驻留在逐个文件表空间中的表被截断或丢弃时，释放出来的空间将返回给操作系统。截断或丢弃驻留在系统表空间中的表只释放了系统表空间中的空间。系统表空间中释放的空间可以再次用于InnoDB数据，但不会返回到操作系统中，因为系统表空间的数据文件永远不会缩小。

  innodb_file_per-table设置不影响临时表的创建。从MySQL 8.0.14开始，临时表是在会话临时表空间中创建的，在这之前是在全局临时表空间中创建。

* [Innodb_page_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_page_size)

  指定InnoDB表空间的页面大小。值可以用字节或千字节来指定。例如，一个16KB的页面大小值可以被指定为16384、16KB或者16k。

  innodb_page_size只能在初始化MySQL实例之前进行配置，之后不能改变。如果没有指定值，实例将使用默认的页面大小进行初始化。

  对于32KB和64KB的页面大小，最大的行长度大约是16000字节。当innodb_page_size被设置为32KB或64KB时，不支持ROW_FORMAT=COMPRESSED。对于innodb_page_size=32KB，范围大小为2MB。当使用32KB或64KB页面大小时，innodb_log_buffer_size应该被设置为至少16M（默认）。

  默认的16KB或更大的页面大小适合于广泛的工作负载，特别是涉及表扫描的查询和涉及批量更新的DML操作。较小的页面大小可能对涉及许多小写的OLTP工作负载更有效，当单个页面包含许多行时，争用可能是一个问题。较小的页面对于通常使用小块大小的SSD存储设备来说也是有效的。保持InnoDB页面大小与存储设备块大小相近，可以最大限度地减少被重写到磁盘上的未改变的数据量。

  第一个系统表空间数据文件（ibdata1）的最小文件大小根据innodb_page_size值的不同而不同。

  使用特定InnoDB页面大小的MySQL实例不能使用来自使用不同页面大小的实例的数据文件或日志文件。

* [innodb_doublewrite](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_doublewrite)

  innodb_doublewrite变量控制双倍写入的缓冲。在大多数情况下，双写缓冲是默认启用的。

  在MySQL 8.0.30之前，你可以在启动服务器时将innodb_doublewrite设置为ON或OFF，以分别启用或禁用双写缓冲。从MySQL 8.0.30开始，innodb_doublewrite还支持DETECT_AND_RECOVER和DETECT_ONLY设置。

  DETECT_AND_RECOVER设置与ON设置相同。在这种设置下，双写缓冲区被完全启用，数据库页面内容被写入双写缓冲区，在恢复过程中被访问以修复不完整的页面写入。

  在DETECT_ONLY设置下，只有元数据被写入到双写缓冲区。数据库页面内容不被写入双写缓冲区，恢复时也不使用双写缓冲区来修复不完整的页面写入。这种轻量级设置仅用于检测不完整的页面写入。

  MySQL 8.0.30以上版本支持在ON、DETECT_AND_RECOVER和DETECT_ONLY之间动态改变启用双写缓冲区的innodb_doublewrite设置。MySQL不支持在启用双写缓冲区的设置和关闭之间的动态变化，反之亦然。

  如果复写缓冲区位于支持原子写入的Fusion-io设备上，复写缓冲区会自动被禁用，数据文件的写入会使用Fusion-io的原子写入进行。然而，请注意innodb_doublewrite设置是全局的。当双写缓冲区被禁用时，它对所有数据文件都是禁用的，包括那些不在Fusion-io硬件上的文件。这个功能只在Fusion-io硬件上支持，并且只在Linux上的Fusion-io NVMFS上启用。为了充分利用这一特性，建议将innodb_flush_method设置为O_DIRECT。
