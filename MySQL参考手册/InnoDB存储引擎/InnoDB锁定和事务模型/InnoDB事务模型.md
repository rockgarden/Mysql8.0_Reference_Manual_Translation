# InnoDB事务模型

- 15.7.2.1 [事务隔离级别](#事务隔离级别)
15.7.2.2 自动提交、提交和回滚
15.7.2.3 一致非锁定读取
15.7.2.4 锁定读数

InnoDB事务模型旨在将[多版本数据库](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_mvcc)的最佳属性与传统的两阶段锁定相结合。InnoDB在行级别执行锁定，默认情况下以Oracle样式作为非锁定[一致读取](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_consistent_read)来运行查询。InnoDB中的锁信息有效地存储了空间，因此不需要升级锁。通常，允许多个用户锁定InnoDB表中的每一行或任何随机的行子集，而不会导致InnoDB内存耗尽。

## 事务隔离级别

事务隔离是数据库处理的基础之一。Isolation是缩写[ACID](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_acid)中的I；隔离级别是当多个事务同时进行更改和执行查询时，微调性能与结果的可靠性、一致性和再现性之间的平衡的设置。

InnoDB提供了SQL:1992标准描述的所有四个事务隔离级别：[READ UNCOMMITTED](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted)、[READ COMMITTED](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)、REPEATABLE READ和[SERIALIZABLE](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_serializable)。InnoDB的默认隔离级别是[REPEATABLE READ](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)。

用户可以使用SETTRANSACTION语句更改单个会话或所有后续连接的隔离级别。要为所有连接设置服务器的默认隔离级别，请在命令行或选项文件中使用--transaction isolation选项。有关隔离级别和级别设置语法的详细信息，请参阅第13.3.7节“[SET TRANSACTION语句](https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html)”。

InnoDB使用不同的锁定策略支持这里描述的每个事务隔离级别。对于ACID遵从性很重要的关键数据上的操作，您可以强制使用默认的REPEATABLE READ级别实现高度一致性。或者，在批量报告等情况下，如果精确的一致性和可重复的结果不如最小化锁定的开销重要，则可以使用READ COMMITTED或甚至READ UNCOMMITTE放宽一致性规则。SERIALIZABLE强制执行比REPEATABLE READ更严格的规则，并且主要用于特殊情况，例如XA事务，以及解决并发和死锁问题。

以下列表描述了MySQL如何支持不同的事务级别。列表从最常用的级别到最少使用的级别。

- 可重复读取

  这是InnoDB的默认隔离级别。同一事务中的一致读取将读取第一次读取所建立的快照。这意味着，如果在同一事务中发出多个普通（非锁定）SELECT语句，这些SELECT声明也会相互一致。参见第15.7.2.3节，“[一致非锁定读取](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)”。

  对于[锁定读取](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking_read)（SELECT with For UPDATE or For SHARE）、UPDATE和DELETE语句，锁定取决于该语句是使用具有唯一搜索条件的唯一索引，还是使用范围类型搜索条件。

  - 对于具有唯一搜索条件的唯一索引，InnoDB只锁定找到的索引记录，而不锁定前面的[间隙](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_gap)。

  - 对于其他搜索条件，InnoDB会锁定扫描的索引范围，使用间隙锁或下一个密钥锁阻止其他会话插入范围覆盖的间隙。

- READ COMMITTED

  即使在同一事务中，每次一致读取也会设置和读取自己的新快照。

  对于锁定读取（SELECT with For UPDATE or For SHARE）、UPDATE语句和DELETE语句，InnoDB只锁定索引记录，而不锁定前面的空白，因此允许在锁定记录旁边自由插入新记录。间隙锁定仅用于外键约束检查和重复键检查。

  由于间隙锁定被禁用，因此可能会出现虚幻行问题，因为其他会话可以将新行插入间隙。

  READ COMMITTED隔离级别仅支持基于行的二进制日志记录。如果将READ COMMITTED与[binlog_format=MIXED](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_format)一起使用，服务器将自动使用基于行的日志记录。

  使用READ COMMITTED具有其他效果：

  - 对于UPDATE或DELETE语句，InnoDB仅对其更新或删除的行持有锁。在MySQL评估WHERE条件后，将释放不匹配行的记录锁。这大大降低了死锁的可能性，但它们仍然可能发生。

  - 对于UPDATE语句，如果一行已经被锁定，InnoDB会执行“半一致”读取，将最新提交的版本返回给MySQL，以便MySQL可以确定该行是否符合UPDATE的WHERE条件。如果行匹配（必须更新），MySQL会再次读取该行，这次InnoDB要么锁定该行，要么等待锁定该行。

  从下表开始，考虑以下示例：

    ```sql
    CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
    INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
    COMMIT;
    ```

  在这种情况下，表没有索引，因此搜索和索引扫描使用隐藏的聚集索引来锁定记录（请参阅第15.6.2.1节“[聚集索引和辅助索引](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)”），而不是索引列。

  假设一个会话使用以下语句执行UPDATE：

    ```sql
    # Session A
    START TRANSACTION;
    UPDATE t SET b = 5 WHERE b = 3;
    ```

  还假设第二个会话通过在第一个会话的语句之后执行这些语句来执行UPDATE：

    ```sql
    # Session B
    UPDATE t SET b = 4 WHERE b = 2;
    ```

  当InnoDB执行每个UPDATE时，它首先为每一行获取一个独占锁，然后决定是否修改它。如果InnoDB不修改该行，则会释放锁。否则，InnoDB将保留锁，直到事务结束。这会对事务处理产生以下影响。

  当使用默认的REPEATABLE READ隔离级别时，第一个UPDATE会在它读取的每一行上获取一个x锁，并且不会释放其中的任何一个：

    ```log
    x-lock(1,2); retain x-lock
    x-lock(2,3); update(2,3) to (2,5); retain x-lock
    x-lock(3,2); retain x-lock
    x-lock(4,3); update(4,3) to (4,5); retain x-lock
    x-lock(5,2); retain x-lock
    ```

  第二个UPDATE在尝试获取任何锁时就会阻塞（因为第一次更新保留了所有行的锁），直到第一次UPDATE提交或回滚后才继续：

    ```log
    x-lock(1,2); block and wait for first UPDATE to commit or roll back
    ```

  如果改用READ COMMITTED，则第一个UPDATE会获取它读取的每一行上的x-lock，并为它未修改的行释放x-lock：

    ```log
    x-lock(1,2); unlock(1,2)
    x-lock(2,3); update(2,3) to (2,5); retain x-lock
    x-lock(3,2); unlock(3,2)
    x-lock(4,3); update(4,3) to (4,5); retain x-lock
    x-lock(5,2); unlock(5,2)
    ```

  对于第二次UPDATE，InnoDB执行“semi-consistent”读取，将读取的每一行的最新提交版本返回给MySQL，以便MySQL可以确定该行是否符合UPDATE的WHERE条件：

    ```log
    x-lock(1,2); update(1,2) to (1,4); retain x-lock
    x-lock(2,3); unlock(2,3)
    x-lock(3,2); update(3,2) to (3,4); retain x-lock
    x-lock(4,3); unlock(4,3)
    x-lock(5,2); update(5,2) to (5,4); retain x-lock
    ```

  但是，如果WHERE条件包含索引列，并且InnoDB使用索引，那么在获取和保留记录锁时，只考虑索引列。在下面的示例中，第一个UPDATE在b=2的每行上获取并保留一个x锁。第二个UPDATE在尝试获取相同记录上的x锁时会阻塞，因为它还使用在列b上定义的索引。

    ```sql
    CREATE TABLE t (a INT NOT NULL, b INT, c INT, INDEX (b)) ENGINE = InnoDB;
    INSERT INTO t VALUES (1,2,3),(2,2,4);
    COMMIT;

    # Session A
    START TRANSACTION;
    UPDATE t SET b = 3 WHERE b = 2 AND c = 3;

    # Session B
    UPDATE t SET b = 4 WHERE b = 2 AND c = 4;
    ```

  READ COMMITTED隔离级别可以在启动时设置，也可以在运行时更改。在运行时，可以为所有会话全局设置，也可以为每个会话单独设置。

- READ UNCOMMITTED

  SELECT语句以非锁定方式执行，但可能会使用行的早期版本。因此，使用这种隔离级别，这样的读取是不一致的。这也称为[脏读](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dirty_read)。否则，此隔离级别的工作方式类似于READ COMMITTED。

- SERIALIZABLE

  此级别类似于REPEATABLE READ，但如果禁用自动提交，InnoDB会隐式将所有普通SELECT语句转换为SELECT…FOR SHARE。如果启用了[自动提交](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_autocommit)，则SELECT是它自己的事务。因此，它是只读的，如果作为一致（非锁定）读取执行，则可以序列化，并且不需要为其他事务阻塞。（若要强制普通SELECT阻止其他事务修改了选定的行，请禁用自动提交。）

> 笔记
从MySQL 8.0.22开始，无论隔离级别如何，从MySQL授权表读取数据（通过联接列表或子查询）但不修改数据的DML操作都不会获取MySQL授权表格上的读锁。有关更多信息，请参阅[Grant Table Concurrency.](https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html#grant-tables-concurrency)。
