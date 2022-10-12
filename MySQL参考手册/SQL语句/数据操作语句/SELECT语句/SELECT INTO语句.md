# SELECT…INTO 语句

SELECT的SELECT…INTO形式允许将查询结果存储在变量中或写入文件：

- SELECT…INTO var_list选择列值并将其存储到变量中。

- SELECT…INTO OUTFILE将所选行写入文件。可以指定列和行终止符来生成特定的输出格式。

- SELECT…INTO DUMPFILE将单行写入文件，而不进行任何格式设置。

给定的SELECT语句最多可以包含一个INTO子句，尽管如SELECT语法描述所示（参见第13.2.10节“[SELECT statement](https://dev.mysql.com/doc/refman/8.0/en/select.html)”），INTO可以出现在不同的位置：

- 在FROM之前。例子：
  `SELECT * INTO @myvar FROM t1;`

- 在尾部锁定子句之前。例子：
  `SELECT * FROM t1 INTO @myvar FOR UPDATE;`

- 在SELECT结束时。例子：
  `SELECT * FROM t1 FOR UPDATE INTO @myvar;`

MySQL 8.0.20支持语句末尾的INTO位置，这是首选位置。从MySQL 8.0.20开始，锁定子句之前的位置已弃用；预计在未来的MySQL版本中将删除对它的支持。换句话说，FROM后面但不是SELECT末尾的INTO会产生警告。

INTO子句不应在嵌套SELECT中使用，因为这样的SELECT必须将其结果返回给外部上下文。在UNION语句中使用INTO也有限制；见第13.2.10.3节，“[联合分句](https://dev.mysql.com/doc/refman/8.0/en/union.html)”。

对于INTO var_list变量：

- var_list 命名一个或多个变量的列表，每个变量都可以是用户定义的变量、存储过程或函数参数或存储程序局部变量。（在准备好的SELECT…INTO var_list语句中，只允许使用用户定义的变量；请参阅第13.6.4.2节“[Local Variable Scope and Resolution](https://dev.mysql.com/doc/refman/8.0/en/local-variable-scope.html)”。）

- 选定的值被分配给变量。变量数必须与列数匹配。查询应返回一行。如果查询未返回任何行，则会出现错误代码为1329的警告（无数据），变量值保持不变。如果查询返回多行，则会发生错误1172（结果由多行组成）。如果语句可能检索多行，可以使用LIMIT1将结果集限制为一行。

  `SELECT id, data INTO @x, @y FROM test.t1 LIMIT 1;`

INTO var_list也可以与TABLE语句一起使用，但受以下限制：

- 变量数必须与表中的列数匹配。

- 如果表包含多行，则必须使用LIMIT 1将结果集限制为单行。LIMIT 1必须位于INTO关键字之前。

下面显示了这样一个语句的示例：

```sql
TABLE employees ORDER BY lname DESC LIMIT 1
    INTO @id, @fname, @lname, @hired, @separated, @job_code, @store_id;
```

您还可以从values语句中选择值，该语句将一行生成一组用户变量。在这种情况下，必须使用表别名，并且必须将值列表中的每个值分配给变量。这里显示的两个语句都等价于 [SET @x=2, @y=4, @z=8](https://dev.mysql.com/doc/refman/8.0/en/set-variable.html)：

`SELECT * FROM (VALUES ROW(2,4,8)) AS t INTO @x,@y,@z;`

`SELECT * FROM (VALUES ROW(2,4,8)) AS t(a,b,c) INTO @x,@y,@z;`

用户变量名不区分大小写。参见第9.4节“[用户定义变量](https://dev.mysql.com/doc/refman/8.0/en/user-variables.html)”。

SELECT的[SELECT…INTO OUTFILE“file_name”](https://dev.mysql.com/doc/refman/8.0/en/select-into.html)形式将所选行写入文件。该文件是在服务器主机上创建的，因此您必须具有file权限才能使用此语法。file_name不能是现有文件，这会阻止修改/etc/passwd和数据库表等文件。 [character_set_filesystem](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_character_set_filesystem) 系统变量控制文件名的解释。

SELECT…INTO OUTFILE 语句旨在启用将表转储到服务器主机上的文本文件。要在其他主机上创建结果文件，SELECT…INTO OUTFILE 通常不适用，因为无法写入相对于服务器主机文件系统的文件路径，除非可以使用服务器主机文件体系上的网络映射路径访问远程主机上文件的位置。

或者，如果MySQL客户端软件安装在远程主机上，则可以使用诸如 `MySQL -e “SELECT…” > file_name` 之类的客户端命令在该主机上生成文件。

SELECT…INTO OUTFILE 是 [LOAD DATA](https://dev.mysql.com/doc/refman/8.0/en/load-data.html) 的补充。列值被写入并转换为 character set 子句中指定的字符集。如果不存在这样的子句，则使用二进制字符集转储值。实际上，没有字符集转换。如果结果集包含多个字符集中的列，输出数据文件也会包含这些列，并且可能无法正确重新加载文件。

语句的export_options部分的语法包含与LOAD DATA语句相同的FIELDS和LINES子句。有关FIELDS和LINES子句的信息，包括其默认值和允许值，请参阅第13.2.7节“[LOAD DATA语句](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)”。

字段ESCAPED BY控制如何写入特殊字符。如果FIELDS ESCAPED BY字符不为空，则在必要时将其用作输出中以下字符之前的前缀，以避免歧义：

- 按字符转义的字段  FIELDS ESCAPED BY character

- 字符封装的字段[可选] FIELDS [OPTIONALLY] ENCLOSED BY character

- 字段 TERMINATED BY 和 行 TERMINITED BY值的第一个字符 The first character of the FIELDS TERMINATED BY and LINES TERMINATED BY values

- ASCII NUL（零值字节；转义字符后面实际写的是ASCII 0，而不是零值字节）

The FIELDS TERMINATED BY, ENCLOSED BY, ESCAPED BY, or LINES TERMINATED BY characters must be escaped so that you can read the file back in reliably. ASCII NUL is escaped to make it easier to view with some pagers.

结果文件不需要符合SQL语法，因此不需要转义其他内容。

如果FIELDS ESCAPED BY字符为空，则不会转义任何字符，NULL将输出为NULL，而不是N。指定空转义字符可能不是一个好主意，特别是如果数据中的字段值包含刚刚给出的列表中的任何字符。

当您想要将表格的所有列转储到文本文件中时，INTO OUTFILE还可以与TABLE语句一起使用。在这种情况下，可以使用ORDER BY和LIMIT控制行的顺序和行数；这些子句必须位于INTO OUTFILE之前。TABLE…INTO OUTFILE支持与SELECT…INTO AUTFILE相同的export_options，并且在写入文件系统时也受到相同的限制。下面显示了这样一个语句的示例：

```sql
TABLE employees ORDER BY lname LIMIT 1000
    INTO OUTFILE '/tmp/employee_data_1.txt'
    FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"', ESCAPED BY '\'
    LINES TERMINATED BY '\n';
```

您还可以使用带有VALUES语句的SELECT…INTO OUTFILE将值直接写入文件。示例如下：

```sql
SELECT * FROM (VALUES ROW(1,2,3),ROW(4,5,6),ROW(7,8,9)) AS t
    INTO OUTFILE '/tmp/select-values.txt';
```

必须使用表别名；还支持列别名，并且可以选择仅用于从所需列中写入值。您还可以使用SELECT…INTO OUTFILE支持的任何或所有导出选项将输出格式化为文件。

以下是一个示例，它以逗号分隔值（CSV）格式生成文件，许多程序都使用这种格式：

```sql
SELECT a,b,a+b INTO OUTFILE '/tmp/result.txt'
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  FROM test_table;
```

如果使用INTO DUMPFILE而不是INTO OUTFILE，MySQL将只向文件中写入一行，没有任何列或行终止，也没有执行任何转义处理。这对于选择BLOB值并将其存储在文件中很有用。

TABLE还支持INTO DUMPFILE。如果表包含多行，还必须使用LIMIT 1将输出限制为单行。INTO DUMPFILE还可以与 `SELECT * FROM (VALUES ROW()[, ...]) AS table_alias [LIMIT 1]` 一起使用。见第13.2.14节“[价值语句](https://dev.mysql.com/doc/refman/8.0/en/values.html)”。

> 笔记
由INTO OUTFILE或INTO DUMPFILE创建的任何文件都归运行mysqld的操作系统用户所有。（出于这个和其他原因，您不应该以root用户身份运行mysqld。）从MySQL 8.0.17开始，用于创建文件的umask是0640；您必须具有足够的访问权限才能操作文件内容。在MySQL 8.0.17之前，umask是0666，服务器主机上的所有用户都可以写入该文件。
如果 [secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_secure_file_priv) 系统变量设置为非空目录名，则要写入的文件必须位于该目录中。

在作为事件调度器执行的事件的一部分发生的SELECT…INTO语句的上下文中，诊断消息（不仅错误，而且警告）会写入错误日志，在Windows上，还会写入应用程序事件日志。有关更多信息，请参阅第25.4.5节“[事件调度器状态](https://dev.mysql.com/doc/refman/8.0/en/events-status-info.html)”。

从MySQL 8.0.22开始，通过设置该版本中引入的 [SELECT_INTO_disk_sync](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_select_into_disk_sync) 服务器系统变量，支持 SELECT INTO OUTFILE 和SELECT INTO DUMPFILE写入的输出文件的定期同步。可以分别使用 [select_into_buffer_size](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_select_into_buffer_size) 和 [select_ito_disk_sync_delay](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_select_into_disk_sync_delay) 设置输出缓冲区大小和可选延迟。有关详细信息，请参见这些系统变量的描述。
