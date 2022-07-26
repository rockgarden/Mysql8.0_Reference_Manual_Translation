# MySQL 服务器使用 InnoDB 与 NDB Cluster 的比较

23.2.6.1 NDB 和 InnoDB 存储引擎的区别
23.2.6.2 NDB 和 InnoDB 工作负载
23.2.6.3 NDB 和 InnoDB 功能使用总结

MySQL Server 在存储引擎中提供了多种选择。由于 NDB 和 InnoDB 都可以作为事务性 MySQL 存储引擎，因此 MySQL Server 的用户有时会对 NDB Cluster 感兴趣。他们将 NDB 视为一种可能的替代方案，或者升级到 MySQL 8.0 中的默认 InnoDB 存储引擎。虽然 NDB 和 InnoDB 有共同的特点，但在架构和实现上存在差异，因此一些现有的 MySQL Server 应用程序和使用场景可以很好地适合 NDB Cluster，但不是全部。

在本节中，我们将讨论和比较 NDB 8.0 使用的 NDB 存储引擎与 MySQL 8.0 中使用的 InnoDB 的一些特性。接下来的几节提供了技术比较。在许多情况下，必须根据具体情况决定何时何地使用 NDB Cluster，并考虑所有因素。虽然为每个可能的使用场景提供细节超出了本文档的范围，但我们也尝试提供一些非常通用的指导，说明 NDB 相对于 InnoDB 后端的一些常见类型的应用程序的相对适用性。

NDB Cluster 8.0 使用基于 MySQL 8.0 的 mysqld，包括对 InnoDB 1.1 的支持。虽然可以将 InnoDB 表与 NDB Cluster 一起使用，但此类表不是集群的。也不能将 NDB Cluster 8.0 发行版中的程序或库与 MySQL Server 8.0 一起使用，反之亦然。

虽然某些类型的常见业务应用程序可以在 NDB Cluster 或 MySQL Server 上运行（最有可能使用 InnoDB 存储引擎）也是事实，但存在一些重要的架构和实现差异。第 23.2.6.1 节，“NDB 和 InnoDB 存储引擎之间的差异”，提供了这些差异的摘要。由于差异，一些使用场景显然更适合一种引擎或另一种；请参阅第 23.2.6.2 节，“NDB 和 InnoDB 工作负载”。这反过来又会影响更适合与 NDB 或 InnoDB 一起使用的应用程序类型。请参阅第 23.2.6.3 节，“NDB 和 InnoDB 功能使用摘要”，以比较每种在常见类型的数据库应用程序中的相对适用性。

有关 NDB 和 MEMORY 存储引擎的相对特征的信息，请参阅何时使用 MEMORY 或 NDB Cluster。

有关 MySQL 存储引擎的更多信息，请参阅第 16 章，替代存储引擎。
