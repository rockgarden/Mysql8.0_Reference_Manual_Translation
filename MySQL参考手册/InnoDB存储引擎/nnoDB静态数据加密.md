# nnoDB静态数据加密

<https://dev.mysql.com/doc/refman/8.0/en/innodb-data-encryption.html>

InnoDB支持对每表文件表空间、一般表空间、mysql系统表空间、重做日志和撤销日志进行静态数据加密。

从MySQL 8.0.16开始，也支持为模式和一般表空间设置加密默认值，这允许DBA控制在这些模式和表空间创建的表是否被加密。

InnoDB静态数据加密的特点和功能在本节的下列主题下描述。

关于静态数据加密

InnoDB使用两层的加密密钥架构，包括一个主加密密钥和表空间密钥。当一个表空间被加密时，一个表空间密钥被加密并存储在表空间头中。当一个应用程序或认证用户想要访问加密的表空间数据时，InnoDB使用主加密密钥来解密表空间密钥。表空间密钥的解密版本永远不会改变，但是主加密密钥可以根据需要改变。这个动作被称为主密钥旋转。

静态数据加密功能依赖于主加密密钥管理的钥匙圈组件或插件。

所有的MySQL版本都提供一个组件_keyring_file组件和keyring_file插件，每个组件都在服务器主机的本地文件中存储钥匙圈数据。

MySQL企业版提供额外的钥匙圈组件和插件：

component_keyring_encrypted_file： 将钥匙圈数据存储在服务器主机本地的一个加密的、受密码保护的文件中。

keyring_encrypted_file： 将钥匙圈数据存储在服务器主机上的一个加密的、受密码保护的文件中。

keyring_okv： 一个KMIP 1.1插件，用于与KMIP兼容的后端钥匙圈存储产品。支持的KMIP兼容产品包括集中式密钥管理解决方案，如Oracle Key Vault, Gemalto KeySecure, Thales Vormetric密钥管理服务器和Fornetix Key Orchestration。

keyring_aws： 与亚马逊网络服务的密钥管理服务（AWS KMS）进行通信，作为密钥生成的后端，并使用本地文件进行密钥存储。

keyring_hashicorp： 与HashiCorp Vault进行通信，用于后端存储。

警告
对于加密密钥管理， component_keyring_file 和 component_keyring_encrypted_file 组件，以及 keyring_file 和 keyring_encrypted_file 插件并不打算作为一个符合法规的解决方案。PCI、FIPS等安全标准要求使用密钥管理系统来确保、管理和保护密钥库或硬件安全模块（HSM）中的加密密钥。

一个安全和强大的加密密钥管理解决方案对安全和符合各种安全标准至关重要。当静态数据加密功能使用集中式密钥管理解决方案时，该功能被称为 "MySQL企业透明数据加密（TDE）"。



静态数据加密功能支持高级加密标准（AES）基于块的加密算法。它使用电子编码簿（ECB）块加密模式进行表空间密钥加密，使用密码块链（CBC）块加密模式进行数据加密。

关于静态数据加密功能的常见问题，见A.17节 "MySQL 8.0常见问题"： InnoDB静态数据加密"。

加密的先决条件

必须在启动时安装和配置一个钥匙圈组件或插件。早期加载确保该组件或插件在InnoDB存储引擎初始化之前可用。关于钥匙圈的安装和配置说明，请参见第6.4.4节，"MySQL钥匙圈"。该说明显示了如何确保所选择的组件或插件是活动的。

一次只能启用一个钥匙圈组件或插件。启用多个钥匙圈组件或插件是不被支持的，结果可能不尽如人意。

重要提示
一旦在MySQL实例中创建了加密表空间，创建加密表空间时加载的钥匙圈组件或插件必须在启动时继续加载。如果不这样做，在启动服务器和InnoDB恢复时就会出现错误。

