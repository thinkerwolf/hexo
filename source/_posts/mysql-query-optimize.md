---
title: MySQL查询优化
date: 2017-06-01 13:07:54
categories: 
- MySQL
tags: 
- MySQL
- 数据库
---

# 查询优化

首先学会分析一个查询语句的执行过程，包括带索引和不带索引的查询区别。
- 为数据表创建索引是数据库服务器更快查阅数据。
- 如何写出以最大程序地利用那些索引，使用EXPLAIN语句检查MySQL服务器如期行事。
- 编写查询影响服务器调度机制。
- 修改服务器的操作以提高工作效率。
- 分析底层硬件做什么，解决物理限制，提高性能。

## 使用索引
通常，能够造成查询速度最大差异的是索引的正确使用。在许多情况下，假如不适用索引，视图通过其他方式提高性能纯粹浪费时间。应该首先使用索引，在看看是否有其他解决办法。

### 索引的优点
一个没有索引的数据表就是无序的数据行集合。如下图没有索引的表，在进行查询时，就需要扫描表的每一行，如果数据量很大，且仅有少数记录匹配，工作过程效率很低。

![image](/images/mysql/query_optimize1.png)

如果为company_num添加索引，如下图所示，如果以company_num = 13进行查询时，先定位到索引表，定位到索引表后会发现3条符合的记录，遇到14时就不在查询，此时就会直接返回数据，避免了全表扫描的问题。

![image](/images/mysql/query_optimize2.png)

不同的存储引擎索引的实现细节也不相同。MyISAM数据表，索引值在索引文件里，一个表可以有多个索引，所有的索引都存储在同一个索引文件里。InnoDB没有按照数据行和索引值分开。InnoDB存储引擎使用的是一个表空间，在这个表空间管理着所有INnoDB类型数据表的数据和索引的存储。可以通过配置让InnoDB为每个数据表分别创建一个它字节表空间。

假定一个带索引查询语句并分析其查询过程：
```
SELECT t1.i1, t2.i2, t2.i3 FROM t1 INNER JOIN t2 INNER JOIN t3 WHERE t1.i1 = t2.i2 AND t2.i2 = t3.i3;
```
1. 从数据表t1选择第一行，看这个数据包含什么样的值。
2. 对t2使用索引，直接找到与数据表t1的值相匹配的数据行，t3类似。
3. 对数据表t1的下一行数据行重复上面过程，知道检查数据表t1的所有数据行。

使用索引就可以避免对t1、t2、t3进行笛卡尔积全表搜索导致搜索效率极其低下。

MySQL使用索引的方式有以下几种。
- 一是把WHERE字句所给的条件相匹配的数据行尽快找出来；二是关联操作中把与其他数据表里的数据行相匹配的数据尽快找出来。
- MIN()和MAX()函数的值可以迅速被找到。
- 迅速完成ORDER BY和GROUP BY子句的分类和分组操作。
- 避免为一个查询整体读取整行。、、、

### 索引的缺点
**索然有缺点，但是使用索引是优点大于缺点。**

首先，索引加快了检索速度，但是却降低了插入、删除和修改数值的速度。出现这种情况是因为写入一条数据行，会要求所有索引都要做出改变，索引越多，需要做出改变就越多。对于写操作多的数据表的索引更新方面的开销会很大。

其次，索引占据磁盘空间，多个索引会占据更大的空间。

### 挑选索引
**尽量为用来搜索，分类或分组的数据列编制索引，不要为输出显示的数据列编制索引。** 换句话说就是尽量在WHERE子句、JOIN子句、GROUP BY和ORDER子句编制索引。
```
SELECT
 t1.col_name, t1.col_name   -> do not add index
FROM t1 
    LEFT JOIN t2
     ON t1.id = t2.id  -> add index
WHERE
    t1.id >= 190       -> add index
ORDER BY 
    t1.socre           -> add index       
```

**综合考虑各数据的维度势。** 数据列的“维度”等于它所能容纳的==非重复值的个数==。维度越高，重复的值越少，索引的使用效果就越好。

**对短小的值进行索引。** 短小的值速度快，体积小，索引块可以容纳更多键值，减少每次需要检索的索引块。

**为字符串值的前缀编索引。**
当为字符串列编索引，尽可能给出前缀长度（前缀长度中的字符都是唯一的或大部分唯一）。可以减少索引长度和体积和不必要的磁盘IO。

