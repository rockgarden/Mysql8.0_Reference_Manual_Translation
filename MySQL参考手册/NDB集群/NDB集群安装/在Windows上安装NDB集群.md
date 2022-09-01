# 在 Windows 上安装 NDB Cluster

23.3.1.1 [在 Windows 上从二进制版本安装 NDB Cluster](#在-windows-上从二进制版本安装-ndb-cluster)
23.3.1.2 在 Windows 上从源代码编译和安装 NDB Cluster
23.3.1.3 NDB Cluster 在 Windows 上的初始启动
23.3.1.4 将 NDB Cluster 进程安装为 Windows 服务

本节介绍 Windows 主机上 NDB Cluster 的安装过程。 可以从 <https://dev.mysql.com/downloads/cluster/> 获得适用于 Windows 的 NDB Cluster 8.0 二进制文件。 有关从 Oracle 提供的二进制版本在 Windows 上安装 NDB Cluster 的信息。

也可以使用 Microsoft Visual Studio 在 Windows 上从源代码编译和安装 NDB Cluster。

## 在 Windows 上从二进制版本安装 NDB Cluster

本节介绍了在 Windows 上使用 Oracle 提供的二进制“免安装”NDB Cluster 版本的 NDB Cluster 的基本安装，使用本节开头概述的相同 4 节点设置（请参阅[“NDB 集群安装”](/MySQL参考手册/NDB集群/NDB集群.md) )，如下表所示：

表 23.5 示例集群中节点的网络地址

| Node                   | IP Address    |
|------------------------|---------------|
| Management node (mgmd) | 198.51.100.10 |
| SQL node ([mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html))      | 198.51.100.20 |
| Data node "A" ([ndbd](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html))   | 198.51.100.30 |
| Data node "B" (ndbd)   | 198.51.100.40 |

与其他平台一样，运行 SQL 节点的 NDB Cluster 主机必须在其上安装 MySQL 服务器二进制文件 ([mysqld.exe](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html))。您还应该在此主机上安装 MySQL 客户端 (mysql.exe)。对于管理节点和数据节点，不需要安装 MySQL Server 二进制文件；但是，每个管理节点都需要管理服务器守护程序 ([ndb_mgmd.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html))；每个数据节点都需要数据节点守护程序（[ndbd.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html) 或 [ndbmtd.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html)）。对于此示例，我们将 ndbd.exe 称为数据节点可执行文件，但您可以以完全相同的方式安装该程序的多线程版本 ndbmtd.exe。您还应该在管理服务器主机上安装管理客户端 ([ndb_mgm.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html))。本节介绍为每种类型的 NDB Cluster 节点安装正确的 Windows 二进制文件所需的步骤。

> 笔记
与其他 Windows 程序一样，NDB Cluster 可执行文件以 .exe 文件扩展名命名。但是，从命令行调用这些程序时，不必包含 .exe 扩展名。因此，我们在本文档中经常将这些程序简称为 mysqld、mysql、ndb_mgmd 等。您应该明白，无论我们（例如）指的是 mysqld 还是 mysqld.exe，这两个名称都意味着相同的东西（MySQL 服务器程序）。

要使用 Oracles 的免安装二进制文件设置 NDB Cluster，安装过程的第一步是从 <https://dev.mysql.com/downloads/cluster/> 下载最新的 NDB Cluster Windows ZIP 二进制存档。该归档文件的文件名为 mysql-cluster-gpl-ver-winarch.zip，其中 ver 是 NDB 存储引擎版本（例如 8.0.28），arch 是架构（32 用于 32 位二进制文​​件，64对于 64 位二进制文​​件）。例如，用于 64 位 Windows 系统的 NDB Cluster 8.0.28 存档名为 mysql-cluster-gpl-8.0.28-win64.zip。

您可以在 32 位和 64 位版本的 Windows 上运行 32 位 NDB Cluster 二进制文件；但是，64 位 NDB Cluster 二进制文件只能在 64 位版本的 Windows 上使用。如果您在具有 64 位 CPU 的计算机上使用 32 位版本的 Windows，则必须使用 32 位 NDB Cluster 二进制文件。

为了尽量减少需要从 Internet 下载或在机器之间复制的文件数量，我们从您打算运行 SQL 节点的计算机开始。

