# InnoDB 集群的用户帐户

InnoDB Cluster 中的成员服务器使用三种类型的用户帐户。一个 InnoDB Cluster 服务器配置帐户用于配置集群的服务器实例。可以创建一个或多个 InnoDB Cluster 管理员帐户，供管理员在集群设置后管理服务器实例。可以为 MySQL Router 实例创建一个或多个 MySQL Router 帐户以连接到集群。每个用户帐户必须存在于 InnoDB Cluster 中的所有成员服务器上，具有相同的用户名和相同的密码。

**InnoDB Cluster 服务器配置账户**
此帐户用于创建和配置 InnoDB Cluster 的成员服务器。每个成员服务器只有一个服务器配置帐户。集群中的每个成员服务器必须使用相同的用户帐户名和密码。为此，您可以使用服务器上的 root 帐户，但如果这样做，集群中每个成员服务器上的 root 帐户必须具有相同的密码。出于安全原因，不建议这样做。

首选方法是使用带有 clusterAdmin 选项的 dba.configureInstance() 命令创建 InnoDB Cluster 服务器配置帐户。为提高安全性，请在交互式提示符处指定密码，否则使用 clusterAdminPassword 选项指定密码。在将成为 InnoDB Cluster 一部分的每个服务器实例上以相同的方式使用相同的用户名和密码创建相同的帐户 - 您连接以创建集群的实例以及将加入集群的实例在那之后。