在对生产数据进行加密时，确保采取步骤防止主加密密钥的丢失。如果主加密密钥丢失，存储在加密的表空间文件中的数据就无法恢复了。如果你使用component_keyring_file或component_keyring_encrypted_file组件，或keyring_file或keyring_encrypted_file插件，在创建第一个加密表空间后，在主密钥旋转前和主密钥旋转后立即创建密钥数据文件的备份。对于每个组件，它的配置文件指出了数据文件的位置。keyring_file_data配置选项定义了keyring_file插件的钥匙圈数据文件位置。keyring_encrypted_file_data配置选项为keyring_encrypted_file插件定义了keyring数据文件的位置。如果你使用keyring_okv或keyring_aws插件，确保你已经进行了必要的配置。有关说明，见第6.4.4节 "MySQL钥匙圈"。

为模式和一般表空间定义加密默认值

从MySQL 8.0.16开始，default_table_encryption系统变量定义了模式和一般表空间的默认加密设置。当没有明确指定ENCRYPTION子句时，CREATE TABLESPACE和CREATE SCHEMA操作应用default_table_encryption设置。

ALTER SCHEMA和ALTER TABLESPACE操作不应用default_table_encryption设置。必须明确指定一个ENCRYPTION子句来改变现有模式或一般表空间的加密。



default_table_encryption变量可以为单个客户连接或使用SET语法在全局范围内设置。例如，下面的语句在全局范围内启用默认模式和表空间加密：

mysql> SET GLOBAL default_table_encryption=ON；
当创建或改变一个模式时，也可以使用DEFAULT ENCRYPTION子句定义一个模式的默认加密设置，如在这个例子：

mysql> CREATE SCHEMA test DEFAULT ENCRYPTION = 'Y'；
如果在创建模式时没有指定DEFAULT ENCRYPTION子句，将应用default_table_encryption设置。必须指定DEFAULT ENCRYPTION子句来改变现有模式的默认加密方式。否则，该模式将保留其当前的加密设置。

默认情况下，表会继承它所创建的模式或一般表空间的加密设置。例如，在一个已启用加密的模式中创建的表，默认是加密的。这种行为使DBA能够通过定义和执行模式和一般表空间的加密默认值来控制表的加密使用。

加密默认值是通过启用table_encryption_privilege_check系统变量来执行的。当table_encryption_privilege_check被启用时，在创建或改变具有不同于default_table_encryption设置的加密设置的模式或一般表空间时，或者在创建或改变具有不同于默认模式加密的加密设置的表时，会进行权限检查。当table_encryption_privilege_check被禁用时（默认情况），权限检查不会发生，前面提到的操作被允许进行，但有警告。

当启用table_encryption_privilege_check时，需要TABLE_ENCRYPTION_ADMIN权限来覆盖默认的加密设置。DBA可以授予这个权限，使用户在创建或改变模式或一般表空间时偏离默认的_table_encryption设置，或者在创建或改变表时偏离默认的模式加密。这个权限不允许在创建或更改表时偏离一般表空间的加密。一个表必须具有与它所在的一般表空间相同的加密设置。

文件-表-表空间加密

从MySQL 8.0.16开始，除非在CREATE TABLE语句中明确指定ENCRYPTION子句，否则逐个文件的表空间会继承创建表的模式的默认加密功能。在MySQL 8.0.16之前，必须指定ENCRYPTION子句以启用加密。

mysql> CREATE TABLE t1 (c1 INT) ENCRYPTION = 'Y'；
要改变现有的按表文件的表空间的加密，必须指定ENCRYPTION子句。


mysql> ALTER TABLE t1 ENCRYPTION = 'Y'；
从MySQL 8.0.16开始，如果table_encryption_privilege_check变量被启用，指定一个ENCRYPTION子句，其设置与默认模式加密不同，需要TABLE_ENCRYPTION_ADMIN权限。参见为模式和一般表空间定义加密默认值。

一般表空间加密

从MySQL 8.0.16开始，default_table_encryption变量决定了新创建的一般表空间的加密，除非在CREATE TABLESPACE语句中明确指定ENCRYPTION子句。在MySQL 8.0.16之前，必须指定一个ENCRYPTION子句以启用加密。

mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' ENCRYPTION = 'Y' Engine=InnoDB；
要改变现有一般表空间的加密，必须指定一个ENCRYPTION子句。

