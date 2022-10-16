# SHOW PROCESSLIST语句

`SHOW [FULL] PROCESSLIST`

MySQL进程列表指示服务器内执行的线程集当前正在执行的操作。SHOW PROCESSLIST语句是过程信息的一个来源。有关此语句与其他源的比较，请参阅[流程信息源](https://dev.mysql.com/doc/refman/8.0/en/processlist-access.html#processlist-sources)。

> 笔记
从MySQL 8.0.22开始，基于Performance Schema进程列表表，可以使用SHOW PROCESSLIST的替代实现，与默认的SHOW PORESSLIST实现不同，它不需要互斥锁，并且具有更好的性能特征。有关详细信息，请参阅第27.12.21.6节“[处理列表表](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-processlist-table.html)”。

如果您具有[PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)权限，则可以看到所有线程，甚至是属于其他用户的线程。否则（没有PROCESS权限），非匿名用户可以访问自己线程的信息，但不能访问其他用户的线程，匿名用户不能访问线程信息。

如果不使用FULL关键字，SHOW PROCESSLIST在Info字段中只显示每条语句的前100个字符。

如果您收到“连接太多”错误消息并想了解发生了什么，SHOW PROCESSLIST语句非常有用。MySQL保留了一个额外的连接，供具有[CONNECTION_ADMIN](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_connection-admin)权限（或不推荐使用的SUPER权限）的帐户使用，以确保管理员始终能够连接和检查系统（假设您没有将此权限授予所有用户）。

可以使用KILL语句终止线程。见 [KILL语句](../其他管理语句/KILL语句.md)。

SHOW PROCESSLIST输出示例：

```sql
mysql> SHOW FULL PROCESSLIST\G
*************************** 1. row ***************************
     Id: 1
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 1030455
  State: Waiting for master to send event
   Info: NULL
*************************** 2. row ***************************
     Id: 2
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 1004
  State: Has read all relay log; waiting for the slave
         I/O thread to update it
   Info: NULL
*************************** 3. row ***************************
     Id: 3112
   User: replikator
   Host: artemis:2204
     db: NULL
Command: Binlog Dump
   Time: 2144
  State: Has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
*************************** 4. row ***************************
     Id: 3113
   User: replikator
   Host: iconnect2:45781
     db: NULL
Command: Binlog Dump
   Time: 2086
  State: Has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
*************************** 5. row ***************************
     Id: 3123
   User: stefan
   Host: localhost
     db: apollon
Command: Query
   Time: 0
  State: NULL
   Info: SHOW FULL PROCESSLIST
```

SHOW PROCESSLIST输出包含以下列：

- Id

  连接标识符。这与INFORMATION_SCHEMA PROCESSLIST表的ID列中显示的值相同，它显示在Performance SCHEMA[线程](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-threads-table.html)表的PROCESSLIST_ID列中，并由线程内的[CONNECTION_ID()](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_connection-id)函数返回。

- User

  发布该语句的MySQL用户。系统用户的值是指服务器生成的用于在内部处理任务的非客户端线程，例如，副本主机上使用的延迟行处理程序线程或I/O（接收器）或SQL（应用程序）线程。对于系统用户，“主机”列中没有指定主机。未经身份验证的用户是指已与客户端连接关联但尚未进行客户端用户身份验证的线程。event_scheduler是指监视计划事件的线程（请参阅第25.4节“[使用事件调度器](https://dev.mysql.com/doc/refman/8.0/en/event-scheduler.html)”）。

  > 笔记
  系统用户的用户值与[SYSTEM_USER](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_system-user)权限不同。前者指定内螺纹。后者区分系统用户和常规用户帐户类别（见第6.2.11节“[帐户类别](https://dev.mysql.com/doc/refman/8.0/en/account-categories.html)”）。

- Host

  发出语句的客户端的主机名（系统用户除外，该用户没有主机）。TCP/IP连接的主机名以host_name:client_port格式报告，以便更容易确定哪个客户端正在执行什么操作。

- db

  线程的默认数据库，如果未选择任何数据库，则为NULL。

- Command

  线程代表客户端执行的命令类型，如果会话处于空闲状态，则为睡眠。有关线程命令的描述，请参阅第8.14节“[检查服务器线程（进程）信息](https://dev.mysql.com/doc/refman/8.0/en/thread-information.html)”。此列的值对应于客户端/服务器协议的COM_xxx命令和COM_xxx状态变量。参见第5.1.10节“[服务器状态变量](https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html)”。

- Time

  线程处于当前状态的时间（以秒为单位）。对于副本SQL线程，该值是上次复制事件的时间戳与副本主机的实时之间的秒数。请参阅第17.2.3节“[复制线程](https://dev.mysql.com/doc/refman/8.0/en/replication-implementation-details.html)”。

- State

  指示线程正在执行的操作、事件或状态。有关状态值的描述，请参阅第8.14节“[检查服务器线程（进程）信息](https://dev.mysql.com/doc/refman/8.0/en/thread-information.html)”。

  大多数状态对应于非常快速的操作。如果线程在给定状态下停留了数秒，则可能存在需要调查的问题。

- Info

  线程正在执行的语句，如果不执行任何语句，则为NULL。该语句可能是发送到服务器的语句，如果该语句执行其他语句，则可能是最内部的语句。例如，如果CALL语句执行正在执行SELECT语句的存储过程，则Info值将显示SELECT声明。