dba.configureInstance() 命令自动授予帐户所需的权限。如果愿意，您可以手动设置帐户，授予其[手动配置 InnoDB 集群管理员帐户](https://dev.mysql.com/doc/mysql-shell/8.0/en/innodb-cluster-user-accounts.html#admin-api-configuring-users)中列出的权限。除了完全的 MySQL 管理员权限外，该帐户还需要对 InnoDB Cluster 元数据表的完全读取和写入权限。

您使用 dba.configureInstance() 操作创建的 InnoDB Cluster 服务器配置帐户不会复制到 InnoDB Cluster 中的其他服务器。 MySQL Shell 禁用 dba.configureInstance() 操作的二进制日志记录。这意味着您必须在每个服务器实例上单独创建帐户。

**InnoDB 集群管理员帐户**
完成配置过程后，这些帐户可用于管理 InnoDB Cluster。您可以设置多个。每个帐户必须以相同的用户名和密码存在于 InnoDB Cluster 中的每个成员服务器上。

要为 InnoDB ClusterSet 部署创建 InnoDB Cluster 管理员帐户，请在将所有实例添加到该集群后发出 cluster.setupAdminAccount() 命令。该命令使用您指定的用户名和密码创建一个帐户，并具有所有必需的权限。使用 cluster.setupAdminAccount() 创建帐户的事务被写入二进制日志并发送到集群中的所有其他服务器实例以在它们上创建帐户。

> 笔记
如果主 InnoDB Cluster 是在 MySQL Shell 8.0.20 之前的版本中设置的，则 cluster.setupAdminAccount() 命令可能已与 update 选项一起使用以更新 InnoDB Cluster 服务器配置帐户的权限。这是不写入二进制日志的命令的特殊用途。

**MySQL 路由器帐户**
MySQL Router 使用这些帐户连接到 InnoDB Cluster 中的服务器实例。您可以设置多个。每个帐户必须以相同的用户名和密码存在于 InnoDB Cluster 中的每个成员服务器上。创建 MySQL 路由器帐户的过程与创建 InnoDB 集群管理员帐户的过程相同，但使用 cluster.setupRouterAccount() 命令。有关创建或升级 MySQL 路由器帐户的说明，请参阅[第 6.10.2 节，“配置 MySQL 路由器用户”](https://dev.mysql.com/doc/mysql-shell/8.0/en/configuring-router-user.html)。

## 手动配置 InnoDB 集群管理员帐户

如果要手动配置 InnoDB Cluster 管理用户，则该用户需要此处列出的权限，所有权限都带有 GRANT OPTION。

> 笔记
此权限列表基于 MySQL Shell 的当前版本。权限可能会在不同版本之间发生变化。因此，推荐的设置管理帐户的方法是使用 dba.configureInstance() 或 cluster.setupAdminAccount() 操作。
> 重要
用于管理 InnoDB Cluster、InnoDB ClusterSet 或 InnoDB ReplicaSet 部署的每个帐户都必须存在于部署中的所有成员服务器实例上，并且具有相同的用户名和相同的密码。

- Global privileges on `*.*` for RELOAD, SHUTDOWN, PROCESS, FILE, SELECT, SUPER, REPLICATION SLAVE, REPLICATION CLIENT, REPLICATION_APPLIER, CREATE USER, SYSTEM_VARIABLES_ADMIN, PERSIST_RO_VARIABLES_ADMIN, BACKUP_ADMIN, CLONE_ADMIN, and EXECUTE.

> 笔记
SUPER 包括以下必需的权限：SYSTEM_VARIABLES_ADMIN、SESSION_VARIABLES_ADMIN、REPLICATION_SLAVE_ADMIN、GROUP_REPLICATION_ADMIN、REPLICATION_SLAVE_ADMIN、ROLE_ADMIN。

- Schema specific privileges for `mysql_innodb_cluster_metadata.*, mysql_innodb_cluster_metadata_bkp.*, and mysql_innodb_cluster_metadata_previous.* are ALTER, ALTER ROUTINE, CREATE, CREATE ROUTINE, CREATE TEMPORARY TABLES, CREATE VIEW, DELETE, DROP, EVENT, EXECUTE, INDEX, INSERT, LOCK TABLES, REFERENCES, SHOW VIEW, TRIGGER, UPDATE; and for mysql.* are INSERT, UPDATE, DELETE`.

如果只需要读取操作，例如创建用于监控目的的用户，则可以使用具有更多受限权限的帐户。为用户 your_user 提供监控 InnoDB Cluster 问题所需的权限：

```sql
GRANT SELECT ON mysql_innodb_cluster_metadata.* TO your_user@'%';
GRANT SELECT ON mysql.slave_master_info TO your_user@'%';
GRANT SELECT ON performance_schema.global_status TO your_user@'%';
GRANT SELECT ON performance_schema.global_variables TO your_user@'%';
GRANT SELECT ON performance_schema.replication_applier_configuration TO your_user@'%';
GRANT SELECT ON performance_schema.replication_applier_status TO your_user@'%';
GRANT SELECT ON performance_schema.replication_applier_status_by_coordinator TO your_user@'%';
GRANT SELECT ON performance_schema.replication_applier_status_by_worker TO your_user@'%';
GRANT SELECT ON performance_schema.replication_connection_configuration TO your_user@'%';
GRANT SELECT ON performance_schema.replication_connection_status TO your_user@'%';
GRANT SELECT ON performance_schema.replication_group_member_stats TO your_user@'%';
GRANT SELECT ON performance_schema.replication_group_members TO your_user@'%';
GRANT SELECT ON performance_schema.threads TO your_user@'%' WITH GRANT OPTION;
```

For more information, see [Account Management Statements](https://dev.mysql.com/doc/refman/8.0/en/account-management-statements.html).

## InnoDB Cluster 创建的内部用户帐户

作为使用 Group Replication 的一部分，InnoDB Cluster 创建内部恢复用户，从而启用集群中服务器之间的连接。这些用户是集群内部的，生成的用户的用户名遵循`mysql_innodb_cluster_server_id@%`的命名方案，其中server_id对于实例是唯一的。在 8.0.17 之前的版本中，生成用户的用户名遵循 `mysql_innodb_cluster_r[10_numbers]` 的命名方案。

用于这些内部用户的主机名设置为“%”。在 v8.0.17 之前，ipAllowlist 通过在 ipAllowlist 中为每个主机创建一个帐户来影响主机名行为。有关更多信息，请参阅[创建服务器白名单](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-innodb-cluster-securing.html#create-allowlist-servers)。

每个内部用户都有一个随机生成的密码。从版本 8.0.18 开始，AdminAPI 允许您更改为内部用户生成的密码。请参阅[重置恢复帐户密码](https://dev.mysql.com/doc/mysql-shell/8.0/en/innodb-cluster-user-accounts.html#reset-recovery-passwords)。随机生成的用户获得以下授权：
`GRANT REPLICATION SLAVE ON *.* to internal_user;`

内部用户帐户在种子实例上创建，然后复制到集群中的其他实例。内部用户是：

- 通过发出 dba.createCluster() 创建新集群时生成

- 通过发出 Cluster.addInstance() 将新实例添加到集群时生成

- 使用主要成员使用的身份验证插件生成

在 v8.0.17 之前，ipAllowlist 导致 Cluster.rejoinInstance() 删除旧的内部用户并生成新用户，而不是重用它们。

有关组复制所需的内部用户的更多信息，请参阅[分布式恢复的用户凭据](https://dev.mysql.com/doc/refman/8.0/en/group-replication-user-credentials.html)。

## 重置恢复帐户密码

从版本 8.0.18 开始，您可以使用 Cluster.resetRecoveryAccountsPassword() 操作来重置 InnoDB Cluster 创建的内部恢复帐户的密码，例如遵循自定义密码生命周期策略。使用 Cluster.resetRecoveryAccountsPassword() 操作重置集群使用的所有内部恢复帐户的密码。该操作为每个在线实例上的内部恢复帐户设置一个新的随机密码。如果无法访问实例，则操作失败。您可以使用 force 选项忽略此类实例，但不建议这样做，并且在使用此操作之前将实例重新联机更安全。此操作仅适用于 InnoDB Cluster 创建的密码，不能用于更新手动创建的密码。

> 笔记
执行此操作的用户必须具有所有必需的管理员权限，特别是 CREATE USER，以确保无论密码验证要求的策略如何都可以更改恢复帐户的密码。换句话说，与 password_require_current 系统变量是否启用无关。
