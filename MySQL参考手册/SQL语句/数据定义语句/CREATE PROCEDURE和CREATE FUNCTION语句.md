# CREATE PROCEDURE 和 CREATE FUNCTION 语句

```sql
CREATE
    [DEFINER = user]
    PROCEDURE [IF NOT EXISTS] sp_name ([proc_parameter[,...]])
    [characteristic ...] routine_body

CREATE
    [DEFINER = user]
    FUNCTION [IF NOT EXISTS] sp_name ([func_parameter[,...]])
    RETURNS type
    [characteristic ...] routine_body

proc_parameter:
    [ IN | OUT | INOUT ] param_name type

func_parameter:
    param_name type

type:
    Any valid MySQL data type

characteristic: {
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
}

routine_body:
    Valid SQL routine statement
```

这些语句用于创建存储例程（存储过程或函数）。也就是说，服务器知道指定的例程。默认情况下，存储的例程与默认数据库关联。要将例程显式地与给定数据库关联，请将名称指定为db_name。创建sp_name时。

CREATE FUNCTION语句在MySQL中也用于支持可加载函数。可加载函数可被视为外部存储函数。存储函数与可加载函数共享其命名空间。有关描述服务器如何解释对不同类型函数的引用的规则，请参见第9.2.5节“[函数名称Parsing和Resolution”,](https://dev.mysql.com/doc/refman/8.0/en/function-resolution.html)”。

要调用存储过程，请使用CALL语句。要调用存储函数，请在表达式中引用它。该函数在表达式求值期间返回一个值。

CREATE PROCEDURE和CREATE FUNCTION需要CREATE ROUTINE权限。如果存在DEFINER子句，所需的权限取决于用户值，如第25.6节“[存储对象访问控制](https://dev.mysql.com/doc/refman/8.0/en/stored-objects-security.html)”所述。如果启用二进制日志记录，CREATE FUNCTION可能需要SUPER权限，如第25.7节“[存储程序二进制日志记录](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html)”所述。

默认情况下，MySQL会自动授予例程创建者ALTER ROUTINE和EXECUTE权限。可以通过禁用automatic_sp_privileges系统变量来更改此行为。参见第25.2.2节，“[存储routines和MySQL特权](https://dev.mysql.com/doc/refman/8.0/en/stored-routines-privileges.html)”。

DEFINER和SQLSECURITY子句指定了在例程执行时检查访问权限时要使用的安全上下文，如本节稍后所述。

如果例程名称与内置SQL函数的名称相同，则会出现语法错误，除非在定义例程或稍后调用例程时在名称和以下括号之间使用空格。因此，请避免在自己的存储例程中使用现有SQL函数的名称。

[IGNORE_SPACE](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_ignore_space) SQL模式适用于内置函数，而不是存储例程。无论是否启用IGNORE_SPACE，都允许在存储的例程名称后使用空格。

如果不存在，如果已经存在同名的例程，则可以防止发生错误。从MySQL 8.0.29开始，CREATE FUNCTION和CREATE PROCEDURE都支持此选项。

如果已经存在同名的内置函数，尝试使用create function…If NOT exists创建存储函数会成功，并显示一条警告，表明它与本机函数同名；这与执行相同的CREATE FUNCTION语句而不指定IF NOT EXISTS时没有区别。

如果已经存在同名的可加载函数，尝试使用If NOT exists创建存储函数会成功，并显示警告。这与不指定IF NOT EXISTS的情况相同。

有关详细信息，请参见[函数名称解析](https://dev.mysql.com/doc/refman/8.0/en/function-resolution.html#function-name-resolution)。

括号内的参数列表必须始终存在。如果没有参数，则应使用空参数列表（）。参数名称不区分大小写。

默认情况下，每个参数都是IN参数。若要为参数指定其他内容，请在参数名称之前使用关键字OUT或INOUT。

> 笔记
将参数指定为IN、OUT或INOUT仅对PROCEDURE有效。对于FUNCTION，参数始终被视为IN参数。

IN参数将值传递到过程中。过程可能会修改该值，但当过程返回时，调用方无法看到该修改。OUT参数将过程中的值传递回调用者。它的初始值在过程中为NULL，当过程返回时，调用方可以看到它的值。INOUT参数由调用方初始化，可以由过程修改，当过程返回时，调用方可以看到过程所做的任何更改。

对于每个OUT或INOUT参数，在调用过程的CALL语句中传递用户定义的变量，以便在过程返回时获得其值。如果要从另一个存储过程或函数中调用该过程，还可以将例程参数或本地例程变量作为OUT或INOUT参数传递。如果您从触发器内调用该过程，还可以传递NEW。col_name作为OUT或INOUT参数。

有关未处理条件对程序参数的影响的信息，请参见第13.6.7.8节“[条件处理和OUT或INOUT参数](https://dev.mysql.com/doc/refman/8.0/en/conditions-and-parameters.html)”。

例程参数不能在例程内准备的语句中引用；参见第25.8节“[存储程序的限制](https://dev.mysql.com/doc/refman/8.0/en/stored-program-restrictions.html)”。

下面的示例显示了一个简单的存储过程，在给定国家代码的情况下，该存储过程统计出现在世界数据库的城市表中的该国家的城市数量。使用IN参数传递国家代码，使用OUT参数返回城市计数：

```sql
mysql> delimiter //

mysql> CREATE PROCEDURE citycount (IN country CHAR(3), OUT cities INT)
       BEGIN
         SELECT COUNT(*) INTO cities FROM world.city
         WHERE CountryCode = country;
       END//
Query OK, 0 rows affected (0.01 sec)

mysql> delimiter ;

mysql> CALL citycount('JPN', @cities); -- cities in Japan
Query OK, 1 row affected (0.00 sec)

mysql> SELECT @cities;
+---------+
| @cities |
+---------+
|     248 |
+---------+
1 row in set (0.00 sec)

mysql> CALL citycount('FRA', @cities); -- cities in France
Query OK, 1 row affected (0.00 sec)

mysql> SELECT @cities;
+---------+
| @cities |
+---------+
|      40 |
+---------+
1 row in set (0.00 sec)
```

该示例使用mysql客户端分隔符命令将语句分隔符从；在定义过程时。这使得：；过程主体中使用的分隔符将传递给服务器，而不是由mysql本身解释。参见第25.1节“[定义存储程序](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-defining.html)”。

只能为FUNCTION指定RETURNS子句，该子句是强制的。它指示函数的返回类型，函数体必须包含return值语句。如果[RETURN](https://dev.mysql.com/doc/refman/8.0/en/return.html)语句返回不同类型的值，则该值将强制为正确的类型。例如，如果函数在RETURNS子句中指定ENUM或SET值，但RETURN语句返回一个整数，则函数返回的值是[SET](https://dev.mysql.com/doc/refman/8.0/en/set.html)成员集合中对应[ENUM](https://dev.mysql.com/doc/refman/8.0/en/enum.html)成员的字符串。

下面的示例函数接受参数，使用SQL函数执行操作，并返回结果。在这种情况下，没有必要使用分隔符，因为函数定义不包含内部；语句分隔符：

```sql
mysql> CREATE FUNCTION hello (s CHAR(20))
mysql> RETURNS CHAR(50) DETERMINISTIC
       RETURN CONCAT('Hello, ',s,'!');
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT hello('world');
+----------------+
| hello('world') |
+----------------+
| Hello, world!  |
+----------------+
1 row in set (0.00 sec)
```

参数类型和函数返回类型可以声明为使用任何有效的数据类型。如果前面有CHARACTER SET规范，则可以使用COLLATE属性。

routine_body由有效的SQL例程语句组成。这可以是SELECT或INSERT等简单语句，也可以是使用BEGIN和END编写的复合语句。复合语句可以包含声明、循环和其他控制结构语句。这些语句的语法见第13.6节“[复合语句语法](https://dev.mysql.com/doc/refman/8.0/en/sql-compound-statements.html)”。实际上，存储函数倾向于使用复合语句，除非主体由单个RETURN语句组成。

MySQL允许例程包含DDL语句，如CREATE和DROP。MySQL还允许存储过程（但不允许存储函数）包含诸如[COMMIT](https://dev.mysql.com/doc/refman/8.0/en/commit.html)之类的SQL事务语句。存储函数不能包含执行显式或隐式提交或回滚的语句。SQL标准不要求支持这些语句，该标准规定每个DBMS供应商可以决定是否允许这些语句。

返回结果集的语句可以在存储过程中使用，但不能在存储函数中使用。此禁止包括没有INTO var_list子句的SELECT语句以及其他语句，如SHOW、EXPLAIN和[CHECK TABLE](https://dev.mysql.com/doc/refman/8.0/en/check-table.html)。对于可以在函数定义时确定返回结果集的语句，将出现`a Not allowed to return a result set from a function`的错误（[ER_SP_NO_RETSET](https://dev.mysql.com/doc/mysql-errors/8.0/en/server-error-reference.html#error_er_sp_no_retset)）。对于只能在运行时确定返回结果集的语句，PROCEDURE发生`%s can't return a result set in the given context`错误（[ER_SP_BADSELECT](https://dev.mysql.com/doc/mysql-errors/8.0/en/server-error-reference.html#error_er_sp_badselect)）。

存储例程中不允许使用[USE](https://dev.mysql.com/doc/refman/8.0/en/use.html)语句。调用例程时，将执行隐式USE db_name（并在例程终止时撤消）。导致例程在执行时具有给定的默认数据库。对例程默认数据库以外的数据库中的对象的引用应使用适当的数据库名称进行限定。

有关存储例程中不允许的语句的其他信息，请参阅第25.8节“[存储程序的限制](https://dev.mysql.com/doc/refman/8.0/en/stored-program-restrictions.html)”。

有关从具有MySQL接口的语言编写的程序中调用存储过程的信息，请参阅 “CALL语句”。

MySQL存储在创建或更改例程时生效的[sql_mode](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_mode)系统变量设置，并始终在该设置生效的情况下执行例程，而不管例程开始执行时的当前服务器sql模式如何（**regardless of the current server SQL mode when the routine begins executing**）。

从调用程序的SQL模式切换到例程的SQL模式发生在对参数进行求值并将结果值分配给例程参数之后。如果您在严格SQL模式下定义例程，但在非严格模式下调用它，则不会在严格模式下将参数分配给例程参数。如果您要求以严格的SQL模式分配传递给例程的表达式，那么您应该以严格的模式调用例程。

COMMENT特性是一个MySQL扩展，可以用来描述存储的例程。此信息由[SHOW CREATE PROCEDURE](https://dev.mysql.com/doc/refman/8.0/en/show-create-procedure.html)和[SHOW CLEATE FUNCTION](https://dev.mysql.com/doc/refman/8.0/en/show-create-function.html)语句显示。

LANGUAGE（语言）特性表示编写例程的语言。服务器忽略此特性；仅支持SQL例程。

如果一个例程总是为相同的输入参数产生相同的结果，那么它被认为是“确定性的(deterministic)”，否则它就“不确定性的”。如果例程定义中既没有给出DETERMINISTIC也没有给出`NOT DETERMNISTIC`，则默认值为NOT DEDERMINISIC。要声明函数是确定性的，必须显式指定deterministic。

对例程性质的评估基于创建者的“诚实”：MySQL不会检查声明为DETERMINISTIC的例程是否没有产生不确定结果的语句。然而，错误声明例程可能会影响结果或影响性能。将非确定性例程声明为DETERMINISTIC可能会导致优化器做出错误的执行计划选择，从而导致意外结果。将确定性例程声明为NONDETERMINISTIC可能会导致不使用可用的优化，从而降低性能。

如果启用了二进制日志记录，DETERMINISTIC特性将影响MySQL接受的例程定义。参见第25.7节，“[存储程序二进制记录](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html)”。

包含NOW()函数（或其同义词）或RAND()的例程是不确定的，但它可能仍然是复制安全的。对于NOW()，二进制日志包含时间戳并正确复制。只要在例程执行期间只调用一次，RAND()也可以正确复制。（您可以将例程执行时间戳和随机数种子视为源和副本上相同的隐式输入。）

一些特征提供了有关例程使用数据的性质的信息。在MySQL中，这些特性仅供参考。服务器不使用它们来约束例程允许执行的语句类型。

- CONTAINS SQL表示例程不包含读取或写入数据的语句。如果没有明确给出这些特征，则这是默认值。这类语句的示例是`SET @x=1`或`DO RELEASE_LOCK('abc')`，它们执行但既不读也不写数据。

- NO SQL表示例程不包含SQL语句。

- READS SQL DATA表示例程包含读取数据的语句（例如，SELECT），但不包含写入数据的语句。

- MODIFIES SQL DATA表示例程包含可能写入数据的语句（例如，INSERT或DELETE）。

SQL安全特性可以是DEFINER或INVOKER来指定安全上下文；也就是说，例程是使用例程DEFINER子句中指定的帐户的权限还是使用调用它的用户执行。此帐户必须具有访问与例程关联的数据库的权限。默认值为DEFINER。调用例程的用户必须具有该例程的EXECUTE特权，如果例程在定义器安全上下文中执行，则DEFINER帐户也必须具有该特权。

DEFINER子句指定在例程执行时检查具有SQL SECURITY DEFINER特性的例程的访问权限时要使用的MySQL帐户。

如果存在DEFINER子句，则用户值应该是指定为“user_name”@“host_name”、“[CURRENT_user](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_current-user)”或“CURRENT_user()”的MySQL帐户。允许的用户值取决于您拥有的特权，如第25.6节“[存储对象访问控制](https://dev.mysql.com/doc/refman/8.0/en/stored-objects-security.html)”所述。有关存储例程安全性的其他信息，请参阅该部分。

如果省略DEFINER子句，则默认定义者是执行CREATE PROCEDURE或CREATE FUNCTION语句的用户。这与显式指定DEFINER=CURRENT_USER相同。

在使用SQL SECURITY DEFINER特性定义的存储例程的主体中，CURRENT_USER函数返回例程的DEFINER值。有关存储例程中用户审核的信息，请参阅第6.2.23节“[基于SQL的帐户活动审核](https://dev.mysql.com/doc/refman/8.0/en/account-activity-auditing.html)”。

考虑下面的过程，它显示了MySQL中列出的MySQL帐户的数量。用户系统表：

```sql
CREATE DEFINER = 'admin'@'localhost' PROCEDURE account_count()
BEGIN
  SELECT 'Number of accounts:', COUNT(*) FROM mysql.user;
END;
```

无论哪个用户定义该过程，都会为该过程分配一个名为“admin”@“localhost”的DEFINER帐户。无论哪个用户调用它，它都以该帐户的权限执行（因为默认的安全特性是DEFINER）。该过程的成功或失败取决于调用程序是否具有对它的EXECUTE特权，以及“admin”@“localhost”是否具有对mysql的SELECT特权。用户表。

现在假设该过程是用SQL SECURITY INVOKER特性定义的：

```sql
CREATE DEFINER = 'admin'@'localhost' PROCEDURE account_count()
SQL SECURITY INVOKER
BEGIN
  SELECT 'Number of accounts:', COUNT(*) FROM mysql.user;
END;
```

该过程的DEFINER仍然为“admin”@“localhost”，但在本例中，它以调用用户的权限执行。因此，该过程的成功或失败取决于调用程序是否具有对它的EXECUTE特权和对mysql的SELECT特权。用户表。

默认情况下，当执行具有SQL SECURITY DEFINER特性的例程时，MySQL Server不会为DEFINER子句中指定的MySQL帐户设置任何活动角色，只设置默认角色。例外情况是，如果启用了[activate_all_roles_on_login](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_activate_all_roles_on_login)系统变量，那么MySQL Server将设置授予DEFINER用户的所有角色，包括强制角色。因此，在发出CREATE PROCEDURE或CREATE FUNCTION语句时，默认情况下不会检查通过角色授予的任何特权。对于存储的程序，如果执行的角色与默认角色不同，程序主体可以执行[SET ROLE](https://dev.mysql.com/doc/refman/8.0/en/set-role.html)来激活所需的角色。必须小心操作，因为分配给角色的权限可以更改。

服务器处理例程参数、使用DECLARE创建的本地例程变量或函数返回值的数据类型，如下所示：

- 检查分配是否存在数据类型不匹配和溢出。在严格的SQL模式下，转换和溢出问题会导致警告或错误。

- 只能指定标量值。例如，SET x = (SELECT 1, 2)等语句无效。

- 对于字符数据类型，如果声明中包含character SET，则使用指定的字符集及其默认排序规则。如果COLLATE属性也存在，则使用该排序规则而不是默认排序规则。

  如果CHARACTER SET和COLLATE不存在，则使用在例程创建时有效的数据库字符集和排序规则。为了避免服务器使用数据库字符集和排序规则，请为字符数据参数提供显式的字符集和COLLATE属性。

  如果更改了数据库默认字符集或排序规则，则必须删除并重新创建要使用新数据库默认值的存储例程。

  数据库字符集和排序规则由[character_set_database](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_character_set_database)和[collation_database](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_collation_database)系统变量的值给出。有关更多信息，请参阅第10.3.3节“[数据库字符集和排序规则](https://dev.mysql.com/doc/refman/8.0/en/charset-database.html)”。

## 存储过程示例

### 传入数组

将数组或项列表（字符串、整数等）传递给存储过程，以便在in运算符中使用。

MySQL没有任何类型Array或List，所以我们需要找到一种解决方法：

- 数组转为字符串（其中每个值用指定分隔符分隔），然后传入存储过程，最后将数组字符串与查询语句的其余部分拼接并执行。
  
    ```sql
    CREATE PROCEDURE `ArrayInDemo`(list_of_ids VARCHAR(1000)) 
    test_proc :
    BEGIN  
    -- # preliminary check
    IF length(list_of_ids) > 1000 THEN
        SELECT 'wrong parameter' as `error`;
        leave test_proc;
    END IF;

    set @sql = concat("UPDATE `TABLE` SET `Column` = 'value'
                        WHERE `TABLE`.Id in (" , list_of_ids , ")");

        PREPARE stmt FROM @sql;
        EXECUTE stmt;
    END; 
    ```

