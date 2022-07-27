# InnoDB ReplicaSet

## 目录

9.1 部署 InnoDB ReplicaSet
9.2 配置 InnoDB ReplicaSet 实例
9.3 创建 InnoDB ReplicaSet
9.4 向 ReplicaSet 添加实例
9.5 采用现有的复制设置
9.6 更改主实例
9.7 强制一个新的主实例
9.8 InnoDB ReplicaSet 锁定
9.9 标记副本集
9.10 检查 InnoDB ReplicaSet 的状态
9.11 升级 InnoDB ReplicaSet

## 概述

MySQL InnoDB ReplicaSet 结合了 MySQL 技术，使您能够部署和管理 Replication 复制。

一个 InnoDB ReplicaSet 至少包含两个 MySQL Server 实例，它提供了您熟悉的所有 MySQL Replication 功能，例如读取横向扩展和数据安全性。 InnoDB ReplicaSet 使用以下 MySQL 技术：

- [MySQL Shell](https://dev.mysql.com/doc/mysql-shell/8.0/en/)，它是 MySQL 的高级客户端和代码编辑器。
- MySQL 服务器 和 复制，它使一组 MySQL 实例能够提供可用性和异步读取横向扩展。 InnoDB ReplicaSet 提供了一种替代的、易于使用的编程方式来使用 Replication。
- [MySQL Router](https://dev.mysql.com/doc/mysql-router/8.0/en/)，一个轻量级的中间件，在你的应用程序和 InnoDB ReplicaSet 之间提供透明的路由。

InnoDB ReplicaSet 的接口类似于 [MySQL InnoDB Cluster](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-innodb-cluster.html)，您使用 MySQL Shell 将 MySQL Server 实例作为 ReplicaSet 使用，并且 MySQL Router 也以与 InnoDB Cluster 相同的方式紧密集成。

基于 MySQL 复制，InnoDB ReplicaSet 有一个主实例，它复制到一个或多个辅助实例。 InnoDB ReplicaSet 不提供 InnoDB Cluster 提供的所有功能，例如自动故障转移或多主模式。但是，它确实支持以类似方式配置、添加和删除实例等功能。您可以手动切换或故障转移到辅助实例，例如在发生故障时。您甚至可以采用现有的 Replication 部署，然后将其作为 InnoDB ReplicaSet 进行管理。

您使用作为 MySQL Shell 的一部分提供的 [AdminAPI](https://dev.mysql.com/doc/mysql-shell/8.0/en/admin-api-overview.html) 使用 InnoDB ReplicaSet。 AdminAPI 在 JavaScript 和 Python 中可用，非常适合脚本和 MySQL 部署的自动化，以实现高可用性和可扩展性。通过使用 MySQL Shell 的 AdminAPI，您可以避免手动配置许多实例的需要。相反，AdminAPI 为 MySQL 实例集提供了一个有效的现代接口，使您能够从一个中央工具配置、管理和监控您的部署。

要开始使用 InnoDB ReplicaSet，您需要[下载](https://dev.mysql.com/downloads/shell/)并[安装](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install.html) MySQL Shell。你需要一些安装了 MySQL Server 实例的主机，你也可以[安装](https://dev.mysql.com/doc/mysql-router/8.0/en/mysql-router-installation.html) MySQL Router。

InnoDB ReplicaSet 支持 [MySQL Clone](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin.html)，使您能够简单地配置实例。过去，要在新实例加入 MySQL 复制部署之前对其进行配置，您需要以某种方式手动将事务传输到加入实例。这可能涉及制作文件副本、手动复制它们等等。您可以简单地向副本集添[加一个实例](https://dev.mysql.com/doc/mysql-shell/8.0/en/add-instance-replicaset.html)，它会自动配置。

同样，InnoDB ReplicaSet 与 [MySQL Router](https://dev.mysql.com/doc/mysql-router/8.0/en/) 紧密集成，您可以使用 AdminAPI 与它们[一起工作](https://dev.mysql.com/doc/mysql-shell/8.0/en/registered-routers.html)。 MySQL Router 可以在称为[引导-bootstrapping](https://dev.mysql.com/doc/mysql-shell/8.0/en/admin-api-bootstrapping-router.html)的过程中基于 InnoDB ReplicaSet 自动配置自己，这消除了您手动配置路由的需要。 MySQL Router 然后透明地将客户端应用程序连接到 InnoDB ReplicaSet，为客户端连接提供路由和负载平衡。这种集成还使您能够管理使用 AdminAPI 针对 InnoDB ReplicaSet 引导的 MySQL 路由器的某些方面。 InnoDB ReplicaSet 状态信息包括有关针对 ReplicaSet 引导的 MySQL 路由器的详细信息。操作使您能够在 ReplicaSet 级别[创建 MySQL Router 用户](https://dev.mysql.com/doc/mysql-shell/8.0/en/configuring-router-user.html)，以使用针对 ReplicaSet 引导的 MySQL Routers，等等。

有关这些技术的更多信息，请参阅描述中链接的用户文档。除了此用户文档之外，MySQL Shell JavaScript API 参考或 MySQL Shell Python API 参考中还有所有 AdminAPI 方法的开发人员文档，可从[连接器和API](https://dev.mysql.com/doc/index-connectors.html) 获得。

AdminAPI 包括对 InnoDB ReplicaSet 的支持，它使您能够管理一组 MySQL 实例，该实例类似地运行基于 GTID 的异步复制，它完全基于事务，到 InnoDB Cluster。 InnoDB ReplicaSet 由单个主节点和多个辅助节点（传统上称为 MySQL 复制源和副本）组成。

您可以使用 ReplicaSet 对象和 AdminAPI 操作来管理 ReplicaSet，例如，检查 InnoDB ReplicaSet 的状态，并在发生故障时手动故障转移到新的主节点。

与 InnoDB Cluster 类似，MySQL Router 支持针对 InnoDB ReplicaSet 的引导，这意味着您可以自动配置 MySQL Router 以使用您的 InnoDB ReplicaSet，而无需手动配置它。这种自动配置使 InnoDB ReplicaSet 成为启动和运行 MySQL 复制和 MySQL Router 的快速简便的方法。它适合在不需要 InnoDB Cluster 提供的高可用性的用例中扩展读取并提供手动故障转移功能。

除了使用 AdminAPI 部署 InnoDB ReplicaSet 之外，您还可以采用现有的复制设置。 AdminAPI 根据复制设置的拓扑配置 InnoDB ReplicaSet。完成复制设置后，您可以像从头开始部署的 InnoDB ReplicaSet 一样管理它。您可以利用 AdminAPI 和 MySQL Router，而无需创建新的 ReplicaSet。有关详细信息，请参阅[第 9.5 节，“采用现有的复制设置”](https://dev.mysql.com/doc/mysql-shell/8.0/en/replicaset-adopting.html)。

您可以在广域网 (WAN) 上使用 InnoDB ReplicaSet，而不会影响写入性能，因为服务器实例通过异步复制通道连接，并且不需要对事务达成共识。但是，复制滞后在 WAN 上更大。这种滞后导致 InnoDB ReplicaSet 中的辅助服务器在主服务器之后更远。

## 限制

InnoDB ReplicaSet 限制。与 InnoDB Cluster 相比，InnoDB ReplicaSet 有几个限制。建议您尽可能部署 InnoDB Cluster。通常，InnoDB ReplicaSet 本身并不能提供高可用性。 InnoDB ReplicaSet 的限制包括：

- 没有自动故障转移。在主服务器不可用的情况下，需要使用 AdminAPI 手动触发故障转移，然后才能再次进行任何更改。但是，辅助实例仍可用于读取。
- 由于意外停止或不可用，无法防止部分数据丢失：意外停止时未完成的事务可能会丢失。
- 在意外退出或不可用后无法防止不一致。如果手动故障转移升级辅助实例，而前一个主实例仍然可用，例如，由于网络分区，脑裂情况可能会导致数据不一致。
- InnoDB ReplicaSet 不支持多主模式。允许写入所有成员的经典复制拓扑无法保证数据一致性。
- 读取横向扩展是有限的。 InnoDB ReplicaSet 基于异步复制，因此无法像 Group Replication 那样调整流控制。
- 所有次要成员都从单一来源复制。对于某些特定的用例，这可能会影响单一来源，例如，大量的小更新。
- 仅支持运行 MySQL 8.0 及更高版本的实例。
- 仅支持基于 GTID 的复制，二进制日志文件位置复制与 InnoDB ReplicaSet 不兼容。
- 仅支持基于行的复制 (RBR)，不支持基于语句的复制 (SBR)。
- 不支持复制过滤器。
- 任何实例都不允许非托管复制通道。
- 一个 ReplicaSet 最多包含一个主实例。支持一个或多个辅助。尽管可以添加到 ReplicaSet 的辅助节点的数量没有限制，但连接到 ReplicaSet 的每个 MySQL 路由器都必须监视每个实例。因此，添加到 ReplicaSet 中的实例越多，监控就越多。

ReplicaSet 必须由 MySQL Shell 管理。例如，复制帐户由 MySQL Shell 创建和管理。不支持在 MySQL Shell 之外对实例进行配置更改，例如，直接使用 SQL 语句更改主实例。始终使用 MySQL Shell 来处理 InnoDB ReplicaSet。
