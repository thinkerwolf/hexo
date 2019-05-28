---
title: MySQL使用SQL管理数据
date: 2017-06-01 13:07:54
categories: 
- MySQL
tags: 
- MySQL
- 数据库
---

# 使用SQL管理数据

## 数据表的创建、删除、索引和变更
MySQL允许创建、删除数据表或改变其结构，相应的SQL语句分别是CREATE TABLE、DROP TABLE、ALTER TABLE。CREATE INDEX和DROP INDEX语句用来给先用的数据表增加或删除索引。

### 存储引擎特征
| Engine             | Support | Commont                  | Transactions |   XA | Savapoints |
| :----------------- | ------- | :----------------------- | -----------: | ---: | ---------: |
| MEMORY             | YES     | 基于Hash，存储在内存中   |           NO |   NO |         NO |
| MRG_MYISAM         | YES     | 同一的MyISAM表集合       |           NO |   NO |         NO |
| CSV                | YES     | CSV存储引擎              |           NO |   NO |         NO |
| FEDERATED          | NO      | 同盟MySQL存储引擎        |         NULL | NULL |       NULL |
| PERFORMANCE_SCHEMA | YES     | 表现模式                 |           NO |   NO |         NO |
| MyISAM             | YES     | MyISAM存储引擎           |           NO |   NO |         NO |
| InnoDM             | DEFAULT | 支持事务、保存点、XA事务 |          YES |  YES |        YES |
| BLOCKHOLE          | YES     | /dev/null存储引擎        |           NO |   NO |         NO |
| ARCHIVE            | YES     | 存档存储引擎             |           NO |   NO |         NO |

#### 存储引擎的可移植性

先用mysqldump工具备份，然后把备份文件梵高另一台服务器主机，并通过加载备份文件重新创建该数据表。可移植性的另一层含义就是 **二进制可移植性**，指的是你可以直接把代表某个数据表的硬盘文件复制到另一台机器，并把它们安装到数据子目录下的地点，那台机器上的MySQL服务器就可以使用该数据表了。

### 创建数据表
CREATE TABLE几种重要的变体，可以灵活地构造数据表。
- 改变存储特性数据表选项。
- 只在数据表不存在才创建。
- 临时数据表，服务器在客户会话结束时自动删除它们。
- 从另一个数据表或是从一次SELECT查询的结果来创建数据表。
- 使用MERGE数据表、分区数据表、FEDERATED数据表
#### 数据表选项
CREATE TABLE mytbl (...) ENGINE = MEMORY;
CREATE TABLE mytbl (...) ENGINE = InnoDB;

服务器使用默认的存储引擎来创建数据表。内建的默认存储引擎时MyISAM，你可以通过--default-storage-engine选项来启动服务器。

#### 临时数据表
CREATE TEMPORARY TABLE tbl_name ... ;

#### 从其他数据表或查询结果创建数据表
MYSQL提供了两条语句帮助我们从其他的数据表或是从查询结果创建新的数据表。
- CREATE TABLE ... LIKE 语句将创建一个新的数据表作为原始数据表的一份空白副本。他将原始数据表的结构丝毫不差赋值过来，包括索引。不够新数据表内容一片空白，需要再使用一条语句(INSERT INTO ... SELECT )
- CREATE TABLE ... SELECT语句可以从任意一条SELECT语句的查询结果创建新数据表。**默认情况这条语句不会赋值所有数据列属性**，比如AUTO_INCREMENT，也不会赋值索引，因为结果集本身不带索引。

方式1Example
```
create table if not exists blog1 like blog;
insert into blog1 select * from blog;
select * from blog1;
```

方式2Example

```
create table if not exists blog_scores select a.*, b.Score from blog a, scores b;
select * from blog_scores;
```

#### 使用MERGE数据表
MERGE存储引擎是把一组MyISAM数据表当作一个逻辑单元来对待，让我们可以同时对它们进行查询，构成一个MERGE数据表的各成员MyISAM数据表必须有完全一样的结构。

