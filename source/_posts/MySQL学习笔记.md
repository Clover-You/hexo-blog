title: MySQL学习笔记
categories: MySQL
tags: [学习笔记,MySQL,SQL]
date: 2022-01-02 17:52:51
---
# 数据库概述

MySQL最初是由“MySQL AB”公司开发的一套关系型数据库管理系统（RDBMS-Relational DataBase Management System）。

 MySQL不仅是最流行的开源数据库，而且是业界成长最快的数据库，每天有超过7万次的下载量，其范围从大企业到专有的嵌入应用系统。

MySQL AB是由两个瑞典人和一个芬兰人：David AxMark、Allan Larsson 和Michael “Monty Widenius”在瑞典举办的。

在2008年初，Sun Microsystems收购了MySQL AB公司。在2009年，Oracle收购了Sun公司，使MySQL并入Oracle的数据库产品线。

## SQL、DB、DBMS分别是什么，他们之间的关系？

- DB
  DataBase ：数据库，数据库实际上在硬盘上以文件的形式存在

- DBMS
  DataBase Management  System：数据库管理系统，常见的有：MySQL、Oracle、DB2、Sybse、SQLServer...

- SQL

  结构化查询语言，是一门标准通用的语言。标准的sql适合于所有的数据库产品。SQL属于高级语言。SQL语句在执行的时候，实际上内部也会先进行编译，然后再执行SQL。（SQL语句的编译由DBMS完成）



DBMS负责执行sql语句，通过执行sql语句来操作DB当中的数据。

DBMS ==(执行)==> SQL ==(操作)==> DB



# 什么是表

- 表：table

  是数据库的基本组成单元，所有的数据都以表格的形式组织，目的是可读性强。

- 一个表包括行和列

  - 行
    被称为数据/记录（data）
  - 列
    被称为字段（column）

- 每一个字段应该包括哪些属性？
  一个字段，应该包括：字段名、数据类型、相关的约束。



# 对SQL语句的分类

- DQL（数据查询语言）
  查询语句，凡是select语句都是DQL

- DML（数据操作语言）
  insert delete update，对表当中的数据进行增删改

- DDL（数据定义语言）

  create drop alert，对表结构的增删改。

- TCL（事务控制语言）
  commit提交事务，rollback回滚事物

- DCL（数据控制语言）
  grant授权、revoke撤销权限等

# MySQL命令

1. 登录mysql数据库管理系统
   dos命令窗口

   ```dos
   mysql -uroot -p[密码]
   ```

   在Mac中如果出现

   ```
   mysql: [Warning] Using a password on the command line interface can be insecure.
   ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
   ```

   可能是mysqld服务没启动，在dos命令窗口输入`mysqld`等待启动即可

2. 查看有哪些数据库

   ```mysql
   show databases;
   ```

   ```sql
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | mysql              |
   | performance_schema |
   | ry-vue             |
   | sys                |
   +--------------------+
   5 rows in set (0.00 sec)
   ```

   这不是SQL语句，而是MySQL的命令

3. 创建属于我们自己的数据库

   ```mysql
   create database upyou_database;
   ```

   这个也不是一个sql语句，还是属于MySQL的命令。

   此时查看我们当前现有的数据库

   ```sql
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | mysql              |
   | performance_schema |
   | ry-vue             |
   | sys                |
   | upyou_database     |
   +--------------------+
   6 rows in set (0.00 sec)
   ```

4. 使用`upyou_database`

   ```mysql
   use upyou_database;
   ```

   这依然不是一个sql语句，这是一个属于mysql的命令。

5. 查看当前使用的数据库中的表

   ```mysql
   show tabels
   ```

   这个也不是一个sql语句，还是属于MySQL的命令。

   由于当前我们没有创建一个表，所以一下结果来源于mysql表

   ```mysql
   +----------------------------------------------+
   | Tables_in_mysql                              |
   +----------------------------------------------+
   | columns_priv                                 |
   | component                                    |
   | db                                           |
   | default_roles                                |
   | engine_cost                                  |
   | func                                         |
   | general_log                                  |
   | global_grants                                |
   +----------------------------------------------+
   34 rows in set (0.00 sec)
   ```