**充分利用最左边的前缀（复合索引）。** 如果创建了一个n个数据的复合索引。一个复合索引工作时就相当于n个索引，索引中最左边的数据列集合能够用于匹配数据行。假设一个索引是(state, city, zip)的组合，可以生效的索引组合为：
```
state, city, zip
state, city
state
```
==MySQL不能使用不包含最左边前缀的索引==。

**让索引的类型与进行比较操作的类型保持匹配。**
InnoDB使用默认BTREE索引，MyISAM默认也是用BTREE索引，还有HASH索引可选择。

**利用慢查询找出性能低劣的查询。** 
使用mysqldumpslow查询日志，找出执行慢的查询语句。

## MySQL查询优化程序
使用***EXPLAIN***会返回SQL语句执行的信息。对分析SQL执行过程和检查优化器的操作过程非常有用。
```
mysql > EXPLAIN select task_id, intro, title, content from task left join blog on task_id = user_id;
+----+-------------+-------+------------+------+---------------+-------------+---------+------------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key         | key_len | ref              | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+-------------+---------+------------------+------+----------+-------------+
|  1 | SIMPLE      | task  | NULL       | ALL  | NULL          | NULL        | NULL    | NULL             |  1   |  100.00  | NULL        |
|  1 | SIMPLE      | blog  | NULL       | ref  | blog_ibfk_1   | blog_ibfk_1 | 5       |test.task.task_id |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+-------------+---------+------------------+------+----------+-------------+
```
### 查询优化器的工作原理
**对数据表进行分析。**
这将生成索引值分布情况的统计数据，可以帮助优化器对索引的使用作出更准确的效果。可以根据数据表的更新频率去手动调用ANALYSE TABLE语句。

**使用EXPLAIN去分析语句。**

**尽量使用数据类型相同的数据列进行比较。** 数据类型不同相比于数据类型相同效率会低一些。比如INT/INT或BIGINT/BIGINT就比INT/BIGINT块。

**使带有索引的列在比较表达式中单独出现。**
如果在函数调用中引用索引数据列，或者在计算表达式中引用索引数据列，会造成索引失效。比如：
```
WHERE index_col * 2 < 4;
WHERE index_col < 4 / 2;
```
两者的效果完全不同，第一行索引失效，第二行索引正常运行。

**不要在LIKE表达式中开始位置使用通配符。**
```
WHERE index_col LIKE '%mac%'
```
这将会使索引失效，可以使用如下方式使索引生效。
```
WHERE index_col LIKE 'mac%'
```

**避免过多使用MySQL的自动类型转换。**

### 用EXPLAIN语句检查优化器操作
EXPLAIN语句提供的信息可以帮助我们了解优化器为处理各种语句而生成的执行计划。
- 连接不同方式编写出来的查询命令是否影响索引使用。
- 了解给数据表增加索引对优化器生成高效率执行计划的能力产生什么影响。

假定一个EXPLAIN语句：
```
EXPLAIN SELECT
	b.user_id,
	b.title,
	b.content,
	b1.title AS title1 
FROM
	blog b
	INNER JOIN blog1 b1 ON b1.user_id = b.user_id ;
```
可以查看在不加索引和加索引在Explain的结果
![image](/images/mysql/query_optimize3.png)

![image](/images/mysql/query_optimize4.png)


### 为提高查询效率挑选数据类型
**尽量使用数值操作，少用字符串操作。**
数值运算通常比字符串运算更快。

**可以使用“小类型”，就不要使用“大类型”。**
“小”类型比“大”的处理速度更快，尤其是字符串类型。而且小类型在磁盘读写上的开销更小。

**选择适用于对应存储引擎的格式。**
对于MyISAM数据表，应该尽量选用固定长度的数据列而不是可变长度的数据列。

**尽量把数据列声明为NOT NULL。**
如果数据列具有NOT NULL属性，对它的处理可以更快完成，因为MySQL不需要在查询处理期间检查该数据列的值是不是NULL。还可以为每个数据航节约一个位的存储空间。

**考虑使用ENUM数据列。**

**利用PROCEDURE ANALYSE()语句。**
分析数据表，看看它对数据的声明提出哪些建议。

