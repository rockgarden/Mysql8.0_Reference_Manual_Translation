# mysqlbinlog 处理二进制日志文件的实用程序

<https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html>

服务器的二进制日志由包含描述数据库内容修改的“事件”的文件组成。服务器以二进制格式写入这些文件。要以文本格式显示它们的内容，请使用 mysqlbinlog 实用程序。您还可以使用 mysqlbinlog 在复制设置中显示由副本服务器写入的中继日志文件的内容，因为中继日志与二进制日志具有相同的格式。二进制日志和中继日志在第 [“二进制日志”](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html) 和第 [“中继日志和复制元数据存储库”](https://dev.mysql.com/doc/refman/8.0/en/replica-logs.html) 中进一步讨论。

像这样调用 mysqlbinlog：

`mysqlbinlog [options] log_file ...`

例如，要显示名为 binlog.000003 的二进制日志文件的内容，请使用以下命令：

`mysqlbinlog binlog.0000003`

输出包括 binlog.000003 中包含的事件。 对于基于语句的日志记录，事件信息包括 SQL 语句、执行它的服务器的 ID、执行语句时的时间戳、花费的时间等等。 对于基于行的日志记录，事件指示行更改而不是 SQL 语句。 有关日志记录模式的信息，请参见第 [“复制格式”](https://dev.mysql.com/doc/refman/8.0/en/replication-formats.html)。

事件前面有提供附加信息的标题注释。 例如：

```txt
# at 141
#100309  9:28:36 server id 123  end_log_pos 245
  Query thread_id=3350  exec_time=11  error_code=0
```

在第一行中，at 后面的数字表示事件在二进制日志文件中的文件偏移量或起始位置。

第二行以日期和时间开头，指示语句何时在事件起源的服务器上开始。 对于复制，此时间戳会传播到副本服务器。 server id 是事件发起的服务器的 server_id 值。 end_log_pos 表示下一个事件的开始位置（即当前事件的结束位置+1）。 thread_id 指示哪个线程执行了该事件。 exec_time 是在复制源服务器上执行事件所花费的时间。 在副本上，它是副本上的结束执行时间减去源上的开始执行时间的差。 差异可作为复制落后于源多少的指标。 error_code 表示执行事件的结果。 零表示没有发生错误。

> 笔记
使用事件组时，可以将事件的文件偏移量归为一组，将事件的注释归为一组。 不要将这些分组事件误认为是空白文件偏移。

windows环境复杂命令示例：

`mysqlbinlog --no-defaults --base64-output=decode-rows -vv --start-datetime="2022-04-14 10:50:00" "D:\Program Files\MySQL8\mysql-8.0.19-winx64\Data\binlog.000128" -r D:\tmp\binlog000128.sql`

mysqlbinlog 的输出可以重新执行（例如，通过将其用作 mysql 的输入）以重做日志中的语句。这对于服务器意外退出后的恢复操作很有用。有关其他使用示例，请参阅本节后面和第 [“时间点（增量）恢复”](https://dev.mysql.com/doc/refman/8.0/en/point-in-time-recovery.html) 中的讨论。要执行 mysqlbinlog 使用的内部使用 BINLOG 语句，用户需要 BINLOG_ADMIN 权限（或已弃用的 SUPER 权限），或 REPLICATION_APPLIER 权限以及执行每个日志事件的适当权限。

您可以使用 mysqlbinlog 直接读取二进制日志文件并将它们应用到本地 MySQL 服务器。您还可以使用 `--read-from-remote-server` 选项从远程服务器读取二进制日志。要读取远程二进制日志，可以给出连接参数选项来指示如何连接到服务器。这些选项是`--host`、`--password`、`--port`、`--protocol`、`--socket` 和`--user`。

当二进制日志文件被加密时，从 MySQL 8.0.14 开始，mysqlbinlog 不能直接读取它们，但可以使用 `--read-from-remote-server` 选项从服务器读取它们。当服务器的 binlog_encryption 系统变量设置为 ON 时，二进制日志文件被加密。 SHOW BINARY LOGS 语句显示特定二进制日志文件是加密的还是未加密的。加密和未加密的二进制日志文件也可以使用加密日志文件（0xFD62696E）的文件头开头的幻数来区分，这与未加密的日志文件（0xFE62696E）不同。请注意，从 MySQL 8.0.14 开始，如果您尝试直接读取加密的二进制日志文件，mysqlbinlog 会返回一个合适的错误，但旧版本的 mysqlbinlog 根本不会将该文件识别为二进制日志文件。有关二进制日志加密的更多信息，请参阅第 [“加密二进制日志文件和中继日志文件”](https://dev.mysql.com/doc/refman/8.0/en/replication-binlog-encryption.html)。

当二进制日志事务有效负载已被压缩时，从 MySQL 8.0.20 开始，该版本的 mysqlbinlog 会自动解压缩和解码事务有效负载，并像未压缩事件一样打印它们。旧版本的 mysqlbinlog 无法读取压缩的事务有效负载。当服务器的 binlog_transaction_compression 系统变量设置为 ON 时，事务有效负载被压缩，然后作为单个事件（Transaction_payload_event）写入服务器的二进制日志文件。使用 `--verbose` 选项，mysqlbinlog 添加注释，说明使用的压缩算法、最初接收的压缩有效负载大小以及解压缩后产生的有效负载大小。

> 笔记
mysqlbinlog 为作为压缩事务有效负载一部分的单个事件声明的结束位置 (end_log_pos) 与原始压缩有效负载的结束位置相同。 因此，多个解压缩事件可以具有相同的结束位置。
如果事务有效负载已经被压缩，mysqlbinlog 自己的连接压缩会减少，但仍对未压缩的事务和标头进行操作。

有关二进制日志事务压缩的更多信息，请参阅第 [“二进制日志事务压缩”](https://dev.mysql.com/doc/refman/8.0/en/binary-log-transaction-compression.html)。

对大型二进制日志运行 mysqlbinlog 时，请注意文件系统有足够的空间用于生成的文件。 要配置 mysqlbinlog 用于临时文件的目录，请使用 TMPDIR 环境变量。

mysqlbinlog 在执行任何 SQL 语句之前将 pseudo_replica_mode 或 pseudo_slave_mode 的值设置为 true。 此系统变量会影响 XA 事务的处理, original_commit_timestamp 复制延迟时间戳和 original_server_version 系统变量以及不受支持的 SQL 模式。

mysqlbinlog 支持以下选项，可以在命令行或选项文件的 mysqlbinlog 和 client 组中指定。 有关 MySQL 程序使用的选项文件的信息，请参阅第 [“使用选项文件”](https://dev.mysql.com/doc/refman/8.0/en/option-files.html)。

如果您需要先修改语句日志（例如，删除由于某种原因不想执行的语句），您也可以将 mysqlbinlog 的输出重定向到文本文件。 编辑文件后，通过将其用作 mysql 程序的输入来执行其中包含的语句：

```bash
mysqlbinlog binlog.000001 > tmpfile
... edit tmpfile ...
mysql -u root -p < tmpfile
```

当使用 `--start-position` 选项调用 mysqlbinlog 时，它仅显示二进制日志中偏移量大于或等于给定位置的那些事件（给定位置必须与一个事件的开始匹配）。 它还具有在看到具有给定日期和时间的事件时停止和启动的选项。 这使您能够使用 `--stop-datetime` 选项执行时间点恢复（例如，可以说“将我的数据库前滚到今天上午 10:30 的状态”）。

**处理多个文件**。 如果您在 MySQL 服务器上执行多个二进制日志，安全的方法是使用与服务器的单个连接来处理它们。 这是一个演示可能不安全的示例：

```bash
mysqlbinlog binlog.000001 | mysql -u root -p # DANGER!!
mysqlbinlog binlog.000002 | mysql -u root -p # DANGER!!
```

如果第一个日志文件包含 CREATE TEMPORARY TABLE 语句并且第二个日志包含使用临时表的语句，那么使用与服务器的多个连接以这种方式处理二进制日志会导致问题。 当第一个 mysql 进程终止时，服务器会删除临时表。 当第二个 mysql 进程尝试使用该表时，服务器报告“未知表”。

为避免此类问题，请使用单个 mysql 进程来执行您要处理的所有二进制日志的内容。 这是一种方法：

`mysqlbinlog binlog.000001 binlog.000002 | mysql -u root -p`

另一种方法是将所有日志写入单个文件，然后处理该文件：

```bash
mysqlbinlog binlog.000001 >  /tmp/statements.sql
mysqlbinlog binlog.000002 >> /tmp/statements.sql
mysql -u root -p -e "source /tmp/statements.sql"
```

从 MySQL 8.0.12 开始，您还可以使用 shell 管道将多个二进制日志文件作为流式输入提供给 mysqlbinlog。 压缩二进制日志文件的存档可以解压缩并直接提供给 mysqlbinlog。 在这个例子中， binlog-files_1.gz 包含多个二进制日志文件进行处理。 管道提取 binlog-files_1.gz 的内容，将二进制日志文件作为标准输入通过管道传输到 mysqlbinlog，并将 mysqlbinlog 的输出通过管道传输到 mysql 客户端执行：

`gzip -cd binlog-files_1.gz | ./mysqlbinlog - | ./mysql -uroot  -p`

您可以指定多个存档文件，例如：

`gzip -cd binlog-files_1.gz binlog-files_2.gz | ./mysqlbinlog - | ./mysql -uroot  -p`

对于流式输入，不要使用 `--stop-position`，因为 mysqlbinlog 无法识别应用此选项的最后一个日志文件。

**加载数据操作**。 mysqlbinlog 可以生成在没有原始数据文件的情况下重现 LOAD DATA 操作的输出。 mysqlbinlog 将数据复制到一个临时文件并写入一个引用该文件的 LOAD DATA LOCAL 语句。 写入这些文件的目录的默认位置是特定于系统的。 要明确指定目录，请使用 `--local-load` 选项。

因为 mysqlbinlog 将 LOAD DATA 语句转换为 LOAD DATA LOCAL 语句（即添加了 LOCAL），所以用于处理语句的客户端和服务器都必须配置为启用 LOCAL 功能。 请参见第 [“LOAD DATA LOCAL 的安全注意事项”](https://dev.mysql.com/doc/refman/8.0/en/load-data-local-security.html)。

> 警告
为 LOAD DATA LOCAL 语句创建的临时文件不会自动删除，因为在您实际执行这些语句之前需要它们。 不再需要语句日志后，应自行删除临时文件。 这些文件可以在临时文件目录中找到，它们的名称类似于 original_file_name-#-#。

## 命令参数

| Option Name                                 | Description                                                                                                                                                                                                | Introduced | Deprecated |
|---------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|------------|
| --base64-output                             | Print binary log entries using base-64 encoding                                                                                                                                                            |            |            |
| --bind-address                              | Use specified network interface to connect to MySQL Server                                                                                                                                                 |            |            |
| --binlog-row-event-max-size                 | Binary log max event size                                                                                                                                                                                  |            |            |
| --character-sets-dir                        | Directory where character sets are installed                                                                                                                                                               |            |            |
| --compress                                  | Compress all information sent between client and server                                                                                                                                                    | 8.0.17     | 8.0.18     |
| --compression-algorithms                    | Permitted compression algorithms for connections to server                                                                                                                                                 | 8.0.18     |            |
| --connection-server-id                      | Used for testing and debugging. See text for applicable default values and other particulars                                                                                                               |            |            |
| --database                                  | List entries for just this database                                                                                                                                                                        |            |            |
| --debug                                     | Write debugging log                                                                                                                                                                                        |            |            |
| --debug-check                               | Print debugging information when program exits                                                                                                                                                             |            |            |
| --debug-info                                | Print debugging information, memory, and CPU statistics when program exits                                                                                                                                 |            |            |
| --default-auth                              | Authentication plugin to use                                                                                                                                                                               |            |            |
| --defaults-extra-file                       | Read named option file in addition to usual option files                                                                                                                                                   |            |            |
| --defaults-file                             | Read only named option file                                                                                                                                                                                |            |            |
| --defaults-group-suffix                     | Option group suffix value                                                                                                                                                                                  |            |            |
| --disable-log-bin                           | Disable binary logging                                                                                                                                                                                     |            |            |
| --exclude-gtids                             | Do not show any of the groups in the GTID set provided                                                                                                                                                     |            |            |
| --force-if-open                             | Read binary log files even if open or not closed properly                                                                                                                                                  |            |            |
| --force-read                                | If mysqlbinlog reads a binary log event that it does not recognize, it prints a warning                                                                                                                    |            |            |
| --get-server-public-key                     | Request RSA public key from server                                                                                                                                                                         |            |            |
| --help                                      | Display help message and exit                                                                                                                                                                              |            |            |
| --hexdump                                   | Display a hex dump of the log in comments                                                                                                                                                                  |            |            |
| --host                                      | Host on which MySQL server is located                                                                                                                                                                      |            |            |
| --idempotent                                | Cause the server to use idempotent mode while processing binary log updates from this session only                                                                                                         |            |            |
| --include-gtids                             | Show only the groups in the GTID set provided                                                                                                                                                              |            |            |
| --local-load                                | Prepare local temporary files for LOAD DATA in the specified directory                                                                                                                                     |            |            |
| --login-path                                | Read login path options from .mylogin.cnf                                                                                                                                                                  |            |            |
| --no-defaults                               | Read no option files                                                                                                                                                                                       |            |            |
| --offset                                    | Skip the first N entries in the log                                                                                                                                                                        |            |            |
| --password                                  | Password to use when connecting to server                                                                                                                                                                  |            |            |
| --plugin-dir                                | Directory where plugins are installed                                                                                                                                                                      |            |            |
| --port                                      | TCP/IP port number for connection                                                                                                                                                                          |            |            |
| --print-defaults                            | Print default options                                                                                                                                                                                      |            |            |
| --print-table-metadata                      | Print table metadata                                                                                                                                                                                       |            |            |
| --protocol                                  | Transport protocol to use                                                                                                                                                                                  |            |            |
| --raw                                       | Write events in raw (binary) format to output files                                                                                                                                                        |            |            |
| --read-from-remote-master                   | Read the binary log from a MySQL replication source server rather than reading a local log file                                                                                                            |            | 8.0.26     |
| --read-from-remote-server                   | Read binary log from MySQL server rather than local log file                                                                                                                                               |            |            |
| --read-from-remote-source                   | Read the binary log from a MySQL replication source server rather than reading a local log file                                                                                                            | 8.0.26     |            |
| --require-row-format                        | Require row-based binary logging format                                                                                                                                                                    | 8.0.19     |            |
| --result-file                               | Direct output to named file                                                                                                                                                                                |            |            |
| --rewrite-db                                | Create rewrite rules for databases when playing back from logs written in row-based format. Can be used multiple times                                                                                     |            |            |
| --server-id                                 | Extract only those events created by the server having the given server ID                                                                                                                                 |            |            |
| --server-id-bits                            | Tell mysqlbinlog how to interpret server IDs in binary log when log was written by a mysqld having its server-id-bits set to less than the maximum; supported only by MySQL Cluster version of mysqlbinlog |            |            |
| --server-public-key-path                    | Path name to file containing RSA public key                                                                                                                                                                |            |            |
| --set-charset                               | Add a SET NAMES charset_name statement to the output                                                                                                                                                       |            |            |
| --shared-memory-base-name                   | Shared-memory name for shared-memory connections (Windows only)                                                                                                                                            |            |            |
| --short-form                                | Display only the statements contained in the log                                                                                                                                                           |            |            |
| --skip-gtids                                | Do not include the GTIDs from the binary log files in the output dump file                                                                                                                                 |            |            |
| --socket                                    | Unix socket file or Windows named pipe to use                                                                                                                                                              |            |            |
| --ssl-ca                                    | File that contains list of trusted SSL Certificate Authorities                                                                                                                                             |            |            |
| --ssl-capath                                | Directory that contains trusted SSL Certificate Authority certificate files                                                                                                                                |            |            |
| --ssl-cert                                  | File that contains X.509 certificate                                                                                                                                                                       |            |            |
| --ssl-cipher                                | Permissible ciphers for connection encryption                                                                                                                                                              |            |            |
| --ssl-crl                                   | File that contains certificate revocation lists                                                                                                                                                            |            |            |
| --ssl-crlpath                               | Directory that contains certificate revocation-list files                                                                                                                                                  |            |            |
| --ssl-fips-mode                             | Whether to enable FIPS mode on client side                                                                                                                                                                 |            |            |
| --ssl-key                                   | File that contains X.509 key                                                                                                                                                                               |            |            |
| --ssl-mode                                  | Desired security state of connection to server                                                                                                                                                             |            |            |
| --ssl-session-data                          | File that contains SSL session data                                                                                                                                                                        | 8.0.29     |            |
| --ssl-session-data-continue-on-failed-reuse | Whether to establish connections if session reuse fails                                                                                                                                                    | 8.0.29     |            |
| --start-datetime                            | Read binary log from first event with timestamp equal to or later than datetime argument                                                                                                                   |            |            |
| --start-position                            | Decode binary log from first event with position equal to or greater than argument                                                                                                                         |            |            |
| --stop-datetime                             | Stop reading binary log at first event with timestamp equal to or greater than datetime argument                                                                                                           |            |            |
| --stop-never                                | Stay connected to server after reading last binary log file                                                                                                                                                |            |            |
| --stop-never-slave-server-id                | Slave server ID to report when connecting to server                                                                                                                                                        |            |            |
| --stop-position                             | Stop decoding binary log at first event with position equal to or greater than argument                                                                                                                    |            |            |
| --tls-ciphersuites                          | Permissible TLSv1.3 ciphersuites for encrypted connections                                                                                                                                                 | 8.0.16     |            |
| --tls-version                               | Permissible TLS protocols for encrypted connections                                                                                                                                                        |            |            |
| --to-last-log                               | Do not stop at the end of requested binary log from a MySQL server, but rather continue printing to end of last binary log                                                                                 |            |            |
| --user                                      | MySQL user name to use when connecting to server                                                                                                                                                           |            |            |
| --verbose                                   | Reconstruct row events as SQL statements                                                                                                                                                                   |            |            |
| --verify-binlog-checksum                    | Verify checksums in binary log                                                                                                                                                                             |            |            |
| --version                                   | Display version information and exit                                                                                                                                                                       |            |            |
| --zstd-compression-level                    | Compression level for connections to server that use zstd compression                                                                                                                                      | 8.0.18     |            |

### 具体说明

#### --base64-output=value

此选项确定何时应使用 BINLOG 语句将事件编码为 base-64 字符串显示。该选项具有以下允许值（不区分大小写）：

AUTO（“自动”）或 UNSPEC（“未指定”）在必要时自动显示 BINLOG 语句（即，用于格式描述事件和行事件）。如果没有给出--base64-output 选项，效果与--base64-output=AUTO 相同。

> 笔记
如果您打算使用 mysqlbinlog 的输出重新执行二进制日志文件内容，则自动显示 BINLOG 是唯一安全的行为。其他选项值仅用于调试或测试目的，因为它们可能会产生不包括可执行形式的所有事件的输出。

NEVER 导致不显示 BINLOG 语句。如果发现必须使用 BINLOG 显示的行事件，mysqlbinlog 将退出并出现错误。

DECODE-ROWS 通过指定 --verbose 选项向 mysqlbinlog 指定您打算将行事件解码并显示为注释的 SQL 语句。与 NEVER 一样，DECODE-ROWS 禁止显示 BINLOG 语句，但与 NEVER 不同，它不会在找到行事件时退出并返回错误。

有关显示 --base64-output 和 --verbose 对行事件输出的影响的示例，请参阅第 4.6.9.2 节，“mysqlbinlog 行事件显示”。

#### --database=db_name, -d db_name

此选项使 mysqlbinlog 输出二进制日志（仅限本地日志）中的条目，这些条目在 USE 选择 db_name 作为默认数据库时发生。

mysqlbinlog 的 --database 选项类似于 mysqld 的 --binlog-do-db 选项，但只能用于指定一个数据库。如果多次给出 --database ，则仅使用最后一个实例。

此选项的效果取决于使用的是基于语句还是基于行的日志记录格式，就像 --binlog-do-db 的效果取决于是基于语句还是基于行的日志记录格式一样采用。

基于语句的日志记录。 --database 选项的工作原理如下：

* 虽然 db_name 是默认数据库，但无论它们修改 db_name 中的表还是不同的数据库，都会输出语句。
* 除非选择 db_name 作为默认数据库，否则不会输出语句，即使它们修改了 db_name 中的表。
* CREATE DATABASE、ALTER DATABASE 和 DROP DATABASE 有一个例外。在确定是否输出语句时，正在创建、更改或删除的数据库被认为是默认数据库。

假设二进制日志是通过使用 statement-based-logging 执行这些语句创建的：

```sql
INSERT INTO test.t1 (i) VALUES(100);
INSERT INTO db2.t2 (j)  VALUES(200);
USE test;
INSERT INTO test.t1 (i) VALUES(101);
INSERT INTO t1 (i)      VALUES(102);
INSERT INTO db2.t2 (j)  VALUES(201);
USE db2;
INSERT INTO test.t1 (i) VALUES(103);
INSERT INTO db2.t2 (j)  VALUES(202);
INSERT INTO t2 (j)      VALUES(203);
```

mysqlbinlog --database=test 不输出前两个 INSERT 语句，因为没有默认数据库。它输出 USE 测试之后的三个 INSERT 语句，但不输出 USE db2 之后的三个 INSERT 语句。

mysqlbinlog --database=db2 不输出前两个 INSERT 语句，因为没有默认数据库。它不会输出 USE test 之后的三个 INSERT 语句，而是输出 USE db2 之后的三个 INSERT 语句。

基于行的日志记录。 mysqlbinlog 仅输出更改属于 db_name 的表的条目。默认数据库对此没有影响。假设刚才描述的二进制日志是使用基于行的日志而不是基于语句的日志创建的。 mysqlbinlog --database=test 仅输出那些在 test 数据库中修改 t1 的条目，无论是否发出 USE 或默认数据库是什么。

如果服务器在 binlog_format 设置为 MIXED 的情况下运行，并且您希望可以通过 --database 选项使用 mysqlbinlog，则必须确保修改的表位于 USE 选择的数据库中。 （特别是，不应使用跨数据库更新。）

与 --rewrite-db 选项一起使用时，首先应用 --rewrite-db 选项；然后使用重写的数据库名称应用 --database 选项。在这方面，提供选项的顺序没有区别。

#### --debug-check

程序退出（事务结束）时打印一些调试信息。

仅当 MySQL 使用 WITH_DEBUG 构建时，此选项才可用。 Oracle 提供的 MySQL 发布二进制文件不是使用此选项构建的。

#### --no-defaults

不要读取任何选项文件。 如果由于从选项文件中读取未知选项而导致程序启动失败，可以使用 --no-defaults 来防止它们被读取。

例外是在所有情况下都会读取 .mylogin.cnf 文件（如果存在）。 这允许以比在命令行上更安全的方式指定密码，即使使用 `--no-defaults` 也是如此。 要创建 .mylogin.cnf，请使用 mysql_config_editor 实用程序。 请参阅第“[mysql_config_editor - MySQL 配置实用程序](https://dev.mysql.com/doc/refman/8.0/en/mysql-config-editor.html)”。

有关此选项文件选项和其他选项文件选项的更多信息，请参阅第“[影响选项文件处理的命令行选项](https://dev.mysql.com/doc/refman/8.0/en/option-file-options.html)”。

#### --short-form, -s

仅显示日志中包含的语句，不显示任何额外信息或基于行的事件。 这仅用于测试，不应在生产系统中使用。 它已被弃用，您应该期望它在未来的版本中被删除。

## mysqlbinlog 十六进制转储格式

--hexdump 选项使 mysqlbinlog 生成二进制日志内容的十六进制转储：

mysqlbinlog --hexdump source-bin.000001
十六进制输出由以 # 开头的注释行组成，因此对于前面的命令，输出可能如下所示：

```log
/*!40019 SET @@SESSION.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
# at 4
#051024 17:24:13 server id 1  end_log_pos 98
# Position  Timestamp   Type   Master ID        Size      Master Pos    Flags
# 00000004 9d fc 5c 43   0f   01 00 00 00   5e 00 00 00   62 00 00 00   00 00
# 00000017 04 00 35 2e 30 2e 31 35  2d 64 65 62 75 67 2d 6c |..5.0.15.debug.l|
# 00000027 6f 67 00 00 00 00 00 00  00 00 00 00 00 00 00 00 |og..............|
# 00000037 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00 |................|
# 00000047 00 00 00 00 9d fc 5c 43  13 38 0d 00 08 00 12 00 |.......C.8......|
# 00000057 04 04 04 04 12 00 00 4b  00 04 1a                |.......K...|
#       Start: binlog v 4, server v 5.0.15-debug-log created 051024 17:24:13
#       at startup
ROLLBACK;
```

十六进制转储输出当前包含以下列表中的元素。 此格式可能会更改。 有关二进制日志格式的更多信息，请参阅 [MySQL 内部：二进制日志](https://dev.mysql.com/doc/internals/en/binary-log.html)。

* 位置：日志文件中的字节位置。
* 时间戳：事件时间戳。 在所示示例中，“9d fc 5c 43”是“051024 17:24:13”的十六进制表示。
* 类型：事件类型代码。
* 主 ID：创建事件的复制源服务器的服务器 ID。
* 大小：事件的大小（以字节为单位）。
* Master Pos：原始源的二进制日志文件中下一个事件的位置。
* 标志：事件标志值。

## mysqlbinlog 行事件展示

以下示例说明了 mysqlbinlog 如何显示指定数据修改的行事件。 这些对应于具有 WRITE_ROWS_EVENT、UPDATE_ROWS_EVENT 和 DELETE_ROWS_EVENT 类型代码的事件。 --base64-output=DECODE-ROWS 和 --verbose 选项可用于影响行事件输出。

假设服务器正在使用基于行的二进制日志记录，并且您执行以下语句序列：

```sql
CREATE TABLE t
(
  id   INT NOT NULL,
  name VARCHAR(20) NOT NULL,
  date DATE NULL
) ENGINE = InnoDB;

START TRANSACTION;
INSERT INTO t VALUES(1, 'apple', NULL);
UPDATE t SET name = 'pear', date = '2009-01-01' WHERE id = 1;
DELETE FROM t WHERE id = 1;
COMMIT;
```

默认情况下，mysqlbinlog 使用 [BINLOG](https://dev.mysql.com/doc/refman/8.0/en/binlog.html) 语句显示编码为 base-64 字符串的行事件。 省略多余的行，前面语句序列产生的行事件的输出如下所示：

```log
$> mysqlbinlog log_file
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAANoAAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBcBAAAAKAAAAAIBAAAQABEAAAAAAAEAA//8AQAAAAVhcHBsZQ==
'/*!*/;
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAC4BAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBgBAAAANgAAAGQBAAAQABEAAAAAAAEAA////AEAAAAFYXBwbGX4AQAAAARwZWFyIbIP
'/*!*/;
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAJABAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBkBAAAAKgAAALoBAAAQABEAAAAAAAEAA//4AQAAAARwZWFyIbIP
'/*!*/;
```

要将行事件视为“pseudo-SQL”语句形式的注释，请使用 --verbose 或 -v 选项运行 mysqlbinlog。 此输出级别还显示适用的表分区信息。 输出包含以 ### 开头的行：

```log
$> mysqlbinlog -v log_file
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAANoAAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBcBAAAAKAAAAAIBAAAQABEAAAAAAAEAA//8AQAAAAVhcHBsZQ==
'/*!*/;
### INSERT INTO test.t
### SET
###   @1=1
###   @2='apple'
###   @3=NULL
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAC4BAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBgBAAAANgAAAGQBAAAQABEAAAAAAAEAA////AEAAAAFYXBwbGX4AQAAAARwZWFyIbIP
'/*!*/;
### UPDATE test.t
### WHERE
###   @1=1
###   @2='apple'
###   @3=NULL
### SET
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAJABAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBkBAAAAKgAAALoBAAAQABEAAAAAAAEAA//4AQAAAARwZWFyIbIP
'/*!*/;
### DELETE FROM test.t
### WHERE
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
```

如果 binlog_rows_query_log_events 系统变量设置为 TRUE，则指定 --verbose 或 -v 两次以显示每列的数据类型和一些元数据，以及信息日志事件，例如行查询日志事件。 输出包含每列更改后的附加注释：

```log
$> mysqlbinlog -vv log_file
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAANoAAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBcBAAAAKAAAAAIBAAAQABEAAAAAAAEAA//8AQAAAAVhcHBsZQ==
'/*!*/;
### INSERT INTO test.t
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='apple' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3=NULL /* VARSTRING(20) meta=0 nullable=1 is_null=1 */
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAC4BAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBgBAAAANgAAAGQBAAAQABEAAAAAAAEAA////AEAAAAFYXBwbGX4AQAAAARwZWFyIbIP
'/*!*/;
### UPDATE test.t
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='apple' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3=NULL /* VARSTRING(20) meta=0 nullable=1 is_null=1 */
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='pear' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3='2009:01:01' /* DATE meta=0 nullable=1 is_null=0 */
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F

BINLOG '
fAS3SBMBAAAALAAAAJABAAAAABEAAAAAAAAABHRlc3QAAXQAAwMPCgIUAAQ=
fAS3SBkBAAAAKgAAALoBAAAQABEAAAAAAAEAA//4AQAAAARwZWFyIbIP
'/*!*/;
### DELETE FROM test.t
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2='pear' /* VARSTRING(20) meta=20 nullable=0 is_null=0 */
###   @3='2009:01:01' /* DATE meta=0 nullable=1 is_null=0 */
```

您可以使用 --base64-output=DECODE-ROWS 选项告诉 mysqlbinlog 禁止行事件的 BINLOG 语句。 这类似于 --base64-output=NEVER 但如果找到行事件，则不会退出并出现错误。 --base64-output=DECODE-ROWS 和 --verbose 的组合提供了一种仅将行事件视为 SQL 语句的便捷方法：

```log
$> mysqlbinlog -v --base64-output=DECODE-ROWS log_file
...
# at 218
#080828 15:03:08 server id 1  end_log_pos 258   Write_rows: table id 17 flags: STMT_END_F
### INSERT INTO test.t
### SET
###   @1=1
###   @2='apple'
###   @3=NULL
...
# at 302
#080828 15:03:08 server id 1  end_log_pos 356   Update_rows: table id 17 flags: STMT_END_F
### UPDATE test.t
### WHERE
###   @1=1
###   @2='apple'
###   @3=NULL
### SET
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
...
# at 400
#080828 15:03:08 server id 1  end_log_pos 442   Delete_rows: table id 17 flags: STMT_END_F
### DELETE FROM test.t
### WHERE
###   @1=1
###   @2='pear'
###   @3='2009:01:01'
```

> 笔记
如果您打算重新执行 mysqlbinlog 输出，则不应禁止 BINLOG 语句。

由 `--verbose` 为行事件生成的 SQL 语句比相应的 BINLOG 语句更具可读性。但是，它们与生成事件的原始 SQL 语句并不完全对应。以下限制适用：

* 原始列名丢失并被 @N 替换，其中 N 是列号。
* 二进制日志中没有字符集信息，影响字符串列显示：
  * 对应的二进制和非二进制字符串类型（BINARY 和 CHAR、VARBINARY 和 VARCHAR、BLOB 和 TEXT）之间没有区别。输出对固定长度字符串使用 STRING 数据类型，对可变长度字符串使用 VARSTRING 数据类型。
  * 对于多字节字符集，二进制日志中不存在每个字符的最大字节数，因此字符串类型的长度以字节而不是字符显示。例如，STRING(4) 用作来自以下任一列类型的值的数据类型：

```sql
CHAR(4) CHARACTER SET latin1
CHAR(2) CHARACTER SET ucs2
````

* 由于 UPDATE_ROWS_EVENT 类型的事件的存储格式，UPDATE 语句会在 SET 子句之前显示 WHERE 子句。

正确解释行事件需要二进制日志开头的格式描述事件的信息。 因为 mysqlbinlog 事先不知道日志的其余部分是否包含行事件，所以默认情况下它在输出的初始部分使用 BINLOG 语句显示格式描述事件。

如果已知二进制日志不包含任何需要 BINLOG 语句的事件（即没有行事件），则可以使用 `--base64-output=NEVER` 选项来防止写入此标头。

## 使用 mysqlbinlog 备份二进制日志文件

默认情况下，mysqlbinlog 读取二进制日志文件并以文本格式显示其内容。这使您能够更轻松地检查文件中的事件并重新执行它们（例如，通过使用输出作为 mysql 的输入）。 mysqlbinlog 可以直接从本地文件系统读取日志文件，或者，使用 `--read-from-remote-server` 选项，它可以连接到服务器并从该服务器请求二进制日志内容。 mysqlbinlog 将文本输出写入其标准输出，或者写入以 `--result-file=file_name` 选项的值命名的文件（如果给定了该选项）。

### mysqlbinlog 备份功能

mysqlbinlog 可以读取二进制日志文件并写入包含相同内容的新文件——即以二进制格式而不是文本格式。此功能使您能够轻松地以原始格式备份二进制日志。 mysqlbinlog 可以进行静态备份，备份一组日志文件并在到达最后一个文件末尾时停止。它还可以进行连续（“实时”）备份，在到达最后一个日志文件的末尾时保持与服务器的连接，并在生成新事件时继续复制它们。在连续备份操作中，mysqlbinlog 一直运行到连接结束（例如，服务器退出时）或mysqlbinlog 被强制终止。当连接结束时，mysqlbinlog 不会等待并重试连接，这与副本服务器不同。要在服务器重新启动后继续实时备份，您还必须重新启动 mysqlbinlog。

> 重要的
mysqlbinlog 可以备份加密和未加密的二进制日志文件。 但是，使用 mysqlbinlog 生成的加密二进制日志文件的副本以未加密的格式存储。

### mysqlbinlog 备份选项

二进制日志备份要求您至少使用两个选项调用 mysqlbinlog：

`--read-from-remote-server`（或 -R）选项告诉 mysqlbinlog 连接到服务器并请求其二进制日志。 （这类似于连接到其复制源服务器的副本服务器。）

`--raw` 选项告诉 mysqlbinlog 写入原始（二进制）输出，而不是文本输出。

与 `--read-from-remote-server` 一起，通常指定其他选项：`--host` 指示服务器运行的位置，您可能还需要指定连接选项，例如 `--user` 和 `--password`。

其他几个选项与 `--raw` 结合使用：

`--stop-never`：到达最后一个日志文件的末尾后保持与服务器的连接并继续读取新事件。

`--connection-server-id=id`：mysqlbinlog连接服务器时报告的服务器ID。当使用 `--stop-never` 时，默认报告的服务器 ID 为 1。如果这导致与副本服务器或另一个 mysqlbinlog 进程的 ID 冲突，请使用 `--connection-server-id` 指定备用服务器 ID。请参阅第 [“指定 mysqlbinlog 服务器 ID”](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-server-id.html)。

`--result-file`：输出文件名的前缀，如后所述。

### 静态和实时备份

要使用 mysqlbinlog 备份服务器的二进制日志文件，您必须指定服务器上实际存在的文件名。 如果您不知道名称，请连接到服务器并使用 SHOW BINARY LOGS 语句查看当前名称。 假设该语句产生以下输出：

```sql
mysql> SHOW BINARY LOGS;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000130 |     27459 | No        |
| binlog.000131 |     13719 | No        |
| binlog.000132 |     43268 | No        |
+---------------+-----------+-----------+
```

有了这些信息，您可以使用 mysqlbinlog 将二进制日志备份到当前目录，如下所示（在一行中输入每个命令）：

要通过 binlog.000132 对 binlog.000130 进行静态备份，请使用以下任一命令：

```bash
mysqlbinlog --read-from-remote-server --host=host_name --raw
  binlog.000130 binlog.000131 binlog.000132

mysqlbinlog --read-from-remote-server --host=host_name --raw
  --to-last-log binlog.000130
```

第一个命令明确指定每个文件名。 第二个仅命名第一个文件并使用 --to-last-log 读取最后一个文件。 这些命令之间的区别在于，如果服务器恰好在 mysqlbinlog 到达 binlog.000132 的末尾之前打开了 binlog.000133，则第一个命令不会读取它，但第二个命令会读取它。

要进行实时备份，其中 mysqlbinlog 以 binlog.000130 开头以复制现有日志文件，然后在服务器生成新事件时保持连接以复制新事件：

```bash
mysqlbinlog --read-from-remote-server --host=host_name --raw
  --stop-never binlog.000130
```

使用--stop-never，不必指定--to-last-log 来读取最后一个日志文件，因为该选项是隐含的。

### 输出文件命名

如果没有 `--raw`，mysqlbinlog 会生成文本输出，如果给定 `--result-file` 选项，则指定将所有输出写入的单个文件的名称。使用 `--raw`，mysqlbinlog 为从服务器传输的每个日志文件写入一个二进制输出文件。默认情况下，mysqlbinlog 会在当前目录中写入与原始日志文件同名的文件。要修改输出文件名，请使用 `--result-file` 选项。与--raw 结合使用时，`--result-file` 选项值被视为修改输出文件名的前缀。

假设服务器当前具有名为 binlog.000999 及以上的二进制日志文件。如果您使用 `mysqlbinlog --raw` 备份文件，则 `--result-file` 选项会生成输出文件名，如下表所示。您可以通过以目录路径开头的 --result-file 值将文件写入特定目录。如果 `--result-file` 值仅包含目录名，则该值必须以路径名分隔符结尾。如果存在，输出文件将被覆盖。

| --result-file Option | Output File Names          |
|----------------------|----------------------------|
| --result-file=x      | xbinlog.000999 and up      |
| --result-file=/tmp/  | /tmp/binlog.000999 and up  |
| --result-file=/tmp/x | /tmp/xbinlog.000999 and up |

### 示例：mysqldump + mysqlbinlog 用于备份和恢复

下面的例子描述了一个简单的场景，展示了如何使用 mysqldump 和 mysqlbinlog 来备份服务器的数据和二进制日志，以及如果发生数据丢失，如何使用备份来恢复服务器。 该示例假定服务器在主机 host_name 上运行，其第一个二进制日志文件名为 binlog.000999。 在一行中输入每个命令。

使用 mysqlbinlog 对二进制日志进行连续备份：

```bash
mysqlbinlog --read-from-remote-server --host=host_name --raw
  --stop-never binlog.000999
```

使用 mysqldump 创建一个转储文件作为服务器数据的快照。 使用--all-databases、--events 和--routines 备份所有数据，使用--master-data=2 将当前二进制日志坐标包含在转储文件中。

```bash
mysqldump --host=host_name --all-databases --events --routines --master-data=2> dump_file
```

定期执行 mysqldump 命令以根据需要创建更新的快照。

如果发生数据丢失（例如，如果服务器意外退出），请使用最近的转储文件来恢复数据：

```bash
mysql --host=host_name -u root -p < dump_file
```

然后使用二进制日志备份重新执行在转储文件中列出的坐标之后写入的事件。 假设文件中的坐标如下所示：

```bash
-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.001002', MASTER_LOG_POS=27284;
```

如果最近备份的日志文件名为 binlog.001004，请重新执行日志事件，如下所示：

```bash
mysqlbinlog --start-position=27284 binlog.001002 binlog.001003 binlog.001004
  | mysql --host=host_name -u root -p
```

您可能会发现将备份文件（转储文件和二进制日志文件）复制到服务器主机更容易执行恢复操作，若是 MySQL 不允许远程 root 访问。

### mysqlbinlog 备份限制

使用 mysqlbinlog 的二进制日志备份受以下限制：

如果连接丢失（例如，如果发生服务器重启或网络中断），mysqlbinlog 不会自动重新连接到 MySQL 服务器。

备份的延迟类似于副本服务器的延迟。
