---
layout: post
title:  "MySQL语法基础"
tag:
- MySQL
- 数据库
comments: true
---

## 常用数据类型

MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。

- 数值类型：常用的有INT、FLOAT、DOUBLE。
- 日期/时间类型：DATETIME、DATE、TIMESTAMP、TIME和YEAR。
- 字符串（字符）类型：常用的有CHAR、VARCHAR、TEXT。VARCHAR与CHAR最大的区别在于VARCHAR是变长的，所以效率不如CHAR，同时必须指定最大长度。

## 建表

### 语法
```sql
CREATE TABLE table_name (column_name column_type);
```

### 实例
```sql
CREATE TABLE my_tbl (
   id INT NOT NULL AUTO_INCREMENT,
   name VARCHAR(40) NOT NULL,
   birthday DATE,
   PRIMARY KEY ( id )
);
```
其中，
- `NOT NULL`禁止字段为NULL，在操作数据库时如果输入该字段的数据为NULL ，就会报错。
- `AUTO_INCREMENT`定义列为自增的属性，一般用于主键，数值会自动加1。
- `PRIMARY KEY`定义列为主键。

## 删表

### 语法
```sql
DROP TABLE table_name ;
```

## 插入

### 语法
```sql
INSERT INTO table_name ( field1, field2,...fieldN )
                       VALUES
                       ( value1, value2,...valueN );
```
注意，如果数据是字符型，必须使用单引号或者双引号，如："value"。

### 实例

```sql
INSERT INTO my_tbl (id, name, birthday)
                   VALUES
                   (1, "Dimitri", "1997-11-02");
```

## 查询

### 语法
```sql
SELECT column_name,column_name
FROM table_name
[WHERE Clause]
[OFFSET M ][LIMIT N]
```
其中，
- OFFSET指定SELECT语句开始查询的数据偏移量。默认情况下偏移量为0。
- LIMIT 属性来设定返回的记录数。

## 更新

### 语法
```sql
UPDATE table_name SET field1=new_value1, field2=new_value2
[WHERE Clause]
```

## 删除

### 语法
```sql
DELETE FROM table_name [WHERE Clause]
```

参考：<https://www.w3cschool.cn/mysql/>
