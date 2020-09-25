## count(1)和count(*) 哪个效率高？

CodeSheep

------

 **count(1)和count(\*)对比** 

当表的数据量大些时，对表作分析之后，使用 `count(1)`还要比使用 `count(*)`用时多了！

从执行计划来看， `count(1)` 和 `count(*)`的效果是一样的。但是在表做过分析之后， `count(1)` 会比 `count(*)`的用时少些（1w以内数据量），不过差不了多少。

如果 `count(1)`是聚索引，那肯定是 `count(1)`快，但是差的很小。因为 `count(*)`自动会优化指定到那一个字段，所以没必要去 `count(1)`，用 `count(*)` sql会帮你完成优化的，因此：`count(1)` 和 `count(*)`基本没有差别！

------

 **count(1)和count(列名)对比** 

两者的主要区别是：

- `count(1)` 会统计表中的所有的记录数，包含字段为 `null` 的记录。
- `count(字段)` 会统计该字段在表中出现的次数，忽略字段为 `null` 的情况。即不统计字段为 `null` 的记录。

------

 **count(\*)、count(1)和count(列名)区别** 

**执行效果上：**

- `count(*)`包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL
- `count(1)`包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL
- `count(列名)`只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

**执行效率上：**

- 列名为主键， `count(列名)` 会比 `count(1)`快
- 列名不为主键， `count(1)` 会比 `count(列名)`快
- 如果表多个列并且没有主键，则 `count(1)` 的执行效率优于 `count(*)`
- 如果有主键，则 `selectcount(主键)` 的执行效率是最优的
- 如果表只有一个字段，则 `selectcount（*）`最优。

**实例分析：**

```mysql
mysql> create table counttest(name char(1), age char(2));Query OK, 0 rows affected (0.03 sec)
mysql> insert into counttest values    -> ('a', '14'),('a', '15'), ('a', '15'),     -> ('b', NULL), ('b', '16'),     -> ('c', '17'),    -> ('d', null),     ->('e', '');Query OK, 8 rows affected (0.01 sec)Records: 8  Duplicates: 0  Warnings: 0
mysql> select * from counttest;+------+------+| name | age  |+------+------+| a    | 14   || a    | 15   || a    | 15   || b    | NULL || b    | 16   || c    | 17   || d    | NULL || e    |      |+------+------+8 rows in set (0.00 sec)
mysql> select name, count(name), count(1), count(*), count(age), count(distinct(age))    -> from counttest    -> group by name;+------+-------------+----------+----------+------------+----------------------+| name | count(name) | count(1) | count(*) | count(age) | count(distinct(age)) |+------+-------------+----------+----------+------------+----------------------+| a    |           3 |        3 |        3 |          3 |                    2 || b    |           2 |        2 |        2 |          1 |                    1 || c    |           1 |        1 |        1 |          1 |                    1 || d    |           1 |        1 |        1 |          0 |                    0 || e    |           1 |        1 |        1 |          1 |                    1 |+------+-------------+----------+----------+------------+----------------------+5 rows in set (0.00 sec)
```