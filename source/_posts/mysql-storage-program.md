---
title: MySQL存储程序
date: 2017-06-01 13:07:54
categories: 
- MySQL
tags: 
- MySQL
- 数据库
---

# 存储程序

- 存储函数（stored function）。返回一个计算结果，结果可用在表达式里。
- 存储过程（stored procedure）。不返回结果，但是可以完成一般的运算或是生成一个结果集并返回客户。
- 触发器（trigger）。与数据表相关联。当某个数据表被INSERT、DELETE、UPDATE语句修改，触发器自动执行。
- 事件（event）。根据时间表预定时刻自动执行。

## 复合语句和语句分隔符
存储程序可以包含多条SQL语句，可以使用局部变量、条件语句、循环和嵌套语句等多种语法。这是就需要用到复合语句。复合语句以BEGIN开头，END结束。
```
CREATE PROCEDURE greetings()
BEGIN
END;
```
复合语句块里以分号（;）隔开。SQL语句也以分号隔开，解决冲突可以使用delimiter。

## 存储函数和存储过程

## 触发器

## 事件

## 存储程序和视图的安全

