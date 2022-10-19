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

## 记录锁

记录锁是索引记录上的锁。例如，SELECT c1 FROM t WHERE c1=10 For UPDATE；防止任何其他事务插入、更新或删除t.c1值为10的行。

记录锁始终锁定索引记录，即使表定义时没有索引。对于这种情况，InnoDB会创建一个隐藏的聚集索引，并使用该索引锁定记录。见第15.6.2.1节，“[聚集索引和二级索引](https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html)”。

记录锁的事务数据在SHOW ENGINE INNODB STATUS和INNODB监视器输出中显示类似于以下内容：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

## 间隙锁

间隙锁是对索引记录之间间隙的锁定，或对第一条索引记录之前或最后一条索引记录之后间隙的锁定。例如，SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 For UPDATE；防止其他事务将值15插入到列t.c1中，无论该列中是否已经存在这样的值，因为范围中所有现有值之间的间隙都被锁定。

间隙可能跨越单个索引值、多个索引值，甚至可能为空。

间隙锁是性能和并发性之间权衡的一部分，在某些事务隔离级别中使用，而在其他级别中不使用。

对于使用唯一索引锁定行以搜索唯一行的语句，不需要进行间隙锁定。（这不包括搜索条件仅包含多列唯一索引的某些列的情况；在这种情况下，会发生间隙锁定。）例如，如果id列具有唯一索引，则以下语句仅对id值为100的行使用索引记录锁定，而其他会话是否在前面的间隙中插入行并不重要：

`SELECT * FROM child WHERE id = 100;`

如果id没有索引或具有非唯一索引，则语句会锁定前面的间隙。

这里还值得注意的是，不同的事务可以在间隙上持有冲突的锁。例如，事务A可以在间隙上持有共享间隙锁（间隙S-锁），而事务B在同一间隙上持有独占间隙锁（差距X-锁）。允许间隙锁冲突的原因是，如果从索引中清除记录，则必须合并不同事务在记录上持有的间隙锁。

InnoDB中的Gap锁是“完全禁止的”，这意味着它们的唯一目的是防止其他事务插入到Gap中。间隙锁可以共存。一个事务使用的间隙锁不会阻止另一个事务对同一间隙使用间隙锁。共享间隙锁和独占间隙锁之间没有区别。它们不会相互冲突，并且执行相同的功能。

间隙锁定可以明确禁用。如果将事务隔离级别更改为[READ COMMITTED](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)，则会发生这种情况。在这种情况下，对搜索和索引扫描禁用间隙锁定，并且仅用于外键约束检查和重复键检查。

使用READ COMMITTED隔离级别还有其他影响。在MySQL评估WHERE条件后，将释放不匹配行的记录锁。对于UPDATE语句，InnoDB执行“半一致”读取，以便将最新提交的版本返回给MySQL，以便MySQL可以确定该行是否符合UPDATE的WHERE条件。

## Next-Key Locks

下一个键锁是索引记录上的记录锁和索引记录之前的间隙上的间隙锁的组合。

InnoDB执行行级锁定的方式是，在搜索或扫描表索引时，它会对遇到的索引记录设置共享或独占锁定。因此，行级锁实际上是索引记录锁。索引记录上的下一个键锁定也会影响该索引记录之前的“间隙”。也就是说，下一个键锁是索引记录锁加上索引记录前面的间隙上的间隙锁。如果一个会话对索引中的记录R具有共享或独占锁定，则另一个会话无法在索引顺序中紧靠R之前的间隙中插入新的索引记录。

假设一个索引包含值10、11、13和20。该索引可能的下一个键锁涵盖以下间隔，其中圆括号表示排除间隔端点，方括号表示包含端点：

```log
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```

对于最后一个间隔，下一个密钥锁将间隙锁定在索引中最大值之上，并锁定值高于索引中实际任何值的“最高”伪记录。上确界不是真正的索引记录，因此，实际上，下一个密钥锁只锁定最大索引值之后的间隙。

默认情况下，InnoDB以[REPEATABLE READ](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)事务隔离级别运行。在这种情况下，InnoDB使用下一个键锁进行搜索和索引扫描，这可以防止虚幻行（请参阅第15.7.4节“[Phantom Rows](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html)”）。

下一个钥匙锁的事务数据在SHOW ENGINE INNODB STATUS和INNODB监视器输出中显示类似于以下内容：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10080 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

## 插入意向锁

插入意图锁是一种间隙锁，由行插入之前的insert操作设置。此锁定表示插入的意图是，如果插入到同一索引间隙中的多个事务没有插入到间隙中的同一位置，则它们不需要等待对方。假设存在值为4和7的索引记录。尝试分别插入值为5和6的单独事务，在获得插入行的排他锁之前，每个事务都使用插入意图锁锁定4和7之间的间隙，但不要相互阻塞，因为这些行是非冲突的。

下面的示例演示了一个事务在获取插入记录的独占锁之前获取插入意图锁。这个例子涉及两个客户，A和B。

客户机A创建一个包含两条索引记录（90和102）的表，然后启动一个事务，该事务对ID大于100的索引记录放置独占锁。独占锁包括记录102之前的间隙锁：

```log
mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
mysql> INSERT INTO child (id) values (90),(102);

mysql> START TRANSACTION;
mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
+-----+
| id  |
+-----+
| 102 |
+-----+
```

客户机B开始一个事务，将一条记录插入间隙。事务在等待获取独占锁时获取插入意图锁。

```sql
mysql> START TRANSACTION;
mysql> INSERT INTO child (id) VALUES (101);
```

插入意图锁的事务数据在SHOW ENGINE INNODB STATUS和INNODB监视器输出中类似于以下内容：

```sql
RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
trx id 8731 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000066; asc    f;;
 1: len 6; hex 000000002215; asc     " ;;
 2: len 7; hex 9000000172011c; asc     r  ;;...
```

## AUTO-INC Locks

AUTO-INC锁是一种特殊的表级锁，由插入具有AUTO_INCREMENT列的表的事务获取。在最简单的情况下，如果一个事务正在向表中插入值，则任何其他事务都必须等待自己向该表中插入，以便由第一个事务插入的行接收连续的主键值。

innodb_autoinc_lock_mode变量控制用于自动增量锁定的算法。它允许您选择如何在可预测的自动递增值序列和插入操作的最大并发性之间进行权衡。

有关更多信息，请参阅第15.6.1.6节“[InnoDB中的AUTO_INCREMENT处理](https://dev.mysql.com/doc/refman/8.0/en/innodb-auto-increment-handling.html)”。

## 空间索引谓词锁

InnoDB支持对包含空间数据的列进行SPATIAL索引（见第11.4.9节“[优化空间分析](https://dev.mysql.com/doc/refman/8.0/en/optimizing-spatial-analysis.html)”）。

为了处理涉及SPATIAL索引的操作的锁定，下一个键锁定无法很好地支持REPEATABLE READ或[SERIALIZABLE](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_serializable)事务隔离级别。多维数据中没有绝对排序概念，因此不清楚哪个是“下一个”键。

为了支持具有SPATIAL索引的表的隔离级别，InnoDB使用谓词锁。SPATIAL索引包含最小边界矩形（MBR）值，因此InnoDB通过对用于查询的MBR值设置谓词锁来强制对索引进行一致读取。其他事务无法插入或修改与查询条件匹配的行。
