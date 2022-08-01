# InnoDB中的自增处理

nnoDB 提供了一种可配置的锁定机制，可以显着提高向具有 AUTO_INCREMENT 列的表添加行的 SQL 语句的可伸缩性和性能。 要将 AUTO_INCREMENT 机制与 InnoDB 表一起使用，必须将 AUTO_INCREMENT 列定义为某个索引的第一列或唯一列，以便可以对表执行等效的索引 SELECT MAX(ai_col) 查找以获得最大值 列值。 索引不需要是 PRIMARY KEY 或 UNIQUE，但为了避免 AUTO_INCREMENT 列中的重复值，建议使用这些索引类型。

本节描述了 AUTO_INCREMENT 锁定模式，不同 AUTO_INCREMENT 锁定模式设置的使用含义，以及 InnoDB 如何初始化 AUTO_INCREMENT 计数器。

## InnoDB AUTO_INCREMENT 锁定模式

本节介绍用于生成自动增量值的 AUTO_INCREMENT 锁定模式，以及每种锁定模式如何影响复制。 自动增量锁定模式在启动时使用 innodb_autoinc_lock_mode 变量进行配置。

以下术语用于描述 innodb_autoinc_lock_mode 设置：

- "INSERT-like"语句

  在一个表中会生成新记录的所有语句，包括`INSERT`,`INSERT ... SELECT`,`REPLACE`,`REPLACE ... SELECT` 和 `LOAD DATA`。包括“simple-inserts”，“bulk-inserts”和“mixed-mode inserts”。

- "Simple inserts"

  指的是插入的数据行数是能够提前确定的语句（当对语句进行最初处理的时候）。这包括单行和多行没有嵌套子查询的`INSERT`和`REPLACE`语句，但`INSERT ... ON DUPLICATE KEY UPDATE` 除外。

- "Bulk inserts"

  插入数据的行数（以及需要自增长的数值）是无法提前预知的语句。这包括`INSERT ... SELECT`,`REPLACE ... SELECT`和 非简单INSERT的`LOAD DATA`语句。InnoDB每次只分配给AUTO_INCREMENT列一个值，就像是一行一行的处理一样。

- "Mixed-mode inserts"

  对新插入数据中部分自增列指定了明确的值的“simple inserts”语句，例如下面这个例子，其中c1是表t1的自增列：

  > INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');

  另一种“混合模式插入”是 INSERT ... ON DUPLICATE KEY UPDATE，在最坏的情况下，它实际上是一个 INSERT 后跟一个 UPDATE，其中为 AUTO_INCREMENT 列分配的值可能会或可能不会在更新阶段。

innodb_autoinc_lock_mode 变量有三种可能的设置。 “traditional-传统”、“consecutive-连续”或“interleaved-交错”锁定模式的设置分别为 0、1 或 2。从 MySQL 8.0 开始，交错锁模式 (innodb_autoinc_lock_mode=2) 是默认设置。在 MySQL 8.0 之前，连续锁定模式是默认的（innodb_autoinc_lock_mode=1）。

MySQL 8.0 中交错锁模式的默认设置反映了从基于语句的复制到基于行的复制作为默认复制类型的变化。基于语句的复制需要连续的自增锁模式，以确保给定的SQL语句序列以可预测和可重复的顺序分配自增值，而基于行的复制对SQL语句的执行顺序不敏感.

