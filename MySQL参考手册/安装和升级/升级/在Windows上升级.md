# 在 Windows 上升级 MySQL

<https://dev.mysql.com/doc/refman/8.0/en/windows-upgrading.html#windows-upgrading-zip-distribution>

您选择的方法取决于现有安装的执行方式。 在继续之前，请查看[第 2.11 节，“升级 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/upgrading.html)以获取有关升级非特定于 Windows 的 MySQL 的更多信息。

> 笔记
无论您选择哪种方法，请始终在执行升级之前备份您当前的 MySQL 安装。 请参见[第 7.2 节，“数据库备份方法”](https://dev.mysql.com/doc/refman/8.0/en/backup-methods.html)。

不支持在非 GA 版本之间进行升级（或从非 GA 版本升级到 GA 版本）。 在非 GA 版本中发生了重大的开发更改，您可能会遇到兼容性问题或启动服务器的问题。

> 笔记
MySQL Installer 不支持社区版本和商业版本之间的升级。 如果您需要这种类型的升级，请使用 ZIP 存档方法执行它。

## 使用 MySQL 安装程序升级 MySQL

当使用 MySQL Installer 执行当前服务器安装并且升级在当前版本系列中时，使用 MySQL Installer 执行升级是最佳方法。 MySQL Installer 不支持版本系列之间的升级，例如从 5.7 升级到 8.0，并且不提供升级指示器提示您升级。有关版本系列之间升级的说明，请使用 Windows ZIP 分发升级 MySQL。

要使用 MySQL 安装程序执行升级：

1、启动 MySQL 安装程序。

2、在仪表板中，单击目录(Catalog)以下载对目录的最新更改。仅当仪表板在服务器版本号旁边显示箭头时，才能升级已安装的服务器。

3、单击升级。所有具有较新版本的产品现在都显示在列表中。

> 笔记
MySQL Installer 取消选择同一版本系列中里程碑版本（Pre-Release）的服务器升级选项。此外，它会显示一条警告以指示不支持升级，识别继续进行的风险，并提供手动执行升级的步骤摘要。您可以重新选择服务器升级并自行承担风险。

4、取消选择除 MySQL 服务器产品之外的所有产品，除非您此时打算升级其他产品，然后单击下一步。

5、单击执行开始下载。下载完成后，单击下一步开始升级操作。

升级到 MySQL 8.0.16 及更高版本可能会显示一个选项来跳过系统表的升级检查和过程。有关此选项的详细信息，请参阅[重要的服务器升级条件](https://dev.mysql.com/doc/refman/8.0/en/mysql-installer-catalog-dashboard.html#mysql-installer-alter-upgrade)。

6、配置服务器。

## 使用 Windows ZIP 分发升级 MySQL

要使用 Windows ZIP 存档分发执行升级：

1、从 <https://dev.mysql.com/downloads/> 下载 MySQL 的最新 Windows ZIP 存档分发。

2、如果服务器正在运行，请停止它。如果服务器作为服务安装，请在命令提示符下使用以下命令停止服务：

`C:\> SC STOP mysqld_service_name`
或者，使用 NET STOP mysqld_service_name 。

如果您没有将 MySQL 服务器作为服务运行，请使用 [mysqladmin](https://dev.mysql.com/doc/refman/8.0/en/mysqladmin.html) 将其停止。例如，在从 MySQL 5.7 升级到 8.0 之前，使用 MySQL 5.7 中的 mysqladmin，如下所示：

`C:\> "C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqladmin" -u root shutdown`

> 笔记
如果 MySQL root 用户帐户有密码，请使用 -p 选项调用 mysqladmin 并在提示时输入密码。

3、提取 ZIP 存档。您可以覆盖现有的 MySQL 安装（通常位于 C:\mysql），或者将其安装到不同的目录，例如 C:\mysql8。建议覆盖现有安装。

4、重新启动服务器。例如，如果您将 MySQL 作为服务运行，则使用 SC START mysqld_service_name 或 NET START mysqld_service_name 命令，否则直接调用 mysqld。

5、在 MySQL 8.0.16 之前，以管理员身份运行 mysql_upgrade 来检查您的表，在必要时尝试修复它们，并在授权表发生更改时更新它们，以便您可以利用任何新功能。请参阅[第 4.4.5 节，“mysql_upgrade - 检查和升级 MySQL 表”](https://dev.mysql.com/doc/refman/8.0/en/mysql-upgrade.html)。从 MySQL 8.0.16 开始，不需要此步骤，因为服务器执行之前由 [mysql_upgrade](https://dev.mysql.com/doc/refman/8.0/en/mysql-upgrade.html) 处理的所有任务。

如果遇到错误，请参阅[第 2.3.5 节，“Microsoft Windows MySQL 服务器安装疑难解答”](https://dev.mysql.com/doc/refman/8.0/en/windows-troubleshooting.html)。