mysql> ALTER TABLESPACE ts1 ENCRYPTION = 'Y'；
从MySQL 8.0.16开始，如果table_encryption_privilege_check变量被启用，指定一个ENCRYPTION子句，其设置与default_table_encryption设置不同，需要有TABLE_ENCRYPTION_ADMIN权限。参见为模式和一般表空间定义加密默认值。

双重写文件加密

从MySQL 8.0.23开始，对双写文件的加密支持已经可用。InnoDB自动对属于加密表空间的双写文件页进行加密。不需要任何操作。双写文件页是使用相关表空间的加密密钥进行加密的。写入表空间数据文件的相同加密页也会被写入双写文件。属于未加密的表空间的双写文件页仍未被加密。

在恢复期间，加密的双写文件页被解密并检查是否有损坏。

mysql系统表空间加密

从MySQL 8.0.16开始，对mysql系统表空间的加密支持是可用的。

mysql系统表空间包含mysql系统数据库和MySQL数据字典表。默认情况下，它是未加密的。要启用mysql系统表空间的加密，在ALTER TABLESPACE语句中指定表空间名称和ENCRYPTION选项。

mysql> ALTER TABLESPACE mysql ENCRYPTION = 'Y'；
要禁用mysql系统表空间的加密，使用ALTER TABLESPACE语句设置ENCRYPTION = 'N'。

mysql> ALTER TABLESPACE mysql ENCRYPTION = 'N'；
启用或禁用mysql系统表空间的加密需要对实例中的所有表进行CREATE TABLESPACE权限（CREATE TABLESPACE on *.*）。

重做日志加密

重做日志数据加密是使用innodb_redo_log_encrypt配置选项启用的。重做日志加密在默认情况下是禁用的。

与表空间数据一样，重做日志数据加密发生在重做日志数据被写入磁盘的时候，而解密发生在重做日志数据被从磁盘读取的时候。一旦重做日志数据被读入内存，它是未加密的形式。重做日志数据的加密和解密是使用表空间的加密功能进行的。



当innodb_redo_log_encrypt被启用时，在磁盘上存在的未加密的重做日志页保持未加密状态，而新的重做日志页以加密的形式写入磁盘。同样，当innodb_redo_log_encrypt被禁用时，在磁盘上存在的加密重做日志页仍然是加密的，而新的重做日志页以未加密的形式写入磁盘。

