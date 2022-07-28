# 在 Linux 上安装 NDB 集群

23.3.1.1 在 Linux 上安装 NDB Cluster 二进制版本
23.3.1.2 从 RPM 安装 NDB Cluster
23.3.1.3 使用 .deb 文件安装 NDB Cluster
23.3.1.4 在 Linux 上从源代码构建 NDB 集群

本节介绍 Linux 和其他类 Unix 操作系统上 NDB Cluster 的安装方法。虽然接下来的几节提到了 Linux 操作系统，但那里给出的说明和过程应该很容易适应其他受支持的类 Unix 平台。

每个 NDB Cluster 主机必须安装正确的可执行程序。运行 SQL 节点的主机必须在其上安装 MySQL 服务器二进制文件 (mysqld)。管理节点需要管理服务器守护进程（ndb_mgmd）；数据节点需要数据节点守护程序（ndbd 或 ndbmtd）。无需在管理节点主机和数据节点主机上安装 MySQL Server 二进制文件。建议您还在管理服务器主机上安装管理客户端 (ndb_mgm)。

在 Linux 上安装 NDB Cluster 可以使用 Oracle 的预编译二进制文件（下载为 .tar.gz 存档）、RPM 包（也可从 Oracle 获得）或源代码来完成。所有这三种安装方法都将在下面的部分中进行描述。

无论使用哪种方法，在安装 NDB Cluster 二进制文件后，仍然需要为所有集群节点创建配置文件，然后才能启动集群。

## 在 Linux 上安装 NDB Cluster 二进制版本

本节介绍从 Oracle 提供的预编译二进制文件为每种类型的集群节点安装正确可执行文件所需的步骤。