- innodb_autoinc_lock_mode = 0 ("traditional"锁定模式)

  传统的锁定模式提供了与引入 innodb_autoinc_lock_mode 变量之前相同的行为。由于语义上可能存在差异，提供传统锁定模式选项是为了向后兼容、性能测试和解决“混合模式插入”问题。

  在这种锁定模式下，所有“INSERT-like”语句都会获得一个特殊的表级 AUTO-INC 锁，用于插入到具有 AUTO_INCREMENT 列的表中。此锁通常保持到语句的末尾（而不是事务的末尾），以确保为给定的 INSERT 语句序列以可预测和可重复的顺序分配自动递增值，并确保自动递增任何给定语句分配的值都是连续的。

  在基于语句的复制的情况下，这意味着当在副本服务器上复制 SQL 语句时，自动增量列使用与源服务器上相同的值。多个 INSERT 语句的执行结果是确定性的，副本复制与源上相同的数据。如果由多个 INSERT 语句生成的自动增量值交错，则两个并发 INSERT 语句的结果将是不确定的，并且无法使用基于语句的复制可靠地传播到副本服务器。

  为了清楚起见，考虑一个使用此表的示例：

  ```sql
  CREATE TABLE t1 (
    c1 INT(11) NOT NULL AUTO_INCREMENT,
    c2 VARCHAR(10) DEFAULT NULL,
    PRIMARY KEY (c1)
  ) ENGINE=InnoDB;
  ```

  假设有两个事务正在运行，每一个都正在向带有自增列的表插入数据。其中一个事务使用`INSERT ... SELECT`语句来插入1000行数据，另一个事务使用简单的`INSERT`语句插入一行数据：

  ```sql
  Tx1: INSERT INTO t1 (c2) SELECT 1000 rows from another table ...
  Tx2: INSERT INTO t1 (c2) VALUES ('xxx');
  ```

  InnoDB 无法提前知道从 Tx1 中的 INSERT 语句中的 SELECT 检索到多少行，并且随着语句的执行，它一次分配一个自动增量值。使用表级锁，保持到语句末尾，一次只能执行一个引用表 t1 的 INSERT 语句，并且不同语句生成自增数不会交错。 Tx1 INSERT ... SELECT 语句生成的自动增量值是连续的，并且 Tx2 中的 INSERT 语句使用的（单个）自动增量值小于或大于用于 Tx1 的所有值，具体取决于哪个语句首先执行。

  只要 SQL 语句在从二进制日志重放时（使用基于语句的复制时，或在恢复场景中）以相同的顺序执行，结果与 Tx1 和 Tx2 首次运行时的结果相同。因此，表级锁一直保持到语句结束，使得使用自动增量的 INSERT 语句可以安全地用于基于语句的复制。但是，当多个事务同时执行插入语句时，这些表级锁会限制并发性和可伸缩性。

  在前面的示例中，如果没有表级锁，则用于 Tx2 中的 INSERT 的自增列的值精确地取决于语句执行的时间。如果 Tx2 的 INSERT 在 Tx1 的 INSERT 运行时执行（而不是在它开始之前或完成之后），则由两个 INSERT 语句分配的特定自动增量值是不确定的，并且可能因运行而异。

  在连续锁模式下，InnoDB 可以避免对预先知道行数的“简单插入”语句使用表级 AUTO-INC 锁，并且仍然为基于语句的复制保留确定性执行和安全性。

  如果您不使用二进制日志作为恢复或复制的一部分重播 SQL 语句，则可以使用交错锁模式来消除所有表级 AUTO-INC 锁的使用，以获得更高的并发性和性能，但代价是允许由语句分配的自动增量编号中的间隙，并且可能具有由同时执行的语句交错分配的编号。

- innodb_autoinc_lock_mode = 1 (“consecutive”锁定模式)

  在这种模式下，“批量插入”使用特殊的 AUTO-INC 表级锁并持有它直到语句结束。这适用于所有 INSERT ... SELECT、REPLACE ... SELECT 和 LOAD DATA 语句。一次只能执行一个持有 AUTO-INC 锁的语句。如果批量插入操作的源表与目标表不同，则在对从源表中选择的第一行进行共享锁定之后，对目标表进行 AUTO-INC 锁定。如果批量插入操作的源和目标是同一个表，则在对所有选定行进行共享锁后，再使用 AUTO-INC 锁。

  “简单插入”（预先知道要插入的行数）通过在互斥锁（轻量级锁）的控制下获得所需数量的自动增量值来避免表级 AUTO-INC 锁仅在分配过程中保留，直到语句完成。除非另一个事务持有 AUTO-INC 锁，否则不使用表级 AUTO-INC 锁。如果另一个事务持有 AUTO-INC 锁，则“简单插入”会等待 AUTO-INC 锁，就好像它是“批量插入”一样。

  这种锁定模式确保在存在预先不知道行数的 INSERT 语句（以及在语句进行时分配自动增量编号）的情况下，由任何“INSERT-like”分配的所有自动增量值语句是连续的，并且操作对于基于语句的复制是安全的。

  简而言之，这种锁定模式显着提高了可伸缩性，同时可以安全地用于基于语句的复制。此外，与“传统”锁定模式一样，任何给定语句分配的自动递增编号都是连续的。对于任何使用自动增量的语句，与“传统”模式相比，语义没有变化，但有一个重要例外。

  例外是“混合模式插入”，其中用户为多行“简单插入”中的某些（但不是全部）行提供 AUTO_INCREMENT 列的显式值。对于此类插入，InnoDB 分配的自动增量值多于要插入的行数。但是，所有自动分配的值都是连续生成的（因此高于）最近执行的前一条语句生成的自动增量值。 “多余”的数字会丢失。

