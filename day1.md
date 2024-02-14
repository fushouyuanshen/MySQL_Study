## Mysql day1课堂笔记

### 定义
* 数据库(database)：
简称DB。按照**一定格式**存储数据的**文件组合**，实际就是一堆文件

* 数据库管理系统(databaseManagement)：
简称DBMS，管理数据库中的数据，可以对数据库中的数据增删改查
常见的数据库管理系统：
MySQL、Oracle、MS Server、DB2、Sybase等……

* 结构化查询语言SQL

三者之间的关系：
DBMS --执行--> SQL --操作--> DB
由MySQL执行SQL语句，从而实现对数据库的增删改查

### 了解SQL
* 数据库database：一组结构化的文件
* 表table：一种结构化的文件，例如顾客表，员工表，订单表等
* 模式schema：数据库和表的布局及特性的信息
* 列column：表中的一个字段，例如顾客表中的姓名字段、编号字段
* 数据类型datatype：所容许的数据类型，每个表列都有对应的数据类型
* 行row：表中的一个记录record，例如顾客表中的一个顾客、员工表上的一名员工
* 主键primary key：唯一且不为null的列，用于标识一个表中的记录
    * 任意两行都不具有相同的主键值
    * 每个行都必须有一个主键值（主键值不允许NULL值）

### 了解MySQL
* 命令输入在mysql>后;
* 命令用;或\g结束，换句话说，仅按Enter不执行命令；
* 输入help或\h获得帮助，也可以输入更多的文本获得特定命令的帮助（入，输入help select获得使用SELECT语句的帮助）
* 输入quit或exit退出mysql


### 使用MySQL
* 使用test数据库
```sql
use test;
```
---
* 获得可用数据库的列表
```sql
show databases;
```
---
* 获得当前数据库内表的列表
```sql
show tables;
```
---
* 获得customers数据表中的列
```sql
show columns from customers;
```
或者
```sql
describe customers;
```
---
* 进一步了解show命令
```sql
help show;
```
---

**自动增量：**
某些表列需要唯一值，mysql自动为每一行分配下一个可用编号。如用户ID，MySQL自动分配1号、2号……

### 检索数据
> 一些注意事项：
mysql命令面板清屏命令：system cls;
SQL语句中SELECT这种语句尽量大写，为了与小写的列名、表名区分

* 检索单个列的值：
```sql
SELECT column_name FROM table_name;
```
从table_name表中得到column_name列的值

* 检索多个列的值：
```sql
SELECT prod_id, prod_name, prod_price FROM products;
```

* 检索所有列的值：
```sql
SELECT * FROM table_name;
```
尽量不要使用*，会降低检索和应用程序性能

* 检索值不同的行：
```sql
SELECT DISTINCT column_name FROM table_name;
```
> 例子：
1001、1001、1001、1002、1002的列值在该语句下输出为1001、1002

**不能部分使用DISTINCT**
```sql
SELECT DISTINCT prod_id, prod_name, prod_price FROM products;
```
此时只有行与行之间当prod_id、prod_name、prod_price三个列的值都相同时，才不会被输出
> 例子：
数据：
1 盆子 12
2 盘子 13
2 盘子 13
2 筷子 13
输出：
1 盆子 12
2 盘子 13
2 筷子 13

* 限制结果
从第1行开始，限制返回5行
```sql
SELECT column_name FROM table_name LIMIT 5;
```

从第6行开始，限制返回5行
```sql
SELECT column_name FROM table_name LIMIT 5, 5;
```

若剩余行数不足，则返回能返回的行数

* 使用完全限定的表名
```sql
SELECT column_name FROM database_name.table_name;
```
等价于
```sql
SELECT column_name FROM table_name;
```