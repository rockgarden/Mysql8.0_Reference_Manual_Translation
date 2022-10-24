# TIMESTAMP列中的索引查找

时间值作为UTC值存储在TIMESTAMP列中，插入TIMESTAMP列和从TIMESTAP列检索的值在会话时区和UTC之间进行转换。（这与CONVERT_TZ（）函数执行的转换类型相同。如果会话时区为UTC，则实际上没有时区转换。）

由于当地时区变化的惯例，如夏令时（DST），UTC和非UTC时区之间的转换不是双向的一对一。不同的UTC值在另一个时区可能不不同。以下示例显示了在非UTC时区中变得相同的不同UTC值：

```sql
mysql> CREATE TABLE tstable (ts TIMESTAMP);
mysql> SET time_zone = 'UTC'; -- insert UTC values
mysql> INSERT INTO tstable VALUES
       ('2018-10-28 00:30:00'),
       ('2018-10-28 01:30:00');
mysql> SELECT ts FROM tstable;
+---------------------+
| ts                  |
+---------------------+
| 2018-10-28 00:30:00 |
| 2018-10-28 01:30:00 |
+---------------------+
mysql> SET time_zone = 'MET'; -- retrieve non-UTC values
mysql> SELECT ts FROM tstable;
+---------------------+
| ts                  |
+---------------------+
| 2018-10-28 02:30:00 |
| 2018-10-28 02:30:00 |
+---------------------+
```

> 笔记
要使用命名时区，如“MET”或“欧洲/阿姆斯特丹”，必须正确设置时区表。有关说明，请参阅第5.1.15节“[MySQL Server时区支持](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html)”。

您可以看到，两个不同的UTC值在转换为“MET”时区时是相同的。这种现象可能导致给定TIMESTAMP列查询的不同结果，这取决于优化器是否使用索引来执行查询。

假设查询使用WHERE子句从前面显示的表中选择值，以在ts列中搜索单个特定值，例如用户提供的时间戳文本：

```sql
SELECT ts FROM tstable
WHERE ts = 'literal';
```

进一步假设查询在以下条件下执行：

- 会话时区不是UTC，并且具有DST偏移。例如：

  `SET time_zone = 'MET';`

- 由于DST偏移，[TIMESTAMP](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)列中存储的唯一UTC值在会话时区中不唯一。（前面显示的示例说明了如何发生这种情况。）

- 该查询指定了在会话时区进入DST后一小时内的搜索值。

在这些条件下，WHERE子句中的比较对于非索引和索引查找以不同的方式进行，并导致不同的结果：

- 如果没有索引或优化器无法使用它，则会在会话时区中进行比较。优化器执行表扫描，检索每个ts列值，将其从UTC转换为会话时区，并将其与搜索值（也在会话时区中解释）进行比较：

    ```sql
    mysql> SELECT ts FROM tstable
        WHERE ts = '2018-10-28 02:30:00';
    +---------------------+
    | ts                  |
    +---------------------+
    | 2018-10-28 02:30:00 |
    | 2018-10-28 02:30:00 |
    +---------------------+
    ```

  由于存储的ts值被转换为会话时区，因此查询可以返回两个与UTC值不同但在会话时区中相等的时间戳值：一个值发生在时钟改变时的DST移位之前，另一个值出现在DST移位之后。

- 如果有可用的索引，则以UTC进行比较。优化器执行索引扫描，首先将搜索值从会话时区转换为UTC，然后将结果与UTC索引条目进行比较：

    ```sql
    mysql> ALTER TABLE tstable ADD INDEX (ts);
    mysql> SELECT ts FROM tstable
        WHERE ts = '2018-10-28 02:30:00';
    +---------------------+
    | ts                  |
    +---------------------+
    | 2018-10-28 02:30:00 |
    +---------------------+
    ```

  在这种情况下，（转换的）搜索值仅与索引条目匹配，并且由于不同存储UTC值的索引条目也是不同的，因此搜索值只能与其中一个匹配。

由于非索引和索引查找的优化器操作不同，查询在每种情况下都会产生不同的结果。非索引查找的结果返回会话时区中匹配的所有值。索引查找无法执行此操作：

- 它在存储引擎中执行，该引擎只知道UTC值。

- 对于映射到同一UTC值的两个不同的会话时区值，索引查找仅匹配相应的UTC索引条目，并仅返回一行。

在前面的讨论中，tstable中存储的数据集恰好由不同的UTC值组成。在这种情况下，使用所示形式的查询的所有索引最多匹配一个索引条目。

如果索引不是UNIQUE，则表（和索引）可以存储给定UTC值的多个实例。例如，ts列可能包含UTC值“2018-10-28 00:30:00”的多个实例。在这种情况下，使用查询的索引将返回它们中的每一个（转换为结果集中的MET值“2018-10-28 02:30:00”）。使用索引的查询将转换后的搜索值与UTC索引项中的单个值匹配，而不是与会话时区中转换为搜索值的多个UTC值匹配。

如果返回会话时区中匹配的所有ts值很重要，解决方法是禁止使用带有IGNORE index提示的索引：

```sql
mysql> SELECT ts FROM tstable
       IGNORE INDEX (ts)
       WHERE ts = '2018-10-28 02:30:00';
+---------------------+
| ts                  |
+---------------------+
| 2018-10-28 02:30:00 |
| 2018-10-28 02:30:00 |
+---------------------+
```

在其他上下文中，也会出现两个方向的时区转换缺少一对一映射的情况，例如使用[FROM_UNIXTIME()](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_from-unixtime)和[UNIX_TIMESTAMP()](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_unix-timestamp)函数执行的转换。参见第12.7节“[日期和时间函数](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)”。