- innodb_autoinc_lock_mode = 2（“interleaved”锁定模式）

  在这种锁模式下，没有“INSERT-like”语句使用表级AUTO-INC锁，多个语句可以同时执行。这是最快且最具可扩展性的锁定模式，但在从二进制日志重放 SQL 语句时使用基于语句的复制或恢复场景时，它并**不安全**。

  在这种锁定模式下，自动增量值保证在所有并发执行的“类似插入”的语句中是唯一的且单调递增的。但是，由于多个语句可以同时生成数字（即，数字的分配在语句之间交错），为任何给定语句插入的行生成的值可能不是连续的。

  如果执行的唯一语句是“简单插入”，其中要插入的行数是提前知道的，那么除了“混合模式插入”之外，为单个语句生成的数字中没有间隙。但是，当执行“批量插入”时，任何给定语句分配的自动增量值可能存在间隙。

## 使用不同InnoDB自增锁定模式的影响

- 将自动增量“auto-increment”与复制一起使用

  如果您使用基于语句的复制，请将 innodb_autoinc_lock_mode 设置为 0 或 1，并在源及其副本上使用相同的值。 如果使用 innodb_autoinc_lock_mode = 2（“interleaved”）或源和副本不使用相同锁定模式的配置，则不能确保副本上的自动增量值与源上的值相同，即**无法保证在主从之间保持一致**。。

  如果您使用的是基于行或混合格式的复制，所有的自动增量锁定模式都是安全的，因为基于行的复制对 SQL 语句的执行顺序不敏感（并且混合格式使用基于行的 对于基于语句的复制不安全的任何语句的复制）。

- 自增值的“Lost-缺失”和自增序列间隙

  在所有锁定模式（0、1 和 2）中，如果生成自增值的事务回滚，则这些自增值将“丢失”。 一旦为自增列生成了值，无论“INSERT-like”语句是否完成，以及包含的事务是否回滚，都无法回滚。 这种丢失的值不会被重用。 因此，存储在表的 AUTO_INCREMENT 列中的值可能存在间隙。

- 为自增列指定NULL或0值

  在所有的自增锁定模式下（0，1或2），如果用户为自增列指定了NULL或者0，那么InnoDB会把该列没有指定值来对待并为之生成一个新值。

- 为自增列分配一个负数值

  在所有锁定模式（0、1 和 2）中，如果将负值分配给 AUTO_INCREMENT 列，则自动增量机制的行为是未定义的，其值是不确定的。

- 自增值已经超过了所指定整形类型的最大值上限

  在所有的自增锁定模式下（0，1或2），如果自增值已经超过所指定的数据类型所能存储的最大值的时候，此时自增机制的行为是不确定的。

- “bulk inserts”时自增值之间间隙

  将 innodb_autoinc_lock_mode 设置为 0（“traditional”）或 1（“consecutive”），任何给定语句生成的自动增量值都是连续的，没有间隙，因为表级 AUTO-INC 锁一直保持到结束语句，并且一次只能执行一个这样的语句。

  将 innodb_autoinc_lock_mode 设置为 2（“interleaved”），“批量插入”生成的自动增量值可能存在间隙，但前提是同时执行“INSERT-like”语句。

  对于锁定模式 1 或 2，连续语句之间可能会出现间隙，因为对于批量插入，可能不知道每个语句所需的自动增量值的确切数量，并且可能会高估。