**SQL node**。我们假设您已在 IP 地址为 198.51.100.20 的计算机上的目录 C:\Documents 和 Settings\username\My Documents\Downloads 中放置了存档的副本，其中用户名是当前用户的名称。 （您可以在命令行上使用 ECHO %USERNAME% 获取此名称。）要将 NDB Cluster 可执行文件作为 Windows 服务安装和运行，此用户应该是管理员组的成员。

从存档中提取所有文件。与 Windows 资源管理器集成的提取向导足以完成此任务。 （如果您使用不同的存档程序，请确保它从存档中提取所有文件和目录，并保留存档的目录结构。）当您被要求输入目标目录时，输入 C:\，这会导致提取将存档解压缩到目录 C:\mysql-cluster-gpl-ver-winarch 的向导。将此目录重命名为 C:\mysql。

可以将 NDB Cluster 二进制文件安装到 C:\mysql\bin 以外的目录；但是，如果这样做，则必须相应地修改此过程中显示的路径。特别是，如果 MySQL 服务器（SQL 节点）二进制文件安装到 C:\mysql 或 C:\Program Files\MySQL\MySQL Server 8.0 以外的位置，或者 SQL 节点的数据目录位于 C 以外的位置:\mysql\data 或 C:\Program Files\MySQL\MySQL Server 8.0\data，额外的配置选项必须在命令行中使用或在启动 SQL 节点时添加到 my.ini 或 my.cnf 文件中。有关将 MySQL 服务器配置为在非标准位置运行的更多信息，请参阅[第 2.3.4 节，“使用 noinstall ZIP 存档在 Microsoft Windows 上安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/windows-install-archive.html)。

要使具有 NDB Cluster 支持的 MySQL 服务器作为 NDB Cluster 的一部分运行，它必须使用选项 --ndbcluster 和 --ndb-connectstring 启动。 虽然您可以在命令行上指定这些选项，但将它们放在选项文件中通常更方便。 为此，请在记事本或其他文本编辑器中创建一个新文本文件。 在此文件中输入以下配置信息：

```bash
[mysqld]
# Options for mysqld process:
ndbcluster                       # run NDB storage engine
ndb-connectstring=198.51.100.10  # location of management server
```

