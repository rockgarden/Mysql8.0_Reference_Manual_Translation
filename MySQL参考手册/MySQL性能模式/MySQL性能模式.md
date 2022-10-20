# MySQL 性能模式

<https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html>

目录

27.1 性能模式快速入门
27.2 性能模式构建配置
27.3 性能模式启动配置
27.4 性能模式运行时配置
27.5 性能模式查询
27.6 性能模式仪器命名约定
27.7 性能模式状态监控
27.8 性能模式原子和分子事件
27.9 当前和历史事件的性能模式表
27.10 性能模式语句摘要和抽样
27.11 性能模式通用表特征
27.12 [性能模式表说明](性能模式表描述/性能模式表描述.md)
27.13 性能模式选项和变量参考
27.14 性能模式命令选项
27.15 性能模式系统变量
27.16 性能模式状态变量
27.17 性能模式内存分配模型
27.18 性能模式和插件
27.19 使用性能模式诊断问题
27.20 性能模式的限制

MySQL Performance Schema是一种低级别监视MySQL Server执行的功能。性能模式具有以下特征：

- 性能模式提供了一种在运行时检查服务器内部执行的方法。它是使用[PERFORMANCE_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html)存储引擎和PERFORMANCE_SCHEMA数据库实现的。性能模式主要关注性能数据。这与用于检查元数据的INFORMATION_SCHEMA不同。

- 性能架构监视服务器事件。“事件”是指服务器所做的任何耗时且已检测的操作，以便收集计时信息。通常，事件可以是函数调用、等待操作系统、SQL语句执行的一个阶段（如解析或排序），或者整个语句或一组语句。事件收集提供了对有关服务器和多个存储引擎的同步调用（例如互斥锁）文件和表I/O、表锁等信息的访问。

- Performance Schema事件与写入服务器二进制日志（描述数据修改）的事件和Event Scheduler事件（存储程序的一种类型）不同。

- 性能模式事件特定于MySQL Server的给定实例。性能架构表被视为服务器的本地表，对它们的更改不会复制或写入二进制日志。

- 当前事件以及事件历史记录和摘要可用。这使您能够确定执行了多少次插入指令的活动以及它们花费了多少时间。事件信息可用于显示特定线程的活动，或与特定对象（如互斥锁或文件）关联的活动。

- PERFORMANCE_SCHEMA存储引擎使用服务器源代码中的“检测点”(instrumentation points)收集事件数据。

- 收集的事件存储在performance_schema数据库的表中。与其他表一样，可以使用SELECT语句查询这些表。

- 通过SQL语句更新Performance_Schema数据库中的表，可以动态修改Performance Schema配置。配置更改会立即影响数据收集。

- 性能架构中的表是内存中的表，不使用磁盘上的持久存储。内容从服务器启动时开始重新填充，在服务器关闭时丢弃。

- MySQL支持的所有平台都可以进行监控。

  Some limitations might apply: The types of timers might vary per platform. Instruments that apply to storage engines might not be implemented for all storage engines. Instrumentation of each third-party engine is the responsibility of the engine maintainer. See also Section 27.20, “[Restrictions on Performance Schema](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-restrictions.html)”.

- 数据收集是通过修改服务器源代码来添加检测来实现的。与其他功能（如复制或事件调度器）不同，没有与性能架构关联的单独线程。

性能模式旨在提供对有关服务器执行的有用信息的访问，同时对服务器性能的影响最小。实施遵循以下设计目标：

- 激活性能架构不会导致服务器行为发生任何变化。例如，它不会导致线程调度更改，也不会导致查询执行计划（如EXPLAIN所示）更改。

- 服务器监控持续而不引人注目，开销很少。激活性能架构不会使服务器不可用。

- 解析器保持不变。没有新的关键字或语句。

- 即使性能模式在内部失败，服务器代码的执行也会正常进行。

- 当可以在最初的事件收集期间或稍后的事件检索期间执行处理之间进行选择时，将优先考虑加快收集速度。这是因为收集正在进行，而检索是按需进行的，可能根本不会发生。

- 大多数Performance Schema表都有索引，这使得优化器可以访问除全表扫描之外的执行计划。有关更多信息，请参阅第8.2.4节“[优化性能模式查询](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-optimization.html)”。

- 添加新的检测点很容易。

- 检测已版本化。如果检测实现发生更改，则以前检测的代码将继续工作。这有利于第三方插件的开发人员，因为不需要升级每个插件来与最新的性能模式更改保持同步。

> 笔记
MySQL系统模式是一组对象，可以方便地访问性能模式收集的数据。默认情况下安装sys架构。有关使用说明，请参阅第28章[MySQL sys Schema](https://dev.mysql.com/doc/refman/8.0/en/sys-schema.html)。
