
# MySQL NDB 集群 8.0

<https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html>

## 目录

[23.1 一般信息](#一般信息)
[23.2 NDB集群概述](NDB集群概述/NDB集群概述.md)
[23.3 NDB集群安装](NDB集群安装/NDB集群安装.md)
23.4 NDB集群的配置
23.5 NDB集群程序
23.6 NDB集群的管理
23.7 NDB集群复制
23.8 NDB集群发行说明

## 概述

本章提供有关 MySQL NDB Cluster 的信息，这是适用于分布式计算环境的 MySQL 的高可用性、高冗余版本。最新的 NDB Cluster 发布系列使用 NDB 存储引擎（也称为 NDBCLUSTER）的版本 8 来支持在集群中运行具有 MySQL 服务器和其他软件的多台计算机。 NDB Cluster 8.0 现在作为通用可用性 (GA) 版本（从版本 8.0.19 开始）提供，包含 NDB 存储引擎的版本 8.0。 NDB Cluster 7.6 和 NDB Cluster 7.5 仍可作为 GA 版本使用，分别使用 NDB 的 7.6 和 7.5 版本。以前的 GA 版本仍可用于生产，NDB Cluster 7.4 和 NDB Cluster 7.3，分别包含 NDB 版本 7.4 和 7.3。 NDB 7.2 和更早的版本系列不再受支持或维护。

本章包含有关 NDB Cluster 8.0 版本到 8.0.30 的信息。 NDB Cluster 8.0 现在作为通用版本发布（从 NDB 8.0.19 开始），建议用于新部署；最新的可用版本是 NDB 8.0.28。

## 一般信息

MySQL NDB Cluster 使用带有 NDB 存储引擎的 MySQL 服务器。 Oracle 构建的标准 MySQL Server 8.0 二进制文件中不包含对 NDB 存储引擎的支持。相反，来自 Oracle 的 NDB Cluster 二进制文件的用户应该升级到最新的 NDB Cluster 二进制版本以支持平台——这些平台包括应该与大多数 Linux 发行版一起使用的 RPM。从源代码构建的 NDB Cluster 8.0 用户应使用为 MySQL 8.0 提供的源代码，并使用提供 NDB 支持所需的选项进行构建。

> 重要的
MySQL NDB Cluster 不支持 InnoDB Cluster，它必须使用带有 InnoDB 存储引擎的 MySQL Server 8.0 以及未包含在 NDB Cluster 分发中的其他应用程序进行部署。 MySQL Server 8.0 二进制文件不能与 MySQL NDB Cluster 一起使用。

支持的平台。 NDB Cluster 目前在许多平台上可用并受支持。有关操作系统版本、操作系统发行版和硬件平台的特定组合可用的确切支持级别，请参阅 <https://www.mysql.com/support/supportedplatforms/cluster.html>。

可用性。 NDB Cluster 二进制和源包可从 <https://dev.mysql.com/downloads/cluster/> 获得支持的平台。

NDB Cluster 版本号。 NDB 8.0 遵循与 MySQL Server 8.0 系列版本相同的发布模式，从 MySQL 8.0.13 和 MySQL NDB Cluster 8.0.13 开始。在本手册和其他 MySQL 文档中，我们使用以“NDB”开头的版本号来标识这些和更高版本的 NDB Cluster 版本。此版本号是 NDB 8.0 版本中使用的 NDBCLUSTER 存储引擎的版本号，与 NDB Cluster 8.0 版本所基于的 MySQL 8.0 服务器版本相同。

NDB Cluster 软件中使用的版本字符串。 MySQL NDB Cluster 发行版提供的 mysql 客户端显示的版本字符串使用以下格式：
`mysql-mysql_server_version-cluster`
mysql_server_version 表示 NDB Cluster 版本所基于的 MySQL 服务器的版本。对于所有 NDB Cluster 8.0 版本，这是 8.0.n，其中 n 是版本号。使用 -DWITH_NDBCLUSTER 或等效项从源代码构建将 -cluster 后缀添加到版本字符串。 （请参阅[第 23.3.1.4 节，“在 Linux 上从源代码构建 NDB 集群](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-source.html)。）您可以在 mysql 客户端中看到这种格式，如下所示：

```sql
$> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 8.0.29-cluster Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SELECT VERSION()\G
*************************** 1. row ***************************
VERSION(): 8.0.29-cluster
1 row in set (0.00 sec)
```

使用 MySQL 8.0 的 NDB Cluster 的第一个通用版本是 NDB 8.0.19，使用 MySQL 8.0.19。

MySQL 8.0 发行版中通常不包含的其他 NDB Cluster 程序显示的版本字符串使用以下格式：
`mysql-mysql_server_version ndb-ndb_engine_version`
mysql_server_version 表示 NDB Cluster 版本所基于的 MySQL 服务器的版本。 对于所有 NDB Cluster 8.0 版本，这是 8.0.n，其中 n 是版本号。 ndb_engine_version 是此版本的 NDB Cluster 软件使用的 NDB 存储引擎的版本。 对于所有 NDB 8.0 版本，此数字与 MySQL 服务器版本相同。 您可以在 ndb_mgm 客户端的 SHOW 命令的输出中看到这种格式，如下所示：

```sql
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @10.0.10.6  (mysql-8.0.29 ndb-8.0.30, Nodegroup: 0, *)
id=2    @10.0.10.8  (mysql-8.0.29 ndb-8.0.30, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=3    @10.0.10.2  (mysql-8.0.29 ndb-8.0.30)

[mysqld(API)]   2 node(s)
id=4    @10.0.10.10  (mysql-8.0.29 ndb-8.0.30)
id=5 (not connected, accepting connect from any host)
```

与标准 MySQL 8.0 版本的兼容性。虽然许多标准 MySQL 模式和应用程序可以使用 NDB Cluster 运行，但未修改的应用程序和数据库模式在使用 NDB Cluster 运行时可能会稍微不兼容或性能欠佳（请参阅[第 23.2.7 节，“NDB Cluster 的已知限制”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations.html))。大多数这些问题都可以克服，但这也意味着您不太可能切换现有的应用程序数据存储（例如，当前使用 MyISAM 或 InnoDB）来使用 NDB 存储引擎，而不允许模式、查询和应用程序的变化。在没有 NDB 支持的情况下编译的 mysqld（即，在没有 -DWITH_NDBCLUSTER_STORAGE_ENGINE 或其别名 -DWITH_NDBCLUSTER 的情况下构建）不能用作使用它构建的 mysqld 的直接替代品。