假设几个日志数据表，内容是几年来每一年的日志记录，定义如下CC代表世纪，YY代表年份。
```
CREATE TABLE log_CCYY
(
    dt DATATIME NOT NULL,
    info VARCHAR(100) NOT NULL,
    INDEX(dt)
) ENGINE = MyISAM;
```
假设日志数据的集合包括log_2004、log_2005、log_2006，而创建一个如下MERGE数据表把他们归拢成一个逻辑单元：

```
CREATE TABLE log_merge 
(
    dt DATATIME NOT NULL,
    info VARCHAR(100) NOT NULL,
    INDEX(dt)
) ENGINE = MERGE UNION = (log_2004, log_2005, log_2006);
```

#### 使用分区数据表
MySQL5.1以上支持分区数据表（partitioned table）。分区在概念上与MERGE存储引擎很相似：他们都可以用来访问被分别存储在不同地点的多个数据表的内容。这两者的区别是：每个分区数据表都是一个货真价实的数据表，而不是一个用来累出各成员数据表的逻辑结构。此外，分区数据表可以使用MyISAM以外的存储引擎，而MERGE只能用MyISAM数据表构成。分区数据表有以下优点。
- 数据表的分区可以分布在多个设备上。
- 优化器可以把检索操作限定在莫尔格特定的分区还是同时搜索多个分区。

```
CREATE TABLE ... PARTITION BY {分区函数}...
```
分区函数把新数据分配到不同分区的依据可以是取值单位、值的列表或散列值。
- 依据取值范围分区，数据行包含的值可以划分为一系列互不冲突的区间，比如日期、收入水平、重量等。
- 根据列表进行分区。每个分区对应一个明确的值的列表，比如邮政码、电话区号、身份证地区编号。
- 根据散列值分区，根据数据行的键字计算出一个散列值，再根据这个散列值把数据分区到各分区，可以自定义散列函数，或者MySQL的内建散列函数。

分区表Example，根据年份把数据行分配到一个给定的分区：

```
CREATE TABLE log_partition 
(
    dt DATATIME NOT NULL,
    info VARCHAR(100) NOT NULL,
    INDEX(dt)
)
PARTITION BY RANGE(YEAR(dt)) 
(
    PARTITION p0 VALUES LESS THAN (2005),
    PARTITION p1 VALUES LESS THAN (2006),
    PARTITION p2 VALUES LESS THAN (2007),
    PARTITION p3 VALUES LESS THAN (2008),
    PARTITION p4 VALUES LESS THAN MAXVALUE
);
```
2009年时看可以对这个分区再进行划分，把2008年记录日志转移到自己分区，把2009年后的日志记录到MAXVALUE分区。

```
ALTER TABLE log_partition REORGANIZE PARTITION p4
INTO (
    PARTITION p4 VALUES LESS THAN (2009),
    PARTITION p5 VALUES LESS THAN (MAXVALUE)
);
```
默认情况下，分区被保存在分区数据表所属于的数据库的子目录里。如果想存储分配到其他位置（如另一个物理设备），需要使用DATA_DIRECTORY和INDEX_DIRECTORY分区选项。

#### 使用FEDRATED数据表
FEDERATED存储引擎可以让你访问其他主机上另一个MySQL服务器实际管理的数据表。

### 为数据表编制索引
#### 存储引擎的索引特性
- 为单个列编制索引，也可以为多个列构造复合索引。
- 索引可以包含独一无二或者重复的值。
- 为同一个表创建多个索引优化不同查询。
- 对于ENUM和SET以外的字符串类型数据，可以只为数据列的一个前缀创建索引，对坐标n个字符创建索引。(对于BLOB和TEXT数据列，必须指定前缀才能创建索引。)**为数据列前缀而不是整个数据列可以让索引本身更小加快访问速度**。

#### 创建索引
- UNIQUE唯一索引。索引项不允许出现重复值，复合索引数据项组合不允许重复。
- INDEX普通索引。允许出现重复值。
- FULLTEXT索引。只适用MyISAM数据表。
- SPATIAL。只适用MyISAM数据表。
- HASH。MEMORY数据表默认索引类型。可以改用BTREE索引代替。

