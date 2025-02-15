# InnoDB锁定和事务模型

<https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-transaction-model.html>

15.7.1 InnoDB锁定
15.7.2 InnoDB事务模型
15.7.3 InnoDB中不同SQL语句设置的锁
15.7.4 虚拟行
15.7.5 InnoDB死锁
15.7.6 交易安排

要实现大型、繁忙或高度可靠的数据库应用程序，从不同的数据库系统移植大量代码，或调整MySQL性能，了解InnoDB锁定和InnoDB事务模型非常重要。

本节讨论与InnoDB锁定和您应该熟悉的InnoDB事务模型相关的几个主题。

- 第15.7.1节“InnoDB锁定”描述了InnoDB使用的锁定类型。

- 第15.7.2节“InnoDB事务模型”描述了事务隔离级别和每个级别使用的锁定策略。它还讨论了自动提交、一致非锁定读取和锁定读取的使用。

- 第15.7.3节“InnoDB中不同SQL语句设置的锁”讨论了InnoDB针对不同语句设置的特定类型的锁。

- 第15.7.4节“虚行”描述了InnoDB如何使用下一个键锁定来避免虚行。

- 第15.7.5节“InnoDB中的死锁”提供了一个死锁示例，讨论了死锁检测，并提供了最小化和处理InnoDB死锁的技巧。
