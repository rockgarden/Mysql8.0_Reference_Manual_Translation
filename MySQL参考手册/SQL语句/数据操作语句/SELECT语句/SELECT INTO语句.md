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

varlist命名一个或多个变量的列表，每个变量都可以是用户定义的变量、存储过程或函数参数或存储程序局部变量。（在准备好的SELECT…INTO var_list语句中，只允许使用用户定义的变量；请参阅第13.6.4.2节“局部变量范围和分辨率”。）

选定的值被分配给变量。变量数必须与列数匹配。查询应返回一行。如果查询未返回任何行，则会出现错误代码为1329的警告（无数据），变量值保持不变。如果查询返回多行，则会发生错误1172（结果由多行组成）。如果语句可能检索多行，可以使用LIMIT1将结果集限制为一行。
