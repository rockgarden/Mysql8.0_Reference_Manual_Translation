# 优化InnoDB磁盘I/O

如果您遵循SQL操作的数据库设计和优化技术的最佳实践，但由于磁盘I/O活动繁重，数据库仍然很慢，请考虑这些磁盘I/O优化。如果Unix顶级工具或Windows任务管理器显示工作负载的CPU使用率低于70%，则工作负载可能是磁盘绑定的。

- 增加缓冲池大小

  当表数据缓存在InnoDB缓冲池中时，可以通过查询重复访问，而无需任何磁盘I/O。使用innodb_buffer_pool_size选项指定缓冲池的大小。此内存区域非常重要，因此通常建议将innodb_buffer_pool_size配置为系统内存的50%至75%。

- 调整冲洗方法

  在GNU/Linux和Unix的某些版本中，使用Unix fsync（）调用（InnoDB默认使用）和类似方法将文件刷新到磁盘的速度惊人地慢。如果数据库写入性能存在问题，请将[innodb_flush_method](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_flush_method)参数设置为O_DSYNC进行基准测试。

- 配置操作系统刷新的阈值

  默认情况下，当InnoDB创建新的数据文件（如新的日志文件或表空间文件）时，文件在刷新到磁盘之前会被完全写入操作系统缓存，这可能会导致大量磁盘写入活动同时发生。要强制从操作系统缓存中定期刷新较小的数据，可以使用[innodb_fsync_threshold](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_fsync_threshold)变量定义一个以字节为单位的阈值。当达到字节阈值时，操作系统缓存的内容将刷新到磁盘。默认值0强制执行默认行为，即仅在文件完全写入缓存后才将数据刷新到磁盘。

  在多个MySQL实例使用同一存储设备的情况下，指定一个阈值以强制较小的定期刷新可能是有益的。例如，创建一个新的MySQL实例及其相关数据文件可能会导致磁盘写入活动激增，从而影响使用相同存储设备的其他MySQL实例的性能。配置阈值有助于避免写入活动中的此类激增。

- 使用fdatasync()而不是fsync()

  在支持fdatasync()系统调用的平台上，MySQL 8.0.26中引入的innodb_use_datasync变量允许使用fdatasyn()而不是fsync()进行操作系统刷新。除非后续数据检索需要，否则fdatasync()系统调用不会刷新对文件元数据的更改，从而提供潜在的性能优势。

  innodb_flush_method设置的子集，如fsync、O_DSYNC和O_DIRECT）使用fsync()系统调用。使用这些设置时，innodb_use_fdatasync变量适用。

