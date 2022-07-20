# InnoDB和MySQL复制

可以以副本上的存储引擎与源上的存储引擎不同的方式使用复制。例如，您可以将对源上的 InnoDB 表的修改复制到副本上的 MyISAM 表。有关更多信息，请参阅“[将复制与不同的源和副本存储引擎一起使用](https://dev.mysql.com/doc/refman/8.0/en/replication-solutions-diffengines.html)”。

有关设置副本的信息，请参阅“[设置副本](https://dev.mysql.com/doc/refman/8.0/en/replication-setup-replicas.html)”和“[为数据快照选择方法](https://dev.mysql.com/doc/refman/8.0/en/replication-snapshot-method.html)”。

要在不删除源或现有副本的情况下创建新副本，请使用 MySQL Enterprise Backup 产品。

源上失败的事务不影响复制。 MySQL 复制基于 MySQL 写入修改数据的 SQL 语句的二进制日志。失败的事务（例如，由于外键违规或回滚）不会写入二进制日志，因此不会发送到副本。请参阅“[START TRANSACTION、COMMIT 和 ROLLBACK 语句](https://dev.mysql.com/doc/refman/8.0/en/commit.html)”。

**Replication** and **CASCADE** (复制和级联)。仅当共享外键关系的表在源和副本上都使用 InnoDB 时，才会在副本上执行源上 InnoDB 表的级联操作。无论您使用的是基于语句的复制还是基于行的复制，都是如此。假设您已经开始复制，然后使用以下 CREATE TABLE 语句在源上创建两个表，其中 InnoDB 被定义为默认存储引擎：

```sql
CREATE TABLE fc1 (
    i INT PRIMARY KEY,
    j INT
);

CREATE TABLE fc2 (
    m INT PRIMARY KEY,
    n INT,
    FOREIGN KEY ni (n) REFERENCES fc1 (i)
        ON DELETE CASCADE
);
```

如果副本将 MyISAM 定义为默认存储引擎，则会在副本上创建相同的表，但它们使用 MyISAM 存储引擎，并且忽略 FOREIGN KEY 选项。 现在我们在源上的表中插入一些行：

```sql
source> INSERT INTO fc1 VALUES (1, 1), (2, 2);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0

source> INSERT INTO fc2 VALUES (1, 1), (2, 2), (3, 1);
Query OK, 3 rows affected (0.19 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

此时，在源和副本上，表 fc1 包含 2 行，表 fc2 包含 3 行，如下所示：

```sql
source> SELECT * FROM fc1;
+---+------+
| i | j    |
+---+------+
| 1 |    1 |
| 2 |    2 |
+---+------+
2 rows in set (0.00 sec)

source> SELECT * FROM fc2;
+---+------+
| m | n    |
+---+------+
| 1 |    1 |
| 2 |    2 |
| 3 |    1 |
+---+------+
3 rows in set (0.00 sec)

replica> SELECT * FROM fc1;
+---+------+
| i | j    |
+---+------+
| 1 |    1 |
| 2 |    2 |
+---+------+
2 rows in set (0.00 sec)

replica> SELECT * FROM fc2;
+---+------+
| m | n    |
+---+------+
| 1 |    1 |
| 2 |    2 |
| 3 |    1 |
+---+------+
3 rows in set (0.00 sec)
```

现在假设您对源执行以下 DELETE 语句：

```sql
source> DELETE FROM fc1 WHERE i=1;
Query OK, 1 row affected (0.09 sec)
```

由于级联，源上的表 fc2 现在仅包含 1 行：

```sql
source> SELECT * FROM fc2;
+---+---+
| m | n |
+---+---+
| 2 | 2 |
+---+---+
1 row in set (0.00 sec)
```

但是，级联不会在副本上传播，因为在副本上 fc1 的 DELETE 不会从 fc2 中删除任何行。 fc2 的副本仍然包含最初插入的所有行：

```sql
replica> SELECT * FROM fc2;
+---+---+
| m | n |
+---+---+
| 1 | 1 |
| 3 | 1 |
| 2 | 2 |
+---+---+
3 rows in set (0.00 sec)
```

这种差异是因为级联删除是由 InnoDB 存储引擎在内部处理的，这意味着没有记录任何更改。
