# WITH（公共表表达式）

公共表表达式（CTE）是一个命名的临时结果集，它存在于单个语句的范围内，稍后可以在该语句中引用，可能多次引用。下面的讨论描述了如何编写使用CTE的语句。

公共表表达式

递归公共表表达式

限制公共表表达式递归

递归公共表表达式示例

与类似构造相比的公共表表达式

有关CTE优化的信息，请参见第8.2.2.4节“通过合并或实体化优化派生表、视图引用和公共表表达式”。

## 其他资源

这些文章包含有关在MySQL中使用CTE的其他信息，包括许多示例：

- MySQL 8.0实验室：[[递归]MySQL中的公共表表达式（CTE）](https://dev.mysql.com/blog-archive/mysql-8-0-labs-recursive-common-table-expressions-in-mysql-ctes/)

- MySQL 8.0实验室：[[递归]MySQL中的通用表表达式（CTE），第二部分——如何生成系列](https://dev.mysql.com/blog-archive/mysql-8-0-labs-recursive-common-table-expressions-in-mysql-ctes-part-two-how-to-generate-series/)

- MySQL 8.0实验室：[[递归]MySQL中的通用表表达式（CTE），第三部分-层次结构](https://dev.mysql.com/blog-archive/mysql-8-0-labs-recursive-common-table-expressions-in-mysql-ctes-part-three-hierarchies/)

- MySQL 8.0.1:[[递归]MySQL中的通用表表达式（CTE），第四部分——深度优先或宽度优先遍历、传递闭包、循环避免](https://dev.mysql.com/blog-archive/mysql-8-0-1-recursive-common-table-expressions-in-mysql-ctes-part-four-depth-first-or-breadth-first-traversal-transitive-closure-cycle-avoidance/)

## 公共表表达式

要指定公共表表达式，请使用具有一个或多个逗号分隔的子子句的WITH子句。每个子子句提供一个子查询，该子查询生成一个结果集，并将名称与子查询相关联。下面的示例在WITH子句中定义了名为cte1和cte2的CTE，并在WITH子句后面的顶级SELECT中引用它们：

```sql
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)
SELECT b, d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```

在包含WITH子句的语句中，可以引用每个CTE名称来访问相应的CTE结果集。

可以在其他CTE中引用CTE名称，从而可以基于其他CTE定义CTE。

CTE可以引用自身来定义递归CTE。递归CTE的常见应用包括分层或树结构数据的序列生成和遍历。

公共表表达式是DML语句语法的可选部分。它们使用WITH子句定义：

```sql
with_clause:
    WITH [RECURSIVE]
        cte_name [(col_name [, col_name] ...)] AS (subquery)
        [, cte_name [(col_name [, col_name] ...)] AS (subquery)] ...
```

cte_name命名一个通用表表达式，可以用作包含WITH子句的语句中的表引用。

AS（子查询）的子查询部分称为“CTE的子查询”，是生成CTE结果集的部分。AS后面的括号是必需的。

如果公共表表达式的子查询引用其自己的名称，则该表达式是递归的。如果WITH子句中的任何CTE是递归的，则必须包含RECURSIVE关键字。有关详细信息，请参见递归公共表表达式。

给定CTE的列名确定如下：

- 如果CTE名称后面有带括号的名称列表，则这些名称为列名：

    ```sql
    WITH cte (col1, col2) AS
    (
    SELECT 1, 2
    UNION ALL
    SELECT 3, 4
    )
    SELECT col1, col2 FROM cte;
    ```

  列表中的名称数必须与结果集中的列数相同。

- 否则，列名来自AS（子查询）部分中第一个select的选择列表：

    ```sql
    WITH cte AS
    (
    SELECT 1 AS col1, 2 AS col2
    UNION ALL
    SELECT 3, 4
    )
    SELECT col1, col2 FROM cte;
    ```

在以下上下文中允许使用WITH子句：

- 在SELECT、UPDATE和DELETE语句的开头。

    ```sql
    WITH ... SELECT ...
    WITH ... UPDATE ...
    WITH ... DELETE ...
    ```

- 在子查询（包括派生表子查询）开始时：

    ```sql
    SELECT ... WHERE id IN (WITH ... SELECT ...) ...
    SELECT * FROM (WITH ... SELECT ...) AS dt ...
    ```

- 对于包含SELECT语句的语句，紧邻SELECT前面：

    ```sql
    INSERT ... WITH ... SELECT ...
    REPLACE ... WITH ... SELECT ...
    CREATE TABLE ... WITH ... SELECT ...
    CREATE VIEW ... WITH ... SELECT ...
    DECLARE CURSOR ... WITH ... SELECT ...
    EXPLAIN ... WITH ... SELECT ...
    ```

同一级别只允许一个WITH子句。不允许WITH后跟WITH处于同一级别，因此这是非法的：

`WITH cte1 AS (...) WITH cte2 AS (...) SELECT ...`

要使语句合法，请使用一个WITH子句，用逗号分隔子句：

`WITH cte1 AS (...), cte2 AS (...) SELECT ...`

但是，一个语句可以包含多个WITH子句，如果它们出现在不同的级别：

```sql
WITH cte1 AS (SELECT 1)
SELECT * FROM (WITH cte2 AS (SELECT 2) SELECT * FROM cte2 JOIN cte1) AS dt;
```

WITH子句可以定义一个或多个公共表表达式，但每个CTE名称必须对该子句唯一。这是非法的：

`WITH cte1 AS (...), cte1 AS (...) SELECT ...`

要使声明合法，请使用唯一名称定义CTE：

`WITH cte1 AS (...), cte2 AS (...) SELECT ...`

CTE可以指自身或其他CTE：

- 自引用CTE是递归的。

- CTE可以引用先前在同一WITH子句中定义的CTE，但不能引用稍后定义的那些CTE。

  此约束排除了相互递归的CTE，其中cte1引用cte2，cte2引用cte1。其中一个引用必须指向稍后定义的CTE（这是不允许的）。

- 给定查询块中的CTE可以指在更外部级别的查询块中定义的CTE，而不是在更内部级别的查询模块中定义的CT。

为了解析对同名对象的引用，派生表隐藏CTE；和CTE隐藏基表、TEMPORARY表和视图。名称解析是通过在同一查询块中搜索对象，然后在没有找到具有名称的对象的情况下依次转到外部块来进行的。

与派生表一样，CTE不能包含MySQL 8.0.14之前的外部引用。这是MySQL 8.00.14中取消的MySQL限制，而不是SQL标准的限制。有关特定于递归CTE的其他语法注意事项，请参见[递归公共表表达式](https://dev.mysql.com/doc/refman/8.0/en/with.html#common-table-expressions-recursive)。
