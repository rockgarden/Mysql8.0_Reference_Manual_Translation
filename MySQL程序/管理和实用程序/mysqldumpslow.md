# mysqldumpslow — 总结慢查询日志文件

MySQL 慢查询日志包含有关需要很长时间执行的查询的信息（“[慢查询日志](https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html)”）。 mysqldumpslow 解析 MySQL 慢查询日志文件并总结它们的内容。

通常，mysqldumpslow 对除了数字和字符串数据值的特定值之外相似的查询进行分组。 在显示摘要输出时，它将这些值“抽象”为 N 和“S”。 要修改值抽象行为，请使用 -a 和 -n 选项。

像这样调用 mysqldumpslow：

`mysqldumpslow [options] [log_file ...]`

没有给出参数选项options的示例输出：

```txt
Reading mysql slow query log from /usr/local/mysql/data/mysqld80-slow.log
Count: 1  Time=4.32s (4s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t2 select * from t1

Count: 3  Time=2.53s (7s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t2 select * from t1 limit N

Count: 3  Time=2.13s (6s)  Lock=0.00s (0s)  Rows=0.0 (0), root[root]@localhost
 insert into t1 select * from t1
```

mysqldumpslow 支持以下选项。

表 4.23 mysqldumpslow 选项

| Option Name | Description                                         |
|-------------|-----------------------------------------------------|
| -a          | 不要将所有数字抽象为 N 并将字符串抽象为 'S' |
| -n          | Abstract numbers with at least the specified digits |
| --debug     | Write debugging information                         |
| -g          | 只考虑匹配 (grep-style) 模式的查询。    |
| --help      | 显示帮助信息并退出                      |
| -h          | Host name of the server in the log file name        |
| -i          |服务器实例的名称（如果使用 mysql.server 启动脚本）               |
| -l          | 不要从总时间中减去锁定时间。         |
| -r          | 颠倒排序顺序。                             |
| -s          | How to sort output                                  |
| -t          | Display only first num queries                      |
| --verbose   | Verbose mode                                        |

--debug, -d

在调试模式下运行。

仅当 MySQL 使用 WITH_DEBUG 构建时，此选项才可用。 Oracle 提供的 MySQL 发布二进制文件不是使用此选项构建的。

 -h host

生成 `*-slow.log` 文件名的 MySQL 服务器主机名。
该值可以包含通配符。
默认值为 *（匹配所有）

 -n N

名称中至少包含 N 个数字的抽象数字

 -s sort_type

如何对输出进行排序。 sort_type 的值应从以下列表中选择：

- t, at：按查询时间或平均查询时间排序
- l, al：按锁定时间或平均锁定时间排序
- r, ar：按发送的行数或发送的平均行数排序
- c：按计数排序

默认情况下，mysqldumpslow 按平均查询时间排序（相当于 -s at）。

 -t N

仅显示输出中的前 N ​​个查询。

 --verbose，-v

详细模式。打印有关程序功能的更多信息。