- “mixed-mode inserts”分配的自增值

  考虑“mixed-mode insert”,一种只给插入的部分数据行（不是全部）指定了自增值的“simple inserts”。这样的语句在锁定模式0，1和2 之间的行为都是不一样的。例如，假设c1是表t1的自增列，并且最近生成的自增值是100。

  ```sql
  mysql> CREATE TABLE t1 (
      -> c1 INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
      -> c2 CHAR(1)
      -> ) ENGINE = INNODB;
  ```

  现在，考虑下面的“mixed-mode insert”语句：

  ```sql
  mysql> INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');
  ```

  当`innodb_autoinc_lock_mode`设置为0（“traditional”），插入的4行数据是：

  ```sql
  mysql> SELECT c1, c2 FROM t1 ORDER BY c2;
  +-----+------+
  | c1  | c2   |
  +-----+------+
  |   1 | a    |
  | 101 | b    |
  |   5 | c    |
  | 102 | d    |
  +-----+------+
  ```

  下一个可用的自动增量值是 103，因为自动增量值一次分配一个，而不是在语句执行开始时一次性分配。 无论是否同时执行“INSERT-like”语句（任何类型），这个结果都是正确的。

  将 innodb_autoinc_lock_mode 设置为 1（“连续”），四个新行也是：

  ```sql
  mysql> SELECT c1, c2 FROM t1 ORDER BY c2;
  +-----+------+
  | c1  | c2   |
  +-----+------+
  |   1 | a    |
  | 101 | b    |
  |   5 | c    |
  | 102 | d    |
  +-----+------+
  ```

  但是，在这种情况下，下一个自增值是105而不是103，因为在语句执行的时候分配了4个自增值但只使用了其中的两个。不管当前是否在并发执行任意的“INSERT-like”语句该结论都成立。

  当 `innodb_autoinc_lock_mode`的值设置为2时（“interleaved”），插入的4行新数据不变，同上。

  x和y的值是唯一的并且大于任何之前创建的值。然而，x和y的具体值依赖于同一时间并发执行的语句所创建的值。

  最后，考虑下面的语句，当最近执行语句产生的自增值是100：

  ```sql
  mysql> INSERT INTO t1 (c1,c2) VALUES (1,'a'), (NULL,'b'), (101,'c'), (NULL,'d');
  ```

  对于任意的`innodb_autoinc_lock_mode`设置值，这个语句都会引发一个duplicate-key 23000的错误码（Can't write;duplicate key in table），由于101已经被分配给数据行（null, 'b'）然后插入数据行（101， 'c'）就失败了。

- 修改一组`INSERT`语句中间的自增值

  在 MySQL5.7以及之前的版本中，修改一组`INSERT`语句中间的自增值会导致`Duplicate entry`错误。例如，你执行一个`UPDATE`操作把一个自增列的值更新为比当前所有自增值还要大的值，随后没有指定一个未使用的自增值的`INSERT`操作会引发`Duplicate entry`错误。在MySQL8.0及以后版本，如果你更新一个自增列的值到一个比当前所有值还要大的值，新值将会被保持下来，随后的`INSERT`操作分配的自增值将从这个更大的新值开始。下面用一个例子来验证这种行为。

  ```sql
  mysql> CREATE TABLE t1 (
      -> c1 INT NOT NULL AUTO_INCREMENT,
      -> PRIMARY KEY (c1)
      ->  ) ENGINE = InnoDB;

  mysql> INSERT INTO t1 VALUES(0), (0), (3);

  mysql> SELECT c1 FROM t1;
  +----+
  | c1 |
  +----+
  |  1 |
  |  2 |
  |  3 |
  +----+

  mysql> UPDATE t1 SET c1 = 4 WHERE c1 = 1;

  mysql> SELECT c1 FROM t1;
  +----+
  | c1 |
  +----+
  |  2 |
  |  3 |
  |  4 |
  +----+

  mysql> INSERT INTO t1 VALUES(0);

  mysql> SELECT c1 FROM t1;
  +----+
  | c1 |
  +----+
  |  2 |
  |  3 |
  |  4 |
  |  5 |
  +----+
  ```

## InnoDB自增计数器初始化

这一小节介绍InnoDB如何初始化AUTO_INCREMENT计数器

如果你为一张InnoDB表指定了自增列，在内存中的表对象包含一个自增计数器，用来给自增列分配自增值。