要使用预编译的二进制文件设置集群，每个集群主机安装过程的第一步是从 [NDB Cluster 下载](https://dev.mysql.com/downloads/cluster/)页面下载二进制存档。 （对于最新的 64 位 NDB 8.0 版本，这是 mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz。）我们假设您已将此文件放在每台机器的 /var /tmp 目录。

如果您需要自定义二进制文件，请参阅[第 2.9.5 节，“使用开发源树安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html)。

> 笔记
完成安装后，不要启动任何二进制文件。我们向您展示如何在节点配置之后执行此操作。

**SQL nodes**。在指定托管 SQL 节点的每台机器上，以系统 root 用户身份执行以下步骤：

1、检查您的 /etc/passwd 和 /etc/group 文件（或使用操作系统提供的任何工具来管理用户和组）以查看系统上是否已经存在 mysql 组和 mysql 用户。一些操作系统发行版在操作系统安装过程中创建这些。如果它们不存在，则创建一个新的 mysql 用户组，然后将 mysql 用户添加到该组：
`$> groupadd mysql`
`$> useradd -g mysql -s /bin/false mysql`
   useradd 和 groupadd 的语法在不同版本的 Unix 上可能略有不同，或者它们可能有不同的名称，例如 adduser 和 addgroup。
2、将位置更改为包含下载文件的目录，解压缩存档，并创建一个名为 mysql 的符号链接到 mysql 目录。实际文件和目录名称因 NDB Cluster 版本号而异。

```bash
$> cd /var/tmp
$> tar -C /usr/local -xzvf mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz
$> ln -s /usr/local/mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64 /usr/local/mysql
```

3、将位置更改为 mysql 目录并使用 mysqld --initialize 设置系统数据库，如下所示：
`$> cd mysql`
`$> mysqld --initialize`
这将为 MySQL root 帐户生成一个随机密码。如果您不希望生成随机密码，您可以用 --initialize-insecure 选项替换 --initialize。在任何一种情况下，您都应该在执行此步骤之前查看[第 2.10.1 节“初始化数据目录”](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html)以获取更多信息。另请参阅[第 4.4.2 节，“mysql_secure_installation - 提高 MySQL 安装安全性”](https://dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html)。

4、为 MySQL 服务器和数据目录设置必要的权限：

```bash
$> chown -R root .
$> chown -R mysql data
$> chgrp -R mysql .
```

5、将 MySQL 启动脚本复制到相应目录，使其可执行，并将其设置为在操作系统启动时启动：

```bash
$> cp support-files/mysql.server /etc/rc.d/init.d/
$> chmod +x /etc/rc.d/init.d/mysql.server
$> chkconfig --add mysql.server
```

（启动脚本目录可能因您的操作系统和版本而异——例如，在某些 Linux 发行版中，它是 /etc/init.d。）

这里我们使用 Red Hat 的 chkconfig 来创建启动脚本的链接；在您的平台上使用适合此目的的任何方法，例如 Debian 上的 update-rc.d。

请记住，必须在 SQL 节点所在的每台机器上重复上述步骤。

**Data nodes**。数据节点的安装不需要 mysqld 二进制文件。仅需要 NDB Cluster 数据节点可执行文件 ndbd（单线程）或 ndbmtd（多线程）。这些二进制文件也可以在 .tar.gz 存档中找到。同样，我们假设您已将此存档放置在 /var/tmp 中。

以系统 root 身份（即，在使用 sudo、su root 或您的系统的等效项来临时承担系统管理员帐户的权限后），执行以下步骤以在数据节点主机上安装数据节点二进制文件：

1、将位置更改为 /var/tmp 目录，并将 ndbd 和 ndbmtd 二进制文件从存档中提取到合适的目录，例如 /usr/local/bin：

```bash
$> cd /var/tmp
$> tar -zxvf mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz
$> cd mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64
$> cp bin/ndbd /usr/local/bin/ndbd
$> cp bin/ndbmtd /usr/local/bin/ndbmtd
```

（一旦 ndb_mgm 和 ndb_mgmd 被复制到可执行文件目录，您就可以安全地从 /var/tmp 中删除通过解压下载的存档创建的目录及其包含的文件。）

2、将位置更改为您将文件复制到的目录，然后使它们都可执行：
`$> cd /usr/local/bin`
`$> chmod +x ndb*`
应在每个数据节点主机上重复上述步骤。

尽管运行 NDB Cluster 数据节点只需要一个数据节点可执行文件，但我们在前面的说明中向您展示了如何安装 ndbd 和 ndbmtd。我们建议您在安装或升级 NDB Cluster 时执行此操作，即使您打算只使用其中一个，因为这样可以在您以后决定从一个更改为另一个时节省时间和麻烦。

> 笔记
托管数据节点的每台机器上的数据目录是 /usr/local/mysql/data。在配置管理节点时，这条信息是必不可少的。

**管理节点**。管理节点的安装不需要 mysqld 二进制文件。只需要 NDB Cluster 管理服务器（ndb_mgmd）；您很可能还想安装管理客户端 (ndb_mgm)。这两个二进制文件也可以在 .tar.gz 存档中找到。同样，我们假设您已将此存档放置在 /var/tmp 中。

以系统 root 身份执行以下步骤，在管理节点主机上安装 ndb_mgmd 和 ndb_mgm：

1、将位置更改为 /var/tmp 目录，并将 ndb_mgm 和 ndb_mgmd 从存档中提取到合适的目录中，例如 /usr/local/bin：

```bash
$> cd /var/tmp
$> tar -zxvf mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz
$> cd mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64
$> cp bin/ndb_mgm* /usr/local/bin
```

（一旦 ndb_mgm 和 ndb_mgmd 被复制到可执行文件目录，您就可以安全地从 /var/tmp 中删除通过解压下载的存档创建的目录及其包含的文件。）

2、将位置更改为您将文件复制到的目录，然后使它们都可执行：
`$> cd /usr/local/bin`
`$> chmod +x ndb_mgm*`

在[第 23.3.3 节，“NDB 集群的初始配置”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html)中，我们为示例 NDB 集群中的所有节点创建配置文件。

## 从 RPM 安装 NDB Cluster

本节介绍使用 Oracle 提供的 RPM 包为每种类型的 NDB Cluster 8.0 节点安装正确可执行文件所需的步骤。

作为本节中描述的方法的替代方法，Oracle 为 NDB Cluster 提供了与许多常见 Linux 发行版兼容的 MySQL 存储库。 此处列出的两个存储库可用于基于 RPM 的发行版：

- 对于使用 yum 或 dnf 的发行版，您可以使用 MySQL Yum Repository for NDB Cluster。 有关说明和其他信息，请参阅[使用 Yum 存储库安装 MySQL NDB Cluster](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/#repo-qg-yum-fresh-cluster-install)。

- 对于 SLES，您可以使用 MySQL SLES Repository for NDB Cluster。 有关说明和其他信息，请参阅[使用 SLES 存储库安装 MySQL NDB Cluster](https://dev.mysql.com/doc/mysql-sles-repo-quick-guide/en/#repo-qg-sles-fresh-cluster-install)。

RPM 可用于 32 位和 64 位 Linux 平台。 这些 RPM 的文件名使用以下模式：

```txt
mysql-cluster-community-data-node-8.0.28-1.el7.x86_64.rpm

mysql-cluster-license-component-ver-rev.distro.arch.rpm

    license:= {commercial | community}

    component: {management-server | data-node | server | client | other—see text}

    ver: major.minor.release

    rev: major[.minor]

    distro: {el6 | el7 | sles12}

    arch: {i686 | x86_64}
```

license 指示 RPM 是 NDB Cluster 的商业版还是社区版的一部分。 在本节的其余部分，我们假设您正在安装社区版本的示例。

可以在下表中找到组件的可能值以及描述：

表 23.6 NDB Cluster RPM 分发的组件

| Component                   | Description                                                                                                                            |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| auto-installer (DEPRECATED) | NDB Cluster Auto Installer program; see Section 23.3.8, “The NDB Cluster Auto-Installer (No longer supported)”, for usage              |
| client                      | MySQL and NDB client programs; includes mysql client, ndb_mgm client, and other client tools                                           |
| common                      | Character set and error message information needed by the MySQL server                                                                 |
| data-node                   | ndbd and ndbmtd data node binaries                                                                                                     |
| devel                       | Headers and library files needed for MySQL client development                                                                          |
| embedded                    | Embedded MySQL server                                                                                                                  |
| embedded-compat             | Backwards-compatible embedded MySQL server                                                                                             |
| embedded-devel              | Header and library files for developing applications for embedded MySQL                                                                |
| java                        | JAR files needed for support of ClusterJ applications                                                                                  |
| libs                        | MySQL client libraries                                                                                                                 |
| libs-compat                 | Backwards-compatible MySQL client libraries                                                                                            |
| management-server           | The NDB Cluster management server (ndb_mgmd)                                                                                           |
| memcached                   | Files needed to support ndbmemcache                                                                                                    |
| minimal-debuginfo           | Debug information for package server-minimal; useful when developing applications that use this package or when debugging this package |
| ndbclient                   | NDB client library for running NDB API and MGM API applications (libndbclient)                                                         |
| ndbclient-devel             | Header and other files needed for developing NDB API and MGM API applications                                                          |
| nodejs                      | Files needed to set up Node.JS support for NDB Cluster                                                                                 |
| server                      | The MySQL server (mysqld) with NDB storage engine support included, and associated MySQL server programs                               |
| server-minimal              | Minimal installation of the MySQL server for NDB and related tools                                                                     |
| test                        | mysqltest, other MySQL test programs, and support files                                                                                |

还可以使用给定平台和架构的所有 NDB Cluster RPM 的单个包（.tar 文件）。此文件的名称遵循此处显示的模式：
`mysql-cluster-license-ver-rev.distro.arch.rpm-bundle.tar`
您可以使用 tar 或您首选的用于提取档案的工具从此文件中提取单个 RPM 文件。

以下列表中给出了安装三种主要类型的 NDB Cluster 节点所需的组件：

- Management node: management-server

- Data node: data-node

- SQL node: server and common

此外，应安装客户端 RPM 以在至少一个管理节点上提供 ndb_mgm 管理客户端。您可能还希望将它安装在 SQL 节点上，以便在这些节点上使用 mysql 和其他 MySQL 客户端程序。我们将在本节后面讨论按类型安装节点。

*ver* 表示 8.0.x 格式的三部分 NDB 存储引擎版本号，在示例中显示为 8.0.28。 rev 以major.minor 格式提供RPM 修订号。在本节所示的示例中，我们使用 1.1 作为该值。

*distro*（Linux 发行版）是 rhel5（Oracle Linux 5、Red Hat Enterprise Linux 4 和 5）、el6（Oracle Linux 6、Red Hat Enterprise Linux 6）、el7（Oracle Linux 7、Red Hat Enterprise Linux 7）、或 sles12 (SUSE Enterprise Linux 12)。对于本节中的示例，我们假设主机运行 Oracle Linux 7、Red Hat Enterprise Linux 7 或同等版本 (el7)。

对于 32 位 RPM，*arch* 是 i686，对于 64 位版本是 x86_64。在此处显示的示例中，我们假设一个 64 位平台。

RPM 文件名中的 NDB Cluster 版本号（此处显示为 8.0.28）可能会根据您实际使用的版本而有所不同。要安装的所有集群 RPM 具有相同的版本号，这一点非常重要。架构也应该适合安装 RPM 的机器；特别是，您应该记住，64 位 RPM (x86_64) 不能与 32 位操作系统一起使用（后者使用 i686）。

**Data nodes**。 在要托管 NDB Cluster 数据节点的计算机上，只需安装数据节点 RPM。 为此，请将此 RPM 复制到数据节点主机，并以系统 root 用户身份运行以下命令，根据需要替换为 RPM 显示的名称以匹配从 MySQL 网站下载的 RPM：
`$> rpm -Uhv mysql-cluster-community-data-node-8.0.28-1.el7.x86_64.rpm`
这会在 /usr/sbin 中安装 ndbd 和 ndbmtd 数据节点二进制文件。 其中任何一个都可用于在此主机上运行数据节点进程。

**SQL nodes**。 将服务器和公用 RPM 复制到每台机器上，以用于托管 NDB Cluster SQL 节点（服务器需要公用）。 通过以系统 root 用户身份执行以下命令来安装服务器 RPM，根据需要替换为 RPM 显示的名称以匹配从 MySQL 网站下载的 RPM 的名称：
`$> rpm -Uhv mysql-cluster-community-server-8.0.28-1.el7.x86_64.rpm`

这会在 /usr/sbin 目录中安装带有 NDB 存储引擎支持的 MySQL 服务器二进制文件 (mysqld)。 它还安装所有需要的 MySQL 服务器支持文件和有用的 MySQL 服务器程序，包括 mysql.server 和 mysqld_safe 启动脚本（分别在 /usr/share/mysql 和 /usr/bin 中）。 RPM 安装程序应该自动处理一般配置问题（例如创建 mysql 用户和组，如果需要）。

> 重要
您必须使用为 NDB Cluster 发布的这些 RPM 的版本； 为标准 MySQL 服务器发布的那些不提供对 NDB 存储引擎的支持。

要管理 SQL 节点（MySQL 服务器），您还应该安装客户端 RPM，如下所示：
`$> rpm -Uhv mysql-cluster-community-client-8.0.28-1.el7.x86_64.rpm`
这会将 mysql 客户端和其他 MySQL 客户端程序（例如 mysqladmin 和 mysqldump）安装到 /usr/bin。

**Management nodes**。要安装 NDB Cluster 管理服务器，只需使用管理服务器 RPM。将此 RPM 复制到用于托管管理节点的计算机，然后以系统 root 用户身份运行以下命令来安装它（根据需要替换为 RPM 显示的名称以匹配从 MySQL 下载的管理服务器 RPM 的名称网站）：
`$> rpm -Uhv mysql-cluster-community-management-server-8.0.28-1.el7.x86_64.rpm`
此 RPM 将管理服务器二进制文件 ndb_mgmd 安装在 /usr/sbin 目录中。虽然这是运行管理节点实际需要的唯一程序，但最好同时使用 ndb_mgm NDB Cluster 管理客户端。您可以通过如前所述安装客户端 RPM 来获取此程序以及其他 NDB 客户端程序，例如 ndb_desc 和 ndb_config。

有关使用 Oracle 提供的 RPM 安装 MySQL 的一般信息，请参阅[第 2.5.4 节，“使用 Oracle 的 RPM 包在 Linux 上安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/linux-installation-rpm.html)。

从 RPM 安装后，还需要配置集群；有关相关信息，请参阅[第 23.3.3 节，“NDB 集群的初始配置”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html)。

要安装的所有集群 RPM 具有相同的版本号，这一点非常重要。架构名称也应该适合安装 RPM 的机器；特别是，您应该记住，64 位 RPM 不能用于 32 位操作系统。

**Data nodes**。 在要托管集群数据节点的计算机上，只需安装服务器 RPM。 为此，请将此 RPM 复制到数据节点主机，并以系统 root 用户身份运行以下命令，根据需要替换为 RPM 显示的名称以匹配从 MySQL 网站下载的 RPM：
`$> rpm -Uhv MySQL-Cluster-server-gpl-8.0.28-1.sles11.i386.rpm`
尽管这会安装所有 NDB Cluster 二进制文件，但实际上只需要程序 ndbd 或 ndbmtd（都在 /usr/sbin 中）来运行 NDB Cluster 数据节点。

**SQL nodes**。 在要用于托管集群 SQL 节点的每台机器上，通过以系统 root 用户身份执行以下命令来安装服务器 RPM，根据需要替换为 RPM 显示的名称以匹配从 MySQL 网站下载的 RPM 的名称：
`$> rpm -Uhv MySQL-Cluster-server-gpl-8.0.28-1.sles11.i386.rpm`
这会在 /usr/sbin 目录中安装带有 NDB 存储引擎支持的 MySQL 服务器二进制文件 (mysqld)，以及所有需要的 MySQL 服务器支持文件。 它还安装 mysql.server 和 mysqld_safe 启动脚本（分别在 /usr/share/mysql 和 /usr/bin 中）。 RPM 安装程序应该自动处理一般配置问题（例如创建 mysql 用户和组，如果需要）。

要管理 SQL 节点（MySQL 服务器），您还应该安装客户端 RPM，如下所示：
`$> rpm -Uhv MySQL-Cluster-client-gpl-8.0.28-1.sles11.i386.rpm`
这将安装 mysql 客户端程序。

**Management nodes**.*。 要安装 NDB Cluster 管理服务器，只需使用服务器 RPM。 将此 RPM 复制到用于托管管理节点的计算机，然后以系统 root 用户身份运行以下命令来安装它（根据需要替换为 RPM 显示的名称以匹配从 MySQL 网站下载的服务器 RPM 的名称） ：
`$> rpm -Uhv MySQL-Cluster-server-gpl-8.0.28-1.sles11.i386.rpm`
尽管此 RPM 安装了许多其他文件，但运行管理节点实际上只需要管理服务器二进制 ndb_mgmd（在 /usr/sbin 目录中）。 服务器 RPM 还安装 NDB 管理客户端 ndb_mgm。

有关所需的安装后配置的信息，请参阅[第 23.3.3 节，“NDB 集群的初始配置”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html)。