NDB Cluster 开发源树。 NDB Cluster 开发树也可以从 <https://github.com/mysql/mysql-server> 访问。

在 <https://github.com/mysql/mysql-server> 维护的 NDB Cluster 开发源在 GPL 下获得许可。有关使用 Git 获取 MySQL 源并自己构建它们的信息，请参阅[第 2.9.5 节，“使用开发源树安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html)。

> 笔记
与 MySQL Server 8.0 一样，NDB Cluster 8.0 版本是使用 CMake 构建的。

NDB Cluster 8.0 从 NDB 8.0.19 开始作为通用版本发布，建议用于新部署。 NDB Cluster 7.6 和 7.5 是以前的 GA 版本，仍然支持生产；有关 NDB Cluster 7.6 的信息，请参阅 NDB Cluster 7.6 中的新增功能。

随着 NDB Cluster 的不断发展，本章的内容可能会进行修订。有关 NDB Cluster 的其他信息可以在 MySQL 网站 <http://www.mysql.com/products/cluster/> 上找到。

其他资源。有关 NDB Cluster 的更多信息可以在以下位置找到：

- 有关 NDB Cluster 的一些常见问题的答案，请参阅[第 A.10 节，“MySQL 8.0 FAQ：NDB Cluster”](https://dev.mysql.com/doc/refman/8.0/en/faqs-mysql-cluster.html)。
- NDB 集群论坛：<https://forums.mysql.com/list.php?25>。
- 许多 NDB Cluster 用户和开发人员在博客中讲述了他们使用 NDB Cluster 的经历，并通过 [PlanetMySQL](http://www.planetmysql.org/) 提供了这些信息。