在 MySQL 5.7 及更早版本中，自动增量计数器存储在主内存中，而不是磁盘上。要在服务器重新启动后初始化自动增量计数器，InnoDB 将在第一次插入包含 AUTO_INCREMENT 列的表时执行与以下语句等效的语句。

```sql
SELECT MAX(ai_col) FROM table_name FOR UPDATE;
```

在 MySQL 8.0 中，这种行为发生了变化。当前最大的自动增量计数器值在每次更改时写入重做日志并保存到每个检查点的数据字典中。这些更改使当前最大的自动增量计数器值在服务器重新启动时保持不变。

在服务器正常关闭后重新启动时，InnoDB 使用存储在数据字典中的当前最大自动增量值初始化内存中的自动增量计数器。

在崩溃恢复期间服务器重新启动时，InnoDB 使用存储在数据字典中的当前最大自动增量值初始化内存中的自动增量计数器，并扫描重做日志以查找自上一个检查点以来写入的自动增量计数器值。如果重做记录的值大于内存中的计数器值，则应用重做记录的值。但是，在服务器意外退出的情况下，无法保证重用先前分配的自动增量值。每次由于 INSERT 或 UPDATE 操作而改变当前最大自增值时，都会将新值写入重做日志，但如果在重做日志刷新到磁盘之前发生意外退出，则先前分配的值可能是在服务器重新启动后初始化自动增量计数器时重用。

InnoDB 使用等效的 SELECT MAX(ai_col) FROM table_name FOR UPDATE 语句来初始化自动增量计数器的唯一情况是在导入没有 .cfg 元数据文件的表时。否则，从 .cfg 元数据文件（如果存在）中读取当前最大自动增量计数器值。除了计数器值初始化之外，当尝试将计数器值设置为小于或等于使用 ALTER TABLE ... AUTO_INCREMENT = N FOR UPDATE 语句持久化计数器值。例如，您可能会在删除一些记录后尝试将计数器值设置为较小的值。在这种情况下，必须查表以确保新的计数器值不小于或等于当前的实际最大计数器值。

在 MySQL 5.7 和更早版本中，服务器重新启动会取消 AUTO_INCREMENT = N 表选项的效果，该选项可以在 CREATE TABLE 或 ALTER TABLE 语句中分别用于设置初始计数器值或更改现有计数器值。在 MySQL 8.0 中，服务器重新启动不会取消 AUTO_INCREMENT = N 表选项的效果。如果将自动增量计数器初始化为特定值，或者将自动增量计数器值更改为更大的值，则新值会在服务器重新启动时保持不变。

> **笔记**
ALTER TABLE ... AUTO_INCREMENT = N 只能将自增计数器值更改为大于当前最大值的值。

在 MySQL 5.7 和更早版本中，在 ROLLBACK 操作之后立即重新启动服务器可能会导致重用先前分配给回滚事务的自动增量值，从而有效地回滚当前最大的自动增量值。在 MySQL 8.0 中，当前最大的自动增量值被持久化，防止重复使用以前分配的值。

如果 SHOW TABLE STATUS 语句在自动增量计数器初始化之前检查表，InnoDB 会打开表并使用存储在数据字典中的当前最大自动增量值初始化计数器值。然后将该值存储在内存中以供以后的插入或更新使用。计数器值的初始化使用对持续到事务结束的表的正常排他锁定读取。 InnoDB 在为用户指定的自增值大于 0 的新创建的表初始化自增计数器时遵循相同的过程。

自增计数器初始化后，如果插入行时没有显式指定自增值，InnoDB 会隐式递增计数器并将新值分配给列。如果插入显式指定自增列值的行，并且该值大于当前最大计数器值，则将计数器设置为指定值。

只要服务器运行，InnoDB 就会使用内存中的自动增量计数器。当服务器停止并重新启动时，InnoDB 重新初始化自动增量计数器，如前所述。

auto_increment_offset 变量确定 AUTO_INCREMENT 列值的起点。默认设置为 1。

auto_increment_increment 变量控制连续列值之间的间隔。默认设置为 1。

> 笔记
当 AUTO_INCREMENT 整数列的值用完时，后续的 INSERT 操作会返回重复键错误。这是一般的 MySQL 行为。