ALTER TABLE比CREATE TABLE语句更灵活多能。比如：
```
ALTER TABLE tbl_name ADD INDEX index_name (index_columns);
ALTER TABLE tbl_name ADD UNIQUE index_name (index_columns);
ALTER TABLE tbl_name ADD PRIPARY KEY (index_columns);
```
```
CREATE TABLE mytbl 
(
   ... column definitions ...
   INDEX index_name (index_columns),
   UNIQUE index_name (index_columns),
   PRIPARY KEY (index_columns),
   ....
)
```
```
CREATE TABLE mytbl 
(
   id int(11) NOT NULL PRIPARY KEY,
   name varchar(100) NOT NULL UNIQUE,
   ....
)
```
等价与
```
CREATE TABLE mytbl 
(
   id int(11) NOT NULL,
   name varchar(100) NOT NULL,
   PRIMARY KEY (id),
   UNIQUE (name),
   ....
)
```
#### 删除索引
可以使用DROP INDEX或ALTER TABLE完成。


```
ALTER TABLE tbl_name DROP INDEX index_name;
ALTER TABLE tbl_name DROP PRIMARY KEY;
```

## 获取数据库元数据


## 利用联结操作对多个数据表进行检索

### 内联结
```
SELECT ... FROM t1 INNER JOIN t2 WHERE t1.i1 = t2.i2;
```
根据某个数据表的每一个数据行与另一个数据表里的每一个数据行得到全部可能的组合的联结操作叫做生成 **笛卡尔积**。这样联结数据表会产生很多结果数据行。
- 可以用ON子句代替WHERE子句，不管被联结的数据列是否同名，ON子句都可以使用。
SELECT ... FROM t1 INNER JOIN t2 ON t1.i1 = t2.i2;
- 另一个语法时使用USING()子句，概念上类似与ON子句，但要求被联结的数据列必须同名。
 SELECT mytbl1.\*, mytbl2.\* from mytbl1 INNER JOIN mytbl2 USING(b);

### 左联结和右联结（外联结）
内联结只显示在两个数据表里都能找到匹配的数据行。外联结除了显示同样的匹配结果，还可以把其中一个数据表在另一个数据表没有匹配的数据行也显示出来。LEFT JOIN把左数据表在右数据表没有匹配的数据行也显示出来。RIGHT JOIN刚好相反。外联结非常适合解决“哪些值事故缺失的”这个问题。
```
select * from blog b LEFT JOIN scores s ON b.id = s.Id;
```
```
select * from blog b LEFT JOIN scores s WHERE b.id = s.Id;
```
> 在进行左联结或者右联结使用WHERE进行两表联结会产生语法错误


```
SELECT 
    *
FROM 
    student s INNER JOIN grade_event g 
    LEFT JOIN socre s ON s.student_id = g.student_id 
                        and g.event_id = s.event_id
WHERE s.socre IS NULL
ORDER BY s.student_id, g.event_id;

```

## 用子查询进行多数据表检索
子查询的结果可以用以下不同的办法进行测试。
- 标量子查询的结果可以用“=”或“<”之类的比较操作求值。
- 可以用In和NOT IN操作符来测试某给定的值是否包含在子查询的结果集里。
- 可以用ALL、ANY和SOME操作符把某给定值与子查询的结果集进行比较。
- 可以用EXISTS和NOT EXISTS操作符来测试子查询的结果集是否为空。

### IN和NOT IN
如果子查询返回多个数据行，可以用IN和NOT IN操作符来构造主查询的检索条件。

```
SELECT last_name, first_name, city, state FROM president 
WHERE (city, state) IN
(SELECT city, state FROM president 
 WHERE last_name = 'Roosevelt');
```

### ALL、ANY和SOME子查询
ALL和ANY操作符的常见用法是结合一个相对比较操作符对一个数据列子查询的结果进行测试。

