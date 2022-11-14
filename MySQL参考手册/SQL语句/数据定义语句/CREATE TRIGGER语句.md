# CREATE TRIGGER 语句

```sql
CREATE
    [DEFINER = user]
    TRIGGER [IF NOT EXISTS] trigger_name
    trigger_time trigger_event
    ON tbl_name FOR EACH ROW
    [trigger_order]
    trigger_body

trigger_time: { BEFORE | AFTER }

trigger_event: { INSERT | UPDATE | DELETE }

trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

这条语句创建了一个新的触发器。触发器是一个命名的数据库对象，它与一个表相关联，当该表发生特定事件时就会激活。触发器与名为tbl_name的表相关联，这个表必须是指一个永久的表。你不能将触发器与临时表或视图相关联。

触发器的名称存在于模式名称空间中，这意味着所有触发器在一个模式中必须有唯一的名称。不同模式中的触发器可以有相同的名称。

IF NOT EXISTS可以防止在同一模式中存在同一表的同名触发器时发生错误。从MySQL 8.0.29开始，CREATE TRIGGER支持这个选项。

本节描述了CREATE TRIGGER的语法。有关其他讨论，见第25.3.1节 "[触发器语法和实例](https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html)"。

CREATE TRIGGER要求与触发器相关的表有[TRIGGER](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_trigger)权限。如果DEFINER子句存在，所需的权限取决于用户值。如果启用了二进制日志，CREATE TRIGGER可能需要SUPER权限，如第25.7节 "[存储程序二进制日志](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html)"中所讨论的。

DEFINER子句决定了在触发器激活时检查访问权限时使用的安全上下文，如本节后面所述。

trigger_time是触发器的动作时间。它可以是BEFORE或AFTER，表示触发器在每条要修改的行之前或之后激活。

**基本的列值检查发生在触发器激活之前，所以不能使用BEFORE触发器将不适合列类型的值转换为有效值。**

trigger_event表示激活触发器的操作种类。这些 trigger_event值是允许的。

- INSERT：每当向表中插入一条新的记录（例如，通过INSERT, [LOAD DATA](https://dev.mysql.com/doc/refman/8.0/en/load-data.html), 和 REPLACE语句），触发器就会激活。

- UPDATE: 每当一条记录被修改（例如，通过UPDATE语句），触发器就会激活。

- DELETE：每当从表中删除一条记录（例如，通过DELETE和REPLACE语句），触发器就会激活。表上的DROP TABLE和TRUNCATE TABLE语句不会激活这个触发器，因为它们不使用DELETE。丢弃一个分区也不会激活DELETE触发器。

trigger_event并不代表激活触发器的SQL语句的字面类型，而是代表一种表操作的类型。例如，INSERT触发器不仅激活INSERT语句，也激活LOAD DATA语句，因为这两个语句都向表中插入行。

一个可能会引起混淆的例子是INSERT INTO ... ON DUPLICATE KEY UPDATE ... 语法：一个BEFORE INSERT触发器为每条记录激活，然后是AFTER INSERT触发器或BEFORE UPDATE和AFTER UPDATE触发器，这取决于该记录是否有一个重复的键。

> 注意
层叠的外键操作不会激活触发器。

我们可以为一个给定的表定义多个具有相同触发事件和操作时间的触发器。例如，你可以为一个表有两个BEFORE UPDATE触发器。默认情况下，具有相同触发事件和动作时间的触发器是按照它们的创建顺序激活的。为了影响触发器的顺序，可以指定一个触发器顺序子句，表示FOLLOWS或PRECEDES，并指定一个具有相同触发事件和动作时间的现有触发器的名称。使用FOLLOWS时，新的触发器会在现有的触发器之后激活。使用PRECEDES时，新的触发器在现有的触发器之前激活。

trigger_body是触发器激活时要执行的语句。要执行多个语句，请使用BEGIN ... END复合语句结构。这也使你可以在存储例程中使用允许的相同语句。参见章节13.6.1, "[BEGIN ... END复合语句](https://dev.mysql.com/doc/refman/8.0/en/begin-end.html)"。有些语句在触发器中是不允许的；参见章节25.8, "[对存储程序的限制](https://dev.mysql.com/doc/refman/8.0/en/stored-program-restrictions.html)"。

在触发器主体中，可以通过使用别名OLD和NEW来引用主题表中的列（与触发器相关的表）。OLD.col_name指的是在更新或删除前的现有记录的列。NEW.col_name指的是要插入的新行或更新后的现有行的列。

触发器不能使用NEW.col_name或者使用OLD.col_name来引用生成的列。关于生成的列的信息，见第13.1.20.8节，"[CREATE TABLE和生成的列](https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html)"。

当触发器被创建时，MySQL存储了有效的[sql_mode](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_mode)系统变量设置，并且总是在该设置有效的情况下执行触发器主体，无论触发器开始执行时当前的服务器SQL模式如何。

DEFINER子句指定了在触发器激活时检查访问权限时要使用的MySQL账户。如果DEFINER子句存在，用户值应该是指定为'user_name'@'host_name'、[CURRENT_USER](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_current-user)或CURRENT_USER()的MySQL账户。允许的用户值取决于你所拥有的权限，如第25.6节 "[存储对象访问控制](https://dev.mysql.com/doc/refman/8.0/en/stored-objects-security.html)"中所讨论的。关于触发器安全的其他信息，也请参见该章节。

如果省略了DEFINER子句，默认定义者是执行CREATE TRIGGER语句的用户。这与明确指定DEFINER = CURRENT_USER相同。

MySQL在检查触发器权限时考虑到DEFINER用户，如下所示。

- 在CREATE TRIGGER时，发布该语句的用户必须有TRIGGER权限。

- 在触发器激活时，权限是根据DEFINER用户检查的。这个用户必须拥有这些权限。

  - 主题表的TRIGGER权限。

  - 如果在触发器主体中使用OLD.col_name或NEW.col_name对表列进行引用，那么主表的SELECT权限。

  - 如果表列是触发器主体中SET NEW.col_name = value分配的目标，那么主表的UPDATE权限。

  - 触发器所执行的语句通常需要的其他权限。

在一个触发器主体中，CURRENT_USER函数返回在触发器激活时用于检查权限的账户。这是DEFINER用户，而不是其行为导致触发器被激活的用户。关于触发器中用户审计的信息，参见章节6.2.23, "基于SQL的账户活动审计"。

如果你使用LOCK TABLES来锁定一个有触发器的表，那么在触发器中使用的表也会被锁定，这在LOCK TABLES和触发器中有所描述。

关于触发器使用的其他讨论，请参见第25.3.1节，"[触发器语法和示例](https://dev.mysql.com/doc/refman/8.0/en/account-activity-auditing.html)"。

如果使用LOCK TABLES锁定具有触发器的表，则触发器中使用的表也会被锁定，如LOCK TALES和触发器中所述。
