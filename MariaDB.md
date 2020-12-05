# MariaDB

<!-- TOC GitLab -->

* [常用操作](#常用操作)
    - [创建一个数据库并为其创建一个表格](#创建一个数据库并为其创建一个表格)
    - [创建用户并授权](#创建用户并授权)
    - [基本查询](#基本查询)
    - [查询-去重](#查询-去重)
    - [连接 CONCAT](#连接-concat)
    - [判断是否为空并执行相关操作](#判断是否为空并执行相关操作)
* [基本命令](#基本命令)
    - [show](#show)
    - [desc](#desc)
    - [create](#create)
    - [insert](#insert)
    - [alter](#alter)
    - [drop](#drop)
    - [truncate](#truncate)
    - [use](#use)
* [运算符](#运算符)
* [函数](#函数)

<!-- /TOC -->

## 常用操作

> 学习要从需求开始

### 创建一个数据库并为其创建一个表格
```sql
create database db1 default character set utf8;
use db1;
create table tb1 (
  id int(3) not null,
  name varchar(20),
  birthday datetime
);
```
> `not null` 说明字段必须非空

### 创建用户并授权
```sql
# 创建用户
CREATE USER 'aloha'@'localhost' IDENTIFIED BY '123456';
# or
CREATE USER 'aloha'@'localhost';
# 授权
GRANT <privileges> ON <database_name>.<table_name> TO 'aloha'@'localhost';
```
- privileges: ALL、SELECT、INSERT、UPDATE ...

**使用**：
```
mariadb -u aloha -D <database_name> -p
```

### 基本查询

`select` 其实相当于 SQL 中的 `print`

```sql
select 'last_name', 'first_name' from employees;
# 查询并用别名输出
select
    'last_name' 姓,
    'first_name' '名',
    'email' as 邮箱
from employees;
```

### 查询-去重

```sql
select DISTINCT 'employee_id',
                'first_name',
                'last_name',
                'email'
from 'employees';
```
> 注意 `DISTINCT` 修饰 `select`，只有后面四列都相同时才会去重。

### 连接 CONCAT

```sql
select concat('first_name', 'last_name') as '姓名' from 'employees';
```

### 判断是否为空并执行相关操作


## 基本命令

### show

- `databases` 查看数据库
- `tables` 查看数据库中的表

### desc

- `<table>` 查看表格结构

### create

- `database <name> <options>` 创建数据库
- `table <name>(<colum_names>)` 创建表

```sql
create database db1 default character set utf8;
use db1;
create table tb1 (id int(3), name varchar(20));
```

### insert

- `into <table>[(<columns>)] <row_values>` 插入一行数据到表格中

```sql
insert into <table> values(1, 'user1');
select * from tb1;
```

### alter

- `table <table>`
    - `add(<colum>)` 新增一列（表字段）
    - `drop <column>` 删除表字段
    - `modify <column> <column_decalration>` 修改表字段类型格式
    - `change <column> <name>` 修改表字段名称
    - `rname <name>` 修改表名称

```sql
alter table tb1 add(age int(3));
alter table tb1 modify age varchar(2);
```

### drop

- `database <database>` 删除数据库

### truncate

- `table <table>` 清空表格

### use

- `use <database>` 进入数据库

## 运算符

- `+` 数字加法、字符连接（有一方为字符串就会将另一方也转换为字符串）

## 函数

- `VERSION()` 返回版本信息
- `CONCAT(<column1>, <column2>)` 将两者连接