警告
在MySQL 8.0.30中引入的一个退步阻止了一旦启用重做日志加密就禁用它。(Bug #108052，Bug #34456802）。

从MySQL 8.0.30开始，重做日志加密元数据，包括表空间加密密钥，被存储在具有最近检查点LSN的重做日志文件的头中。在MySQL 8.0.30之前，重做日志加密元数据，包括表空间加密密钥，被存储在第一个重做日志文件（ib_logfile0）的标题中。如果带有加密元数据的重做日志文件被删除，重做日志加密将被禁用。

一旦启用重做日志加密，在没有钥匙圈组件或插件或没有加密钥匙的情况下，正常重启是不可能的，因为InnoDB必须能够在启动期间扫描重做页面，如果重做日志页面被加密，这是不可能的。如果没有钥匙圈组件或插件或加密钥匙，只可能在没有重做日志的情况下强制启动（SRV_FORCE_NO_LOG_REDO）。参见第15.21.3节，"强制InnoDB恢复"。

撤销日志加密

使用innodb_undo_log_encrypt配置选项可以启用撤销日志数据加密功能。撤销日志加密适用于驻留在撤销表空间中的撤销日志。参见章节15.6.3.4, "撤销表空间"。默认情况下，撤销日志数据加密是禁用的。

与表空间数据一样，撤销日志数据加密在撤销日志数据写入磁盘时发生，而解密在撤销日志数据从磁盘读取时发生。一旦撤销日志数据被读入内存，它就处于未加密的状态。撤销日志数据是使用表空间的加密密钥进行加密和解密的。

当 innodb_undo_log_encrypt 启用时，磁盘上未加密的撤销日志页保持未加密状态，而新的撤销日志页以加密的形式写入磁盘。同样的，当innodb_undo_log_encrypt被禁用时，磁盘上已加密的撤销日志页仍然是加密的，而新的撤销日志页则以未加密的形式写入磁盘。

撤销日志的加密元数据，包括表空间的加密密钥，被存储在撤销日志文件的头部。

注意
当撤销日志加密被禁用时，服务器会继续要求使用用于加密撤销日志数据的钥匙圈组件或插件，直到包含加密撤销日志数据的撤销表空间被截断。(只有当撤销表空间被截断时，加密头才会从撤销表空间中被删除）。有关截断撤销表空间的信息，请参见截断撤销表空间。

主密钥的轮换

应定期轮换主加密密钥，只要你怀疑密钥已被破坏，就应轮换。



主密钥旋转是一个原子性的、实例级的操作。每次主加密密钥被旋转时，MySQL实例中的所有表空间密钥都会被重新加密并保存回它们各自的表空间头。作为一个原子操作，一旦启动旋转操作，所有表空间密钥的重新加密必须成功。如果主密钥轮换被服务器故障打断，InnoDB会在服务器重启时将操作向前滚动。更多信息，请参阅加密和恢复。

旋转主加密密钥只是改变主加密密钥和重新加密表空间密钥。它不会解密或重新加密相关的表空间数据。

旋转主加密密钥需要 ENCRYPTION_KEY_ADMIN 权限（或已被废弃的 SUPER 权限）。

要旋转主加密密钥，运行

mysql> ALTER INSTANCE ROTATE INNODB MASTER KEY；
ALTER INSTANCE ROTATE INNODB MASTER KEY支持并发的DML。然而，它不能与表空间加密操作同时运行，并采取锁来防止并发执行可能产生的冲突。如果ALTER INSTANCE ROTATE INNODB MASTER KEY操作正在运行，它必须在表空间加密操作进行之前完成，反之亦然。

加密和恢复

如果在加密操作过程中发生服务器故障，当服务器重新启动时，该操作会向前滚动。对于一般的表空间，加密操作会在后台线程中从最后处理的页面开始恢复。

如果在主密钥旋转过程中发生服务器故障，InnoDB会在服务器重启时继续进行操作。

在存储引擎初始化之前，必须加载钥匙圈组件或插件，以便在InnoDB初始化和恢复活动访问表空间数据之前，可以从表空间标题中获取解密表空间数据页的必要信息。(参见加密的先决条件）。

当InnoDB初始化和恢复开始时，主密钥旋转操作恢复。由于服务器故障，一些表空间密钥可能已经使用新的主加密密钥进行了加密。InnoDB从每个表空间头读取加密数据，如果数据表明表空间密钥是使用旧的主加密密钥加密的，InnoDB从钥匙圈中检索旧的密钥并使用它来解密表空间密钥。然后InnoDB使用新的主加密密钥对表空间密钥进行重新加密，并将重新加密的表空间密钥保存到表空间头。

导出加密的表空间

表空间导出只支持按表文件的表空间。



当一个加密的表空间被导出时，InnoDB会生成一个传输密钥，用来加密表空间密钥。加密的表空间密钥和传输密钥被存储在tablespace_name.cfp文件中。这个文件和加密的表空间文件一起被要求执行导入操作。在导入时，InnoDB使用传输密钥来解密tablepace_name.cfp文件中的表空间密钥。相关信息请参见第15.6.1.3节，"导入InnoDB表"。

加密和复制

ALTER INSTANCE ROTATE INNODB MASTER KEY语句仅在复制环境中得到支持，其中源和复制运行支持表空间加密的 MySQL 版本。

成功的ALTER INSTANCE ROTATE INNODB MASTER KEY语句会被写入二进制日志，以便在副本上进行复制。

如果ALTER INSTANCE ROTATE INNODB MASTER KEY语句失败，它不会被记录到二进制日志，也不会被复制到副本上。

如果钥匙圈组件或插件安装在源文件上，但没有安装在副本上，则ALTER INSTANCE ROTATE INNODB MASTER KEY操作的复制失败。

如果keyring_file或keyring_encrypted_file插件同时安装在源文件和副本上，但副本没有keyring数据文件，复制的ALTER INSTANCE ROTATE INNODB MASTER KEY语句会在副本上创建keyring数据文件，假设keyring文件数据没有在内存中缓存。ALTER INSTANCE ROTATE INNODB MASTER KEY使用内存中缓存的钥匙圈文件数据，如果可用的话。

识别加密的表空间和模式

在MySQL 8.0.13中引入的信息模式INNODB_TABLESPACES表包括一个ENCRYPTION列，可用于识别加密的表空间。

mysql> SELECT SPACE, NAME, SPACE_TYPE, ENCRYPTION FROM INFORMATION_SCHEMA.INNODB_TABLESPACES
       where encryption='y'/g
*************************** 1.行 ***************************
     空间：4294967294
      名称：mysql
SPACE_TYPE： 一般
ENCRYPTION: Y
*************************** 2.行 ***************************
     空间：2
      名称：test/t1
空间_类型： 单一
ENCRYPTION: Y
*************************** 3. 行 ***************************
     空间：3
      名称：TS1
空间_类型： 一般
ENCRYPTION: Y
当ENCRYPTION选项在CREATE TABLE或ALTER TABLE语句中被指定时，它被记录在INFORMATION_SCHEMA.TABLES的CREATE_OPTIONS列。这个列可以被查询，以确定驻留在加密的逐表文件表空间中的表。


mysql> SELECT TABLE_SCHEMA, TABLE_NAME, CREATE_OPTIONS FROM INFORMATION_SCHEMA.TABLES
       where create_options like '%encryption%'；
+--------------+------------+----------------+
| table_schema | table_name | create_options |
+--------------+------------+----------------+
| 测试 | t1 | ENCRYPTION="Y" |
+--------------+------------+----------------+
查询信息模式INNODB_TABLESPACES表，检索与特定模式和表相关的表空间信息。

mysql> SELECT SPACE, NAME, SPACE_TYPE FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE NAME='test/t1'；
+-------+---------+------------+
| 空间 | 名称 | 空间_类型 |
+-------+---------+------------+
| 3 | test/t1 | 单一 |
+-------+---------+------------+
你可以通过查询信息模式SCHEMATA表来识别加密启用的模式。

mysql> SELECT SCHEMA_NAME, DEFAULT_ENCRYPTION FROM INFORMATION_SCHEMA.SCHEMATA
       where default_encryption='yes'；
+-------------+--------------------+
| 架构名称 | 默认_加密 |
+-------------+--------------------+
| 测试 | 是
+-------------+--------------------+
SHOW CREATE SCHEMA也显示DEFAULT ENCRYPTION子句。

监控加密进度

你可以使用Performance Schema监控一般表空间和mysql系统表空间的加密进度。

stage/innodb/alter tablespace (encryption) stage事件工具报告一般表空间加密操作的WORK_ESTIMATED和WORK_COMPLETED信息。

下面的例子演示了如何启用stage/innodb/alter tablespace (encryption)阶段事件工具和相关的消费者表来监控一般表空间或mysql系统表空间的加密进度。关于Performance Schema阶段事件工具和相关消费者的信息，参见第27.12.5节 "Performance Schema阶段事件表"。

启用阶段/innodb/alter表空间（加密）工具：

mysql> 使用performance_schema；
mysql> UPDATE setup_instruments SET ENABLED = 'YES
       WHERE NAME LIKE 'stage/innodb/alter tablespace (encryption)' ；
启用舞台事件消费者表，其中包括events_stages_current、events_stages_history和events_stages_history_long。

mysql> UPDATE setup_consumers SET ENABLED = 'YES' WHERE NAME LIKE '%stages%' ；
运行一个表空间加密操作。在这个例子中，一个名为ts1的普通表空间被加密了。

mysql> ALTER TABLESPACE ts1 ENCRYPTION = 'Y'；
通过查询Performance Schema events_stages_current表来检查加密操作的进度。WORK_ESTIMATED 报告了表空间的总页数。WORK_COMPLETED报告处理的页面数量。


mysql> SELECT EVENT_NAME, WORK_ESTIMATED, WORK_COMPLETED FROM events_stages_current；
+--------------------------------------------+----------------+----------------+
| 事件_名称 | 工作_完成 | 工作_估计 |
+--------------------------------------------+----------------+----------------+
| stage/innodb/alter tablespace (encryption) | 1056 | 1407 |
+--------------------------------------------+----------------+----------------+
如果加密操作已经完成，events_stages_current表返回一个空集。在这种情况下，你可以检查events_stages_history表来查看已完成操作的事件数据。比如说

mysql> SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM events_stages_history；
+--------------------------------------------+----------------+----------------+
| 事件_名称 | 工作_完成 | 工作_估计 |
+--------------------------------------------+----------------+----------------+
| stage/innodb/alter tablespace (encryption) | 1407 | 1407 !
+--------------------------------------------+----------------+----------------+
加密使用说明

当使用ENCRYPTION选项改变现有的逐个文件的表空间时，要适当地计划。驻留在逐个文件表空间中的表是使用COPY算法重建的。当改变一般表空间或mysql系统表空间的ENCRYPTION属性时，使用INPLACE算法。INPLACE算法允许对驻扎在一般表空间的表进行并发的DML。并发的DDL被阻止了。

当一个普通表空间或mysql系统表空间被加密时，驻留在该表空间的所有表都被加密了。同样，在加密的表空间中创建的表也被加密。

如果服务器在正常运行期间退出或停止，建议使用先前配置的相同加密设置重新启动服务器。

当第一个新的或现有的表空间被加密时，会产生第一个主加密密钥。

主密钥旋转重新加密表空间密钥，但不改变表空间密钥本身。要改变表空间密钥，你必须停用和重新启用加密功能。对于逐个文件的表空间，重新加密表空间是一个重建表的ALGORITHM=COPY操作。对于一般的表空间和mysql系统表空间，它是一个ALGORITHM=INPLACE操作，它不需要重建驻留在表空间中的表。

如果用COMPRESSION和ENCRYPTION选项创建表，在表空间数据被加密之前会进行压缩。

如果钥匙圈数据文件（由keyring_file_data或keyring_encrypted_file_data命名的文件）为空或丢失，第一次执行ALTER INSTANCE ROTATE INNODB MASTER KEY会创建一个主加密钥匙。

卸载 component_keyring_file 或 component_keyring_encrypted_file 组件不会删除现有的钥匙圈数据文件。卸载keyring_file或keyring_encrypted_file插件并不会删除现有的钥匙圈数据文件。

建议你不要把钥匙圈数据文件和表空间数据文件放在同一个目录下。

在运行时或重启服务器时修改keyring_file_data或keyring_encrypted_file_data设置，会导致以前加密的表空间无法访问，导致数据丢失。

InnoDB FULLTEXT索引表支持加密，在添加FULLTEXT索引时隐式创建。相关信息，请参见InnoDB全文字索引表。

加密限制

高级加密标准（AES）是唯一支持的加密算法。InnoDB表空间加密使用电子密码本（ECB）块加密模式进行表空间密钥加密，使用密码块链（CBC）块加密模式进行数据加密。CBC块加密模式不使用填充。相反，InnoDB确保要加密的文本是块大小的倍数。

加密只支持每个文件的表空间、一般表空间和mysql系统表空间。对一般表空间的加密支持是在MySQL 8.0.13中引入的。对mysql系统表空间的加密支持从MySQL 8.0.16开始提供。其他表空间类型不支持加密，包括InnoDB系统表空间。

你不能从加密的逐表文件表空间、一般表空间或mysql系统表空间移动或复制一个表到不支持加密的表空间类型。

你不能从一个加密的表空间移动或复制一个表到一个未加密的表空间。然而，允许将表从一个未加密的表空间移动到一个加密的表空间。例如，你可以将一个表从未加密的按表文件或一般表空间移动或复制到加密的一般表空间。

默认情况下，表空间加密只适用于表空间中的数据。重做日志和撤销日志数据可以通过启用innodb_redo_log_encrypt和innodb_undo_log_encrypt进行加密。参见重做日志加密，和撤销日志加密。关于二进制日志文件和中继日志文件加密的信息，参见第17.3.2节，"加密二进制日志文件和中继日志文件"。

不允许改变驻留在或以前驻留在加密表空间中的表的存储引擎。
