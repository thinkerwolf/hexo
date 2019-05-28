---
title: MySQL数据类型
date: 2017-06-01 13:08:54
categories: 
- MySQL
tags: 
- MySQL
- 数据库
---
# MySQL数据类型
要理解透彻MySQL如何处理数据，必须注意一下问题。
- 包括NULL值在内，MySQL能够表达哪几大类型的值
- MySQL有哪些数据类型及其属性和特点，有哪些是常用的。有哪些有着特殊的行为特点。
- 服务器的SQL模式如何影响坏数据的处理方式。严格模式拒绝坏数据。
- MySQL的表达式求值规则。类型转换规则是MySQL表达式求值规则的重要组成部分之一。
- 如何为数据列选择正确的数据类型。

## MySQL的数据类型
在创建表列时你可以确定以下问题。
- 哪种类型值保存其中。
- 类型值占用多少存储空间。
- 值的长度固定不变还是可变
- 类型的值如何比较和存储。
- 能否对这种类型编制索引。

### 数据类型概述
#### 数值数据类型



| 数据类型  | 含义           |
| --------- | -------------- |
| TINYINT   | 非常小的整数   |
| SMALLINT  | 小整数         |
| MEDIUMINT | 中等大小的整数 |
| INT       | 标准的整数     |
| BIGINT    | 大整数         |
| DECIMAL   | 定点数         |
| FLOAT     | 单精度浮点数   |
| DOUBLE    | 双精度浮点数   |
| BIT       | 位字段         |

#### 字符串数据类型

| 数据类型    | 含义                                               |
| ----------- | -------------------------------------------------- |
| CHAR        | 固定长度的非二进制（字符）字符串                   |
| VARCHAR     | 可变长度的非二进制字符串                           |
| BINARY      | 固定长度的二进制字符串                             |
| VARBINARY   | 可变长度的二进制字符串                             |
| TINYBLOB    | 非常小的BLOB（二进制大对象）                       |
| BLOB        | BLOB                                               |
| MEDIUMBLOB  | 中等大小的BLOB                                     |
| LONGBLOB    | 大BLOB                                             |
| TINYTEXT    | 非常小的非二进制文本                               |
| TEXT        | 小文本字字符串                                     |
| MEFDIUMTEXT | 中等大小的非二进制字符串                           |
| LONGTEXT    | 大的非二进制字符串                                 |
| ENUM        | 枚举集合                                           |
| SET         | 集合。数据列的取值可以是零或者这个集合中的多个元素 |

#### 日期数据类型

| 数据类型  | 含义                                      |
| --------- | ----------------------------------------- |
| DATE      | 日期值，格式为‘CCYY-MM-DD’                |
| TIME      | 时间值，格式为‘hh:mm:ss’                  |
| DATATIME  | 日期加时间值，格式为‘CCYY-MM-DD hh:mm:ss’ |
| TIMESTAMP | 时间戳值，格式为‘CCYY-MM-DD hh:mm:ss’     |
| YEAR      | 年份值，格式为CCYY或YY                    |

### 数值数据类型
MySQL的数值类型分3大类。
- 精确值类型，包括整数类型和DECIMAL。
- 浮点类型，分为单精度（FLOAT）和双精度（DOUBLE）
- BIT保存位字段值。

可以与Java的基本数据类型进行类比了解其内存占用和取值范围。

INT[(M)]、FLOAT[(M, D)]等，M代表整数类型**最大显示宽度**（与整数需要多少字节毫不相干）、浮点类型和DECIMAL类型的精度（小数点后面的数字）以及BIT类位数。对有小数部分的数据类型，D代表数学精确度（小数点后面个数）。M取值范围1-255之间。D是一个0-30之间的整数值且不能大于M。

UNSIGNED属性不允许数据列里出现负数值。可以修饰出BIT外所有数值类型。UNSIGNED属性不会改变数据列的取值范围“长度”，只是把范围朝着正数方向平移了。
比如TINYINT加上UNSIGNED后从-2<sup>7</sup>-1到2<sup>7</sup>变为了0到2<sup>8</sup>

### 字符串数据类型
#### CHAR和VARCHAR
CHAR和VARCHAR用来保存非二进制字符串，因此与字符集和排序方式相关联。
- CHAR是固定长度，VARCHAR是可变长度的。
- 从CHAR检索出来的尾椎会被去掉。
- VARCHAR尾缀空格在存储和检索时都会被保留。

#### BINARY和VARBINARY类型
BINARY和VARBINARY与CHAR和VARCHAR相似。但是存在以下区别：
- VARCHAR和CHAR是非二进制的，在列中中以字符串形式存储。
- BINARY和VARBINARY是二进制的，在列中以是一串字节，不涉及字符集和排序方式，比较的是字节大小。

