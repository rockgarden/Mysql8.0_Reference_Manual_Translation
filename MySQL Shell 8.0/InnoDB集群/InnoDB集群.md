# InnoDB 集群

本章介绍 MySQL InnoDB Cluster，它结合了 MySQL 技术，使您能够为 MySQL 部署和管理一个完整的集成高可用性解决方案。

## 目录

7.1 InnoDB 集群要求
7.2 InnoDB 集群限制
7.3 InnoDB Cluster 的用户账户
7.4 部署生产 InnoDB 集群
7.5 配置 InnoDB 集群
7.6 保护 InnoDB 集群
7.7 监控 InnoDB 集群
7.8 恢复和重启 InnoDB 集群
7.9 修改或解散 InnoDB 集群
7.10 升级 InnoDB 集群

## 概述

一个 InnoDB Cluster 至少由 **三个 MySQL 服务器实例** 组成，它提供了高可用性和扩展特性。 InnoDB Cluster 使用以下 MySQL 技术：

- MySQL Shell，它是 MySQL 的高级客户端和代码编辑器。
- MySQL 服务器和组复制，它使一组 MySQL 实例能够提供高可用性。 InnoDB Cluster 提供了一种替代的、易于使用的编程方式来处理 Group Replication。
- MySQL Router，一个轻量级的中间件，在你的应用程序和 InnoDB Cluster 之间提供透明的路由。

