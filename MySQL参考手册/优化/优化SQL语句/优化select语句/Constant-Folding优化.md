# Constant-Folding优化

常量值与列值之间的比较（其中常量值超出范围或列类型的类型错误）现在在查询优化期间处理一次，而不是在执行期间逐行处理。可以用这种方式处理的比较是>，>=，<，<=，<>/！=，=，和<=>。

考虑以下语句创建的表：

`CREATE TABLE t (c TINYINT UNSIGNED NOT NULL);`

查询`SELECT*FROM t WHERE c<256`中的WHERE条件包含整数常量256，该常量超出了TINYINT UNSIGNED列的范围。以前，这是通过将两个操作数都视为较大的类型来处理的，但现在，由于c的任何允许值都小于常量，WHERE表达式可以改为折叠为WHERE 1，这样查询将重写为SELECT*FROM t WHERE。

这使得优化器可以完全删除WHERE表达式。如果列c可为空（即，仅定义为TINYINT UNSIGNED），查询将被重写如下：

`SELECT * FROM t WHERE ti IS NOT NULL`

与支持的MySQL列类型相比，对常量执行折叠，如下所示：

- 整数列类型。整数类型与以下类型的常量进行比较，如下所述：

  - 整数值 Integer。如果常量超出列类型的范围，则比较将折叠为1或为NOT NULL，如前所示。

    如果常量是范围边界，则比较将折叠为=。例如（使用已定义的相同表）：

    ```sql
    mysql> EXPLAIN SELECT * FROM t WHERE c >= 255;
    *************************** 1. row ***************************
            id: 1
    select_type: SIMPLE
            table: t
    partitions: NULL
            type: ALL
    possible_keys: NULL
            key: NULL
        key_len: NULL
            ref: NULL
            rows: 5
        filtered: 20.00
            Extra: Using where
    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS;
    *************************** 1. row ***************************
    Level: Note
    Code: 1003
    Message: /* select#1 */ select `test`.`t`.`ti` AS `ti` from `test`.`t` where (`test`.`t`.`ti` = 255)
    1 row in set (0.00 sec)
    ```

  - 浮点或定点值 Floating- or fixed-point。如果常量是十进制类型之一（如decimal、REAL、DOUBLE或FLOAT），并且具有非零小数部分，则它不能相等；相应地折叠。对于其他比较，根据符号向上或向下舍入到一个整数值，然后执行范围检查和处理，如前面对整数比较所述。

    如果REAL值太小，无法表示为小数，则将其舍入为.01或 -.01，然后作为十进制处理。

  - 字符串类型。尝试将字符串值解释为整数类型，然后处理整数值之间的比较。如果失败，请尝试将该值处理为REAL。

- DECIMAL或REAL列。十进制类型与以下类型的常量进行比较，如下所述：

  - 整数值。对列值的整数部分执行范围检查。如果没有折叠结果，请将常量转换为小数位数与列值相同的DECIMAL，然后将其检查为DECIMAL（请参见下一步）。

  - DECIMAL或REAL值。检查溢出（即，常量的整数部分是否比列的十进制类型允许的数字多）。如果是，请折叠。

    如果常量的有效小数位数大于列的类型，请截断该常量。如果比较运算符为=或<>，请折叠。如果运算符>=或<=，则由于截断而调整运算符。例如，如果列的类型为DECIMAL（3,1），`SELECT*FROM t WHERE f>=10.13`将变为`SELECT*FROM t WHERE f>10.1`。

    如果常量的小数位数少于列的类型，请将其转换为具有相同位数的常量。对于REAL值的下溢（即小数位数太少，无法表示它），请将常量转换为小数0。

  - 字符串值。如果该值可以解释为整数类型，则按此处理。否则，试着把它当作真实的。

- FLOAT或DOUBLE列。与常量比较的FLOAT（m，n）或DOUBLE（m，n）值处理如下：

  如果值超出列的范围，请折叠。

  如果值的小数位数超过n，则截断，在折叠期间进行补偿。对于=和<>比较，如前所述，折叠为TRUE、FALSE或IS[NOT]NULL；对于其他操作员，调整操作员。

  如果该值的整数位数超过m，请折叠。

局限性此优化不能用于以下情况：

1. 使用BETWEEN或IN进行比较。

2. 具有BIT列或使用日期或时间类型的列。

3. 在准备语句的准备阶段，尽管它可以在优化阶段应用，当准备语句实际执行时。这是因为在语句准备期间，常数的值还不知道。