**对容易产生碎片的数据表进行整理。**
数据表-尤其是哪些包含可变长度数据列的数据表-往往因为数据的大量修改产生碎片。随着时间推移，你将不得不读取更多的存储块才能把数据行读入内存，无疑会对性能产生影响。定期使用mysqldump工具删除表再重新创建是一个解决碎片的方法。
```
mysqldump -uroot -p123 db_name tbl_name > dump.sql
```

**把数据压缩到BLOB或TEXT数据列。**
将数据转换成XML或者JSON字符串进行存储到BLOB或者TEXT中。

**\*\*使用人造索引。**
具体做法是根据其他数据列计算出一个散列值存储在另一个数据列中，然后通过搜索散列值去检索数据。（这种方法只是用精确查找，不适用>=、<=、<、>查找）可以使用MD5()、SHA1()或CRC32()都可以生产散列。数据类型的散列值的存储效率是非常高效的。人造散列对BLOB或TEXT数据列的检索非常有效。

**尽量把BLOB或TEXT数据剥离到单独的一张表中。**
既能减少原始数据表的碎片，享受固定长度带来的好处。又你会在SELECT *原始表时把大量的数据通过网络传输给客户端。

## 有效加载数据（插入数据）
- 批量加载的效率比单数据行加载效率高。因为键缓存在每一次输入记录加载之后都不需要刷新，可以在批量记录结束的时候再刷新。越是减少键缓存的刷新次数，数据加载越快。
- 加载有索引的数据表比加载无索引的数据表慢一些。
- 较短的SQL语句的数据加载比较长的语句快。

如果使用多个INSERT语句，尽可能对他们分组来减少索引的刷新次数。对于事务性存储引擎，可以通过发起事务来实现：
```
START TRANSACTION;
INSERT INTO tbl_name ...;
INSERT INTO tbl_name ...;
INSERT INTO tbl_name ...;
...
COMMIT;
```
对于非事务形存储引擎，可以采取锁定的方式实现：
```
LOCL TABLES `tbl_name` WRITE;
INSERT INTO tbl_name ...;
INSERT INTO tbl_name ...;
INSERT INTO tbl_name ...;
...
UNLOCKTABLES;
```

对于MyISAM数据表，减少索引刷新次数的另一个策略是使用DELAY_KEY_WRITE数据表选项。使用这个选项后，数据行会像往常写入数据，但是键缓存只在有必要时刷新。想要使用DELAY_KEY_WRITE，必须在启动mysqld是给出--delay-key-write=ALL选项。

在为MyISMA数据表开启“键值缓写”的功能时，在服务器挂掉后可能会让索引值丢失，但是MyISAM可以根据数据行修复。应该在启动服务器给出--myisam-recover=FORCE选项。

你通常希望避免被写入的数据表长时间运行SELECT查询，这样会导致争用问题和较差的性能。如果写入大部分是INSERT操作，可以先将数据写到辅助表中，然后定期将这些数据写到主表中，使用这样办法的前提是对SELECT查询数据实时性要求不高。

## 调度和锁定问题
可以改变语句的调度优先权，可以使查询更协调的工作。改变优先权可以用于改变这个策略的选项。MySQL的默认调度策略如下。
- 写入比读取有更高的优先权。
- 对数据表的写操作必须按照请求顺序来。
- 对同一个数据表进行的读操作可以同时进行。

对于MyISAM、MERGE、MEMORY存储引擎，调度策略是由数据表帮助完成的。只要有客户程序访问数据表，必须先锁定它。客户端完成操作，锁定才能解除。

MySQL提供一些语句修饰符改变调度策略。一是DELETE、INSERT、LOAD DATA、REPLACE、UPDATE中使用LOW_PRIORITY。而是在SELECT和INSERT语句中使用HIGH_PRIORITY关键字，三是在INSERT和REPLACE语句中使用DELAYED关键字。

LOW_PRIORITY和HIGH_PRIORITY只对支持锁定功能的存储引擎（MyISAM、MERGE、MEMORY）有效果。DELAYED限定在MyISAM、MEMORY、ARCHIVE。

### 改变语句的执行优先级
通过改变SELECT和INSERT的优先级来确定SELECT和INSERT的顺序。通过LOW_PRIORITY和HIGH_PRIORITY关键字来确定SELECT和INSERT语句的优先级。

### 使用延迟插入
//TODO
### 使用并发插入
//TODO
### 锁定级别与并发性
//TODO