# 在 Windows 上安装 NDB Cluster

23.3.2.1 [在 Windows 上从二进制版本安装 NDB Cluster](#在-windows-上从二进制版本安装-ndb-cluster)
23.3.2.2 [在 Windows 上从源代码编译和安装 NDB Cluster](#在-windows-上从源代码编译和安装-ndb-cluster)
23.3.2.3 [NDB Cluster 在 Windows 上的初始启动](#windows-上-ndb-集群的初始启动)
23.3.2.4 [将 NDB Cluster 进程安装为 Windows 服务](#将ndb群集进程安装为windows服务)

本节介绍 Windows 主机上 NDB Cluster 的安装过程。 可以从 <https://dev.mysql.com/downloads/cluster/> 获得适用于 Windows 的 NDB Cluster 8.0 二进制文件。 有关从 Oracle 提供的二进制版本在 Windows 上安装 NDB Cluster 的信息。

也可以使用 Microsoft Visual Studio 在 Windows 上从源代码编译和安装 NDB Cluster。

## 在 Windows 上从二进制版本安装 NDB Cluster

本节介绍了在 Windows 上使用 Oracle 提供的二进制“免安装”NDB Cluster 版本的 NDB Cluster 的基本安装，使用本节开头概述的相同 4 节点设置（请参阅[“NDB 集群安装”](/MySQL参考手册/NDB集群/NDB集群.md) )，如下表所示：

表 23.5 示例集群中节点的网络地址

| Node                   | IP Address    |
|------------------------|---------------|
| Management node ([mgmd](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html)) | 198.51.100.10 |
| SQL node ([mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html))      | 198.51.100.20 |
| Data node "A" ([ndbd](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html))   | 198.51.100.30 |
| Data node "B" (ndbd)   | 198.51.100.40 |

与其他平台一样，运行 SQL 节点的 NDB Cluster 主机必须在其上安装 MySQL 服务器二进制文件 ([mysqld.exe](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html))。您还应该在此主机上安装 MySQL 客户端 ([mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html))。对于管理节点和数据节点，不需要安装 MySQL Server 二进制文件；但是，每个管理节点都需要管理服务器守护程序 ([ndb_mgmd.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html))；每个数据节点都需要数据节点守护程序（[ndbd.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html) 或 [ndbmtd.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html)）。对于此示例，我们将 ndbd.exe 称为数据节点可执行文件，但您可以以完全相同的方式安装该程序的多线程版本 ndbmtd.exe。您还应该在管理服务器主机上安装管理客户端 ([ndb_mgm.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html))。

本节介绍为每种类型的 NDB Cluster 节点安装正确的 Windows 二进制文件所需的步骤。

> 笔记
与其他 Windows 程序一样，NDB Cluster 可执行文件以 .exe 文件扩展名命名。但是，从命令行调用这些程序时，不必包含 .exe 扩展名。因此，我们在本文档中经常将这些程序简称为 mysqld、mysql、ndb_mgmd 等。您应该明白，无论我们（例如）指的是 mysqld 还是 mysqld.exe，这两个名称都意味着相同的东西（MySQL 服务器程序）。

要使用 Oracles 的免安装二进制文件设置 NDB Cluster，安装过程的第一步是从 <https://dev.mysql.com/downloads/cluster/> 下载最新的 NDB Cluster Windows ZIP 二进制存档。该归档文件的文件名为 mysql-cluster-gpl-ver-winarch.zip，其中 ver 是 NDB 存储引擎版本（例如 8.0.28），arch 是架构（32 用于 32 位二进制文​​件，64对于 64 位二进制文​​件）。例如，用于 64 位 Windows 系统的 NDB Cluster 8.0.28 存档名为 mysql-cluster-gpl-8.0.28-win64.zip。

您可以在 32 位和 64 位版本的 Windows 上运行 32 位 NDB Cluster 二进制文件；但是，64 位 NDB Cluster 二进制文件只能在 64 位版本的 Windows 上使用。如果您在具有 64 位 CPU 的计算机上使用 32 位版本的 Windows，则必须使用 32 位 NDB Cluster 二进制文件。

为了尽量减少需要从 Internet 下载或在机器之间复制的文件数量，我们从您打算运行 SQL 节点的计算机开始。

**SQL node**。我们假设您已在 IP 地址为 198.51.100.20 的计算机上的目录 C:\Documents 和 Settings\username\My Documents\Downloads 中放置了存档的副本，其中用户名是当前用户的名称。 （您可以在命令行上使用 `ECHO %USERNAME%` 获取此名称。）要将 NDB Cluster 可执行文件作为 Windows 服务安装和运行，此用户应该是管理员组的成员。

从存档中提取所有文件。与 Windows 资源管理器集成的提取向导足以完成此任务。 （如果您使用不同的存档程序，请确保它从存档中提取所有文件和目录，并保留存档的目录结构。）当您被要求输入目标目录时，输入 C:\，这会导致提取将存档解压缩到目录 C:\mysql-cluster-gpl-ver-winarch 的向导。将此目录重命名为 C:\mysql。

可以将 NDB Cluster 二进制文件安装到 C:\mysql\bin 以外的目录；但是，如果这样做，则必须相应地修改此过程中显示的路径。特别是，如果 MySQL 服务器（SQL 节点）二进制文件安装到 C:\mysql 或 C:\Program Files\MySQL\MySQL Server 8.0 以外的位置，或者 SQL 节点的数据目录位于 C 以外的位置:\mysql\data 或 C:\Program Files\MySQL\MySQL Server 8.0\data，额外的配置选项必须在命令行中使用或在启动 SQL 节点时添加到 my.ini 或 my.cnf 文件中。有关将 MySQL 服务器配置为在非标准位置运行的更多信息，请参阅[第 2.3.4 节，“使用 noinstall ZIP 存档在 Microsoft Windows 上安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/windows-install-archive.html)。

要使具有 NDB Cluster 支持的 MySQL 服务器作为 NDB Cluster 的一部分运行，它必须使用选项 `--ndbcluster` 和 `--ndb-connectstring` 启动。 虽然您可以在命令行上指定这些选项，但将它们放在选项文件中通常更方便。 为此，请在记事本或其他文本编辑器中创建一个新文本文件。 在此文件中输入以下配置信息：

```ini
[mysqld]
# Options for mysqld process:
ndbcluster                       # run NDB storage engine
ndb-connectstring=198.51.100.10  # location of management server
```

如果需要，您可以添加此 MySQL 服务器使用的其他选项（请参阅[第 2.3.4.2 节，“创建选项文件”](https://dev.mysql.com/doc/refman/8.0/en/windows-create-option-file.html)），但该文件必须至少包含显示的选项。将此文件另存为 C:\mysql\my.ini。这样就完成了 SQL 节点的安装和设置。

**Data nodes**。 Windows 主机上的 NDB Cluster 数据节点只需要一个可执行文件，ndbd.exe 或 ndbmtd.exe 之一。对于此示例，我们假设您使用的是 ndbd.exe，但使用 ndbmtd.exe 时也适用相同的说明。在您希望运行数据节点的每台计算机（IP 地址为 198.51.100.30 和 198.51.100.40 的计算机）上，创建目录 C:\mysql、C:\mysql\bin 和 C:\mysql\cluster-数据；然后，在您下载并解压缩免安装存档的计算机上，在 C:\mysql\bin 目录中找到 ndbd.exe。将此文件复制到两个数据节点主机中的每一个上的 C:\mysql\bin 目录。

要作为 NDB Cluster 的一部分，必须为每个数据节点提供管理服务器的地址或主机名。您可以在启动每个数据节点进程时使用 [`--ndb-connectstring`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-config.html#option_ndb_config_ndb-connectstring) 或 `-c` 选项在命令行上提供此信息。但是，通常最好将此信息放在选项文件中。为此，请在记事本或其他文本编辑器中创建一个新文本文件，然后输入以下文本：

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
  
    ```ini
    [mysql_cluster]
    # Options for management node process
    config-file=C:/mysql/bin/config.ini
    ```

    将此文件保存为文本文件 C:\mysql\bin\my.ini。

2. 一个全局配置文件，管理节点可以从中获取管理 NDB Cluster 整体的配置信息。此文件至少必须包含 NDB Cluster 中每个节点的部分，以及管理节点和所有数据节点的 IP 地址或主机名（HostName 配置参数）。还建议包括以下附加信息：

    - 任何 SQL 节点的 IP 地址或主机名
    - 分配给每个数据节点的数据内存和索引内存（[DataMemory](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-datamemory) 和 [IndexMemory](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-indexmemory) 配置参数）
    - 片段副本的数量，使用 [NoOfReplicas](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-noofreplicas) 配置参数（请参阅[“NDB 集群节点、节点组、片段副本和分区”](../NDB集群概述/NDB节点和分区.md)）
    - 每个数据节点存储其数据和日志文件的目录，以及管理节点保存其日志文件的目录（在这两种情况下，[DataDir](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbd-datadir) 配置参数）

   使用记事本等文本编辑器新建一个文本文件，并输入以下信息：
  
    ```ini
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

## 在 Windows 上从源代码编译和安装 NDB Cluster

Oracle 为 Windows 提供了预编译的 NDB Cluster 二进制文件，对于大多数用户来说应该足够了。但是，如果您愿意，也可以从源代码编译 NDB Cluster for Windows。执行此操作的过程几乎与用于为 Windows 编译标准 MySQL 服务器二进制文件的过程相同，并且使用相同的工具。但是，有两个主要区别：

- 构建 MySQL NDB Cluster 8.0 需要使用 MySQL Server 8.0 源。这些可从位于 <https://dev.mysql.com/downloads/> 的 MySQL 下载页面获得。归档源文件的名称应类似于 mysql-8.0.28.tar.gz。您还可以在 <https://github.com/mysql/mysql-server> 从 GitHub 获取源代码。
- 除了您希望与 CMake 一起使用的任何其他构建选项之外，您还必须使用 [WITH_NDB](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html#option_cmake_with_ndb) 选项配置构建。 [WITH_NDBCLUSTER](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html#option_cmake_with_ndbcluster) 也支持向后兼容，但自 NDB 8.0.31 起已弃用。

  > 重要
  [WITH_NDB_JAVA](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html#option_cmake_with_ndb_java) 选项默认启用。 这意味着，默认情况下，如果 CMake 在您的系统上找不到 Java 的位置，则配置过程会失败； 如果您不希望启用 Java 和 ClusterJ 支持，则必须通过使用 -DWITH_NDB_JAVA=OFF 配置构建来明确指出这一点。 （错误 #12379735）如果需要，使用 [WITH_CLASSPATH](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html#option_cmake_with_classpath) 提供 Java 类路径。

有关特定于构建 NDB Cluster 的 CMake 选项的更多信息，请参阅[用于编译 NDB Cluster 的 CMake 选项](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html#cmake-mysql-cluster-options)。

构建过程完成后，您可以创建一个包含已编译二进制文件的 Zip 存档； [第 2.9.4 节，“使用标准源分发安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/installing-source-distribution.html)提供了在 Windows 系统上执行此任务所需的命令。 NDB Cluster 二进制文件可以在生成的存档的 bin 目录中找到，这相当于 no-install 存档，并且可以以相同的方式安装和配置。

## Windows 上 NDB 集群的初始启动

一旦 NDB Cluster 可执行文件和所需的配置文件就位，执行集群的初始启动只需为集群中的所有节点启动 NDB Cluster 可执行文件即可。 每个集群节点进程必须单独启动，并且在它所在的主机上启动。 管理节点应该首先启动，然后是数据节点，最后是任何 SQL 节点。

1. 在管理节点主机上，从命令行发出以下命令以启动管理节点进程。 输出应类似于此处显示的内容：

   ```txt
   C:\mysql\bin> ndb_mgmd
   2010-06-23 07:53:34 [MgmtSrvr] INFO -- NDB Cluster Management Server. mysql-8.0.29-ndb-8.0.30
   2010-06-23 07:53:34 [MgmtSrvr] INFO -- Reading cluster configuration from 'config.ini'
   ```

   管理节点进程继续将日志记录输出打印到控制台。这是正常的，因为管理节点没有作为 Windows 服务运行。 （如果您在 Linux 等类 Unix 平台上使用过 NDB Cluster，您可能会注意到管理节点在这方面在 Windows 上的默认行为实际上与其在 Unix 系统上的行为相反，它在默认情况下作为 Unix 运行daemon 进程。在 Windows 上运行的 NDB Cluster 数据节点进程也是如此。）因此，不要关闭运行 ndb_mgmd.exe 的窗口；这样做会杀死管理节点进程。参阅[将NDB集群进程安装为Windows服务](#将ndb群集进程安装为windows服务)。

   必需的 `-f` 选项告诉管理节点在哪里可以找到全局配置文件 (config.ini)。此选项的长格式是 [--config-file](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_config-file)。

   > 重要
   NDB Cluster 管理节点缓存它从 config.ini 读取的配置数据；一旦它创建了一个配置缓存，它会在随后的启动中忽略 config.ini 文件，除非被迫这样做。这意味着，如果管理节点由于该文件中的错误而无法启动，则必须在纠正其中的任何错误后让管理节点重新读取 config.ini。您可以通过在命令行上使用 [--reload](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_reload) 或 [--initial](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_initial) 选项启动 ndb_mgmd.exe 来执行此操作。这些选项中的任何一个都可以刷新配置缓存。
   没有必要也不建议在管理节点的 my.ini 文件中使用这些选项中的任何一个。

2. 在每个数据节点主机上，运行此处显示的命令以启动数据节点进程：

   ```txt
   C:\mysql\bin> ndbd
   2010-06-23 07:53:46 [ndbd] INFO -- Configuration fetched from 'localhost:1186', generation: 1
   ```

   在每种情况下，数据节点进程的第一行输出应该类似于前面示例中所示，后面是额外的日志输出行。与管理节点进程一样，这是正常的，因为数据节点不是作为Windows服务运行的。因此，不要关闭正在运行数据节点进程的控制台窗口；这样做会杀死ndbd.exe。

3. 尚未启动SQL节点；在数据节点完成启动之前，它无法连接到群集，这可能需要一些时间。相反，在管理节点主机上的新控制台窗口中，启动NDB群集管理客户端 NDB_mgm.exe，它应该位于管理节点主机上的C:\mysql\bin中。（不要尝试通过键入CTRL+C来重复使用控制台窗口，其中ndb_mgmd.exe正在运行，因为这会杀死管理节点。）结果输出应如下所示：

   ```txt
   C:\mysql\bin> ndb_mgm
   -- NDB Cluster -- Management Client --
   ndb_mgm>
   ```

   当提示 ndb_mgm> 出现时，这表明管理客户端已准备好接收 NDB Cluster 管理命令。 您可以通过在管理客户端提示符下输入 ALL STATUS 来观察数据节点启动时的状态。 此命令会生成数据节点启动序列的运行报告，应如下所示：

   ```txt
   ndb_mgm> ALL STATUS
   Connected to Management Server at: localhost:1186
   Node 2: starting (Last completed phase 3) (mysql-8.0.29-ndb-8.0.30)
   Node 3: starting (Last completed phase 3) (mysql-8.0.29-ndb-8.0.30)
   Node 2: starting (Last completed phase 4) (mysql-8.0.29-ndb-8.0.30)
   Node 3: starting (Last completed phase 4) (mysql-8.0.29-ndb-8.0.30)
   Node 2: Started (version 8.0.30)
   Node 3: Started (version 8.0.30)

   ndb_mgm>
   ```

   > 笔记
   管理客户端发出的命令不区分大小写； 我们使用大写作为这些命令的规范形式，但在将它们输入到 ndb_mgm 客户端时，您不需要遵守此约定。 有关更多信息，请参阅[第 23.6.1 节，“NDB Cluster Management Client 中的命令”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mgm-client-commands.html)。

   ALL STATUS 产生的输出可能与此处显示的不同，具体取决于数据节点能够启动的速度、您正在使用的 NDB Cluster 软件的发布版本号以及其他因素。 重要的是，当您看到两个数据节点都已启动时，您就可以启动 SQL 节点了。

   您可以让 ndb_mgm.exe 继续运行； 它对 NDB Cluster 的性能没有负面影响，我们在下一步中使用它来验证 SQL 节点是否已连接到集群后启动它。

4. 在指定为 SQL 节点主机的计算机上，打开控制台窗口并导航到解压 NDB Cluster 二进制文件的目录（如果您按照我们的示例进行操作，则为 C:\mysql\bin）。

   通过从命令行调用 mysqld.exe 来启动 SQL 节点，如下所示：
   `C:\mysql\bin> mysqld --console`

   [--console](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_console) 选项会导致将日志信息写入控制台，这在出现问题时会很有帮助。 （一旦您对 SQL 节点以令人满意的方式运行感到满意，您可以停止它并在不使用 --console 选项的情况下重新启动它，以便正常执行日志记录。）

   在管理节点主机上运行管理客户端 (ndb_mgm.exe) 的控制台窗口中，输入 [SHOW](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mgm-client-commands.html#ndbclient-show) 命令，该命令应产生类似于此处所示的输出：

   ```txt
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

   您还可以使用 SHOW ENGINE NDB STATUS 语句验证 SQL 节点是否已连接到 mysql 客户端 (mysql.exe) 中的 NDB Cluster。

您现在应该准备好使用 NDB Cluster 的 NDBCLUSTER 存储引擎处理数据库对象和数据。 有关更多信息和示例，请参阅[第 23.3.5 节，“带有表和数据的 NDB 集群示例”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-example-data.html)。

您还可以将 ndb_mgmd.exe、ndbd.exe 和 ndbmtd.exe 安装为 Windows 服务。

## 将NDB群集进程安装为Windows服务

一旦您确信NDB群集正在按所需运行，就可以将管理节点和数据节点安装为Windows服务，以便在Windows启动或停止时自动启动和停止这些进程。这还可以使用适当的SC START和SC STOP命令或使用Windows图形服务实用程序从命令行控制这些进程。也可以使用净启动和净停止命令。

将程序安装为Windows服务通常必须使用具有系统管理员权限的帐户。

要将管理节点安装为Windows上的服务，请调用ndb_mgmd。使用 [--install](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_install) 选项从托管管理节点的计算机上的命令行执行，如下所示：

```txt
C:\> C:\mysql\bin\ndb_mgmd.exe --install
Installing service 'NDB Cluster Management Server'
  as '"C:\mysql\bin\ndbd.exe" "--service=ndb_mgmd"'
Service successfully installed.
```

> 重要
将NDB群集程序安装为Windows服务时，应始终指定完整路径；否则，服务安装可能会失败，错误是系统找不到指定的文件。

必须首先使用 --install 选项，然后是可能为 ndb_mgmd.exe 指定的任何其他选项。 但是，最好在选项文件中指定此类选项。 如果您的选项文件不在 ndb_mgmd.exe [--help](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_help) 的输出中所示的默认位置之一，您可以使用 [--config-file](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_config-file) 选项指定位置。

现在您应该能够像这样启动和停止管理服务器：

`C:\> SC START ndb_mgmd`

`C:\> SC STOP ndb_mgmd`

> 笔记
如果使用 NET 命令，您还可以使用描述性名称将管理服务器作为 Windows 服务启动或停止，如下所示：

  ```txt
  C:\> NET START 'NDB Cluster Management Server'
  The NDB Cluster Management Server service is starting.
  The NDB Cluster Management Server service was started successfully.

  C:\> NET STOP  'NDB Cluster Management Server'
  The NDB Cluster Management Server service is stopping..
  The NDB Cluster Management Server service was stopped successfully.
  ```

通常更简单的是指定一个简短的服务名称或允许在安装服务时使用默认服务名称，然后在启动或停止服务时引用该名称。 要指定 ndb_mgmd 以外的服务名称，请将其附加到 --install 选项，如下例所示：

```txt
C:\> C:\mysql\bin\ndb_mgmd.exe --install=mgmd1
Installing service 'NDB Cluster Management Server'
  as '"C:\mysql\bin\ndb_mgmd.exe" "--service=mgmd1"'
Service successfully installed.
```

现在您应该能够使用您指定的名称启动或停止服务，如下所示：

`C:\> SC START mgmd1`

`C:\> SC STOP mgmd1`
要删除管理节点服务，请使用 SC DELETE service_name：

`C:\> SC DELETE mgmd1`
或者，使用 [--remove](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html#option_ndb_mgmd_remove) 选项调用 ndb_mgmd.exe，如下所示：

```txt
C:\> C:\mysql\bin\ndb_mgmd.exe --remove
Removing service 'NDB Cluster Management Server'
Service successfully removed.
```

如果您使用非默认服务名称安装服务，请将服务名称作为 ndb_mgmd.exe --remove 选项的值传递，如下所示：

```txt
C:\> C:\mysql\bin\ndb_mgmd.exe --remove=mgmd1
Removing service 'mgmd1'
Service successfully removed.
```

可以使用 ndbd.exe（或 ndbmtd.exe）的 --install 选项以类似的方式将 NDB Cluster 数据节点进程安装为 Windows 服务，如下所示：

```txt
C:\> C:\mysql\bin\ndbd.exe --install
Installing service 'NDB Cluster Data Node Daemon' as '"C:\mysql\bin\ndbd.exe" "--service=ndbd"'
Service successfully installed.
```

现在您可以启动或停止数据节点，如下例所示：
`C:\> SC START ndbd`
`C:\> SC STOP ndbd`
要删除数据节点服务，请使用 SC DELETE service_name：
`C:\> SC DELETE ndbd`
或者，使用 --remove 选项调用 ndbd.exe，如下所示：

```txt
C:\> C:\mysql\bin\ndbd.exe --remove
Removing service 'NDB Cluster Data Node Daemon'
Service successfully removed.
```

与ndb_mgmd.exe（和mysqld.exe）一样，在将ndbd.exe安装为Windows服务时，也可以为服务指定一个名称作为--install的值，然后在启动或停止服务时使用，像这样：

```txt
C:\> C:\mysql\bin\ndbd.exe --install=dnode1
Installing service 'dnode1' as '"C:\mysql\bin\ndbd.exe" "--service=dnode1"'
Service successfully installed.

C:\> SC START dnode1

C:\> SC STOP dnode1
```

如果您在安装数据节点服务时指定了服务名称，则在删除它时也可以使用此名称，如下所示：
`C:\> SC DELETE dnode1`
或者，您可以将服务名称作为 ndbd.exe --remove 选项的值传递，如下所示：

```txt
C:\> C:\mysql\bin\ndbd.exe --remove=dnode1
Removing service 'dnode1'
Service successfully removed.
```

使用 mysqld --install、SC START、SC STOP 和 SC DELETE（或 mysqld --remove ）。 NET 命令也可用于启动或停止服务。有关其他信息，请参阅[第 2.3.4.8 节，“将 MySQL 作为 Windows 服务启动”](https://dev.mysql.com/doc/refman/8.0/en/windows-start-service.html)。
