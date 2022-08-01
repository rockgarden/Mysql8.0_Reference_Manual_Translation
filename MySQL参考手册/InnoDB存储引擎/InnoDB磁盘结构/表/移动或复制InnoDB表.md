# 移动或复制InnoDB表

这一小节讲解移动或拷贝部分或全部InnoDB表到另一个服务器或实例的技术。例如，你可能需要把一个MySQL实例迁移到一个更大更快的服务器上；你也可能会克隆一个完整的数据库实例到一个MySQL从库上；你可能会拷贝单独的几张表到另一个实例来开发或测试一个应用，或者导入到数仓中来生成报告。

在Winwods上，InnoDB总是以小写字母来存储数据库和表名。为了从Unix迁移一个二进制格式的数据库到Windows或者反过来，你应该总是使用小写字母来命名数据库和表。达到此目的一种简便方法是在创建表之前添加如下配置项到my.cnf或者my.ini配置文件中的[mysqld]配置段。

```ini
[mysqld]
lower_case_table_names=1
```

> **Note**
禁止使用和初始化服务器时不同的`lower_case_table_names`值来启动服务器。

移动和拷贝InnoDB表的技术包括以下：

- 导入表
- MySQL 企业备份
- 复制数据文件（冷备份方法）
- 从逻辑备份恢复

## 导入表

驻留在每表文件表空间中的表可以从另一个 MySQL 服务器实例或使用可传输表空间功能的备份导入。 请参阅[第 15.6.1.3 节，“导入 InnoDB 表”](https://dev.mysql.com/doc/refman/8.0/en/innodb-table-import.html)。

可传输的表空间特性主要使用`FLUSH TABLES ... FOR EXPORT`来让一张InnoDB表准备从一个数据库实例拷贝到另一个。为了使用这个特性，InnoDB表创建时必须打开`innodb_file_per_table`选项以便每张InnoDB表都有它自己的表空间。

## 商业版MySQL备份

MySQL企业版备份产品能够让你在生成一个一致性读快照时对数据库操作造成最少干扰的情况下来备份一个正在运行的数据库。当MySQL企业版备份正在拷贝表时，能够继续响应读写操作。而且，MySQL企业版备份能够压缩备份文件和备份表的部分数据。和MySQL的bin log结合，你能够执行某一时间点的备份。MySQL企业版备份作为MySQL企业版订阅的一部分。
更多有关企业版的信息，请查看[第30.2节 MySQL企业版版备份工具总览](https://dev.mysql.com/doc/refman/8.0/en/mysql-enterprise-backup.html)。

## 拷贝数据文件（冷备份）

您只需复制[第 15.18.1 节“InnoDB 备份”](https://dev.mysql.com/doc/refman/8.0/en/innodb-backup.html)中“冷备份”下列出的所有相关文件即可移动 InnoDB 数据库。

InnoDB 数据和日志文件在具有相同浮点数格式的所有平台上都是二进制兼容的。 如果浮点格式不同但您没有在表中使用 FLOAT 或 DOUBLE 数据类型，则过程相同：只需复制相关文件即可。

当您移动或复制每表文件的 .ibd 文件时，源系统和目标系统上的数据库目录名称必须相同。 存储在 InnoDB 共享表空间中的表定义包括数据库名称。 存储在表空间文件中的事务 ID 和日志序列号也因数据库而异。

要将 .ibd 文件和关联表从一个数据库移动到另一个数据库，请使用 RENAME TABLE 语句：

```sql
RENAME TABLE db1.tbl_name TO db2.tbl_name;
```

如果你有一个.ibd文件“干净”的备份，你可以按以下方法把它按照原来的方式还原到MySQL安装：

1. 拷贝.ibd文件之后表必须没被删除或清空，因为这些操作会让存储在表空间的表ID发生变化。
2. 使用 `ALTER TABLE` 语句来删除当前的.ibd文件。

   ```sql
   ALTER TABLE tbl_name DISCARD TABLESPACE;
   ```

3. 拷贝备份的.ibd 文件到合适的数据库目录。
4. 使用`ALTER TABLE`语句来告诉InnoDB使用新的 .ibd文件:

   ```sql
   ALTER TABLE tbl_name IMPORT TABLESPACE;
   ```

   > **Note**
   > `ALTER TABLE ... IMPORT TABLESPACE` 特性不会对导入的数据做强制外键约束。

在本文中，一份“干净”的.ibd 文件需要满足下列条件：

- .ibd 文件中的事务没有未提交的修改。

- .ibd 文件中没有未合并的插入缓冲区条目。

- Purge 已从 .ibd 文件中删除所有带有删除标记的索引记录。

- mysqld 已将 .ibd 文件的所有修改页面从缓冲池刷新到文件中。

您可以使用以下方法制作干净的备份 .ibd 文件：

1. 停止MySQL的所有操作并提交所有的事务。

2. 等待 `SHOW_ENGINE_INNODB_STATUS` 显示数据库中没有正在执行的事务，并且InnoDB的主线程状态状态是等待 `Waiting for server activity`。然后你就可以复制.ibd文件了。

另一个获得.ibd“干净”备份的方法是使用MySQL企业级备份产品：

1. 使用MySQL企业级备份来备份InnoDB安装。
2. 在备份机上开启另一个mysqld服务并让它清理这个.ibd文件。

## 导出导入（mysqldump）

你可以使用mysqldump在一台机器上导出你的表然后在另一台机器上导入这个备份文件。使用这个方法，表是否格式不同或者表有浮点数都没有关系。

一种可以提高这种导入性能的方法是关闭autocommit，假设表空间有足够的空间来存放导入事务时产生的庞大的回滚数据段。
当导入一整张表或者表的一段数据后再行提交。