#### BLOB和TEXT类型
BLOB是binary large Object的缩写（二进制大对象）。BLOB存储的是二进制字符串，用来存储图像、视频、图像等大对象比较合适。

TEXT与BLOB有很多相似之处，但是其存储的是非二进制字符串。能否在BLOB和TEXT建立索引取决以下条件。

#### 挑选字符串类型
选用字符串类型时需要考虑一下问题。

**应该使用二进制类型还是非二进制类型。**

**比较操作需要区分字母大小情况吗？**

**需要尽量占用少的空间？** 使用可变长度类型。

**数据列的可选值总是某几个值的集合？** 选用ENUM或SET比较好。

**尾缀的空格（或者零值字节）是否很重要？** 选用VARCHAR或者VARBINARY

### 日期、时间数据类型
MySQL提供DATE、TIME、DATETIME、TIMESTAMP和YEAR几种数据类型。

类型定义 | 取值范围
---|---
DATE | '1000-01-01' 到 '9999-12-31'
DATETIME | '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'
TIME | '-838:59:59' 到 '838:59:59'
TIMESTAMP | '1970-01-01 00:00:01' 到 '2038-01-19 03:14:07'
YEAR | YEAR(4)1901-2155; YEAR(2)1970-2069

#### DATE、TIME和DATAEIME
DATETIME类型代表是几点几分，必须落在''00:00:00'到''23:59:59'范围内。TIME值代表是一段逝去的时间。
```
mysql> create table t (dt DATETIME, d DATE, t TIME) engine = MyISAM;
mysql> INSERT INTO t values (now(), now(), now());
mysql> SELECT * from t;
+---------------------+------------+----------+
| dt                  | d          | t        |
+---------------------+------------+----------+
| 2019-03-18 14:20:58 | 2019-03-18 | 14:20:58 |
+---------------------+------------+----------+
```

#### TIMESTAMP类型
TIMESTAMP用来保存日期和时间的组合。TIMESTAMP以标准UTC格式存放，在存储时自动把别的时区转化为UTC，在查询时转化为原始的时区。

TIMESTAMP具有自动初始化和自动更新属性，可以设置其有其中一个或者两个属性。
- 自动初始化：在INSERT没有给出值或者NULL，自动取值为当前时间戳。
- 自动更新：当修改其他数据列的值时TIMESTAMP自动更新为当前时间戳。

> ts TIMESTAMP [DEFAULT constant_value] [ON UPDATE CURRENT_TIMESTAMP]

- 如果给出DEFAULT CURRENT_TIMESTAMP短语，数据列自动具备自动初始化属性。如果还有ON UPDATE CURRENT_TIMESTAMP短语，还具备自动更新属性。
- 如果省略两个属性，默认定义为DEFAULT CURRENT_TIMESTAMP和ON UPDATE CURRENT_TIMESTAMP属性。
- 如果DEFAULT constant_value给出常数值，将不具备自动初始化属性。

如果想让数据表里第二个或更靠后的某个TIMESTAMP数据列具备自动初始化或自动更新属性，就必须在定义第一个TIMESTAMP时明确给出DEFAULT constant_value属性且不能给出ON UPDATE CURRENT_TIMESTAMP属性。
？？？？新版本貌似不是这样。。。

TIMESTAMP数据列的定义可以包含NULL或NOT NULL。默认这是NOT NULL，效果是明确把TIMESTAMP数据列设置为NULL，MYSQL会把它设置为当前的时间戳值（插入和修改操作）。如果TIMESTAMP定义了NULL属性，把数据列设置为NULL，MySQL将把NULL而不是当前时间戳存入该数据列。

#### 日期/时间数据类型属性
以下适用于除TIMESTAMP以外的所有日期/时间类型。
- 可以加上通用的NULL或NOT NULL属性，没设定默认NULL。
- DEFAULT子句设置一个默认值。

## MySQL处理非法数据类型
MySQL正常模式下对“坏数据”是以一种尽量兼容方式或者截取的方式来存储。执行完INSERT、REPLACE、UPDATE、ALTER TABLE等语句是可能发生坏数据的情况，这是用SHOW WARNINGS查看警告内容。

如果想在插入或修改采用更严格是检查，可以启用一下两种SQL模式之一：

```
SET sql_mode = 'STRICT_ALL_TABLES'; /** 严格模式 */
SET sql_mode = 'STRICT_TRANS_TABLES'; /** 严格事务模式 */
SET sql_mode = 'TRADITIONAL'; /* 严格模式和附加检查 */
```

## 序列
//TODO  很重要 之后研究

## 表达式求值和类型转换

## 数据类型的选用