如果需要，您可以添加此 MySQL 服务器使用的其他选项（请参阅[第 2.3.4.2 节，“创建选项文件”](https://dev.mysql.com/doc/refman/8.0/en/windows-create-option-file.html)），但该文件必须至少包含显示的选项。将此文件另存为 C:\mysql\my.ini。这样就完成了 SQL 节点的安装和设置。

**Data nodes**。 Windows 主机上的 NDB Cluster 数据节点只需要一个可执行文件，ndbd.exe 或 ndbmtd.exe 之一。对于此示例，我们假设您使用的是 ndbd.exe，但使用 ndbmtd.exe 时也适用相同的说明。在您希望运行数据节点的每台计算机（IP 地址为 198.51.100.30 和 198.51.100.40 的计算机）上，创建目录 C:\mysql、C:\mysql\bin 和 C:\mysql\cluster-数据;然后，在您下载并解压缩免安装存档的计算机上，在 C:\mysql\bin 目录中找到 ndbd.exe。将此文件复制到两个数据节点主机中的每一个上的 C:\mysql\bin 目录。

要作为 NDB Cluster 的一部分，必须为每个数据节点提供管理服务器的地址或主机名。您可以在启动每个数据节点进程时使用 [--ndb-connectstring](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-config.html#option_ndb_config_ndb-connectstring) 或 -c 选项在命令行上提供此信息。但是，通常最好将此信息放在选项文件中。为此，请在记事本或其他文本编辑器中创建一个新文本文件，然后输入以下文本：

```bash
[mysql_cluster]
# Options for data node process:
ndb-connectstring=198.51.100.10  # location of management server
```

在数据节点主机上将此文件另存为 C:\mysql\my.ini。创建另一个包含相同信息的文本文件并将其保存为 C:mysql\my.ini 在另一个数据节点主机上，或将 my.ini 文件从第一个数据节点主机复制到第二个数据节点主机，确保将复制到第二个数据节点的 C:\mysql 目录。两个数据节点主机现在都可以在 NDB Cluster 中使用，只剩下要安装和配置的管理节点。

**Management node**。用于托管 NDB Cluster 管理节点的计算机上唯一需要的可执行程序是管理服务器程序 ndb_mgmd.exe。但是，为了在 NDB Cluster 启动后对其进行管理，您还应该在与管理服务器相同的机器上安装 NDB Cluster 管理客户端程序 ndb_mgm.exe。在您下载并解压缩免安装存档的机器上找到这两个程序；这应该是 SQL 节点主机上的目录 C:\mysql\bin。在 IP 地址为 198.51.100.10 的计算机上创建目录 C:\mysql\bin，然后将两个程序复制到此目录。

您现在应该创建两个供 ndb_mgmd.exe 使用的配置文件：

1. 提供特定于管理节点本身的配置数据的本地配置文件。通常，此文件只需要提供 NDB Cluster 全局配置文件的位置（请参阅第 2 项）。
  要创建此文件，请在记事本或其他文本编辑器中新建一个文本文件，然后输入以下信息：
  
    ```bash
    [mysql_cluster]
    # Options for management node process
    config-file=C:/mysql/bin/config.ini
    ```

    将此文件保存为文本文件 C:\mysql\bin\my.ini。

2. 一个全局配置文件，管理节点可以从中获取管理 NDB Cluster 整体的配置信息。此文件至少必须包含 NDB Cluster 中每个节点的部分，以及管理节点和所有数据节点的 IP 地址或主机名（HostName 配置参数）。还建议包括以下附加信息：

    - 任何 SQL 节点的 IP 地址或主机名
    - 分配给每个数据节点的数据内存和索引内存（DataMemory 和 IndexMemory 配置参数）
    - 片段副本的数量，使用 NoOfReplicas 配置参数（请参阅[“NDB 集群节点、节点组、片段副本和分区”](../NDB集群概述/NDB节点和分区.md)）
    - 每个数据节点存储其数据和日志文件的目录，以及管理节点保存其日志文件的目录（在这两种情况下，DataDir 配置参数）

   使用记事本等文本编辑器新建一个文本文件，并输入以下信息：
  
    ```bash
    [ndbd default]
    # Options affecting ndbd processes on all data nodes:
    NoOfReplicas=2
    # Number of fragment replicas
    DataDir=C:/mysql/cluster-data
    # Directory for each data node's data files
    # Forward slashes used in directory path,
    # rather than backslashes. This is correct;
    # see Important note in text
    DataMemory=80M
    # Memory allocated to data storage
    IndexMemory=18M
    # Memory allocated to index storage
    # For DataMemory and IndexMemory, we have used the
    # default values. Since the "world" database takes up
    # only about 500KB, this should be more than enough for
    # this example Cluster setup.
    [ndb_mgmd]
    # Management process options:
    HostName=198.51.100.10
    # Hostname or IP address of management node
    DataDir=C:/mysql/bin/cluster-logs
    # Directory for management node log files
    [ndbd]
    # Options for data node "A":
    # (one [ndbd] section per data node)
    HostName=198.51.100.30
    # Hostname or IP address
    [ndbd]
    # Options for data node "B":
    HostName=198.51.100.40
    # Hostname or IP address
    [mysqld]
    # SQL node options:
    HostName=198.51.100.20
    # Hostname or IP address
    ```

    将此文件保存为文本文件 C:\mysql\bin\config.ini。

> **重要**
在 Windows 上 NDB Cluster 使用的程序选项或配置文件中指定目录路径时，不能使用单个反斜杠字符 (\)。 相反，您必须使用第二个反斜杠 (\\) 对每个反斜杠字符进行转义，或者将反斜杠替换为正斜杠字符 (/)。 例如，来自 NDB Cluster config.ini 文件的 [ndb_mgmd] 部分的以下行不起作用：
`DataDir=C:\mysql\bin\cluster-logs`
相反，您可以使用以下任一方法：
`DataDir=C:\\mysql\\bin\\cluster-logs # 转义反斜杠`
`DataDir=C:/mysql/bin/cluster-logs #正斜杠`
出于简洁和易读性的原因，我们建议您在 Windows 上的 NDB Cluster 程序选项和配置文件中使用的目录路径中使用正斜杠。
