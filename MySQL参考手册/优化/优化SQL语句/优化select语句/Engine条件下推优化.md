# Engine条件下推优化

这种优化提高了无索引列和常量之间直接比较的效率。在这种情况下，条件被“下推”到存储引擎进行评估。此优化只能由NDB存储引擎使用。

对于NDB Cluster，此优化可以消除在集群的数据节点和发出查询的MySQL服务器之间通过网络发送不匹配行的需要，并可以在可以使用但未使用条件下推的情况下，将使用查询的查询速度提高5到10倍。

假设NDB Cluster表定义如下：

```sql
CREATE TABLE t1 (
    a INT,
    b INT,
    KEY(a)
) ENGINE=NDB;
```

引擎条件下推可用于此处显示的查询，其中包括无索引列和常量之间的比较：

`SELECT a, b FROM t1 WHERE b = 10;`

在EXPLAIN的输出中可以看到发动机条件下压的使用：

```sql
mysql> EXPLAIN SELECT a, b FROM t1 WHERE b = 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
        Extra: Using where with pushed condition
```

但是，引擎条件下推不能用于以下查询：

`SELECT a,b FROM t1 WHERE a = 10;`

引擎条件下推不适用于此处，因为列a上存在索引。（索引访问方法将更有效，因此将优先于条件下推。）

当使用>或<运算符将索引列与常量进行比较时，也可以使用发动机状态下推：

```sql
mysql> EXPLAIN SELECT a, b FROM t1 WHERE a < 2\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: range
possible_keys: a
          key: a
      key_len: 5
          ref: NULL
         rows: 2
        Extra: Using where with pushed condition
```

发动机状况下推的其他支持比较包括：

- `column [NOT] LIKE pattern`

  模式必须是包含要匹配的模式的字符串文字；有关语法，请参阅第12.8.1节“[字符串比较函数和运算符](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html)”。

- column IS [NOT] NULL

- column IN (value_list)

  value_list中的每个项都必须是常量文本值。

- column BETWEEN constant1 AND constant2

  constant1和constant2都必须是常量，文字值。

默认情况下启用发动机状况下推。要在服务器启动时禁用它，请将optimizer_switch系统变量的engine_condition_pushdown标志设置为off。例如，在my.cnf文件中，使用以下行：

```config
[mysqld]
optimizer_switch=engine_condition_pushdown=off
```

运行时，禁用条件下推如下：

`SET optimizer_switch='engine_condition_pushdown=off';`

局限性. Engine 条件下推受到以下限制：

- 只有NDB存储引擎支持引擎条件下推。

- 在NDB 8.0.18之前，可以将列与仅计算为常量值的常量或表达式进行比较。在NDB 8.0.18及更高版本中，只要列的类型完全相同，包括相同的符号、长度、字符集、精度和比例（如果适用），就可以相互比较。

- 比较中使用的列不能是任何BLOB或TEXT类型。此排除也扩展到JSON、[BIT](https://dev.mysql.com/doc/refman/8.0/en/bit-type.html)和[ENUM](https://dev.mysql.com/doc/refman/8.0/en/enum.html)列。

- 要与列进行比较的字符串值必须使用与列相同的排序规则。

- 不直接支持连接；如果可能，涉及多个表的条件将被单独推送。使用扩展的EXPLAIN输出确定哪些条件实际上被下推。

以前，发动机条件下推仅限于引用条件被推到的同一表中的列值的术语。从NDB 8.0.16开始，也可以从推式条件引用查询计划中前面表中的列值。这减少了连接处理期间SQL节点必须处理的行数。筛选也可以在LDM线程中并行执行，而不是在单个mysqld进程中执行。这有可能显著提高查询性能。

从NDB 8.0.20开始，如果在同一联接嵌套中使用的任何表，或者在其上面的联接嵌套中它所依赖的任何表上都没有不可推送的条件，则可以推送使用扫描的外部联接。如果采用的优化策略是firstMatch，则半联接也是如此。

在以下两种情况下，联接算法不能与引用前一个表中的列相结合：

1. 当前面提到的任何表位于联接缓冲区中时。在这种情况下，从扫描筛选表中检索的每一行都与缓冲区中的每一行都匹配。这意味着在生成扫描筛选器时，不存在可以从中提取列值的单个特定行。

2. 当列源自推式联接中的子操作时。这是因为在生成扫描筛选器时，尚未检索到联接中祖先操作引用的行。

从NDB 8.0.27开始，连接中祖先表的列可以下推，前提是它们满足前面列出的要求。下面显示了使用前面创建的表t1进行查询的示例：

```sql
mysql> EXPLAIN 
    ->   SELECT * FROM t1 AS x 
    ->   LEFT JOIN t1 AS y 
    ->   ON x.a=0 AND y.b>=3\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: x
   partitions: p0,p1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: y
   partitions: p0,p1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using where; Using pushed condition (`test`.`y`.`b` >= 3); Using join buffer (hash join)
2 rows in set, 2 warnings (0.00 sec)
```