- 在Linux上使用带有本地AIO的noop或截止期I/O调度程序

  InnoDB在Linux上使用异步I/O子系统（本机AIO）来执行数据文件页的读写请求。此行为由[innodb_use_native_aio](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_use_native_aio)配置选项控制，该选项默认启用。对于本机AIO，I/O调度器的类型对I/O性能有更大的影响。通常，建议使用noop和截止期I/O调度程序。执行基准测试，以确定哪个I/O调度器为您的工作负载和环境提供了最佳结果。有关更多信息，请参阅第15.8.6节“[在Linux上使用异步I/O](https://dev.mysql.com/doc/refman/8.0/en/innodb-linux-native-aio.html)”。

- 在Solaris 10 for x86_64体系结构上使用直接I/O

  在Solaris 10 for x86_64体系结构（AMD Opteron）上使用InnoDB存储引擎时，请对InnoDB相关文件使用直接I/O，以避免InnoDB性能下降。要对用于存储InnoDB相关文件的整个UFS文件系统使用直接I/O，请使用forcedirection选项进行安装；参见mount_ufs（1M）。（Solaris 10/x86_64上的默认设置是不使用此选项。）要仅将直接I/O应用于InnoDB文件操作，而不是整个文件系统，请将 InnoDB_flush_method = O_DIRECT 。使用此设置，InnoDB调用direction()而不是fcntl()来对数据文件进行I/O（而不是对日志文件进行I/O）。

- Solaris 2.6或更高版本使用原始存储存储数据和日志文件

  在Solaris 2.6及更高版本的任何版本和任何平台（sparc/x86/x64/amd64）上使用具有较大InnoDB_buffer_pool_size值的InnoDB存储引擎时，使用前面描述的forcedirectio mount选项，在原始设备或单独的直接I/O UFS文件系统上使用InnoDB数据文件和日志文件进行基准测试。（如果需要日志文件的直接I/O，则必须使用mount选项而不是设置innodb_flush_method。）Veritas文件系统VxFS的用户应使用convosync=direct装载。

  不要将其他MySQL数据文件（例如MyISAM表的数据文件）放置在直接I/O文件系统上。可执行文件或库不得放置在直接I/O文件系统上。

- 使用其他存储设备

  可以使用其他存储设备来设置RAID配置。有关相关信息，请参阅第8.12.1节“[优化磁盘I/O](https://dev.mysql.com/doc/refman/8.0/en/disk-issues.html)”。

  或者，InnoDB表空间数据文件和日志文件可以放在不同的物理磁盘上。有关详细信息，请参阅以下章节：

  - 第15.8.1节，“[InnoDB启动配置](https://dev.mysql.com/doc/refman/8.0/en/innodb-init-startup-configuration.html)”

  - 第15.6.1.2节，“[外部创建表](https://dev.mysql.com/doc/refman/8.0/en/innodb-create-table-external.html)”

  - [创建常规表空间](https://dev.mysql.com/doc/refman/8.0/en/general-tablespaces.html#general-tablespaces-creating)

  - 第15.6.1.4节，“[移动或复制InnoDB表](https://dev.mysql.com/doc/refman/8.0/en/innodb-migration.html)”

- 考虑非旋转存储

  非旋转non-rotational存储通常为随机I/O操作提供更好的性能；以及用于顺序I/O操作的旋转存储。在旋转和非旋转存储设备之间分发数据和日志文件时，请考虑主要在每个文件上执行的I/O操作的类型。

  面向I/O的随机文件通常包括[每个表的文件](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_file_per_table)、[通用表空间](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_general_tablespace)数据文件、[撤消表空间](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_undo_tablespace)文件和[临时表空间](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_temporary_tablespace)文件。面向I/O的顺序文件包括InnoDB[系统表空间](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_system_tablespace)文件（由于MySQL 8.0.20之前的[双写缓冲](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_doublewrite_buffer)和[更改缓冲](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_change_buffer)）、MySQL 8.0.20中引入的双写文件，以及二进制日志文件和重做日志文件等日志文件。

  使用非旋转存储时，请查看以下配置选项的设置：

  - [innodb_checksum_algorithm](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_checksum_algorithm)

    crc32选项使用更快的校验和算法，建议用于快速存储系统。

  - [innodb_flush_neighbors](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_flush_neighbors)

    优化旋转存储设备的I/O。将其禁用为非旋转存储或旋转和非旋转存储的混合。默认情况下，它被禁用。

  - [innodb_idle_flush_pct](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_idle_flush_pct)

    允许在空闲期间限制页面刷新，这有助于延长非旋转存储设备的寿命。在MySQL 8.0.18中引入。

  - [innodb_io_capacity](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_io_capacity)

    默认设置200对于低端非旋转存储设备通常是足够的。对于高端总线连接设备，请考虑更高的设置，如1000。

  - [innodb_io_capacity_max](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_io_capacity_max)

    默认值2000适用于使用非旋转存储的工作负载。对于高端、总线连接的非旋转存储设备，请考虑更高的设置，如2500。

  - [innodb_log_compressed_pages](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_compressed_pages)

    如果重做日志位于非旋转存储上，请考虑禁用此选项以减少日志记录。请参阅[禁用压缩页面的日志记录](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-diskio.html#innodb-disable-log-compressed-pages)。

  - [innodb_log_file_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_file_size)（在MySQL 8.0.30中已弃用）

    如果重做日志位于非旋转存储上，请配置此选项以最大化缓存和写入组合。

  - [innodb_redo_log_capacity](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_redo_log_capacity)

    如果重做日志位于非旋转存储上，请配置此选项以最大化缓存和写入组合。

  - [innodb_page_size](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_page_size)

    考虑使用与磁盘内部扇区大小匹配的页面大小。早期的SSD设备通常具有4KB的扇区大小。一些较新的设备具有16KB的扇区大小。默认InnoDB页面大小为16KB。将页面大小保持在接近存储设备块大小的位置，可将重写到磁盘的未更改数据量降至最低。

  - [binlog_row_image](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_row_image)

    如果二进制日志位于非循环存储上，并且所有表都有主键，请考虑将此选项设置为最小值以减少日志记录。

  确保为您的操作系统启用了TRIM支持。默认情况下，它通常处于启用状态。

- 增加I/O容量以避免积压

  如果吞吐量由于InnoDB检查点操作而周期性下降，请考虑增加InnoDB_io_capacity配置选项的值。较高的值会导致更频繁的刷新，从而避免可能导致吞吐量下降的工作积压。

- 如果冲洗未落后，则I/O容量降低

  如果系统没有落后于InnoDB刷新操作，请考虑降低InnoDB_io_capacity配置选项的值。通常，您将此选项值保持为尽可能低的值，但不要太低，以免导致吞吐量周期性下降，如前一个项目所述。在您可以降低选项值的典型场景中，您可能会在SHOW ENGINE INNODB STATUS的输出中看到这样的组合：

  - 历史列表长度低，低于几千。

  - 插入缓冲区合并靠近插入的行。

  - 缓冲池中修改的页面始终低于缓冲池的[innodb_max_dirty_pages_pct](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_max_dirty_pages_pct)。（在服务器未进行大容量插入时进行测量；在大容量插入期间，修改的页面百分比会显著上升，这是正常的。）

  - 日志序列号-最后一个检查点小于InnoDB日志文件总大小的7/8或理想情况下小于6/8。

- 在Fusion io设备上存储系统表空间文件

  通过在支持原子写入的Fusion io设备上存储包含双写存储区域的文件，可以利用与双写缓冲区相关的I/O优化。（在MySQL 8.0.20之前，双写缓冲区存储驻留在系统表空间数据文件中。从MySQL 8.0.20开始，存储区驻留在双写文件中。请参阅第15.6.4节“双写缓冲”。）当双写存储区文件放置在支持原子写的Fusion io设备上时，双写缓冲区被自动禁用，Fusion io原子写入用于所有数据文件。此功能仅在Fusion io硬件上受支持，仅在Linux上的Fusion io NVMFS上启用。为了充分利用此功能，建议将innodb_flush_method设置为O_DIRECT。

  > 笔记
  因为双写缓冲区设置是全局的，所以对于不在Fusion io硬件上的数据文件，双写缓冲也被禁用。

- 禁用压缩页面的日志记录

  使用InnoDB表压缩功能时，当对压缩数据进行更改时，重新压缩的页面的图像将写入重做日志。此行为由innodb_log_compressed_pages控制，默认情况下启用此功能可防止在恢复期间使用不同版本的zlib压缩算法时可能发生的损坏。如果您确定zlib版本不会更改，请禁用innodb_log_compressed_pages以减少修改压缩数据的工作负载的重做日志生成  
