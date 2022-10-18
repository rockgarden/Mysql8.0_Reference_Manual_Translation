# InnoDB锁

使用的锁类型

- Shared and Exclusive Locks

- Intention Locks

- Record Locks

- Gap Locks

- Next-Key Locks

- Insert Intention Locks

- AUTO-INC Locks

- Predicate Locks for Spatial Indexes

## 共享锁和独占锁

InnoDB实现了标准的行级锁定，其中有两种类型的锁，共享（S）锁和独占（X）锁。

- [shared (S) lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_shared_lock)允许持有该锁的事务读取行。

- [exclusive (X) lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_exclusive_lock)允许持有该锁的事务更新或删除行。

如果事务T1持有第r行上的共享（S）锁，那么来自某个不同事务T2的请求将按如下方式处理第r行的锁：

- T2对S锁的请求可以立即获得批准。因此，T1和T2在r上都持有S锁。

- T2对X锁的请求不能立即获得批准。

如果事务T1在第r行上持有独占（X）锁，则无法立即批准某个不同事务T2对r上任一类型锁的请求。相反，事务T2必须等待事务T1释放其对第r行的锁定。

## 意向锁

InnoDB支持多粒度锁定，允许行锁和表锁共存。例如，诸如LOCK TABLES…WRITE之类的语句在指定的表上采用独占锁（X锁）。为了实现多粒度级别的锁定，InnoDB使用意向锁。意向锁是表级锁，它指示事务稍后对表中的行需要哪种类型的锁（共享或独占）。有两种类型的意图锁：

- [intention shared lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_shared_lock) (IS)表示事务打算在表中的各行上设置共享锁。

- [intention exclusive lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_exclusive_lock) (IX)表示事务打算在表中的各行上设置独占锁。

例如，[SELECT…FOR SHARE](https://dev.mysql.com/doc/refman/8.0/en/select.html)设置IS锁，[SELECT…FOR UPDATE](https://dev.mysql.com/doc/refman/8.0/en/select.html)设置IX锁。

意图锁定协议如下：

- 在事务可以获取表中行的共享锁之前，它必须首先获取表上的IS锁或更强的锁。

- 在事务可以获取表中某行的独占锁之前，它必须首先获取表的IX锁。

表级锁类型兼容性总结在以下矩阵中。

| X  | IX       | S          | IS         |            |
|----|----------|------------|------------|------------|
| X  | Conflict | Conflict   | Conflict   | Conflict   |
| IX | Conflict | Compatible | Conflict   | Compatible |
| S  | Conflict | Conflict   | Compatible | Compatible |
| IS | Conflict | Compatible | Compatible | Compatible |

如果请求事务与现有锁兼容，则会将锁授予该事务，但如果与现有锁冲突，则不会授予该事务。事务将一直等待，直到冲突的现有锁被释放。如果锁请求与现有锁冲突，并且由于会导致[死锁](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_deadlock)而无法授予，则会发生错误。

意向锁不会阻止除全表请求之外的任何内容（例如，LOCK TABLES…WRITE）。意图锁的主要目的是显示有人正在锁定一行，或者打算锁定表中的一行。

意向锁的事务数据在SHOW ENGINE INNODB STATUS和INNODB监视器输出中显示类似于以下内容：

```sql
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