```
SELECT last_name, first_name, birth FROM president
WHERE birth <= ANY (SELECT birth FROM president);
```

当ALL、ANY或SOME操作符与“=”操作符配合使用，子查询可以是一个数据表子查询。此时，你需要使用一个数据行构造器来提供与子查询返回的数据进行比较的比较值。
```
SELECT last_name, first_name, city, state FROM president
WHERE (city, state) = ANY 
(SELECT city, state FROM president 
WHERE last_name = 'Roosevelt');
```
IN和NOT IN操作符是=ANY和<>ALL的简写。IN的含义“等于子查询所返回的某个数据行”，NOT IN是“不等于子查询返回的所有数据行”。

### EXISTS和NOT EXISTS子查询
这两个操作符只测试某个子查询是否返回了数据行。
```
SELECT EXISTS (SELECT * FROM absence);
SELECT NOT EXISTS (SELECT * FROM absence);
```

### 与主查询相关的子查询
子查询可以与主查询相关，也可以与之无关。
- 与主查询无关的子查询不引用主查询的任何值。
- 与主查询相关的子查询需要引用主查询里的值，所以必须依赖主查询。

### 把子查询改写为联结查询
有时候，联结查询比子查询执行效率高。如果一条使用了子查询的SELECT语句需要执行很长时间，就应该尝试把它改写成一个联结查询，看它是不是执行得更好。

#### 改写用来选取匹配值的子查询
下面例子从score数据表把考试（不包含小测验）成绩筛选出来：
```
SELECT * FROM score WHERE event_id IN (SELECT event_id FROM grade_event WHERE category='T');
```
可以改写成
```
SELECT s.* FROM score s INNER JOIN grade_vent g ON s.event_id = g.event_id WHERE g.category = 'T';
```

#### 改写用来选取非匹配（确实）值的子查询
子查询语句的另一种常见的用法是检索一个数据表有，另一个数据表没有的值。与“哪些没有出现”的有关问题可以用LEFT JOIN来解决。

下面例子是查询没有出现在absence表的学生：
```
SELECT * FROM student WHERE student_id NOT IN (select student_id FROM absence);
```
可以改写成
```
SELECT s.* FROM student LEFT JOIN absence a ON s.student_id = a.student_id WHERE a.student_id IS NULL;
```

## 用UNION语句进行多数据表检索
UNION语句有以下特性。

**数据列的名字和数据类型**。UNION结果集里的数据列名字来自第一个SELECT语句里的数据列的名字。UNION中第二个和后面必须选取个数相同的数据列，但各个有段数据列不必有相同的名字和数据类型。MySQL会自动进行类型转换。
```
select task_id, intro from task UNION select user_id, create_time from blog;

+---------+---------------------+
| task_id | intro               |
+---------+---------------------+
| 2321    | 击毙本拉登          |
| 21      | 2018-12-28 10:23:39 |
| 42      | 2019-03-18 11:25:00 |
+---------+---------------------+
```

重复数据行的处理。在默认的情况下，UNION将从结果集剔除重复的数据行。如果像保留重复的数据行，需要把每个UNION都改为UNION ALL。**如果把UNION和UNION ALL混杂使用，每个UNION优先于左边任何UNION ALL操作**。

**ORDER BY和LIMIT处理。**

```
select task_id, intro from task UNION select user_id, create_time from blog ORDER BY intro LIMIT 1;
```

## 使用视图
视图是一种虚拟的数据表，它们的行为和数据表一样，并不真正包含数据。它们是永底层（真正的）数据表或其他视图定义出来的“假”数据表，用来提供查看数据表数据的另一种方法，可以简化应用程序。

## 涉及多个数据表的删除和更新操作
可以根据某个数据表里数据是否在另一个数据表里有匹配来删除它们。联结概念在用来完成这些操作的语句扮演十分重要的角色。
```
DELETE FROM t1 LEFT JOIN t2 ON t1.id = t2.id WHERE intro IS NULL;
DELETE FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```
## 事务处理

## 外键和引用完整性