建立在 MySQL [Group Replication](https://dev.mysql.com/doc/refman/8.0/en/group-replication.html) 之上，提供了自动成员管理、容错、自动故障转移等功能。 InnoDB Cluster 通常以单主模式运行，具有一个主实例（读写）和多个辅助实例（只读）。高级用户还可以利用多主模式[multi-primary](https://dev.mysql.com/doc/refman/8.0/en/group-replication-multi-primary-mode.html)，其中所有实例都是主实例。您甚至可以在 InnoDB Cluster 在线时更改集群的拓扑，以确保尽可能高的可用性。

您使用作为 MySQL Shell 的一部分提供的 [AdminAPI](https://dev.mysql.com/doc/mysql-shell/8.0/en/admin-api-overview.html) 使用 InnoDB Cluster。 AdminAPI 在 JavaScript 和 Python 中可用，非常适合脚本和 MySQL 部署的自动化，以实现高可用性和可扩展性。通过使用 MySQL Shell 的 AdminAPI，您可以避免手动配置许多实例的需要。相反，AdminAPI 为 MySQL 实例集提供了一个有效的现代接口，使您能够从一个中央工具配置、管理和监控您的部署。

要开始使用 InnoDB Cluster，您需要[下载](https://dev.mysql.com/downloads/shell/)并[安装](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install.html) MySQL Shell。你需要一些安装了 MySQL Server 实例的主机，你也可以[安装](https://dev.mysql.com/doc/mysql-router/8.0/en/mysql-router-installation.html) MySQL Router。

InnoDB Cluster 支持 [MySQL 克隆](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin.html)，这使您能够简单地配置实例。过去，要在新实例加入一组 MySQL 实例之前配置它，您需要以某种方式手动将事务传输到加入实例。这可能涉及制作文件副本、手动复制它们等等。使用 InnoDB Cluster，您可以简单地向集群添加一个实例，它会自动配置。

同样，InnoDB Cluster 与 MySQL Router 紧密集成，您可以使用 AdminAPI 将它们一起使用。 MySQL Router 可以在称为引导的过程中基于 InnoDB Cluster 自动配置自身，这消除了您手动配置路由的需要。 MySQL Router 然后透明地将客户端应用程序连接到 InnoDB Cluster，为客户端连接提供路由和负载平衡。这种集成还使您能够管理使用 AdminAPI 针对 InnoDB 集群引导的 MySQL 路由器的某些方面。 InnoDB Cluster 状态信息包括有关针对集群引导的 MySQL 路由器的详细信息。操作使您能够在集群级别创建 MySQL 路由器用户，使用针对集群引导的 MySQL 路由器，等等。

MySQL InnoDB Cluster 为 MySQL 提供了完整的高可用性解决方案。通过使用 MySQL Shell 中包含的 AdminAPI，您可以轻松配置和管理一组至少三个 MySQL 服务器实例，以充当 InnoDB Cluster。

InnoDB Cluster 中的每个 MySQL 服务器实例都运行 MySQL Group Replication，它提供了在 InnoDB Cluster 内复制数据的机制，并具有内置的故障转移功能。 AdminAPI 消除了在 InnoDB 集群中直接使用 Group Replication 的需要，但有关更多信息，请参阅解释详细信息的 Group Replication。从 MySQL 8.0.27 开始，您还可以设置 InnoDB ClusterSet（请参阅[第 8 章，MySQL InnoDB ClusterSet](https://dev.mysql.com/doc/mysql-shell/8.0/en/innodb-clusterset.html)），通过将主 InnoDB Cluster 与其在备用位置（例如不同位置）的一个或多个副本链接，为 InnoDB Cluster 部署提供容灾能力。数据中心。

MySQL Router 可以根据您部署的集群自动配置自身，将客户端应用程序透明地连接到服务器实例。如果服务器实例发生意外故障，集群会自动重新配置。在默认的单主模式下，InnoDB Cluster 有一个读写服务器实例 - 主服务器。多个辅助服务器实例是主服务器的副本。如果主服务器发生故障，辅助服务器会自动提升为主服务器的角色。 MySQL 路由器检测到这一点并将客户端应用程序转发到新的主节点。高级用户还可以将集群配置为具有多个主节点。

下图显示了这些技术如何协同工作的概述：

![innodb_cluster_overview](../../resources/innodb_cluster_overview.png)

> 重要的
InnoDB Cluster 不提供对 MySQL NDB Cluster 的支持。 NDB Cluster 依赖于 NDB 存储引擎以及一些特定于 NDB Cluster 的程序，这些程序未随 MySQL Server 8.0 提供； NDB 仅作为 MySQL NDB Cluster 分发的一部分提供。 此外，MySQL Server 8.0 提供的 MySQL 服务器二进制文件 (mysqld) 不能与 NDB Cluster 一起使用。 有关 MySQL NDB Cluster 的更多信息，请参阅[第 23 章，MySQL NDB Cluster 8.0](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html) 和[使用 InnoDB 的 MySQL 服务器与 NDB Cluster 的比较](../../MySQL参考手册/NDB集群/NDB集群概述/InnoDB与NDB的比较.md)。

## InnoDB 集群要求

在安装 InnoDB Cluster 的生产部署之前，请确保您打算使用的服务器实例满足以下要求。

- InnoDB Cluster 使用组复制，因此您的服务器实例必须满足相同的要求。请参阅[组复制要求](https://dev.mysql.com/doc/refman/8.0/en/group-replication-requirements.html)。 AdminAPI 提供 dba.checkInstanceConfiguration() 方法来验证实例是否满足 Group Replication 要求，并提供 dba.configureInstance() 方法来配置实例以满足要求。
  > 笔记
  使用沙盒部署时，实例被配置为自动满足这些要求。
- 用于 Group Replication 的数据，以及用于 InnoDB Cluster 的数据，必须存储在 InnoDB 事务存储引擎中。使用其他存储引擎，包括临时 MEMORY 存储引擎，可能会导致 Group Replication 出错。在将实例与 Group Replication 和 InnoDB Cluster 一起使用之前，将其他存储引擎中的任何表转换为使用 InnoDB。您可以通过在服务器实例上设置 disabled_storage_engines 系统变量来阻止使用其他存储引擎，例如：
`disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"`
- 设置集群时，任何服务器实例上都不得有入站复制通道。由 Group Replication 自动创建的通道（group_replication_applier 和 group_replication_recovery）在正在采用的复制组上是允许的。 InnoDB Cluster 不支持在使​​用 AdminAPI 管理的通道之外手动配置的异步复制通道。如果您将现有复制拓扑迁移到 InnoDB Cluster 部署并且需要在设置过程中暂时跳过此验证，您可以在创建集群时使用 force 选项绕过它。
- group_replication_tls_source 不得设置为 mysql_admin。
- 必须在要与 InnoDB Cluster 一起使用的任何实例上启用 Performance Schema。
- MySQL Shell 用于配置服务器以在 InnoDB Cluster 中使用的供应脚本需要访问 Python。在 Windows 上，MySQL Shell 包括 Python，不需要用户配置。在 Unix 上，Python 必须作为 shell 环境的一部分。要检查您的系统是否正确配置了 Python，请执行以下操作：
`$ /usr/bin/env python`
如果 Python 解释器启动，则无需进一步操作。如果前面的命令失败，请在 /usr/bin/python 和您选择的 Python 二进制文件之间创建一个软链接。
- 从版本 8.0.17 开始，实例必须在 InnoDB Cluster 中使用唯一的 server_id。使用 Cluster.addInstance(instance) 操作时，如果 instance 的 server_id 已被集群中的某个实例使用，则操作失败并出现错误。
- 从版本 8.0.23 开始，应将实例配置为使用并行复制应用程序。请参阅[配置并行复制应用程序](https://dev.mysql.com/doc/mysql-shell/8.0/en/configuring-innodb-cluster.html#configuring-parallel-applier)。
- 在为 InnoDB Cluster 配置实例的过程中，配置了使用实例所需的大部分系统变量。但是 AdminAPI 没有配置 transaction_isolation 系统变量，这意味着它默认为 REPEATABLE READ。这不会影响单主集群，但如果您使用的是多主集群，那么除非您在应用程序中依赖 REPEATABLE READ 语义，否则我们建议使用 READ COMMITTED 隔离级别。请参阅[组复制限制](https://dev.mysql.com/doc/refman/8.0/en/group-replication-limitations.html)。
- 实例的相关配置选项，尤其是 Group Replication 配置选项，必须位于单个选项文件中。 InnoDB Cluster 仅支持服务器实例的单个选项文件，不支持使用 --defaults-extra-file 选项指定附加选项文件。对于使用实例选项文件的任何 AdminAPI 操作，必须指定主文件。如果您想为与 InnoDB Cluster 无关的配置选项使用多个选项文件，则必须手动配置文件，确保根据使用多个选项文件的优先规则正确更新它们，并确保相关的设置到 InnoDB Cluster 不会被额外无法识别的选项文件中的选项错误地覆盖。

## InnoDB 集群限制

本节介绍 InnoDB Cluster 的已知限制。由于 InnoDB Cluster 使用 Group Replication，您还应该了解它的限制，请参阅 [Group Replication Limitations](https://dev.mysql.com/doc/refman/8.0/en/group-replication-limitations.html)。

> **重要**
由于元数据查询中的错误，MySQL Shell 8.0.27 无法用于管理运行 MySQL Server 8.0.25 的 InnoDB Cluster。要解决此问题，请先将 InnoDB Cluster 成员实例上的 MySQL Server 升级到 8.0.26 或 8.0.27 版本，然后再将 MySQL Shell 8.0.27 与集群一起使用。该问题将在 MySQL Shell 8.0.28 中修复。

- InnoDB Cluster 不管理手动配置的异步复制通道。 Group Replication 和 AdminAPI 不能确保异步复制仅在主节点上处于活动状态，并且状态不会跨实例复制。这可能导致复制不再起作用的各种情况，并可能导致脑裂(split brain)。一个 InnoDB Cluster 和另一个 InnoDB Cluster 之间的复制仅受 InnoDB ClusterSet 支持，它可从 MySQL 8.0.27 获得，并管理从活动主读写 InnoDB Cluster 到多个只读副本集群的复制。有关该解决方案的信息，请参阅第 8 章，MySQL InnoDB ClusterSet。

- InnoDB Cluster 旨在部署在局域网中。在广域网上部署单个 InnoDB Cluster 对写入性能有显着影响。稳定且低延迟的网络对于 InnoDB Cluster 成员服务器使用底层组复制技术相互通信以就事务达成共识非常重要。然而，InnoDB ClusterSet 设计为跨多个数据中心部署，每个 InnoDB Cluster 位于一个数据中心，异步复制通道将它们连接起来。

- 对于 AdminAPI 操作，您只能使用 TCP/IP 连接和经典 MySQL 协议连接到 InnoDB Cluster 中的服务器实例。 AdminAPI 操作不支持使用 Unix 套接字和命名管道，AdminAPI 操作不支持使用 X 协议。同样的限制也适用于服务器实例本身之间的连接。

> 笔记
客户端应用程序可以使用 X 协议和 Unix 套接字和命名管道连接到 InnoDB 集群中的实例。这些限制仅适用于使用 AdminAPI 命令的管理操作，以及实例之间的连接。

- AdminAPI 和 InnoDB Cluster 支持使用运行 MySQL Server 5.7 的实例。但是，这些实例还有其他限制，并且描述的某些功能在您使用它们时不适用。[第 6.2.1 节，“使用运行 MySQL 5.7 的实例”](https://dev.mysql.com/doc/mysql-shell/8.0/en/using-version-5-7.html)列出了其他限制。

- 使用多主模式时，不支持针对同一对象但在不同服务器上发出的并发数据定义语句和数据操作语句。在对象上发布数据定义语言 (DDL) 语句期间，在同一对象上但来自不同服务器实例的并发数据操作语言 (DML) 存在未检测到在不同实例上执行的 DDL 冲突的风险。有关详细信息，请参阅[组复制限制](https://dev.mysql.com/doc/refman/8.0/en/group-replication-limitations.html)。
