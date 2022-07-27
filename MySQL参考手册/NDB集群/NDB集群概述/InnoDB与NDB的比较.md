# MySQL 服务器使用 InnoDB 与 NDB Cluster 的比较

23.2.6.1 NDB 和 InnoDB 存储引擎的区别
23.2.6.2 NDB 和 InnoDB 工作负载
23.2.6.3 NDB 和 InnoDB 功能使用总结

MySQL Server 在存储引擎中提供了多种选择。由于 NDB 和 InnoDB 都可以作为事务性 MySQL 存储引擎，因此 MySQL Server 的用户有时会对 NDB Cluster 感兴趣。他们将 NDB 视为一种可能的替代方案，或者升级到 MySQL 8.0 中的默认 InnoDB 存储引擎。虽然 NDB 和 InnoDB 有共同的特点，但在架构和实现上存在差异，因此一些现有的 MySQL Server 应用程序和使用场景可以很好地适合 NDB Cluster，但不是全部。

在本节中，我们将讨论和比较 NDB 8.0 使用的 NDB 存储引擎与 MySQL 8.0 中使用的 InnoDB 的一些特性。接下来的几节提供了技术比较。在许多情况下，必须根据具体情况决定何时何地使用 NDB Cluster，并考虑所有因素。虽然为每个可能的使用场景提供细节超出了本文档的范围，但我们也尝试提供一些非常通用的指导，说明 NDB 相对于 InnoDB 后端的一些常见类型的应用程序的相对适用性。

NDB Cluster 8.0 使用基于 MySQL 8.0 的 mysqld，包括对 InnoDB 1.1 的支持。虽然可以将 InnoDB 表与 NDB Cluster 一起使用，但此类表不是集群的。也不能将 NDB Cluster 8.0 发行版中的程序或库与 MySQL Server 8.0 一起使用，反之亦然。

