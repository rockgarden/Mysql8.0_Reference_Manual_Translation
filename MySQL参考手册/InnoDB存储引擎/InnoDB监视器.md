# InnoDB监视器

15.17.1 InnoDB监视器类型
15.17.2 启用InnoDB监视器
15.17.3 InnoDB标准监视器和锁定监视器输出

InnoDB监视器提供有关InnoDB内部状态的信息。此信息对于性能调整很有用。

## InnoDB监视器类型

InnoDB监视器有两种类型：

- 标准InnoDB监视器显示以下类型的信息：

  - 主后台线程完成的工作

  - 信号等待

  - 有关最近的外键和死锁错误的数据

  - 锁定等待事务

  - 活动事务持有的表和记录锁

  - 挂起的I/O操作和相关统计信息

  - 插入缓冲区和自适应哈希索引统计信息

  - 重做日志数据

  - 缓冲池统计

  - 行操作数据

- InnoDB锁定监视器打印额外的锁定信息，作为标准InnoDB监视器输出的一部分。

## 启用InnoDB监视器

当InnoDB监视器启用定期输出时，InnoDB大约每隔15秒将输出写入mysqld服务器标准错误输出（stderr）。

InnoDB将监视器输出发送到stderr，而不是stdout或固定大小的内存缓冲区，以避免潜在的缓冲区溢出。

在Windows上，除非另有配置，否则stderr将定向到默认日志文件。如果要将输出定向到控制台窗口而不是错误日志，请使用[--console](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_console)选项从控制台窗口中的命令提示符启动服务器。有关详细信息，请参阅Windows上的[默认错误日志目标](https://dev.mysql.com/doc/refman/8.0/en/error-log-destination-configuration.html#error-log-destination-configuration-windows)。

在Unix和类Unix系统上，除非另有配置，否则stderr通常定向到终端。有关更多信息，请参阅[Unix和类Unix系统上的默认错误日志目标](https://dev.mysql.com/doc/refman/8.0/en/error-log-destination-configuration.html#error-log-destination-configuration-unix)。

InnoDB监视器只应在您真正想查看监视器信息时启用，因为输出生成会导致性能下降。此外，如果监视器输出被定向到错误日志，那么如果您稍后忘记禁用监视器，日志可能会变得很大。

> 笔记
为了帮助故障排除，InnoDB在某些情况下会暂时启用标准InnoDB监视器输出。有关更多信息，请参阅第15.21节 [InnoDB故障排除](https://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting.html)。

`SELECT @@innodb_status_output;`

InnoDB监视器输出以包含时间戳和监视器名称的标题开始。例如：

```log
=====================================
2014-10-16 18:37:29 0x7fc2a95c1700 INNODB监视器输出
=====================================
```

标准InnoDB监视器（InnoDB Monitor OUTPUT）的标头也用于锁定监视器，因为后者通过添加额外的锁定信息生成相同的输出。

[innodb_status_output](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_status_output) [和innodb_status_output_locks](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_status_output_locks) 系统变量用于启用标准innodb监视器和innodb锁定监视器。

启用或禁用InnoDB监视器需要[PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)权限。

### 启用标准InnoDB监视器

通过将InnoDB_status_output系统变量设置为ON，启用标准InnoDB监视器。

```sql
mysql> SELECT @@innodb_status_output;
+------------------------+
| @@innodb_status_output |
+------------------------+
|                      0 |
+------------------------+
1 row in set (0.01 sec)
 
mysql> SET GLOBAL innodb_status_output=ON;
Query OK, 0 rows affected (0.01 sec)
 
mysql> SELECT @@innodb_status_output;
+------------------------+
| @@innodb_status_output |
+------------------------+
|                      1 |
+------------------------+
1 row in set (0.01 sec)
```

要禁用标准InnoDB监视器，请将InnoDB_status_output设置为OFF。

关闭服务器时，innodb_status_output变量被设置为默认的OFF值。

### 启用InnoDB锁定监视器

InnoDB Lock Monitor数据通过InnoDB Standard Monitor输出打印。必须启用InnoDB Standard Monitor和InnoDB Lock Monitor，才能定期打印InnoDB锁定监视器数据。

要启用InnoDB锁监视器，请将InnoDB_status_output_locks系统变量设置为ON。InnoDB标准监视器和InnoDB锁定监视器都必须启用，才能定期打印InnoDB Lock Monitor数据：

```sql
SET GLOBAL innodb_status_output=ON;
SET GLOBAL innodb_status_output_locks=ON;
```

要禁用InnoDB锁定监视器，请将InnoDB_status_output_locks设置为OFF。将innodb2_status_ output设置为OFF，也可以禁用Inno数据库标准监视器。

关闭服务器时，innodb_status_output和innodb_status_output_locks变量被设置为默认的OFF值。

> 笔记
要为SHOW ENGINE InnoDB STATUS输出启用InnoDB锁定监视器，只需启用innodb_status_output_locks。

### 按需获取标准InnoDB监视器输出

作为启用标准InnoDB Monitor进行定期输出的替代方法，您可以使用SHOW ENGINE InnoDB STATUS SQL语句按需获取标准InnoDB-Monitor输出，该语句将输出提取到客户端程序。如果您使用的是mysql交互式客户端，如果将常用的分号语句终止符替换为\G，则输出更具可读性：

`mysql> SHOW ENGINE INNODB STATUS\G`

如果启用INNODB锁定监视器，则SHOW ENGINE INNODB STATUS输出还包括INNODB Lock Monitor数据。

## 将标准InnoDB监视器输出定向到状态文件

通过在启动时指定 `--innodb-status-file` 选项，可以启用标准InnoDB监视器输出并将其定向到状态文件。使用此选项时，InnoDB会创建一个名为InnoDB_status的文件。pid并大约每隔15秒向其写入输出。

InnoDB会在服务器正常关闭时删除状态文件。如果发生异常关机，可能需要手动删除状态文件。

`--innodb-status-file` 选项用于临时使用，因为输出生成可能会影响性能和 innodb_status.pid 文件可能会随着时间的推移变得非常大。