6. 为刚刚创建的数据库`upyou_database`初始化数据。

   使用mysql的`source`命令来导入一个sql脚本，如果你有

   ```mysql
   source 
   ```

   [学习数据库脚本下载](http://www.bjpowernode.com/javavideo/111.html?bili)

   导入成功后查看当前数据库

   ```mysql
   show tables;
   +--------------------------+
   | Tables_in_upyou_database |
   +--------------------------+
   | DEPT                     |
   | EMP                      |
   | SALGRADE                 |
   +--------------------------+
   3 rows in set (0.00 sec)
   ```




## 删除数据库

尝试将刚刚创建的`upyou_database`删除

```mysql
drop database upyou_database;
```

查看当前数据库列表

```mysql
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| ry-vue             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```



## 查看表结构

当前数据库为`upyou_database`

```mysql
+--------------------------+
| Tables_in_upyou_database |
+--------------------------+
| DEPT                     |	(部门表)
| EMP                      |	(员工表)
| SALGRADE                 |	(工资等级表)
+--------------------------+
3 rows in set (0.00 sec)
```

使用`desc`命令查看表结构，DEPT是一个表名

```mysql
desc DEPT
```

```mysql
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| DEPTNO | int         | NO   | PRI | NULL    |       |	(部门编号)
| DNAME  | varchar(14) | YES  |     | NULL    |       |	(部门名称)
| LOC    | varchar(13) | YES  |     | NULL    |       |	(部门位置)
+--------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

```mysql
desc EMP
```

```mysql
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| EMPNO    | int         | NO   | PRI | NULL    |       |	（员工编号）
| ENAME    | varchar(10) | YES  |     | NULL    |       |	（员工姓名）
| JOB      | varchar(9)  | YES  |     | NULL    |       |	（工作岗位）
| MGR      | int         | YES  |     | NULL    |       |	（上级领导编号）
| HIREDATE | date        | YES  |     | NULL    |       |	（入职日期）
| SAL      | double(7,2) | YES  |     | NULL    |       |	（月薪）
| COMM     | double(7,2) | YES  |     | NULL    |       |	（补助）
| DEPTNO   | int         | YES  |     | NULL    |       |	（部门编号）
+----------+-------------+------+-----+---------+-------+
8 rows in set (0.00 sec)
```

```mysql
desc SALGRADE
```

```mysql
+-------+------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+-------+------+------+-----+---------+-------+
| GRADE | int  | YES  |     | NULL    |       |	（等级）
| LOSAL | int  | YES  |     | NULL    |       |	（最低薪资）
| HISAL | int  | YES  |     | NULL    |       |	（最高薪资）
+-------+------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```



## 查询当前使用的数据库

```mysql
select database();
```

```mysql
+----------------+
| database()     |
+----------------+
| upyou_database |
+----------------+
1 row in set (0.00 sec)
```



## 查看MySQL本号

```mysql
select version();
```

```mysql
+-----------+
| version() |
+-----------+
| 8.0.22    |
+-----------+
```



## 结束一条语句

当想结束一条语句时使用`\c`命令，例如想结束以下语句

```mysql
mysql> selece
    -> *
    -> from
    -> 
    -> 
    -> 
    -> 
```

输入`\c`后回车即可结束语句。



## 查看创建表的语句：

如果想查看某个表是用哪条语句创建的，可以使用`show create table 表名`

```mysql
 show create table DEPT
```

```sql
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                     |
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| DEPT  | CREATE TABLE `DEPT` (
  `DEPTNO` int NOT NULL,
  `DNAME` varchar(14) DEFAULT NULL,
  `LOC` varchar(13) DEFAULT NULL,
  PRIMARY KEY (`DEPTNO`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```



# SQL语句

## SQL脚本

以`.sql`结尾的文件，这样的文件被称为“sql脚本”。什么是sql脚本呢？

当一个文件的拓展名是`.sql`，并且该文件中编写了大量的sql语句，我们称这样的文件为sql脚本。

> sql脚本中的数据量太大的时候，无法打开，请使用source命令完成初始化。

## 查看表格中的数据

查询某个表中所有的数据

```sql
select * from DEPT
```

```mysql
+--------+------------+----------+
| DEPTNO | DNAME      | LOC      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.00 sec)
```



## 简单的查询语句（DQL）

语法格式：

```sql
select 字段名1, 字段名2, 字段名3, ... from 表名;
```

- 任何一条sql语句以分号结尾“;”。
- sql 语句不区分大小写。



### 例子

1. 查询员工名字

   ```sql
   select ENAME from EMP;
   ```

   ```mysql
   +--------+
   | ENAME  |
   +--------+
   | SMITH  |
   | ALLEN  |
   | WARD   |
   | JONES  |
   | MARTIN |
   | BLAKE  |
   | CLARK  |
   | SCOTT  |
   | KING   |
   | TURNER |
   | ADAMS  |
   | JAMES  |
   | FORD   |
   | MILLER |
   +--------+
   ```



2. 查询员工的年薪

   数据库中记录的是月薪，年薪是需要乘12

   ```sql
   select ENAME, (SAL * 12) from emp;
   ```

   ```mysql
   +--------+------------+
   | ENAME  | (SAL * 12) |
   +--------+------------+
   | SMITH  |    9600.00 |
   | ALLEN  |   19200.00 |
   | WARD   |   15000.00 |
   | JONES  |   35700.00 |
   | MARTIN |   15000.00 |
   | BLAKE  |   34200.00 |
   | CLARK  |   29400.00 |
   | SCOTT  |   36000.00 |
   | KING   |   60000.00 |
   | TURNER |   18000.00 |
   | ADAMS  |   13200.00 |
   | JAMES  |   11400.00 |
   | FORD   |   36000.00 |
   | MILLER |   15600.00 |
   +--------+------------+
   ```

   如果由于`SAL * 12`后字段名变为SAL * 12，如需重新给该字段名明，可以使用`as`关键字命名

   ```sql
   select ENAME, (SAL * 12) as SAL from EMP;
   ```

   ```java
   +--------+----------+
   | ENAMW  | SAL      |
   +--------+----------+
   | SMITH  |  9600.00 |
   | ALLEN  | 19200.00 |
   | WARD   | 15000.00 |
   | JONES  | 35700.00 |
   | MARTIN | 15000.00 |
   | BLAKE  | 34200.00 |
   | CLARK  | 29400.00 |
   | SCOTT  | 36000.00 |
   | KING   | 60000.00 |
   | TURNER | 18000.00 |
   | ADAMS  | 13200.00 |
   | JAMES  | 11400.00 |
   | FORD   | 36000.00 |
   | MILLER | 15600.00 |
   +--------+----------+
   ```


通过以上可以看出来，在`sql`中字段可以参与数学运算。

如果字段想显示中文，则需要使用单引号包裹（标准sql语句中要求字符串使用单引号括起来。）

```sql
select ENAME as '员工名', SAL * 12 as '年薪' from EMP;
```

```mysql
+-----------+----------+
| 员工名    | 年薪     |
+-----------+----------+
| SMITH     |  9600.00 |
| ALLEN     | 19200.00 |
| WARD      | 15000.00 |
| JONES     | 35700.00 |
| MARTIN    | 15000.00 |
| BLAKE     | 34200.00 |
| CLARK     | 29400.00 |
| SCOTT     | 36000.00 |
| KING      | 60000.00 |
| TURNER    | 18000.00 |
| ADAMS     | 13200.00 |
| JAMES     | 11400.00 |
| FORD      | 36000.00 |
| MILLER    | 15600.00 |
+-----------+----------+
14 rows in set (0.00 sec)
```

**在SQL Server、Oracle中仅支持并必须使用单引号将中文包裹，而MySQL可以使用双引号和单引号，所以以后写SQL语句不要出现双引号，如果出现双引号意味着这个SQL脚本只能在MySQL中使用。**

### 查询所有字段

在sql中可以使用`*`代表所有字段，如果需要查询所有字段，可以使用`*`

```sql
select * from EMP;
```

```java
+-------+--------+-----------+------+------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+------------+---------+---------+--------+
14 rows in set (0.00 sec)
```

> 不推荐在线上项目中使用，因为这会折扣一部分效率。

## 条件查询

语法格式：

```sql
select 字段, 字段, ... from 表名 where 条件;
```

执行顺序：from -> where -> select

| 运算符           | 说明                                       |
| ---------------- | ------------------------------------------ |
| =                | 等于                                       |
| <> 或 !=         | 不等于                                     |
| <                | 小于                                       |
| <=               | 小于等于                                   |
| >                | 大于                                       |
| >=               | 大于等于                                   |
| between...and... | 两个值之间，等同于 `>= and <=`             |
| is null          | 为null（is not null 不为空）               |
| and              | 并且                                       |
| or               | 或者                                       |
| in               | 包含，相当于多个or（not in不在这个范围中） |
| not              | not 可以取非，主要用在is或in中             |
| like             | like称为模糊查询，支持%或下划线匹配        |



1. 查询工资等于5000的员工姓名

   ```sql
   select ENAME from EMP where SAL = 5000;
   ```
   
   ```mysql
   +-------+
   | ENAME |
   +-------+
   | KING  |
   +-------+
   ```
   
2. 查询SMITH的工资

   ```sql
   select ENAME, SAL from EMP where ENAME = 'SMITH';
   ```

   ```mysql
   +-------+--------+
   | ENAME | SAL    |
   +-------+--------+
   | SMITH | 800.00 |
   +-------+--------+
   ```

3. 找出工资高于3000的员工

   ```sql
   select ENAME, SAL from EMP where SAL > 3000;
   ```

   ```mysql
   +-------+---------+
   | ENAME | SAL     |
   +-------+---------+
   | KING  | 5000.00 |
   +-------+---------+
   ```

4. 找出工资不等于3000的员工

   ```sql
   select ENAME, SAL from EMP where SAL <> 3000;
   ```

   ```mysql
   +--------+---------+
   | ENAME  | SAL     |
   +--------+---------+
   | SMITH  |  800.00 |
   | ALLEN  | 1600.00 |
   | WARD   | 1250.00 |
   | JONES  | 2975.00 |
   | MARTIN | 1250.00 |
   | BLAKE  | 2850.00 |
   | CLARK  | 2450.00 |
   | KING   | 5000.00 |
   | TURNER | 1500.00 |
   | ADAMS  | 1100.00 |
   | JAMES  |  950.00 |
   | MILLER | 1300.00 |
   +--------+---------+
   ```

5. 找出工资在1100和3000之间的员工

   ```sql 
   select ENAME, SAL from EMP where SAL >= 1100 and SAL <= 3000;
   ```

   ```mysql
   +--------+---------+
   | ENAME  | SAL     |N
   +--------+---------+
   | ALLEN  | 1600.00 |
   | WARD   | 1250.00 |
   | JONES  | 2975.00 |
   | MARTIN | 1250.00 |
   | BLAKE  | 2850.00 |
   | CLARK  | 2450.00 |
   | SCOTT  | 3000.00 |
   | TURNER | 1500.00 |
   | ADAMS  | 1100.00 |
   | FORD   | 3000.00 |
   | MILLER | 1300.00 |
   +--------+---------+
   ```

   ```sql
   select ENAME, SAL from EMP where SAL between 1100 and 3000;
   ```

   



### between and

between ... and ...是闭区间查询语法，执行结果与`SAL >= 1100 and SAL <= 3000`一致。`between and`在使用的时候必须左小右大。

```sql
select ENAME, SAL from EMP where SAL between 1100 and 3000;
```

```mysql
+--------+---------+
| ENAME  | SAL     |
+--------+---------+
| ALLEN  | 1600.00 |
| WARD   | 1250.00 |
| JONES  | 2975.00 |
| MARTIN | 1250.00 |
| BLAKE  | 2850.00 |
| CLARK  | 2450.00 |
| SCOTT  | 3000.00 |
| TURNER | 1500.00 |
| ADAMS  | 1100.00 |
| FORD   | 3000.00 |
| MILLER | 1300.00 |
+--------+---------+
```

`between and` 除了用在数字方面还可以用在字符串方面。

```sql
select ENAME from EMP where ENAME between 'A' and 'C';
```

```mysql
+-------+
| ENAME |
+-------+
| ALLEN |
| BLAKE |
| ADAMS |
+-------+
```

在数据库中，有这么一条数据，但我们却没有查出来，因为`between and`在字符串中是左闭右开的。

```mysql
+-------+
| ENAME |
+-------+
| CLARK |
+-------+
```



### is null 和 is not null

例子

1. 找出哪些人津贴为null？在数据库当中**NULL**不是一个值，代表什么也没有，为空。空不是一个值，不能使用等号衡量，必须使用`is null` 或者`is not null`

   ```sql
   select ENAME, COMM from EMP where COMM is null;
   ```

   ```mysql
   +--------+------+
   | ENAME  | COMM |
   +--------+------+
   | SMITH  | NULL |
   | JONES  | NULL |
   | BLAKE  | NULL |
   | CLARK  | NULL |
   | SCOTT  | NULL |
   | KING   | NULL |
   | TURNER | 0.00 |
   | ADAMS  | NULL |
   | JAMES  | NULL |
   | FORD   | NULL |
   | MILLER | NULL |
   +--------+------+
   ```

2. 找出哪些人津贴不为NULL

   ```sql
   select ENAME, COMM from EMP where COMM is not null;
   ```

   ```mysql
   +--------+---------+
   | ENAME  | COMM    |
   +--------+---------+
   | ALLEN  |  300.00 |
   | WARD   |  500.00 |
   | MARTIN | 1400.00 |
   | TURNER |    0.00 |
   +--------+---------+
   ```

3. 找出哪些人没有津贴？

   ```sql
   select ENAME, COMM from EMP where COMM is null or COMM = 0;
   ```

   ```mysql
   +--------+------+
   | ENAME  | COMM |
   +--------+------+
   | SMITH  | NULL |
   | JONES  | NULL |
   | BLAKE  | NULL |
   | CLARK  | NULL |
   | SCOTT  | NULL |
   | KING   | NULL |
   | TURNER | 0.00 |
   | ADAMS  | NULL |
   | JAMES  | NULL |
   | FORD   | NULL |
   | MILLER | NULL |
   +--------+------+
   ```



### or和and的优先级问题

找出薪资大于1000并且部门编号是20或30部门的员工。

```sql
select ENAME, SAL, DEPTNO from EMP where SAL > 1000 and DEPTNO = 20 or DEPTNO = 30;
```

```java
+--------+---------+--------+
| ENAME  | SAL     | DEPTNO |
+--------+---------+--------+
| ALLEN  | 1600.00 |     30 |
| WARD   | 1250.00 |     30 |
| JONES  | 2975.00 |     20 |
| MARTIN | 1250.00 |     30 |
| BLAKE  | 2850.00 |     30 |
| SCOTT  | 3000.00 |     20 |
| TURNER | 1500.00 |     30 |
| ADAMS  | 1100.00 |     20 |
| JAMES  |  950.00 |     30 |
| FORD   | 3000.00 |     20 |
+--------+---------+--------+
```

以上例子在and和or联合使用时，and的优先级比较高，and会将左右两边的条件联合使用，再判判断or，所以需要使用小括号将or语句括起来

```sql
select ENAME, SAL, DEPTNO from EMP where SAL > 1000 and (DEPTNO = 20 or DEPTNO = 30);
```

> 当运算符的优先级不确定的时候，添加小括号。



### in

in等同于or：找出工作岗位是MANAGER和SALESMAN的员工

```sql
select ENAME, JOB from EMP where JOB = 'MANAGER' or JOB = 'SALESMAN';
```

```mysql
+--------+----------+
| ENAME  | JOB      |
+--------+----------+
| ALLEN  | SALESMAN |
| WARD   | SALESMAN |
| JONES  | MANAGER  |
| MARTIN | SALESMAN |
| BLAKE  | MANAGER  |
| CLARK  | MANAGER  |
| TURNER | SALESMAN |
+--------+----------+
```

在以上例子中，`JOB = xxx or JOB = xxx`这样的语句可以使用in简化

```sql
select ENAME, JOB from EMP where JOB in ('MANAGER', 'SALESMAN');
```

```mysql
+--------+----------+
| ENAME  | JOB      |
+--------+----------+
| ALLEN  | SALESMAN |
| WARD   | SALESMAN |
| JONES  | MANAGER  |
| MARTIN | SALESMAN |
| BLAKE  | MANAGER  |
| CLARK  | MANAGER  |
| TURNER | SALESMAN |
+--------+----------+
```



### not in 

not in：不在这几个值当中。

```sql
select ENAME, JOB from EMP where JOB not in ('MANAGER', 'SALESMAN');
```

```mysql
+--------+-----------+
| ENAME  | JOB       |
+--------+-----------+
| SMITH  | CLERK     |
| SCOTT  | ANALYST   |
| KING   | PRESIDENT |
| ADAMS  | CLERK     |
| JAMES  | CLERK     |
| FORD   | ANALYST   |
| MILLER | CLERK     |
+--------+-----------+
```

等价于

```sql
select ENAME, JOB from EMP where JOB != 'MANAGER' and JOB != 'SALESMAN';
```



### 模糊查询like

在模糊查询当中，必须掌握两个特殊的符号，一个是“%”，一个是“\_”，“%”代表的是任意多个字符，“\_”代表任意1个字符。如果需要找的是已""%""或者"_"，需要使用转义字符转义。



#### 找出名字当中含有O的？

```sql
select ENAME from EMP where ENAME like '%O%';
```

```mysql
+-------+
| ENAME |
+-------+
| JONES |
| SCOTT |
| FORD  |
+-------+
```



#### 找出名字中第二个字母是A的？

```sql
select ENAME from EMP where ENAME like '_A%';
```

```mysql
+--------+
| ENAME  |
+--------+
| WARD   |
| MARTIN |
| JAMES  |
+--------+
```



## 排序

1. 按照工资升序排序，找出员工名和薪资。

   ```sql
   select ENAME, SAL from EMP order by SAL;
   ```

   ```mysql
   +--------+---------+
   | ENAME  | SAL     |
   +--------+---------+
   | SMITH  |  800.00 |
   | JAMES  |  950.00 |
   | ADAMS  | 1100.00 |
   | WARD   | 1250.00 |
   | MARTIN | 1250.00 |
   | MILLER | 1300.00 |
   | TURNER | 1500.00 |
   | ALLEN  | 1600.00 |
   | CLARK  | 2450.00 |
   | BLAKE  | 2850.00 |
   | JONES  | 2975.00 |
   | SCOTT  | 3000.00 |
   | FORD   | 3000.00 |
   | KING   | 5000.00 |
   +--------+---------+
   
   ```

   `order by`默认就是升序排序。怎么指定升序或者降序呢？ `asc` 表示升序，`desc` 表示降序。

2. 按照工资降序排序，找出员工名和薪资。

   ```sql
   select ENAME, SAL from EMP order by SAL desc;
   ```

   ```mysql
   +--------+---------+
   | ENAME  | SAL     |
   +--------+---------+
   | KING   | 5000.00 |
   | SCOTT  | 3000.00 |
   | FORD   | 3000.00 |
   | JONES  | 2975.00 |
   | BLAKE  | 2850.00 |
   | CLARK  | 2450.00 |
   | ALLEN  | 1600.00 |
   | TURNER | 1500.00 |
   | MILLER | 1300.00 |
   | WARD   | 1250.00 |
   | MARTIN | 1250.00 |
   | ADAMS  | 1100.00 |
   | JAMES  |  950.00 |
   | SMITH  |  800.00 |
   +--------+---------+
   ```

3. 按照工资的降序排列，当工资相同的时候再按照名字的升序排列。

   ```sql
   select ENAME, SAL from EMP order by SAL desc, ENAME asc;
   ```

   ```mysql
   +--------+---------+
   | ENAME  | SAL     |
   +--------+---------+
   | KING   | 5000.00 |
   | FORD   | 3000.00 |
   | SCOTT  | 3000.00 |
   | JONES  | 2975.00 |
   | BLAKE  | 2850.00 |
   | CLARK  | 2450.00 |
   | ALLEN  | 1600.00 |
   | TURNER | 1500.00 |
   | MILLER | 1300.00 |
   | MARTIN | 1250.00 |
   | WARD   | 1250.00 |
   | ADAMS  | 1100.00 |
   | JAMES  |  950.00 |
   | SMITH  |  800.00 |
   +--------+---------+
   ```

   >  越来靠前的字段越能起到主导作用。只有当前面的字段无法完成排序的时候，才会启用后面的字段。

   

4. 找出工作岗位是SALESMAN的员工，并且要求按照薪资的降序排列

   ```sql
   select ENAME, JOB, SAL from EMP where JOB = 'SALESMAN' order by SAL desc;
   ```

   ```mysql
   +--------+----------+---------+
   | ENAME  | JOB      | SAL     |
   +--------+----------+---------+
   | ALLEN  | SALESMAN | 1600.00 |
   | TURNER | SALESMAN | 1500.00 |
   | WARD   | SALESMAN | 1250.00 |
   | MARTIN | SALESMAN | 1250.00 |
   +--------+----------+---------+
   ```
   
   

   > 无论何种排序，只有在结果查询出来时才能进行排序。也就是说order by是最后执行的。



# 分组函数

### 多行处理函数

| 函数名 | 说明   |
| ------ | ------ |
| count  | 计数   |
| sum    | 求和   |
| avg    | 平均值 |
| max    | 最大值 |
| min    | 最小值 |

> 所有的分组函数都是对某一组数据进行操作

1. 找出员工工资总和

   ```sql
   select sum(SAL) from EMP;
   ```


   ```mysql
   +----------+
   | sum(SAL) |
   +----------+
   | 29025.00 |
   +----------+
   ```

2. 找出最高工资

   ```sql
   select max(SAL) from EMP;
   ```

   ```mysql
   +----------+
   | max(SAL) |
   +----------+
   |  5000.00 |
   +----------+
   ```

3. 找出最低工资

   ```sql
   select min(SAL) from EMP;
   ```

   ```mysql
   +----------+
   | min(SAL) |
   +----------+
   |   800.00 |
   +----------+
   ```

4. 找出平均工资

   ```sql
   select avg(SAL) from EMP;
   ```

   ```mysql
   +-------------+
   | avg(SAL)    |
   +-------------+
   | 2073.214286 |
   +-------------+
   ```

5. 找出总人数

   ```sql
   select count(ENAME) from EMP;
   ```

   ```mysql
   +--------------+
   | count(ENAME) |
   +--------------+
   |           14 |
   +--------------+
   ```

   count(\*)和count(具体的某个字段)的区别是：count(\*)查询的是总的条数，而count(字段)查询的是某个字段不为NULL的数据条数。

> 分组函数一共5个，分组函数还有另一个名字：**多行处理函数**。
>
> 多行处理函数的特点是：输入多行，输出一行。分组函数自动忽略NULL



### 单行处理行数

单行处理函数是：输入行，输出一行。

> 只要表达式中参与运算的成员有一个是NULL，无论式子怎么列，结果都为NULL（任何数据库）

| 函数   | 说明                                                   |
| ------ | ------------------------------------------------------ |
| ifnull | 空处理函数（ifnull(可能为NULL的数据，被当做什么处理)） |



1. 计算每个员工的年薪

   ```sql
   select ENAME, (SAL + ifnull(COMM, 0)) * 12 as yearsal from EMP;
   ```

   ```mysql
   +--------+----------+
   | ENAME  | yearsal  |
   +--------+----------+
   | SMITH  |  9600.00 |
   | ALLEN  | 22800.00 |
   | WARD   | 21000.00 |
   | JONES  | 35700.00 |
   | MARTIN | 31800.00 |
   | BLAKE  | 34200.00 |
   | CLARK  | 29400.00 |
   | SCOTT  | 36000.00 |
   | KING   | 60000.00 |
   | TURNER | 18000.00 |
   | ADAMS  | 13200.00 |
   | JAMES  | 11400.00 |
   | FORD   | 36000.00 |
   | MILLER | 15600.00 |
   +--------+----------+
   ```

2. 找出工资高于平均工资的员工？

   ```sql
   select ENAME, SAL from EMP where SAL > avg(SAL);
   ```

   ```mysql
   ERROR 1111 (HY000): Invalid use of group function
   ```

   > SQL语句当中有一个语法规则，分组函数不可直接使用在where子句中。因为group by是在where后执行，而分组函数是在group by后执行。
   
   **下面有**



### 分组查询

| 关键字 | 说明                               |
| ------ | ---------------------------------- |
| group  | 按照某个字段或者某些字段进行分组。 |
| having | 对分组之后的数据进行再次过滤。     |

> 分组函数一般都会和group by联合使用。这也是为什么它被称为分组函数的原因。并且任何一个分组函数（count sum avg max min）都是在group by语句执行结束之后才会执行的。当一条sql语句没有group by的话，整张表的数据会自成一组。

1. 找出每个工作岗位的最高薪资。

   ```sql
   select JOB, max(SAL) from EMP group by JOB;
   ```

   ```mysql
   +-----------+---------+
   | JOB       | MAXSAL  |
   +-----------+---------+
   | CLERK     | 1300.00 |
   | SALESMAN  | 1600.00 |
   | MANAGER   | 2975.00 |
   | ANALYST   | 3000.00 |
   | PRESIDENT | 5000.00 |
   +-----------+---------+
   ```

   执行顺序：from -> group by -> select
   **当一条sql语句出现group by语句时，select语句只能出现分组语句以及参与分组字段，别的都不能出现。**
   
   ```sql
   select ENAME, JOB, max(SAL) from EMP group by JOB;
   ```
   
   以上语句在低版本**MySQL**可能不会出现以上错误并可查询出结果。
   
   ```mysql
   ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'upyou_database.EMP.ENAME' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
   ```
   
2. 查询每个工作岗位的平均工资。

   ```sql
   select JOB, avg(SAL) from EMP group by JOB;
   ```

   ```mysql
   +-----------+-------------+
   | JOB       | avg(SAL)    |
   +-----------+-------------+
   | CLERK     | 1037.500000 |
   | SALESMAN  | 1400.000000 |
   | MANAGER   | 2758.333333 |
   | ANALYST   | 3000.000000 |
   | PRESIDENT | 5000.000000 |
   +-----------+-------------+
   ```

3. 找出不同部门不同工作岗位的最高工资

   ```sql
   select DEPTNO, JOB, max(SAL) from EMP group by DEPTNI, JOB;
   ```

   ```mysql
   +--------+-----------+----------+
   | DEPTNO | JOB       | max(SAL) |
   +--------+-----------+----------+
   |     20 | CLERK     |  1100.00 |
   |     30 | SALESMAN  |  1600.00 |
   |     20 | MANAGER   |  2975.00 |
   |     30 | MANAGER   |  2850.00 |
   |     10 | MANAGER   |  2450.00 |
   |     20 | ANALYST   |  3000.00 |
   |     10 | PRESIDENT |  5000.00 |
   |     30 | CLERK     |   950.00 |
   |     10 | CLERK     |  1300.00 |
   +--------+-----------+----------+
   ```

4. 找出每个部门的最高薪资，要求显示薪资大于2900的数据。

   1. 第一步，找出每个部门的最高薪资
      
      ```sql
           select max(SAL) from EMP group by DEPTNO;
      ```
      
      ```mysql
      +----------+
      | max(SAL) |
      +----------+
      |  3000.00 |
      |  2850.00 |
      |  5000.00 |
      +----------+
      ```
      
      
      
   2. 找出薪资大于2900的员工信息。
   
      ```sql
      select max(SAL) from EMP group by DEPTNO having max(SAL) > 2900;
      ```
   
      ```mysql
      +----------+
      | max(SAL) |
      +----------+
      |  3000.00 |
      |  5000.00 |
      +----------+
      ```
   
      以上使用`having`这种方式效率较低，可以使用`where`先过滤掉低于2900的数据后再分组。
   
      ```sql
      select max(SAL) from EMP where SAL > 2900 group by DEPTNO;
      ```
   
5. 找出每个部门的平均薪资，要求显示薪资大于2000的数据。
   
   ```sql
   select avg(SAL) from EMP group by DEPTNO having svg(SAL) > 2000;
   ```
   
   
   
      

### 去除重复记录

如果查询emp表的JOB字段，会出现很多没必要的重复数据：

```sql
select JOB from EMP;
```

```mysql
+-----------+
| job       |
+-----------+
| CLERK     |
| SALESMAN  |
| SALESMAN  |
| MANAGER   |
| SALESMAN  |
| MANAGER   |
| MANAGER   |
| ANALYST   |
| PRESIDENT |
| SALESMAN  |
| CLERK     |
| CLERK     |
| ANALYST   |
| CLERK     |
+-----------+
```

如果不需要重复数据可以使用`distinct`关键字去除重复数据(需要去重复的字段前使用)

```sql
select distinct JOB from EMP;
```

```mysql
+-----------+
| JOB       |
+-----------+
| CLERK     |
| SALESMAN  |
| MANAGER   |
| ANALYST   |
| PRESIDENT |
+-----------+
```

以下的SQL语句是错误的，没有这种语法机制

```sql
select ENAME, distinct JOB from EMP;
```

```mysql
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'distinct job from emp' at line 1
```

> distinct字段只能出现在所有字段的最前面。distinct后面的所有字段联合起来去重。
> select distinct JOB, ENAME from EMP;

### 一个完整的DQL语句怎么写以及执行顺序

```sql
select 					5
	... 
from 						1
	... 
where 					2
	...
group by 				3
	...
having					4
	...
order by				6
```









# 连接查询

## 什么是连接查询？

在实际开发中，大部分情况下都不是从单表中查询数据，一般都是多张表联合查询取出最终的结果。在实际开发中，一个业务都会对应多张表，比如：学生和班级：

```mysql
stuno		stuname		classno								classname
---------------------------------------------------------------
1					zs				1					北京大兴区亦庄经济开发区第二中学高三1班
2					ls				1					北京大兴区亦庄经济开发区第二中学高三1班
...
```

学生和班级信息存储到一张表中，结果就像上面一样，数据会存在大量的重复，导致数据的**冗余**。



### 连接查询的分类

1. 根据语法出现的年代来划分的话，包括：

   - SQL92

   - SQL99（比较新的语法）

2. 根据表的连接方式来划分，包括：

   - 内连接
     - 等值连接
     - 非等值连接
       `between and`
     - 自连接
   - 外连接
     - 左外连接
     - 右外连接
   - 全连接



### 笛卡尔积现象

在表的连接查询方面有一种现象被称为：笛卡尔积现象。（笛卡尔乘积现象）



1. 找出每一个员工的部门名称，要求显示员工名和部门名称。
   EMP表

   ```sql
   select ENAME, DEPTNO from EMP;
   ```

   ```mysql
   +--------+--------+
   | ENAME  | DEPTNO |
   +--------+--------+
   | SMITH  |     20 |
   | ALLEN  |     30 |
   | WARD   |     30 |
   | JONES  |     20 |
   | MARTIN |     30 |
   | BLAKE  |     30 |
   | CLARK  |     10 |
   | SCOTT  |     20 |
   | KING   |     10 |
   | TURNER |     30 |
   | ADAMS  |     20 |
   | JAMES  |     30 |
   | FORD   |     20 |
   | MILLER |     10 |
   +--------+--------+
   ```

   DEPT表

   ```sql
   select * from DEPT;
   ```

   ```mysql
   +--------+------------+----------+
   | DEPTNO | DNAME      | LOC      |
   +--------+------------+----------+
   |     10 | ACCOUNTING | NEW YORK |
   |     20 | RESEARCH   | DALLAS   |
   |     30 | SALES      | CHICAGO  |
   |     40 | OPERATIONS | BOSTON   |
   +--------+------------+----------+
   ```

   通过from查询多张表：

   ```sql
   select ENAME, DNAME from EMP, DEPT;
   ```

   ```mysql
   +--------+------------+
   | ename  | dname      |
   +--------+------------+
   | SMITH  | ACCOUNTING |
   | SMITH  | RESEARCH   |
   | SMITH  | SALES      |
   | SMITH  | OPERATIONS |
   | ALLEN  | ACCOUNTING |
   | ALLEN  | RESEARCH   |
   | ALLEN  | SALES      |
   | ALLEN  | OPERATIONS |
   | WARD   | ACCOUNTING |
   | WARD   | RESEARCH   |
   | WARD   | SALES      |
   | WARD   | OPERATIONS |
   | JONES  | ACCOUNTING |
   | JONES  | RESEARCH   |
   |...									|
   +--------+------------+
   56 rows in set (0.00 sec)
   ```

   以上sql语句没有过滤条件，意味着EMP中的每一项都可以和DEPT匹配。

   > 如果两张表连接的话如果没有任何条件限制，最终的查询结果条数是两张表记录条数的乘积。这种现象被称为笛卡尔乘积现象。



### 关于表的别名

```sql
select E.ENAME, D.DNAME from EMP E, DEPT D;
```

#### 表的别名有什么好处？

1. 执行效率高。
2. 可读性好



### 如何避免笛卡尔积现象？

若想要避免笛卡尔积现象，当然是加条件进行过滤。

那么避免了笛卡尔积现象，会减少记录的匹配次数吗？不会，匹配的次数还是两张表记录条数的乘积，只不过显示的是有效记录。

1. 找出每一个员工的部门名称，要求显示员工名和部门名称。
   EMP表
   
   ```sql
    select
    		e.ename, d.dname
    from
    		emp e, dept d
    where 
    		d.deptno = e.deptno;
   ```
   
   ```mysql
   +--------+------------+
   | ename  | dname      |
   +--------+------------+
   | SMITH  | RESEARCH   |
   | ALLEN  | SALES      |
   | WARD   | SALES      |
   | JONES  | RESEARCH   |
   | MARTIN | SALES      |
   | BLAKE  | SALES      |
   | CLARK  | ACCOUNTING |
   | SCOTT  | RESEARCH   |
   | KING   | ACCOUNTING |
   | TURNER | SALES      |
   | ADAMS  | RESEARCH   |
   | JAMES  | SALES      |
   | FORD   | RESEARCH   |
   | MILLER | ACCOUNTING |
   +--------+------------+
   14 rows in set (0.00 sec)
   ```

> 以上SQL语句为SQL92的写法，以后不用。



## 内连接

### 等值连接

等值连接最大的特点是**条件是等量关系**

1. 查询每个员工的部门名称，要求显示员工名和部门名。

   ### SQL92

   ```sql
   select E.ENAME, D.DNAME from EMP E, DEPT D where E.DEPTNO = D.DEPTNO; 
   ```

   ```mysql
   +--------+------------+
   | ENAME  | DNAME      |
   +--------+------------+
   | SMITH  | RESEARCH   |
   | ALLEN  | SALES      |
   | WARD   | SALES      |
   | JONES  | RESEARCH   |
   | MARTIN | SALES      |
   | BLAKE  | SALES      |
   | CLARK  | ACCOUNTING |
   | SCOTT  | RESEARCH   |
   | KING   | ACCOUNTING |
   | TURNER | SALES      |
   | ADAMS  | RESEARCH   |
   | JAMES  | SALES      |
   | FORD   | RESEARCH   |
   | MILLER | ACCOUNTING |
   +--------+------------+
   14 rows in set (0.00 sec)
   ```

   ### SQL99

   ```sql
   select E.ENAME, D.DNAME from EMP E inner join DEPT D on E.DEPTNO = D.DEPTNO;
   ```

   `join`后面跟的是需要连接的表，`on`后面跟的是表的连接条件

   ```mysql
   +--------+------------+
   | ENAME  | DNAME      |
   +--------+------------+
   | SMITH  | RESEARCH   |
   | ALLEN  | SALES      |
   | WARD   | SALES      |
   | JONES  | RESEARCH   |
   | MARTIN | SALES      |
   | BLAKE  | SALES      |
   | CLARK  | ACCOUNTING |
   | SCOTT  | RESEARCH   |
   | KING   | ACCOUNTING |
   | TURNER | SALES      |
   | ADAMS  | RESEARCH   |
   | JAMES  | SALES      |
   | FORD   | RESEARCH   |
   | MILLER | ACCOUNTING |
   +--------+------------+
   14 rows in set (0.00 sec)
   ```

   语法

   ```mysql
   ...
   	A
   inner join
   	B
   on
   	连接条件
   where
   	...
   ```

   SQL99语法的结构更清晰一些，因为表的连接条件和后来的where条件分离了。

   > inner join中的inner是可以缺省的。带着inner目的性是可读性高一些。



### 非等值连接

非等值连接最大的特点是：连接条件中的关系是非等量关系。

1. 找出每个员工的工资等级，要求显示员工名、工资、工资等级。

   ```sql
   select E.ENAME, E.SAL, S.GRADE from EMP E inner join SALGRADE S on E.SAL between S.LOSAL and S.HISAL;,
   ```

   ```mysql
   +--------+---------+-------+
   | ENAME  | SAL     | GRADE |
   +--------+---------+-------+
   | SMITH  |  800.00 |     1 |
   | ALLEN  | 1600.00 |     3 |
   | WARD   | 1250.00 |     2 |
   | JONES  | 2975.00 |     4 |
   | MARTIN | 1250.00 |     2 |
   | BLAKE  | 2850.00 |     4 |
   | CLARK  | 2450.00 |     4 |
   | SCOTT  | 3000.00 |     4 |
   | KING   | 5000.00 |     5 |
   | TURNER | 1500.00 |     3 |
   | ADAMS  | 1100.00 |     1 |
   | JAMES  |  950.00 |     1 |
   | FORD   | 3000.00 |     4 |
   | MILLER | 1300.00 |     2 |
   +--------+---------+-------+
   ```

   

### 自连接

自连接最大的特点是：一张表看做两张表。自己连接自己。

1. 找出每个员工的上级领导，要求显示员工名和对应的领导名。

   ```sql
   select * from EMP;
   ```
   
   EMP A员工表
   
   ```mysql
   +-------+--------+------+
   | EMPNO | ENAME  | MGR  |
   +-------+--------+------+
   |  7369 | SMITH  | 7902 |
   |  7499 | ALLEN  | 7698 |
   |  7521 | WARD   | 7698 |
   |  7566 | JONES  | 7839 |
   |  7654 | MARTIN | 7698 |
   |  7698 | BLAKE  | 7839 |
   |  7782 | CLARK  | 7839 |
   |  7788 | SCOTT  | 7566 |
   |  7839 | KING   | NULL |
   |  7844 | TURNER | 7698 |
   |  7876 | ADAMS  | 7788 |
   |  7900 | JAMES  | 7698 |
   |  7902 | FORD   | 7566 |
   |  7934 | MILLER | 7782 |
   +-------+--------+------+
   ```
   
   EMP B领导表
   
   ```mysql
   +-------+--------+
   | EMPNO | ENAME  |
   +-------+--------+
   |  7566 | JONES  |
   |  7698 | BLAKE  |
   |  7782 | CLARK  |
   |  7788 | SCOTT  |
   |  7839 | KING   |
   |  7902 | FORD   |
   +-------+--------+
   ```
   
   ```sql
   select A.ENAME as '员工名', B.ENAME as '领导名' from EMP A inner join EMP B on A.mgr = B.empno;
   ```
   
   ```mysql
   +--------+--------+
   | 员工   | 领导    |
   +--------+--------+
   | SMITH  | FORD   |
   | ALLEN  | BLAKE  |
   | WARD   | BLAKE  |
   | JONES  | KING   |
   | MARTIN | BLAKE  |
   | BLAKE  | KING   |
   | CLARK  | KING   |
   | SCOTT  | JONES  |
   | TURNER | BLAKE  |
   | ADAMS  | SCOTT  |
   | JAMES  | BLAKE  |
   | FORD   | JONES  |
   | MILLER | CLARK  |
   +--------+--------+
   13 rows in set (0.00 sec)
   ```
   





## 外连接

什么是外连接，和内连接有什么区别？

1. 内连接
   假设A和B表进行连接，使用内连接的话，凡是A表和B表能够匹配的记录查询出来。AB两张表没有主副之分，两张表是平等的。

2. 外连接
   假设A和B表进行连接，使用外连接的话，AB两张表中有一张表是主表，一张是福表，主要查询主表中的数据，捎带查询副表。当副表中的数据没有和主表中的数据匹配上，附表自动模拟出NULL与之匹配。

3. 外连接的分类

   - 左外连接（左连接）
     左边那张表是主表
   - 右外连接（右连接）
     右边那张表是主表

   > 左连接有右连接的写法，右连接也会有对应的左连接的写法。

外连接最重要的特点是：主表的数据无条件的全部查询出来。



1. 找出每个员工的上级领导（所有员工必须全部查询出来）

   ### 左外连接

   ```sql
   select A.ENAME as '员工', B.ENAME as '领导' from EMP A left outer join EMP B on A.MGR=B.EMPNO;
   ```

   ```mysql
   +--------+--------+
   | 员工    | 领导   |
   +--------+--------+
   | SMITH  | FORD   |
   | ALLEN  | BLAKE  |
   | WARD   | BLAKE  |
   | JONES  | KING   |
   | MARTIN | BLAKE  |
   | BLAKE  | KING   |
   | CLARK  | KING   |
   | SCOTT  | JONES  |
   | KING   | NULL   |
   | TURNER | BLAKE  |
   | ADAMS  | SCOTT  |
   | JAMES  | BLAKE  |
   | FORD   | JONES  |
   | MILLER | CLARK  |
   +--------+--------+
   14 rows in set (0.00 sec)
   ```

   `left join`左外连接，A表是主表

   ### 右外连接

   ```sql
   select A.ENAME as '员工', B.ENAME as '领导' from EMP B right outer join EMP A on A.MGR=B.EMPNO;
   ```
   
   ```mysql
   +--------+--------+
   | 员工    | 领导   |
   +--------+--------+
   | SMITH  | FORD   |
   | ALLEN  | BLAKE  |
   | WARD   | BLAKE  |
   | JONES  | KING   |
   | MARTIN | BLAKE  |
   | BLAKE  | KING   |
   | CLARK  | KING   |
   | SCOTT  | JONES  |
   | KING   | NULL   |
   | TURNER | BLAKE  |
   | ADAMS  | SCOTT  |
   | JAMES  | BLAKE  |
   | FORD   | JONES  |
   | MILLER | CLARK  |
   +--------+--------+
   14 rows in set (0.00 sec)
   ```
   
   > 无论是left outer join还是right outer join，其中的outer是可以缺省的。
   
2. 找出哪个部门没有员工

   ```sql
   select A.* from DEPT A left outer join EMP B on A.DEPTNO=B.DEPTNO where B.DEPTNO is null;
   ```

   ```mysql
   +--------+------------+--------+
   | DEPTNO | DNAME      | LOC    |
   +--------+------------+--------+
   |     40 | OPERATIONS | BOSTON |
   +--------+------------+--------+
   1 row in set (0.00 sec)
   ```

   



### 三张表连接查询

1. 找出每一个员工的部门名称以及工资等级。

   ```sql
   select e.ename, d.dname, s.grade from emp e 
   left outer join dept d on e.deptno = d.deptno
   left outer join salgrade s on e.sal between s.losal and s.hisal;
   ```

   ```mysql
   +--------+------------+-------+
   | ename  | dname      | grade |
   +--------+------------+-------+
   | SMITH  | RESEARCH   |     1 |
   | ALLEN  | SALES      |     3 |
   | WARD   | SALES      |     2 |
   | JONES  | RESEARCH   |     4 |
   | MARTIN | SALES      |     2 |
   | BLAKE  | SALES      |     4 |
   | CLARK  | ACCOUNTING |     4 |
   | SCOTT  | RESEARCH   |     4 |
   | KING   | ACCOUNTING |     5 |
   | TURNER | SALES      |     3 |
   | ADAMS  | RESEARCH   |     1 |
   | JAMES  | SALES      |     1 |
   | FORD   | RESEARCH   |     4 |
   | MILLER | ACCOUNTING |     2 |
   +--------+------------+-------+
   ```

   

2. 找出每一个员工的部门名称、工资等级、以及上级领导。

   ```sql
   select e.ename, b.ename as '领导', d.dname, s.grade from emp e 
   left outer join emp b on e.mgr=b.empno
   left outer join dept d on e.deptno = d.deptno
   left outer join salgrade s on e.sal between s.losal and s.hisal;
   ```

   ```mysql
   +--------+--------+------------+-------+
   | ename  | 领导   | dname      | grade |
   +--------+--------+------------+-------+
   | SMITH  | FORD   | RESEARCH   |     1 |
   | ALLEN  | BLAKE  | SALES      |     3 |
   | WARD   | BLAKE  | SALES      |     2 |
   | JONES  | KING   | RESEARCH   |     4 |
   | MARTIN | BLAKE  | SALES      |     2 |
   | BLAKE  | KING   | SALES      |     4 |
   | CLARK  | KING   | ACCOUNTING |     4 |
   | SCOTT  | JONES  | RESEARCH   |     4 |
   | KING   | NULL   | ACCOUNTING |     5 |
   | TURNER | BLAKE  | SALES      |     3 |
   | ADAMS  | SCOTT  | RESEARCH   |     1 |
   | JAMES  | BLAKE  | SALES      |     1 |
   | FORD   | JONES  | RESEARCH   |     4 |
   | MILLER | CLARK  | ACCOUNTING |     2 |
   +--------+--------+------------+-------+
   ```

   

# 子查询

## 什么是子查询？

什么是子查询？子查询都可以出现在哪里？

- 什么是子查询？
  select语句当中嵌套select语句，被嵌套的select语句是子查询。

- 子查询可以出现在哪里？

  ```sql
  select
  	...(select)
  from
  	...(select)
  where
  	...(select)
  ```

  

## 在where语句中使用子查询

1. 找出高于平均工资的员工信息。

   ```sql
   select * from emp where sal > (select avg(sal) from emp);
   ```

   ```mysql
   +-------+-------+-----------+------+------------+---------+------+--------+
   | EMPNO | ENAME | JOB       | MGR  | HIREDATE   | SAL     | COMM | DEPTNO |
   +-------+-------+-----------+------+------------+---------+------+--------+
   |  7566 | JONES | MANAGER   | 7839 | 1981-04-02 | 2975.00 | NULL |     20 |
   |  7698 | BLAKE | MANAGER   | 7839 | 1981-05-01 | 2850.00 | NULL |     30 |
   |  7782 | CLARK | MANAGER   | 7839 | 1981-06-09 | 2450.00 | NULL |     10 |
   |  7788 | SCOTT | ANALYST   | 7566 | 1987-04-19 | 3000.00 | NULL |     20 |
   |  7839 | KING  | PRESIDENT | NULL | 1981-11-17 | 5000.00 | NULL |     10 |
   |  7902 | FORD  | ANALYST   | 7566 | 1981-12-03 | 3000.00 | NULL |     20 |
   +-------+-------+-----------+------+------------+---------+------+--------+
   ```

2. 找出每个部门的平均薪资等级。

   ```sql
   select d.dname, s.grade, (select avg(sal) from emp e where e.deptno=d.deptno ) as avgsal from dept d 
   left join salgrade s on 
   (select avg(sal) from emp e where e.deptno=d.deptno ) between s.losal and s.hisal;
   ```

   ```mysql
   +------------+-------+-------------+
   | dname      | grade | avgsal      |
   +------------+-------+-------------+
   | ACCOUNTING |     4 | 2916.666667 |
   | RESEARCH   |     4 | 2175.000000 |
   | SALES      |     3 | 1566.666667 |
   | OPERATIONS |  NULL |        NULL |
   +------------+-------+-------------+
   ```

   

   以上代码是我写的，下面的是教程里的(我做了些小变动)，对比之后惨不忍睹

   ```sql
   select d.dname, t.avgsal, s.grade from 
   	(select deptno, avg(sal) as avgsal from emp group by deptno) t
   left join dept d on d.deptno = t.deptno
   left join salgrade s on t.avgsal between s.losal and hisal;
   ```

   ```mysql
   +------------+-------------+-------+
   | dname      | avgsal      | grade |
   +------------+-------------+-------+
   | RESEARCH   | 2175.000000 |     4 |
   | SALES      | 1566.666667 |     3 |
   | ACCOUNTING | 2916.666667 |     4 |
   +------------+-------------+-------+
   ```

3. 找出每个部门薪资等级的平均等级。

   ```sql
   select e1.deptno, avg(s.GRADE) from emp e1 join salgrade s on e1.sal between s.losal and s.hisal group by e1.deptno;
   ```

   ```mysql
   +--------+--------------+
   | deptno | avg(s.GRADE) |
   +--------+--------------+
   |     20 |       2.8000 |
   |     30 |       2.5000 |
   |     10 |       3.6667 |
   +--------+--------------+
   3 rows in set (0.00 sec)
   ```

   

## 在select后面嵌套子查询

1. 找出每个员工所在的部门名称，要求显示员工名和部门名。

   - 找出员工

     ```sql
     select e1.ename from emp e1;
     ```

   - 找出部门

     ```sql
     select d.dname from dept d;
     ```

   - 显示员工名和部门名

     ```sql
     select
     	e1.ename,
     	(select d.dname from dept d where d.deptno = e1.deptno) as dname
     from emp e1;
     ```

   ```mysql
   +--------+------------+
   | ename  | dname      |
   +--------+------------+
   | SMITH  | RESEARCH   |
   | ALLEN  | SALES      |
   | WARD   | SALES      |
   | JONES  | RESEARCH   |
   | MARTIN | SALES      |
   | BLAKE  | SALES      |
   | CLARK  | ACCOUNTING |
   | SCOTT  | RESEARCH   |
   | KING   | ACCOUNTING |
   | TURNER | SALES      |
   | ADAMS  | RESEARCH   |
   | JAMES  | SALES      |
   | FORD   | RESEARCH   |
   | MILLER | ACCOUNTING |
   +--------+------------+
   ```





# union的用法

union可以将查询结果相加，在进行相加时两条语句的查询结果列数必须一样。

1. 找出工作岗位是SALESMAN和MANAGER的员工。

   第一种

   ```sql
   select * from emp where job in('SALESMAN', 'MANAGER');
   ```

   第二种

   ```sql
   select * from emp where job = 'SALESMAN' or job = 'MANAGER';
   ```

   第三种使用union

   ```sql
   select * from emp where job = 'SALESMAN'
   union 
   select * from emp where job = 'MANAGER';
   ```

   



# limit

limit是mysql特有的，其它数据库没有，不通用。limit取结果集中的部分数据

- 语法机制：limit startIndex，length
  startIndex表示起使位置,从0开始。
  length表示取几个



1. 取出工资前五名的员工。

   ```sql
   select * from emp order by sal desc limit 0, 5;
   ```

   ```mysql
   +-------+-------+-----------+------+------------+---------+------+--------+
   | EMPNO | ENAME | JOB       | MGR  | HIREDATE   | SAL     | COMM | DEPTNO |
   +-------+-------+-----------+------+------------+---------+------+--------+
   |  7839 | KING  | PRESIDENT | NULL | 1981-11-17 | 5000.00 | NULL |     10 |
   |  7788 | SCOTT | ANALYST   | 7566 | 1987-04-19 | 3000.00 | NULL |     20 |
   |  7902 | FORD  | ANALYST   | 7566 | 1981-12-03 | 3000.00 | NULL |     20 |
   |  7566 | JONES | MANAGER   | 7839 | 1981-04-02 | 2975.00 | NULL |     20 |
   |  7698 | BLAKE | MANAGER   | 7839 | 1981-05-01 | 2850.00 | NULL |     30 |
   +-------+-------+-----------+------+------------+---------+------+--------+
   ```

   > 如果limit startIndex第一个数字没写，那么默认值就是0。limit是sql语句最后执行的一个环节。

2. 取出工资排名在第4到第9名的员工

   ```sql
   select * from emp order by sal desc limit 3, 6;
   ```

   ```mysql
   +-------+--------+----------+------+------------+---------+--------+--------+
   | EMPNO | ENAME  | JOB      | MGR  | HIREDATE   | SAL     | COMM   | DEPTNO |
   +-------+--------+----------+------+------------+---------+--------+--------+
   |  7566 | JONES  | MANAGER  | 7839 | 1981-04-02 | 2975.00 |   NULL |     20 |
   |  7698 | BLAKE  | MANAGER  | 7839 | 1981-05-01 | 2850.00 |   NULL |     30 |
   |  7782 | CLARK  | MANAGER  | 7839 | 1981-06-09 | 2450.00 |   NULL |     10 |
   |  7499 | ALLEN  | SALESMAN | 7698 | 1981-02-20 | 1600.00 | 300.00 |     30 |
   |  7844 | TURNER | SALESMAN | 7698 | 1981-09-08 | 1500.00 |   0.00 |     30 |
   |  7934 | MILLER | CLERK    | 7782 | 1982-01-23 | 1300.00 |   NULL |     10 |
   +-------+--------+----------+------+------------+---------+--------+--------+
   ```





# 表

建表语句的语法格式：

```sql
create table 表名 (
	字段名1 数据类型，
  字段名2 数据类型，
  字段名3 数据类型，
);
```



## 关于MySQL当中常用字段的数据类型。

常用的数据类型有：

| 类型    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| int     | 整数型（java中的int）                                        |
| bigint  | 长整形（java中的Long）                                       |
| float   | 浮点型（java中的float、double）                              |
| char    | 定长字符串（String）                                         |
| varchar | 可变长字符串（StringBuffer/StringBuilder）                   |
| data    | 日期类型（对应Java中的java.sql.Date类型）                    |
| BLOB    | 二进制大对象（存储图片、视频等流媒体信息）Binary Large Object（对应java中的Object） |
| CLOB    | 字符大对象（存储较大文本，比如：可以存储4G的字符串）Character Large Object（对应java中的Object） |



### char和varchar如何选择？

在实际开发中，当某个字段中的数据长度不发生改变的时候，是定长的，例如：性别、生日等都是采用char。

当一个字段的数据长度不确定，例如：简介、姓名等都是采用varchar。





### 建表

> 表名在数据库当中一般建议以**t_**或者**tbl_**开始。

创建学生表：

	学生信息有：

 -  学号
    bigint

 -  姓名
    varchar

 -  性别

    char

 -  班级编号

    int

 -  生日

    char



```sql
create table t_student(
	stuno bigint,
  name varchar(255),
  sex char(1),
  classno varchar(255),
  birth char(10)
);
```

```mysql
show tables;
+--------------------------+
| Tables_in_upyou_database |
+--------------------------+
| DEPT                     |
| EMP                      |
| SALGRADE                 |
| t_student                |
+--------------------------+

```
查看刚刚建的表的结构
```mysql
desc t_student;
```

```mysql
+---------+--------------+------+-----+---------+-------+
| Field   | Type         | Null | Key | Default | Extra |
+---------+--------------+------+-----+---------+-------+
| stuno   | bigint       | YES  |     | NULL    |       |
| name    | varchar(255) | YES  |     | NULL    |       |
| sex     | char(1)      | YES  |     | NULL    |       |
| classno | varchar(255) | YES  |     | NULL    |       |
| birth   | char(10)     | YES  |     | NULL    |       |
+---------+--------------+------+-----+---------+-------+
```



## insert语句插入数据

语法格式：

```sql
insert into 表名(字段名1, 字段名2, 字段名3) values(值1, 值2, 值3);
```

> 字段的数量和值的数量相同，并且数据类型要对应相同。

```sql
insert into t_student(stuno, name, sex, classno, birth) values(020413, 'UpYou', '男', 2020414, '2002-04-13');
```

若显示一行受影响表示操作成功

```mysql
Query OK, 1 row affected (0.00 sec)
```

查询刚刚插入的数据

```sql
select * from t_student;
```

```mysql
+-------+-------+------+---------+------------+
| stuno | name  | sex  | classno | birth      |
+-------+-------+------+---------+------------+
| 20413 | UpYou | 男   | 2020414 | 2002-04-13 |
+-------+-------+------+---------+------------+
```

如果字段可以为空，并且你没有给该字段插入值时，会字段将NULL插入到该字段。(插入NULL说的并不准确，因为是不会给未指定的字段插入数据，查询的时候是将为**空**的数据显示为NULL)

```sql
insert into t_student(name)values('UoYou');
```

```mysql
+-------+-------+------+---------+------------+
| stuno | name  | sex  | classno | birth      |
+-------+-------+------+---------+------------+
| 20413 | UpYou | 男   | 2020414 | 2002-04-13 |
|  NULL | UoYou | NULL | NULL    | NULL       |
+-------+-------+------+---------+------------+
```



> 当一条insert语句执行成功之后，表格当中必然会多一条记录。即使多的这一行记录当中某些字段是NULL，后期也没有办法再执行insert语句插入数据了，只能使用update进行更新。



如果不指定插入的字段时，values中列的字段、顺序必须与表格结构一致(多一个不行，少一个不行)。

```sql
insert into t_student values(020413, 'UpYou', '男', 2020414, '2002-04-13');
```



一次插入多行数据

```sql
insert into
	t_student(stuno, name, sex, classno, birth) 
values 
	(020413, 'UpYou一号', '男', 2020414, '2002-04-13'),
	(020413, 'UpYou二号', '男', 2020414, '2002-04-13'),
	(020413, 'UpYou三号', '男', 2020414, '2002-04-13');
```





### 表的复制以及批量插入

#### 表的复制

语法：
将查询结果当作表创建出来。

```sql
create table 表名 as DQL语句;
```




```sql
create table emp2 as select * from emp;
```

```mysql
Query OK, 14 rows affected, 2 warnings (0.01 sec)
Records: 14  Duplicates: 0  Warnings: 2
```

```sql
select * from emp2;
```

```mysql
+-------+--------+-----------+------+------------+---------+---------+--------+
| EMPNO | ENAME  | JOB       | MGR  | HIREDATE   | SAL     | COMM    | DEPTNO |
+-------+--------+-----------+------+------------+---------+---------+--------+
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 |  800.00 |    NULL |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 |  300.00 |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 |  500.00 |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 |    NULL |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 |    NULL |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 |    NULL |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 | 3000.00 |    NULL |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 |    NULL |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 |    0.00 |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-05-23 | 1100.00 |    NULL |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 |  950.00 |    NULL |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 |    NULL |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 |    NULL |     10 |
+-------+--------+-----------+------+------------+---------+---------+--------+
14 rows in set (0.00 sec)
```



#### 批量插入

将查询结果批量插入到某张表中。

语法

```sql
insert into 表名 as DQL语句;
```

```sql
show create table emp;
```

```mysql
CREATE TABLE `emp2` (
  `EMPNO` int NOT NULL,
  `ENAME` varchar(10) DEFAULT NULL,
  `JOB` varchar(9) DEFAULT NULL,
  `MGR` int DEFAULT NULL,
  `HIREDATE` date DEFAULT NULL,
  `SAL` double(7,2) DEFAULT NULL,
  `COMM` double(7,2) DEFAULT NULL,
  `DEPTNO` int DEFAULT NULL,
  PRIMARY KEY (`EMPNO`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

```sql
insert into emp2 as select * from emp;
```

```mysql
Query OK, 14 rows affected (0.00 sec)
Records: 14  Duplicates: 0  Warnings: 0
```

```sql
select * from emp2 limit 0, 2;
```

```mysql
+-------+-------+----------+------+------------+---------+--------+--------+
| EMPNO | ENAME | JOB      | MGR  | HIREDATE   | SAL     | COMM   | DEPTNO |
+-------+-------+----------+------+------------+---------+--------+--------+
|  7369 | SMITH | CLERK    | 7902 | 1980-12-17 |  800.00 |   NULL |     20 |
|  7499 | ALLEN | SALESMAN | 7698 | 1981-02-20 | 1600.00 | 300.00 |     30 |
+-------+-------+----------+------+------------+---------+--------+--------+
```







## 定义默认值

将`t_student`删掉，重新创建一张表，并且将sex字段默认值设为`男`

```sql
drop table if exists t_student;
```

```sql
create table t_student(
	stuno bigint,
  name varchar(255),
  sex char(1) default '男',
  classno varchar(255),
  birth char(10)
);
```

```sql
desc t_student;
```

此时Default栏就有了一个默认值

```mysql
+---------+--------------+------+-----+---------+-------+
| Field   | Type         | Null | Key | Default | Extra |
+---------+--------------+------+-----+---------+-------+
| stuno   | bigint       | YES  |     | NULL    |       |
| name    | varchar(255) | YES  |     | NULL    |       |
| sex     | char(1)      | YES  |     | 男      |       |
| classno | varchar(255) | YES  |     | NULL    |       |
| birth   | char(10)     | YES  |     | NULL    |       |
+---------+--------------+------+-----+---------+-------+
```

再插入数据，不设置`sex`

```sql
insert into t_student(stuno, name, classno, birth) values(020413, 'UpYou', 2020414, '2002-04-13');
```

```sql
select * from t_student;
```

```mysql
+-------+-------+------+---------+------------+
| stuno | name  | sex  | classno | birth      |
+-------+-------+------+---------+------------+
| 20413 | UpYou | 男   | 2020414 | 2002-04-13 |
+-------+-------+------+---------+------------+
1 row in set (0.00 sec)
```

再插入一条数据，设置`sex`

```sql
insert into t_student(stuno, name, sex, classno, birth) values(020413, 'UpYou那并不存在的妹妹', '女', 2020414, '2003-04-13');
```

```sql
select * from t_student;
```

```mysql
+-------+----------------------------+------+---------+------------+
| stuno | name                       | sex  | classno | birth      |
+-------+----------------------------+------+---------+------------+
| 20413 | UpYou                      | 男   | 2020414 | 2002-04-13 |
| 20413 | UpYou那并不存在的妹妹         | 女   | 2020414 | 2003-04-13 |
+-------+----------------------------+------+---------+------------+
```





## update 修改数据

语法格式

```sql
update 表名 set 字段名1=值1， 字段名2=值2... where 条件;
```

> **没有条件整张表数据全部更新。**



1. 将部门1o的LOC修改为SHANGHAI，将部门名称修改为RENSHIBU

   ```sql
   update dept1 set DNAME='RENSHIBU', LOC='SHANGHAI' where deptno=10;
   ```
   
   ```mysql
   Query OK, 1 row affected (0.00 sec)
   Rows matched: 1  Changed: 1  Warnings: 0
   ```
   
```sql
   select * from dept1;
```

   ```mysql
   +--------+------------+----------+
   | DEPTNO | DNAME      | LOC      |
   +--------+------------+----------+
   |     10 | RENSHIBU   | SHANGHAI |
   |     20 | RESEARCH   | DALLAS   |
   |     30 | SALES      | CHICAGO  |
   |     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
   4 rows in set (0.00 sec)
   ```

   



如果没**where**或者**忘记**给**where**。。。。。那就完蛋了，跑路吧！！！

```sql
update dept1 set DNAME='RENSHIBU', LOC='SHANGHAI';
```

```mysql
Query OK, 3 rows affected (0.00 sec)
Rows matched: 4  Changed: 3  Warnings: 0
```

```sql
select * from dept1;
```

```mysql
+--------+----------+----------+
| DEPTNO | DNAME    | LOC      |
+--------+----------+----------+
|     10 | RENSHIBU | SHANGHAI |
|     20 | RENSHIBU | SHANGHAI |
|     30 | RENSHIBU | SHANGHAI |
|     40 | RENSHIBU | SHANGHAI |
+--------+----------+----------+
```





## 删除数据

语法格式

```sql
delete from 表名 where 条件;
```

> 没有条件**全部删除**。



1. 删除10部门数据

   ```mysql
   +--------+------------+----------+
   | DEPTNO | DNAME      | LOC      |
   +--------+------------+----------+
   |     10 | RENSHIBU   | SHANGHAI |
   |     20 | RESEARCH   | DALLAS   |
   |     30 | SALES      | CHICAGO  |
   |     40 | OPERATIONS | BOSTON   |
   +--------+------------+----------+
   ```

   ```sql
   delete from dept1 where deptno = 10;
   ```

   ```mysql
   Query OK, 1 row affected (0.00 sec)
   ```

   ```sql
   select * from dept1;
   ```

   ```mysql
   +--------+------------+---------+
   | DEPTNO | DNAME      | LOC     |
   +--------+------------+---------+
   |     20 | RESEARCH   | DALLAS  |
   |     30 | SALES      | CHICAGO |
   |     40 | OPERATIONS | BOSTON  |
   +--------+------------+---------+
   3 rows in set (0.00 sec)
   ```

   

由于delete删除数据较慢，所以删除全部数据的时候使用`truncate`删除大表。(重点)

```mysql
truncate table dept1;
```

```sql
select * from dept1;
```

```mysql
 Empty set (0.00 sec)
```

> truncate是将表截断（就剩下表头），不可回滚。truncate删除之后不可回滚，所以必须确认该表数据是否真的删除。



## 表约束

约束，英语单词（Constraint）



### 什么是约束？

在创建表的时候，可以给表的字段添加约束，添加约束是为了保证数据的完整性、合法性、有效性。

常见的约束有：

- 非空约束（not null）

  约束的字段不能为NULL

- 唯一约束（unique）
  约束的字段不能重复

- 主键约束（primary key，简称PK）
  约束的字段既不能为NULL也不能重复

- 外键约束（foreign key，简称FK）

- 检查约束（check）
  Oracle数据库有check约束，但是mysql没有，目前mysql不支持该约束。



> 增删改查术语：CRUD
> Create Retrieve Update Delete

### 非空约束

```sql
drop table if exists t_user;
create table t_user(
	id int ,
  user_name varchar(255) not null,
  password varchar(255)
);
```

```mysql
desc t_user;
```

```mysql
+-----------+--------------+------+-----+---------+-------+
| Field     | Type         | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+-------+
| id        | int          | YES  |     | NULL    |       |
| user_name | varchar(255) | NO   |     | NULL    |       |
| password  | varchar(255) | YES  |     | NULL    |       |
+-----------+--------------+------+-----+---------+-------+
```

`user_name`字段设置了非空约束，现在插入一条`user_name`为空的数据

```sql
insert into t_user(id, password) values(1, '123456789');
```

插入异常

```mysql
Field 'user_name' doesn't have a default value
```



### 唯一性约束

唯一约束修饰的字段具有唯一性，不能为空， 不能重复。但可以为NULL。

单个字段约束专业术语叫：列级约束

多个字段约束专业术语叫：表级约束

表级约束：

```sql
create table 表名(
  字段1,
  字段2,
  字段3,
  unique(字段1, 字段2)
);
```



---



```sql
drop table if exists t_user;
create table t_user(
	id int,
  user_name varchar(255) unique,
  password varchar(255)
);
```

```mysql
desc t_user;
```

```mysql
+-----------+--------------+------+-----+---------+-------+
| Field     | Type         | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+-------+
| id        | int          | YES  |     | NULL    |       |
| user_name | varchar(255) | YES  | UNI | NULL    |       |
| password  | varchar(255) | YES  |     | NULL    |       |
+-----------+--------------+------+-----+---------+-------+
```

插入两条数据数据

```sql
insert into t_user(id, user_name, password) 
values
	(1, 'UpYou', '123'),
	(2, 'UpYou', '123456');
```

出现异常

```mysql
ERROR 1062 (23000): Duplicate entry 'UpYou' for key 't_user.user_name'
```

插入两条数据，让`user_name`为NULL

```sql
insert into t_user(id, password) 
values
	(1, '123'),
	(2, '123456');
```

插入成功

```mysql
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

```sql
select * from t_user;
```

```mysql
+------+-----------+----------+
| id   | user_name | password |
+------+-----------+----------+
|    1 | NULL      | 123      |
|    2 | NULL      | 123456   |
+------+-----------+----------+
2 rows in set (0.00 sec)
```

> 一张表中如果有多个字段有唯一性约束（unique），那么意思是联合起来约束。



### 主键约束

怎么给一张表添加主键约束？

```sql
drop table if exists t_user;
create table t_user(
	id int primary key,
  user_name varchar(255),
  password varchar(255)
);
```

```mysql
desc t_user;
```

```mysql
+-----------+--------------+------+-----+---------+-------+
| Field     | Type         | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+-------+
| id        | int          | NO   | PRI | NULL    |       |
| user_name | varchar(255) | YES  |     | NULL    |       |
| password  | varchar(255) | YES  |     | NULL    |       |
+-----------+--------------+------+-----+---------+-------+
```

在表中插入一条数据。

```sql
insert into t_user values(1, 'UpYou', '123');
```

再插入一条`id`为1的数据

```sql
insert into t_user values(1, 'UpYou那个不存在的妹妹', '123456');
```

错误，id重复

```mysql
ERROR 1062 (23000): Duplicate entry '1' for key 't_user.PRIMARY'
```

不插入id试试

```sql
insert into t_user(user_name, password) values('UpYou那个不存在的妹妹', '123456');
```

错误，没有指定默认值，也就是说不能为NULL

```mysql
ERROR 1364 (HY000): Field 'id' doesn't have a default value
```



#### 主键相关术语

- 主键约束

  primary key

- 主键字段
  id字段添加primary key之后，id叫做主键字段

- 主键值
  id字段添加primary key之后，id的每一个值都是主键值。



#### 主键有什么用？

- 表的设计三范式中有要求，第一范式要求任何一张表都应该有主键。
- 主键的作用：主键值是这行记录在这张表中的唯一标识。（就像一个人的身份证号码一样）



#### 主键的分类

工具主键字段的字段数量来划分：

- 单一主键
  比较常用的

- 复合主键

  多个字段联合起来添加一个主键约束。复合主键不建议使用，因为复合主键违背三范式

根据主键性质来划分：

- 自然主键
  主键值最好就是一个和业务没有任何关系的自然数
- 业务主键（不推荐用）
  主键值和系统的业务挂钩，例如：拿着银行卡的卡号做主键，拿着身份证号做主键。



> 一张表的主键约束只能有一个



#### 主键值自增

`auto_increment`将`primary key`修饰的字段自动维护一个自增的数，从`1`开始以`1`递增。

```sql
drop table if exists t_user;
create table t_user(
	id int primary key auto_increment,
  user_name varchar(255)
);
```

```sql
desc t_user;
```

```mysql
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| id        | int          | NO   | PRI | NULL    | auto_increment |
| user_name | varchar(255) | YES  |     | NULL    |                |
+-----------+--------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)
```

现在插入一个值，但不设置`id`

```sql
insert into t_user(user_name) values ('UpYou'), ('UpYou那并不存在的妹妹');
```

```sql
select * from t_user;
```

```mysql
+----+-------------------------------+
| id | user_name                     |
+----+-------------------------------+
|  1 | UpYou                         |
|  2 | UpYou那并不存在的妹妹        		|
+----+-------------------------------+
```

> Oracle当中也提供了一个自增机制，叫做：序列（sequence）





### 外键约束

关于外键约束的术语：

- 外键约束：
  foreign key

- 外键字段
  添加有外键约束的字段

- 外键值
  外键字段中的每一个值

  

#### 业务背景

1. 请设计数据库表，用来维护学生和班级的信息。

   1. 一张表存储所有数据

      ```mysql
      +----------+-------------------------------+---------+------------------------------+
      | no(PK)   | name                          | classno | classname                    |
      +----------+-------------------------------+---------+------------------------------+
      |    1     | UpYou                         |     101 | 传说中的高中高三1班    		      |
      |    2     | UpYou那并不存在的妹妹     		    |     102 | 传说中的高中高三1班   		       |
      +----------+-------------------------------+---------+------------------------------+
      ```

      这种方式会导致数据冗余。不推荐

   2. 两张表（班级表和学生表）
      `t_class` 班级表

      ```mysql
      +------+------------------------------+
      | cno  | cname                        |
      +------+------------------------------+
      |  101 | 传说中的高中高三1班         	  |
      |  102 | 传说中的高中高三2班      		    |
      |  103 | 传说中的高中高三3班       	    |
      +------+------------------------------+
      ```

      `t_student`学生表

      ```mysql
      +------+----------------------------------+
      | no   | name                             |
      +------+----------------------------------+
      |    1 | UpYou                            |
      |    2 | UpYou那并不存在的妹妹              |
      |    3 | UpYou那并不存在的二妹妹          		|
      +------+----------------------------------+
      ```

      以上两张表看不出他们之间的关系，而第一种方法可以很清晰的看出这个学生所在班级。

      给`t_student`学生表添加一个字段`cno`并且给该字段添加外键约束`FK`，如果不给该字段添加外键约束，那么这个字段中的值可以随便填，如果加上外键约束就表示这个字段的值是某一张表中的某个主键值。

      ```mysql
      +------+-----------------------------------------------------+------+---+
      | no   | name                                                | cno(FK)  |
      +------+-----------------------------------------------------+------+---+
      |    1 | UpYou                                               |    101   |
      |    2 | UpYou那并不存在的妹妹                                  |    101   |
      |    3 | UpYou那并不存在的二妹妹(不在同个班表示很生气)             |    103   |
      +------+-----------------------------------------------------+------+---+
      
      ```

      如果此时修改学生表中的第一条数据`cno`字段为110时，修改出错，因为`t_class`表中没有这个班级。
      `t_student`中的`cno`字段引用`t_class`表中的`cno`字段，此时`t_student`表叫做子表。`t_class`表叫做父表。





- 删除数据的时候先删除子表再删除父表。

- 添加数据的时候，先添加父表再添加子表。

- 创建表的时候，先创建父表再创建子表。

- 删除表的时候先删除子表，再删除父表。



将解决方案二中的表创建出来

```sql
drop table if exists t_student;
drop table if exists t_class;

create table t_class(
	cno int primary key,
  cname varchar(255)
);

create table t_student(
	sno int primary key auto_increment,
  sname varchar(255),
  classno int,
  foreign key(classno) references t_class(cno)
);

```

`t_class`班级表

```mysql
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| cno   | int          | NO   | PRI | NULL    |       |
| cname | varchar(255) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
```

`t_student` 学生表

```mysql
+---------+--------------+------+-----+---------+----------------+
| Field   | Type         | Null | Key | Default | Extra          |
+---------+--------------+------+-----+---------+----------------+
| sno     | int          | NO   | PRI | NULL    | auto_increment |
| sname   | varchar(255) | YES  |     | NULL    |                |
| classno | int          | YES  | MUL | NULL    |                |
+---------+--------------+------+-----+---------+----------------+
```

插入数据，应该先插入父表

```sql
insert into t_class values 
(101, '传说中的高中高三1班'),
(102, '传说中的高中高三2班'),
(103, '传说中的高中高三3班'),
(104, '传说中的高中高三4班'),
(105, '传说中的高中高三5班'),
(106, '传说中的高中高三6班'),
(107, '传说中的高中高三7班');
```

```sql
select * from t_class;
```

```mysql
+-----+------------------------------+
| cno | cname                        |
+-----+------------------------------+
| 101 | 传说中的高中高三1班             |
| 102 | 传说中的高中高三2班             |
| 103 | 传说中的高中高三3班             |
| 104 | 传说中的高中高三4班             |
| 105 | 传说中的高中高三5班             |
| 106 | 传说中的高中高三6班             |
| 107 | 传说中的高中高三7班             |
+-----+------------------------------+
```

再给子表插入数据

```sql
insert into t_student(sname, classno) values 
('UpYou', 102),
('UpYou那并不存在的妹妹', 102),
('UpYou那并不存在的二妹妹（不在同个班表示很生气）', 103);
```

```sql
select * from t_student;
```

```mysql
+-----+--------------------------------------------------------+---------+
| sno | sname                                                  | classno |
+-----+--------------------------------------------------------+---------+
|   1 | UpYou                                                  |     102 |
|   2 | UpYou那并不存在的妹妹                              	      |     102 |
|   3 | UpYou那并不存在的二妹妹（不在同个班表示很生气）               |     103 |
+-----+--------------------------------------------------------+---------+
```

此时如果添加一条不存在的班级

```sql
insert into t_student(sname, classno) values 
('UpYou', 1020);
```

出现错误

```mysql
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`upyou_database`.`t_student`, CONSTRAINT `t_student_ibfk_1` FOREIGN KEY (`classno`) REFERENCES `t_class` (`cno`))
```

新增一个外键为空的数据

```sql
insert into t_student(sname) values('UpYou那并不存在的三妹妹');
```

```sql
select * from t_student;
```

外键可以为空

```mysql
+-----+-------------------------------------------------------+---------+
| sno | sname                                                 | classno |
+-----+-------------------------------------------------------+---------+
|   1 | UpYou                                                 |     102 |
|   2 | UpYou那并不存在的妹妹                                    |     102 |
|   3 | UpYou那并不存在的二妹妹（不在同个班表示很生气）              |     103 |
|   5 | UpYou那并不存在的三妹妹                                  |    NULL |
+-----+-------------------------------------------------------+---------+
```



> - 外键字段引用其他表的某个字段的时候，被引用的字段必须是主键吗？
>
>   被引用的字段不一定是主键，但至少具有 `unique` 约束。



# 存储引擎

## 什么是存储引擎

储存引擎描述的是表的存储方式。在MySQL当中，不同的存储引擎会有不同的存储方式。存储引擎这个名字只有在mysql中存在。（Oracle中有对应的机制，但是不叫做存储引擎，Oracle中没有特殊的名字，就叫：表的存储方式）



MySQL支持很多存储引擎，每一种存储引擎都对应了一种不同的存储方式。每一个存储引擎都有自己的优缺点，需要在合适的时机选择合适的存储引擎。

## 查看存储引擎

查看建表语句可以查看一张表使用的是怎样的存储引擎。

```mysql
show create table emp;
```

`ENGINE=InnoDB` 指明了这张表用的是`InnoDB`存储引擎。下面这条代码才是一个完整的建表语句

```mysql
| emp   | CREATE TABLE emp (
  ......
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
```

建表的时候可以指定存储引擎和字符集，如果创建表的时候没有指定存储引擎，默认使用的是**InnoDB**存储引擎，字符集默认采用的是**utf8mb4**（在我MacOS中是如此，我MySQL版本8.0+）



## 常见的存储引擎

MySQL8.0.22中支持的存储引擎

```mysql
show engines;
```

```mysql
*************************** 1. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 2. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 4. row ***************************
      Engine: FEDERATED
     Support: NO
     Comment: Federated MySQL storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
*************************** 5. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 8. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 9. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
9 rows in set (0.00 sec)
```



### MyISAM存储引擎

- MyISAM存储引擎是MySQL最常用的引擎。但是这种引擎不是默认的。
- MyISAM存储引擎不支持事务。
- 它管理的表具有以下特征：
  1. 使用三个文件表示每张表
     - 格式文件 - 存储表结构的定义（mytable.frm）
     - 数据文件 - 存储表行的内容（mytable.MYD）
     - 索引文件 - 存储表上索引（mytable.MYI）
  2. 灵活的 **AUTO_INCREMENT** 字段处理。
  3. 可被压缩，节省存储空间。并且可以转换为只读表，提高检索效率。



### InnoDB存储引擎

- InnoDB存储引擎是MySQL的缺省引擎。
- 它管理的表具有下列主要特征：
  1. 每个InnoDB表在数据库目录中以`.frm`格式文件表示
  2. InnoDB表空间tablespace被用于存储表的内容，tablespace只是一个逻辑概念，他不是一个具体的文件
  3. 提供一组用来记录事务性活动的日志文件
  4. 用**COMMIT（提交）**、**SAVEPOINT**及**ROLLBACK（回滚）**支持事务处理
  5. 提供全ACID兼容
  6. 在MySQL服务器崩溃后提供自动恢复
  7. 多版本（MVCC）和行级锁定
  8. 支持外键及引用的完整性，包括级联删除和更新。



> 支持事务、行级锁、外键等。这种存储引擎数据的安全得到保障。



### MEMORY 存储引擎

- 使用MEMORY存储引擎的表，其数据存在内存中，且行的长度固定，这两个特点使得MEMORY存储引擎非常快。
- 不支持事务且数据容易丢失，因为所有数据和索引都是存储在内存当中。查询速度最快。以前叫HEPA引擎。
- 在数据库目录内，每个表均以`.frm`格式的文件表示。
- 支持表级锁机制
- 不能存储BLOB字段和TEXT字段



# 事务

事务英语单词：Transaction



## 什么是事务

一个事务是一个完整的业务逻辑单元，不可再分。
比如：银行账户转账，从A账户向B账户转账10000。需要执行两条update语句。

```sql
update t_act set balance = balance - 10000 where actno = 'act-001';
update t_act set balance = balance + 10000 where actno = 'act-002';
```

以上两条DML语句必须同时成功，或者同时失败，不允许出现一条成功，一条失败。要想保证以上的两条DML语句同时成功或者同时失败，那么就需要使用数据库的“事务机制”。

> 和事务相关的语句只有：DML语句。因为DML语句都是和数据库当中的“数据”相关的。事务的存在是为了保证数据的完整性、安全性。

假设所有的业务都能用一条DML语句搞定，这种情况是不需要事务。但实际情况是一个“需求（事务）”需要多条DML语句共同联合完成。



## 事务原理

假设有一个业务，需要先执行一条insert，再执行update，最后执行一条delete，这个业务才算完成。



```txt
开启事务机制(开始)：

执行insert语句 -- > insert...（这条DML语句执行成功之后，把执行记录到数据库的操作历史当中，并不会向文件中保存一条数据，不会真正的修改硬盘上的数据。）

执行update语句 -- > update...（记录历史操作，不会真正修改硬盘上的数据）

执行delete语句 -- > delete..（这个执行也是记录历史操作「记录到缓存」，不会修改硬盘上的数据）

提交事务或回滚(结束)
```

无论是回滚还是提交事务，都会将历史记录给抹除掉，区别在于提交事务后是将硬盘上的数据真正的修改。



## 事务的四大特性

事务包括四大特性：ACID

- A
  原子性：事务是最小的工作单元，不可再分。

- C
  一致性：事务必须保证多条DML语句同时成功或者同时失败

- I
  隔离性：事务A与事务B之间具有隔离。

- D

  持久性：持久性说的是最终数据必须持久化到硬盘文件中，事务才算成功的结束。



### 事务之间的隔离性

事务隔离性存在隔离级别，理论上隔离级别包括4个：

- 第一级别
  读未提交（read uncommitted）：对方事务还未提交，我们当前事务可以读取到对方未提交的数据。读未提交存在脏读现象（Dirty Read）现象：表示读到了脏的数据「数据不稳定」。

- 第二级别
  读已提交（read committed）：对方事务提交之后的数据我方可以读取到。读已提交存在的问题是：不可重复读。

  这种隔离级别解决了脏读问题

- 第三级别
  可重复读（repeatable read）：这种隔离级别解决了：不可重复读问题。
  这种隔离级别存在的问题是：读取到的数据是幻象。

- 第四级别：
  序列化读、串行化读，解决了所有问题，效率低，需要排队。



> Oracle数据库默认的隔离级别是：读已提交。
>
> MySQL数据库默认的隔离级别是：可重复读。



## 演示事务

MySQL中默认情况下是自动提交的。（只要执行任意一条DML语句就提交一次。）

使用`start transaction `可关闭自动提交。



### **准备表**

```sql
drop table if exists t_user;
create table t_user(
	id int primary key auto_increment,
  user_name varchar(255)
);
```

```sql
desc t_user;
```

```mysql
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| id        | int          | NO   | PRI | NULL    | auto_increment |
| user_name | varchar(255) | YES  |     | NULL    |                |
+-----------+--------------+------+-----+---------+----------------+
```



### **演示**

插入一条数据

```sql
insert into t_user(user_name) values('UpYou');
```

事务回滚操作

```mysql
rollback;
```

再查询回滚之后的数据

```sql
select * from t_user;
```

```mysql
+----+-----------+
| id | user_name |
+----+-----------+
|  1 | UpYou     |
+----+-----------+
```

发现数据没有回滚，这说明了MySQL已经默认提交了事务。关闭自动提交机制

```mysql
start transaction;
```

再插入一条数据

```sql
insert into t_user(user_name) values('uncommitted');
```

```sql
select * from t_user;
```

```mysql
+----+-------------+
| id | user_name   |
+----+-------------+
|  1 | UpYou       |
|  2 | uncommitted |
+----+-------------+
```

回滚事务

```mysql
rollback;
```

再查询回滚之后的数据

```sql
select * from t_user;
```

```mysql
+----+-----------+
| id | user_name |
+----+-----------+
|  1 | UpYou     |
+----+-----------+
```

无论是rollback还是committed之后，事务无论如何都是结束了。此时需要重新开启事务。

```mysql
start transaction;
```

插入数据

```sql
insert into t_user(user_name) values('UpYou那个并不存在的妹妹！');
```

```sql
select * from t_user;
```

```mysql
+----+-------------------------------------+
| id | user_name                           |
+----+-------------------------------------+
|  1 | UpYou                               |
|  3 | UpYou那个并不存在的妹妹！              |
+----+-------------------------------------+
```

提交事务

```MYSQL
commit;
```

再尝试回滚

```mysql
rollback;
```

```sql
select * from t_user;
```

```mysql
+----+-------------------------------------+
| id | user_name                           |
+----+-------------------------------------+
|  1 | UpYou                               |
|  3 | UpYou那个并不存在的妹妹！           		|
+----+-------------------------------------+
```







# 索引

## 什么是索引？有什么用？

索引就相当于一本书的目录，通过目录可以快速的找到对应的资源。在数据库方面，查询一张表的时候，有两种检索方式：

- 第一种是：全表扫描
- 第二种方式是**根据索引检索（效率很高）**



索引为什么可以提高检索效率呢？其实最根本的原理是缩小了扫描的范围。索引虽然可以提高索引效率，但是不能随意添加索引，因为索引也是数据库当中的对象，也需要数据库不断的维护。是有维护成本的。比如：
表中的数据经常被修改，这样就不适合添加索引，因为数据一旦修改，索引需要重新排序，进行维护。

> 添加索引是给某一个字段或者说某些字段添加索引。

```sql
select ename from emp where ename = 'SMITH';
```

当ename字段上没有添加索引的时候，以上sql语句会进行全表扫描，扫描ename字段中所有的值。

当ename字段上添加索引的时候，以上sql语句会根据索引扫描，快速定位。

## 什么时候考虑给字段添加索引？

- 当数据量庞大的时候。（根据客户需求，根据线上环境）
- 该字段很少的DML操作。
- 该字段经常出现在where子句中。（经常根据哪个字段查询）



**主键和具有unique约束的字段自动会添加索引。根据主键查询效率较高。**



## 添加索引

语法格式：

```MYSQL
create index 索引名 on 表名(字段名);
```



使用`explain`关键字查看一条查询语句的执行计划。

```mysql
explain select * from emp where sal = 5000;
```

```mysql
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | emp   | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   14 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

结果中有个`type`字段，他的值是`ALL`这意味着它将全表扫描，`rows`是扫描次数。自己给薪资`sal`字段添加索引。

```mysql
create index EMP_SAL_INDEX on emp(sal);
```

```mysql
explain select * from emp where sal = 5000;
```

```mysql
+----+-------------+-------+------------+------+---------------+---------------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key           | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+---------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | emp   | NULL       | ref  | EMP_SAL_INDEX | EMP_SAL_INDEX | 9       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+---------------+---------+-------+------+----------+-------+
```

这次通过索引去检索时，检索的次数只检索了1次



## 删除索引兑现

语法格式：

```mysql
drop index 索引名 on 表名;
```

删除刚刚创建的索引

```mysql
drop index EMP_SAL_INDEX on emp;
```

```mysql
explain select * from emp where sal = 5000;
```

```mysql
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | emp   | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   14 |    10.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
```



## 索引实现原理

索引底层采用的数据结构是：B + Tree;



```sql
select ename from emp where ename = 'SMITH';
```

当ename字段上没有索引的时候，会进行全表扫描，效率较低。给ename字段添加索引：

```mysql
create index EMP_SAL_INDEX on emp(ename);
```

以上sql语句执行的时候，会在内存或者硬盘文件当中生成一个索引，到底是在硬盘还是内存，得看你存储引擎。底层索引进行了排序、分区，索引会携带数据在表中的“物理地址”，最终通过索引检索到数据之后，获取到关联的物理地址，通过物理地址定位到表中的数据，效率是最高的。

```sql
select ename from emp where ename = 'SMITH';
```

以上sql语句会通过索引转换为：

```sql
select ename from emp where 物理地址 = 0x3;
```



## 索引的分类

1. 单一索引
   给单个字段添加索引
2. 复合索引
   给多个字段联合起来添加1个索引
3. 主键索引
   主键上会自动添加索引
4. 唯一索引
   有unique约束的字段上会自动添加索引。



## 索引什么时候失效？

当遇到以下sql语句的时候，sql语句不会走索引

因为索引是通过第一个字母在字典中查找，而以下sql根本不知道第一个字母是什么。无法定位。

```sql
select ename from emp where ename like '%A%';
```







# 视图

## 什么是视图？

视图就是：站在不同的角度去看数据。（同一张表的数据，通过不同的角度去看待）。



## 视图的创建

语法格式：

```mysql
create view 视图名 as DQL语句。
```




```mysql
create view myview as select empno, enmae from emp;
```

> 只有DQL语句才能以视图对象的方式创建。

## 删除视图

语法格式：

```mysql
drop view 视图名称;
```

```mysql
drop view myview;
```



## 使用

对视图进行增删改查，会影响到原表数据。（通过视图影响原表数据的，不是直接操作原表），可以对视图进行CRUD操作



```SQL
SELECT * FROM myview;
```

```MYSQL
+-------+--------+
| empno | ename  |
+-------+--------+
|  7369 | SMITH  |
|  7499 | ALLEN  |
|  7521 | WARD   |
|  7566 | JONES  |
...
+-------+--------+
14 rows in set (0.01 sec)
```

---

拷贝一个备用表

```MYSQL
CREATE TABLE EMP_BAK AS SELECT * FROM EMP;
```

创建这个表的视图

```MYSQL
CREATE VIEW EMP_BAK_VIEW AS SELECT * FROM EMP_BAK;
```

```SQL
SELECT ENAME, EMPNO FROM EMP_BAK_VIEW;
```

```MYSQL
+--------+-------+
| ENAME  | EMPNO |
+--------+-------+
| SMITH  |  7369 |
| ALLEN  |  7499 |
| WARD   |  7521 |
| BLAKE  |  7698 |
...
+--------+
14 rows in set (0.00 sec)
```

更新视图

```SQL
UPDATE EMP_BAK_VIEW SET ENAME = 'UpYou' WHERE EMPNO = 7369;
```

查看视图

```SQL
SELECT ENAME, EMPNO FROM EMP_BAK_VIEW;
```

```MYSQL
+--------+-------+
| ENAME  | EMPNO |
+--------+-------+
| UpYou  |  7369 |
| ALLEN  |  7499 |
| WARD   |  7521 |
| BLAKE  |  7698 |
...
+--------+
14 rows in set (0.00 sec)
```

以上操作都是对视图进行修改，并没有对原表进行修改，查询一下原表的数据是否受到影响

```SQL
SELECT ENAME, DEPTNO FROM EMP_BAK;
```

```MYSQL
+--------+--------+
| ENAME  | DEPTNO |
+--------+--------+
| UpYou  |     20 |
| ALLEN  |     30 |
| WARD   |     30 |
...
+--------+--------+
14 rows in set (0.00 sec)
```



## 视图的作用

视图可以隐藏表的实现细节。保密级别较高的系统，数据库只对外提供视图，程序员只对视图CRUD。





# DBA命令



## 导出数据

在dos命令窗口中执行

导出整个数据库语法格式

```SQL
mysqldump 数据库名>输出路劲 -uroot -p数据库密码
```

```SQL
mysqldump upyou_database >  /Users/yct/Desktop/upyou_database_bak.sql -uroot -p123456;
```



导出指定表语法格式

```mysql
mysqldump 数据库名 表名>路径 -uroot -p数据库密码
```

```mysql
mysqldump upyou_database emp >  /Users/yct/Desktop/upyou_database_bak.sql -uroot -p123456;
```



## 导入数据

若想要导入数据，那么你的数据库必须存在：

```mysql
create database upyou_database;
```

使用这个数据库：

```mysql
use upyou_database;
```

导入数据

```mysql
source /Users/yct/Desktop/upyou_database_bak.sql;
```





# 数据库三范式

## 什么是设计范式

设计表的依据。按照这个三范式设计的表不会出现数据冗余。



## 三范式都是哪些？

1. 第一范式

   任何一张表都应该有主键，并且每一个字段原子性不可再分。

2. 第二范式

   第二范式建立在第一范式的基础之上，所有非主键字段必须完全依赖主键，不能产生部分依赖。（多对多三张表一个关系两个外键）

3. 第三范式
   建立在第二范式的基础之上，所有非主键字段直接依赖主键，不能产生传递依赖。（一对多，两张表，多的表加外键）



> 实际开发中，以满足业务需求为主，有时会拿冗余换执行速度。