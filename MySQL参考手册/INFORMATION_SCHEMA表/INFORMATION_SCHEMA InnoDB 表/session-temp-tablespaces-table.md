# INFORMATION_SCHEMA INNODB_SESSION_TEMP_TABLESPACES 表

INNODB_SESSION_TEMP_TABLESPACES 表提供了用于内部临时表和用户创建的临时表的会话临时表空间的元数据。该表是在 MySQL 8.0.13 中添加的。

INNODB_SESSION_TEMP_TABLESPACES 表有以下列：

- ID

  进程或会话 ID。

- SPACE 表空间

  表空间 ID。为会话临时表空间保留了 40 万个空间 ID。每次启动服务器时，都会重新创建会话临时表空间。空间 ID 在服务器关闭时不会被持久化，可以重复使用。

- PATH 路径

  表空间数据文件路径。会话临时表空间的扩展名为 ibt 文件。

- SIZE 大小

  表空间的大小，以字节为单位。

- STATE 状态

  表空间的状态。ACTIVE 表示当前会话正在使用该表空间。INACTIVE 表示该表空间在可用会话临时表空间池中。

- PURPOSE 目的

  表空间的用途。INTRINSIC 表示该表空间用于优化器优化内部临时表。SLAVE 表示表空间被分配用于在复制从属设备上存储用户创建的临时表。USER 表示表空间用于存储用户创建的临时表。NONE 表示未使用表空间。

> 备注
  必须具有 [PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process) 权限才能查询此表。
  使用 INFORMATION_SCHEMA [COLUMNS](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html) 表或 [SHOW COLUMNS](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html) 语句可查看有关此表列的其他信息，包括数据类型和默认值。
