# NDB 集群的初始启动

配置好后启动集群并不是很困难。 每个集群节点进程必须单独启动，并且在它所在的主机上启动。 管理节点应该首先启动，然后是数据节点，最后是任何 SQL 节点：

1. 在管理主机上，从系统 shell 发出以下命令以启动管理节点进程：
`$> ndb_mgmd --initial -f /var/lib/mysql-cluster/config.ini`
第一次启动时，必须使用 -f 或 --config-file 选项告诉 ndb_mgmd 在哪里可以找到其配置文件。 此选项要求还指定 --initial 或 --reload； 有关详细信息，请参阅[第 23.5.4 节，“ndb_mgmd - NDB Cluster Management Server Daemon”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html)。

2. 在每个数据节点主机上，运行以下命令来启动 ndbd 进程：
`$> ndbd`

3. 如果您使用 RPM 文件在 SQL 节点所在的集群主机上安装 MySQL，您可以（并且应该）使用提供的启动脚本在 SQL 节点上启动 MySQL 服务器进程。

如果一切顺利，并且集群设置正确，那么集群现在应该可以运行了。 您可以通过调用 ndb_mgm 管理节点客户端对此进行测试。 输出应该看起来像这里显示的那样，尽管您可能会看到输出中的一些细微差异，具体取决于您使用的 MySQL 的确切版本：

```bash
$> ndb_mgm
-- NDB Cluster -- Management Client --
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=2    @198.51.100.30  (Version: 8.0.29-ndb-8.0.30, Nodegroup: 0, *)
id=3    @198.51.100.40  (Version: 8.0.29-ndb-8.0.30, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @198.51.100.10  (Version: 8.0.29-ndb-8.0.30)

[mysqld(API)]   1 node(s)
id=4    @198.51.100.20  (Version: 8.0.29-ndb-8.0.30)
```

SQL 节点在此处引用为 [mysqld(API)]，这反映了 mysqld 进程充当 NDB Cluster API 节点的事实。

> 笔记
在 SHOW 的输出中为给定的 NDB Cluster SQL 或其他 API 节点显示的 IP 地址是 SQL 或 API 节点用于连接到集群数据节点而不是任何管理节点的地址。

您现在应该准备好使用 NDB Cluster 中的数据库、表和数据。 有关简要讨论，请参阅[第 23.3.5 节，“带有表和数据的 NDB 集群示例”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-example-data.html)。
