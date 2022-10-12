# UPDATE语句

UPDATE是一个DML语句，用于修改表中的行。

UPDATE语句可以以with子句开头，以定义UPDATE中可访问的公共表表达式。见第13.2.15节，“[WITH（公共表表达式）](https://dev.mysql.com/doc/refman/8.0/en/with.html)”。

单表语法：

```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
    SET assignment_list
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]

value:
    {expr | DEFAULT}

assignment:
    col_name = value

assignment_list:
    assignment [, assignment] ...
```

多表语法：

```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_references
    SET assignment_list
    [WHERE where_condition]
```

对于单表语法，UPDATE语句使用新值更新命名表中现有行的列。SET子句指示要修改的列及其应给定的值。每个值都可以作为表达式给定，也可以使用关键字DEFAULT将列显式设置为其默认值。如果给定WHERE子句，则指定标识要更新哪些行的条件。如果没有WHERE子句，则更新所有行。如果指定了ORDER BY子句，则按指定的顺序更新行。LIMIT子句限制可以更新的行数。

对于多表语法，UPDATE更新table_references中指定的每个表中满足条件的行。每个匹配行更新一次，即使它多次匹配条件。对于多表语法，不能使用ORDER BY和LIMIT。

对于分区表，此语句的单表形式和多表形式都支持将PARTITION子句用作表引用的一部分。此选项采用一个或多个分区或子分区（或两者）的列表。只检查列出的分区（或子分区）是否匹配，不在这些分区或子分区中的行不会更新，无论它是否满足where_condition。

> 笔记
与将PARTITION与INSERT或REPLACE语句一起使用时的情况不同，即使列出的分区（或子分区）中没有行与where_condition匹配，否则有效的UPDATE…PARTITION语句也被视为成功。
Unlike the case when using PARTITION with an INSERT or REPLACE statement, an otherwise valid UPDATE ... PARTITION statement is considered successful even if no rows in the listed partitions (or subpartitions) match the where_condition.

有关更多信息和示例，请参阅第24.5节“[分区选择](https://dev.mysql.com/doc/refman/8.0/en/partitioning-selection.html)”。

- where_condition 是一个表达式，对于要更新的每一行，其计算结果为true。有关表达式语法，请参阅第9.5节“[表达式](https://dev.mysql.com/doc/refman/8.0/en/expressions.html)”。

- table_references 和 where_condition 如第13.2.10节“[SELECT语句](https://dev.mysql.com/doc/refman/8.0/en/select.html)”所述。

您只需要对UPDATE中引用的实际更新的列使用UPDATE特权。对于已读取但未修改的任何列，您只需要SELECT权限。

UPDATE语句支持以下修饰符：

- 使用 LOW_PRIORITY 修饰符，UPDATE的执行将被延迟，直到没有其他客户机从表中读取数据为止。这只影响只使用表级锁定的存储引擎（如MyISAM、MEMORY和MERGE）。

- 使用IGNORE修饰符，即使更新期间发生错误，更新语句也不会中止。在唯一键值上发生重复键值冲突的行不会更新。更新为可能导致数据转换错误的值的行将更新为最接近的有效值。有关更多信息，请参阅[IGNORE对语句执行的影响](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#ignore-effect-on-execution)。

[UPDATE IGNORE](https://dev.mysql.com/doc/refman/8.0/en/update.html) 语句（包括具有ORDER BY子句的语句）对于基于语句的复制来说被标记为不安全。（这是因为行的更新顺序决定了忽略哪些行。）当使用基于语句的模式时，这些语句会在错误日志中生成警告，当使用MIXED模式时，会使用基于行的格式写入二进制日志。（错误号11758262，错误号50439）有关更多信息，请参阅第17.2.1.3节“[二进制日志中安全和不安全声明的确定](https://dev.mysql.com/doc/refman/8.0/en/replication-rbr-safe-unsafe.html)”。

如果从表中访问要在表达式中更新的列，UPDATE将使用该列的当前值。例如，以下语句将col1设置为比其当前值多一个值：

`UPDATE t1 SET col1 = col1 + 1;`

下面语句中的第二个赋值将col2设置为当前（更新的）col1值，而不是原始col1值。结果是col1和col2具有相同的值。此行为与标准SQL不同。

`UPDATE t1 SET col1 = col1 + 1, col2 = col1;`

单表UPDATE赋值通常从左到右进行计算。对于多个表更新，不能保证按任何特定顺序执行分配。

如果将列设置为当前的值，MySQL会注意到这一点，并且不会更新它。

如果通过将设置为NULL来更新已声明为NOT NULL的列，则如果启用严格SQL模式，则会发生错误；否则，该列将设置为列数据类型的隐式默认值，警告计数将递增。对于数字类型，隐式默认值为0；对于字符串类型，为空字符串('')；对于日期和时间类型，为“zero”值。参见第11.6节“[数据类型默认值](https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html)”。

如果显式更新生成的列，则唯一允许的值是 `DEFAULT` 。有关生成的列的信息，请参阅第13.1.20.8节“[CREATE TABLE and generated columns](https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html)”。

UPDATE返回实际更改的行数。[mysql_info()](https://dev.mysql.com/doc/c-api/8.0/en/mysql-info.html) C API函数返回匹配和更新的行数以及UPDATE期间出现的警告数。

可以使用 LIMIT row_count 限制UPDATE的范围。LIMIT子句是行匹配限制。该语句在找到满足WHERE子句的row_count行后立即停止，无论这些行是否实际被更改。

如果UPDATE语句包含ORDER BY子句，则按子句指定的顺序更新行。这在某些可能导致错误的情况下非常有用。假设表t包含具有唯一索引的列id。以下语句可能会因重复的键错误而失败，具体取决于行的更新顺序：

`UPDATE t SET id = id + 1;`

例如，如果表的id列中包含1和2，并且1在2更新为3之前更新为2，则会发生错误。要避免此问题，请添加ORDER BY子句，以使id值较大的行在值较小的行之前更新：

`UPDATE t SET id = id + 1 ORDER BY id DESC;`

还可以执行覆盖多个表的UPDATE操作。但是，不能对多表UPDATE使用ORDER BY或LIMIT。table_references子句列出了联接中涉及的表。其语法如第13.2.10.2节“[JOIN子句](https://dev.mysql.com/doc/refman/8.0/en/join.html)”所述。下面是一个示例：

```sql
UPDATE items,month SET items.price=month.price
WHERE items.id=month.id;
```

前面的示例显示了使用逗号运算符的内部联接，但多个表UPDATE语句可以使用SELECT语句中允许的任何类型的联接，例如LEFT join。

如果使用涉及有外键约束的InnoDB表的多表UPDATE语句，MySQL优化器可能会以不同于父/子关系的顺序处理表。在这种情况下，语句失败并回滚。相反，更新一个表，并依赖InnoDB提供的 ON UPDATE 功能来相应地修改其他表。见第13.1.20.5节，“[FOREIGN KEY Constraints](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)”。

不能更新表并直接从子查询中的同一表中进行选择。您可以通过使用多表更新来解决这个问题，其中一个表是从您实际希望更新的表派生出来的，并使用别名引用派生表。假设您希望更新一个名为items的表，该表使用如下所示的语句定义：

```sql
CREATE TABLE items (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    wholesale DECIMAL(6,2) NOT NULL DEFAULT 0.00,
    retail DECIMAL(6,2) NOT NULL DEFAULT 0.00,
    quantity BIGINT NOT NULL DEFAULT 0
);
```

要降低加价为30%或以上且库存不足100件的任何商品的零售价，可以尝试使用UPDATE语句，如下面的语句，该语句在WHERE子句中使用子查询。如图所示，此语句不起作用：

```sql
mysql> UPDATE items
     > SET retail = retail * 0.9
     > WHERE id IN
     >     (SELECT id FROM items
     >         WHERE retail / wholesale >= 1.3 AND quantity > 100);
ERROR 1093 (HY000): You can't specify target table 'items' for update in FROM clause
```

相反，您可以使用多表更新，其中子查询被移动到要更新的表列表中，使用别名在最外层的WHERE子句中引用它，如下所示：

```sql
UPDATE items,
       (SELECT id FROM items
        WHERE id IN
            (SELECT id FROM items
             WHERE retail / wholesale >= 1.3 AND quantity < 100))
        AS discounted
SET items.retail = items.retail * 0.9
WHERE items.id = discounted.id;
```

由于优化器默认尝试将折扣后的派生表合并到最外层的查询块中，因此只有在强制实现派生表时才有效。您可以在运行更新之前将optimizer_switch系统变量的derived_merge标志设置为off，或者使用NO_merge优化器提示，如下所示：

```sql
UPDATE /*+ NO_MERGE(discounted) */ items,
       (SELECT id FROM items
        WHERE retail / wholesale >= 1.3 AND quantity < 100)
        AS discounted
    SET items.retail = items.retail * 0.9
    WHERE items.id = discounted.id;
```

在这种情况下使用优化器提示的优点是，它只应用于使用它的查询块中，因此在执行UPDATE之后，不必再次更改优化器switch的值。

另一种可能性是重写子查询，使其不使用IN或EXISTS，如下所示：

```sql
UPDATE items,
       (SELECT id, retail / wholesale AS markup, quantity FROM items)
       AS discounted
    SET items.retail = items.retail * 0.9
    WHERE discounted.markup >= 1.3
    AND discounted.quantity < 100
    AND items.id = discounted.id;
```

在这种情况下，子查询在默认情况下是具体化(materialized)的，而不是合并(merged)的，因此没有必要禁用派生表的合并。
