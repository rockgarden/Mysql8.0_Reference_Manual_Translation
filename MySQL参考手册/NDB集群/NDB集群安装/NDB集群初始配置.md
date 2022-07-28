# NDB Cluster 的初始配置

在本节中，我们通过创建和编辑配置文件来讨论已安装的 NDB Cluster 的手动配置。

对于我们的[四节点、四主机 NDB 集群](NDB集群安装.md#安装节点主机)，需要编写四个配置文件，每个节点主机一个。

- 每个数据节点或 SQL 节点都需要一个 my.cnf 文件，该文件提供两条信息：一个告诉节点在哪里找到管理节点的连接字符串，以及一个告诉该主机（托管数据节点的机器）上的 MySQL 服务器的行) 以启用 NDBCLUSTER 存储引擎。

  有关连接字符串的更多信息，请参阅[第 23.4.3.3 节，“NDB 集群连接字符串”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-connection-strings.html)。

- 管理节点需要一个 config.ini 文件，告诉它要维护多少个片段副本，为每个数据节点上的数据和索引分配多少内存，在哪里找到数据节点，在每个数据节点上将数据保存到磁盘的哪里，以及在哪里可以找到任何 SQL 节点。

**配置数据节点和SQL节点**。数据节点所需的 my.cnf 文件相当简单。配置文件应该位于 /etc 目录中，并且可以使用任何文本编辑器进行编辑。 （如果文件不存在，则创建该文件。）例如：
`$> vi /etc/my.cnf`

> 笔记
我们在这里展示了 vi 用于创建文件，但任何文本编辑器都应该可以正常工作。

对于我们示例设置中的每个数据节点和 SQL 节点，my.cnf 应如下所示：

```ini
[mysqld]
# Options for mysqld process:
ndbcluster                      # run NDB storage engine

[mysql_cluster]
# Options for NDB Cluster processes:
ndb-connectstring=198.51.100.10  # location of management server
```

输入上述信息后，保存该文件并退出文本编辑器。 对托管数据节点“A”、数据节点“B”和 SQL 节点的机器执行此操作。

> 重要
如前所示，在 my.cnf 文件的 [mysqld] 和 [mysql_cluster] 部分中使用 ndbcluster 和 ndb-connectstring 参数启动 mysqld 进程后，如果没有实际启动，就无法执行任何 CREATE TABLE 或 ALTER TABLE 语句 簇。 否则，这些语句将失败并出现错误。 这是设计使然。

**配置管理节点**。 配置管理节点的第一步是创建可以找到配置文件的目录，然后创建文件本身。 例如（以 root 身份运行）：

```bash
$> mkdir /var/lib/mysql-cluster
$> cd /var/lib/mysql-cluster
$> vi config.ini
```

对于我们的典型设置，config.ini 文件应如下所示：

```ini
[ndbd default]
# Options affecting ndbd processes on all data nodes:
NoOfReplicas=2    # Number of fragment replicas
DataMemory=98M    # How much memory to allocate for data storage

[ndb_mgmd]
# Management process options:
HostName=198.51.100.10          # Hostname or IP address of management node
DataDir=/var/lib/mysql-cluster  # Directory for management node log files

[ndbd]
# Options for data node "A":
                                # (one [ndbd] section per data node)
HostName=198.51.100.30          # Hostname or IP address
NodeId=2                        # Node ID for this data node
DataDir=/usr/local/mysql/data   # Directory for this data node's data files

[ndbd]
# Options for data node "B":
HostName=198.51.100.40          # Hostname or IP address
NodeId=3                        # Node ID for this data node
DataDir=/usr/local/mysql/data   # Directory for this data node's data files

[mysqld]
# SQL node options:
HostName=198.51.100.20          # Hostname or IP address
                                # (additional mysqld connections can be
                                # specified for this node for various
                                # purposes such as running ndb_restore)
```

> 笔记
世界数据库可以从 <https://dev.mysql.com/doc/index-other.html> 下载。

在创建了所有配置文件并指定了这些最小选项之后，您就可以继续启动集群并验证所有进程是否正在运行。 我们在[NDB 集群的初始启动](NDB集群的初始启动.md)中讨论了这是如何完成的。

有关可用 NDB Cluster 配置参数及其用途的更多详细信息，请参阅[第 23.4.3 节，“NDB 集群配置文件”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-config-file.html)和[第 23.4 节，“NDB 集群的配置”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-configuration.html)。 有关与备份相关的 NDB Cluster 配置，请参阅[第 23.6.8.3 节，“NDB Cluster 备份的配置”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup-configuration.html)。

> 笔记
Cluster 管理节点的默认端口是 1186； 数据节点的默认端口是 2202。但是，集群可以从已经空闲的数据节点中自动为数据节点分配端口。
