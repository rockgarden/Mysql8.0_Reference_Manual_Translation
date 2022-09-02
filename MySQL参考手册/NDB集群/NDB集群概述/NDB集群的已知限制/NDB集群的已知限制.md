# NDB Cluster 的已知限制

<https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations.html>

## 目录

23.2.7.1 [NDB Cluster 中不符合 SQL 语法](不符合NDB集群中的SQL语法.md)
23.2.7.2 [NDB Cluster 与标准 MySQL 限制的限制和差异](#ndb集群与标准mysql限制的限制和差异)
23.2.7.3 [NDB Cluster 中与事务处理相关的限制](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-transactions.html)
23.2.7.4 NDB 集群错误处理
23.2.7.5 与 NDB Cluster 中的数据库对象相关的限制
23.2.7.6 NDB Cluster 中不支持或缺少的功能
23.2.7.7 与 NDB Cluster 中的性能相关的限制
23.2.7.8 NDB Cluster 独有的问题
23.2.7.9 与 NDB Cluster 磁盘数据存储相关的限制
23.2.7.10 与多个 NDB Cluster 节点相关的限制
23.2.7.11 NDB Cluster 8.0 中解决的以前的 NDB Cluster 问题

## 概述

在接下来的部分中，我们将讨论当前 NDB Cluster 版本中的已知限制，与使用 MyISAM 和 InnoDB 存储引擎时可用的功能进行比较。如果您在 <http://bugs.mysql.com> 上的 MySQL 错误数据库中检查“集群”类别，您可以在 <http://bugs.mysql.com> 上的 MySQL 错误数据库中的“MySQL 服务器”下找到以下类别中的已知错误.mysql.com，我们打算在即将发布的 NDB Cluster 版本中更正：

- NDB Cluster

- Cluster Direct API (NDBAPI)

- Cluster Disk Data

- Cluster Replication

- ClusterJ

此信息旨在就刚刚提出的条件而言是完整的。您可以使用[第 1.6 节“如何报告错误或问题”](https://dev.mysql.com/doc/refman/8.0/en/bug-reports.html) 中给出的说明向 MySQL 错误数据库报告遇到的任何差异。我们不打算在 NDB Cluster 8.0 中修复的任何问题都会添加到列表中。

有关 NDB Cluster 8.0 中已解决的早期版本中的问题列表，请参阅[第 23.2.7.11 节，“NDB Cluster 8.0 中已解决的先前 NDB Cluster 问题”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-resolved.html)。

> 笔记
[第 23.7.3 节“NDB Cluster 复制中的已知问题”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-replication-issues.html)中描述了特定于 NDB Cluster 复制的限制和其他问题。

## NDB集群与标准MySQL限制的限制和差异

在本节中，我们列出了NDB集群中发现的限制，这些限制与标准MySQL中发现的或未发现的限制不同。

内存使用和恢复。将数据插入NDB表时消耗的内存在删除时不会自动恢复，这与其他存储引擎一样。相反，以下规则适用：

- NDB表上的DELETE语句使被删除行以前使用的内存仅可供同一表上的INSERT重新使用。然而，通过执行优化表，可以使该内存可用于一般重用。
  集群的滚动重新启动也会释放已删除行使用的所有内存。参见[第23.6.5节，“执行NDB集群的滚动重启”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-rolling-restart.html)。
- NDB表上的DROP TABLE或TRUNCATE TABLE操作释放该表使用的内存，供任何NDB表（同一表或另一NDB表）重用。
  >笔记
  回想一下，TRUNCATE TABLE删除并重新创建表。见[第13.1.37节“截断表语句”](https://dev.mysql.com/doc/refman/8.0/en/truncate-table.html)。
- 群集配置施加的限制。存在许多可配置的硬限制，但集群中的可用主内存设置了限制。请参阅[第23.4.3节“NDB群集配置文件”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-config-file.html)中的完整配置参数列表。大多数配置参数可以在线升级。这些硬限制包括：
  - 数据库内存大小和索引内存大小（分别为 [DataMemory](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-datamemory) 和 [IndexMemory](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-indexmemory) ）。
  - 数据内存分配为32KB页面。当使用每个DataMemory页时，它被分配给一个特定的表；一旦分配，除非删除表，否则无法释放该内存。
  更多信息，请参见[第23.4.3.6节“定义NDB集群数据节点”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html)。
  - 每个事务可以执行的最大操作数是使用配置参数 [MaxNoOfConcurrentOperations](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-maxnoofconcurrentoperations) 和 [MaxNoOfLocalOperations](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-maxnooflocaloperations) 设置的。
    > 笔记
    大容量加载、截断表和ALTER表是通过运行多个事务作为特殊情况处理的，因此不受此限制。
  - 与表和索引相关的不同限制。例如，集群中的最大有序索引数由 [MaxNooforderedIndex](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-maxnooforderedindexes) 确定，每个表的最大有序指数数为16。
- 节点和数据对象最大值。以下限制适用于群集节点和元数据对象的数量：
  - 最大数据节点数为145个（在NDB 7.6及更早版本中为48个）
  数据节点的节点ID必须在1到144（包括1到144）的范围内。
  管理和API节点可以使用范围为1到255（包括1到255）的节点ID。
  - NDB集群中的最大节点总数为255。这个数字包括所有SQL节点（MySQL服务器）、API节点（除MySQL服务器之外访问集群的应用程序）、数据节点和管理服务器。
  - NDB群集当前版本中的最大元数据对象数为20320。此限制是硬编码的。

  有关更多信息，请参阅[第23.2.7.11节，“NDB集群8.0中解决的以前NDB集群问题”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-resolved.html)。