虽然某些类型的常见业务应用程序可以在 NDB Cluster 或 MySQL Server 上运行（最有可能使用 InnoDB 存储引擎）也是事实，但存在一些重要的架构和实现差异。[NDB 和 InnoDB 存储引擎之间的差异](#ndb-和-innodb-存储引擎之间的差异)，提供了这些差异的摘要。由于差异，一些使用场景显然更适合一种引擎或另一种；请参阅[NDB 和 InnoDB 工作负载](#ndb-和-innodb-工作负载)。这反过来又会影响更适合与 NDB 或 InnoDB 一起使用的应用程序类型。请参阅[NDB 和 InnoDB 功能使用摘要](#ndb-和-innodb-功能使用总结)，以比较每种在常见类型的数据库应用程序中的相对适用性。

有关 NDB 和 MEMORY 存储引擎的相对特征的信息，请参阅[何时使用 MEMORY 或 NDB Cluster](https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html#memory-storage-engine-compared-cluster)。

有关 MySQL 存储引擎的更多信息，请参阅[第 16 章，替代存储引擎](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)。

## NDB 和 InnoDB 存储引擎之间的差异

NDB 存储引擎是使用分布式、无共享架构实现的，这导致它在许多方面的行为与 InnoDB 不同。 对于那些不习惯使用 NDB 的人来说，由于其在事务、外键、表限制和其他特性方面的分布式特性，可能会出现意想不到的行为。 这些如下表所示：

表 23.2 InnoDB 和 NDB 存储引擎的区别

| Feature                                              | InnoDB (MySQL 8.0)                                                                            | NDB 8.0                                                                                                                                                                        |
|------------------------------------------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| MySQL Server Version                                 | 8                                                                                             | 8                                                                                                                                                                              |
| InnoDB Version                                       | InnoDB 8.0.30                                                                                 | InnoDB 8.0.30                                                                                                                                                                  |
| NDB Cluster Version                                  | N/A                                                                                           | NDB 8.0.30/8.0.30                                                                                                                                                              |
| Storage Limits                                       | 64TB                                                                                          | 128TB                                                                                                                                                                          |
| Foreign Keys                                         | Yes                                                                                           | Yes                                                                                                                                                                            |
| Transactions                                         | All standard types                                                                            | READ COMMITTED                                                                                                                                                                 |
| MVCC.                                                | Yes                                                                                           | No                                                                                                                                                                             |
| Data Compression                                     | Yes                                                                                           | No (NDB checkpoint and backup files can be compressed)                                                                                                                         |
| Large Row Support (> 14K)                            | Supported for VARBINARY, VARCHAR, BLOB, and TEXTcolumns                                       | Supported for BLOB and TEXT columns only (Using these types to store very large amounts of data can lower NDB performance)                                                     |
| Replication Support                                  | Asynchronous and semisynchronous replication using MySQL Replication; MySQL Group Replication | Automatic synchronous replication within an NDB Cluster; asynchronous replication between NDB Clusters, using MySQL Replication (Semisynchronous replication is not supported) |
| Scaleout for Read Operations                         | Yes (MySQL Replication)                                                                       | Yes (Automatic partitioning in NDB Cluster; NDB Cluster Replication)                                                                                                           |
| Scaleout for Write Operations                        | Requires application-level partitioning (sharding)                                            | Yes (Automatic partitioning in NDB Cluster is transparent to applications)                                                                                                     |
| High Availability (HA)                               | Built-in, from InnoDB cluster                                                                 | Yes (Designed for 99.999% uptime)                                                                                                                                              |
| Node Failure Recovery and Failover                   | From MySQL Group Replication                                                                  | Automatic (Key element in NDB architecture)                                                                                                                                    |
| Time for Node Failure Recovery                       | 30 seconds or longer                                                                          | Typically < 1 second                                                                                                                                                           |
| Real-Time Performance                                | No                                                                                            | Yes                                                                                                                                                                            |
| In-Memory Tables                                     | No                                                                                            | Yes (Some data can optionally be stored on disk; both in-memory and disk data storage are durable)                                                                             |
| NoSQL Access to Storage Engine                       | Yes                                                                                           | Yes (Multiple APIs, including Memcached, Node.js/JavaScript, Java, JPA, C++, and HTTP/REST)                                                                                    |
| Concurrent and Parallel Writes                       | Yes                                                                                           | Up to 48 writers, optimized for concurrent writes                                                                                                                              |
| Conflict Detection and Resolution (Multiple Sources) | Yes (MySQL Group Replication)                                                                 | Yes                                                                                                                                                                            |
| Hash Indexes                                         | No                                                                                            | Yes                                                                                                                                                                            |
| Online Addition of Nodes                             | Read/write replicas using MySQL Group Replication                                             | Yes (all node types)                                                                                                                                                           |
| Online Upgrades                                      | Yes (using replication)                                                                       | Yes                                                                                                                                                                            |
| Online Schema Modifications                          | Yes, as part of MySQL 8.0                                                                     | Yes                                                                                                                                                                            |

## NDB 和 InnoDB 工作负载

NDB Cluster 具有一系列独特的属性，使其非常适合为需要高可用性、快速故障转移、高吞吐量和低延迟的应用程序提供服务。 由于其分布式架构和多节点实现，NDB Cluster 还具有特定的约束条件，可能会导致某些工作负载无法正常运行。 下表显示了 NDB 和 InnoDB 存储引擎在一些常见类型的数据库驱动应用程序工作负载方面的一些主要行为差异：

表 23.3 InnoDB 和 NDB 存储引擎之间的差异，常见类型的数据驱动应用程序工作负载。

| Workload                                         | InnoDB | NDB Cluster (NDB)                                                                    |
|--------------------------------------------------|--------|--------------------------------------------------------------------------------------|
| High-Volume OLTP Applications                    | Yes    | Yes                                                                                  |
| DSS Applications (data marts, analytics)         | Yes    | Limited (Join operations across OLTP datasets not exceeding 3TB in size)             |
| Custom Applications                              | Yes    | Yes                                                                                  |
| Packaged Applications                            | Yes    | Limited (should be mostly primary key access); NDB Cluster 8.0 supports foreign keys |
| In-Network Telecoms Applications (HLR, HSS, SDP) | No     | Yes                                                                                  |
| Session Management and Caching                   | Yes    | Yes                                                                                  |
| E-Commerce Applications                          | Yes    | Yes                                                                                  |
| User Profile Management, AAA Protocol            | Yes    | Yes                                                                                  |

## NDB 和 InnoDB 功能使用总结

在将应用程序功能要求与 InnoDB 与 NDB 的功能进行比较时，一些显然与一种存储引擎比另一种更兼容。

下表根据每个功能通常更适合的存储引擎列出了支持的应用程序功能。

表 23.4 支持的应用程序功能（根据每个功能通常更适合的存储引擎）

InnoDB 的首选应用要求

- 外键(笔记NDB Cluster 8.0 支持外键)
- 全表扫描
- 非常大的数据库、行或事务
- READ COMMITTED 以外的事务

NDB的首选申请要求

- 写入缩放
- 99.999% 的正常运行时间
- 在线添加节点和在线模式操作
- 多个 SQL 和 NoSQL API（请参阅 NDB Cluster API：概述和概念）
- 实时性能
- 限制使用 BLOB 列
- 支持外键，尽管它们的使用可能会影响高吞吐量时的性能
