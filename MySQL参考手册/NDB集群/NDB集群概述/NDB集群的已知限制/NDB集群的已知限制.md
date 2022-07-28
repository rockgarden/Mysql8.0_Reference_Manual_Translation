# NDB Cluster 的已知限制

<https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations.html>
## 目录

[23.2.7.1 NDB Cluster 中不符合 SQL 语法](不符合NDB集群中的SQL语法.md)
23.2.7.2 NDB Cluster 与标准 MySQL 限制的限制和差异
23.2.7.3 NDB Cluster 中与事务处理相关的限制
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
