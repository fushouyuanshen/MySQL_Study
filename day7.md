## MySQL day7 课堂笔记

### 使用游标

#### 游标

游标（cursor）：一个存储在MySQL服务器上的数据库查询，不是SELECT语句，而是SELECT语句返回的结果集

用途：得到结果集的第一行、前进一行或后退一行、多行。

> MySQL游标只能用于存储过程（和函数）



#### 使用游标

* 声明游标，定义游标要使用的SELECT语句
* 一旦声明游标，则必须使用游标，用定义的SELECT语句检索
* 对应填有数据的游标，取出数据
* 结束游标使用后，关闭游标



#### 创建游标

```sql
CREATE PROCEDURE processorders()
BEGIN
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
END;
```

创建游标并定义游标对应的SELECT语句检索订单号



#### 打开和关闭游标

```sql
OPEN ordernumbers;
```

打开游标



```sql
CLOSE ordernumbers;
```

关闭游标



> 关闭游标释放其使用的内部内存和资源，因此当游标不再使用时应该关闭
>
> 关闭游标后将不能被使用，如确需使用，可以再次OPEN
>
> 如果不明确关闭游标，在到达END语句时，MySQL会自动关闭它



```sql
CREATE PROCEDURE processorders()
BEGIN
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	-- Open the cursor
	OPEN ordernumbers;
	
	-- Close the cursor
	CLOSE ordernumbers;
	
END;
```



#### 使用游标数据

```sql
CREATE PROCEDURE processorders()
BEGIN
	-- Declare local variables
	DECLARE o INT;
	
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	-- Open the cursor
	OPEN ordernumbers;
	
	-- Get order number
	FETCH ordernumbers INTO o;
	
	-- Close the cursor
	CLOSE ordernumbers;
	
END;
```

检索订单号第一行数据

FETCH用来将检索到的订单号存入局部声明o中，默认从第一行开始，并只存入一次



```sql
CREATE PROCEDURE processorders()
BEGIN
	-- Declare local variables
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	-- Declare continue handler
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
	
	-- Open the cursor
	OPEN ordernumbers;
	
	-- Loop through all rows
	REPEAT
	
		-- Get order number
		FETCH ordernumbers INTO o;
		
	-- End of loop
	UNTIL done END REPEAT;
	
	-- Close the cursor
	CLOSE ordernumbers;
	
END;
```

循环检索数据，从第一行到最后一行

```sql
DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
```

这句话定义了一个CONTINUE HANDLER，它是在条件出现时被执行的代码，即SQLSTATE'02000'出现时，SET done = 1。SQLSTATE '02000'是未找到条件，也就是行已经找完了。

> DECLARE语句的次序
>
> 局部变量必须在游标、句柄前定义，句柄必须在游标后定义



**最终样例：**

```sql
CREATE PROCEDURE processorders()
BEGIN
	-- Declare local variables
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	DECLARE t DECIMAL(8,2);
	
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	-- Declare continue handler
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
	
	-- Create a table to store the results
	CREATE TABLE IF NOT EXISTS ordertotals
		(order_num INT, total DECIMAL(8,2));
		
	-- Open the cursor
	OPEN ordernumbers;
	
	-- Loop through all rows
	REPEAT
	
		-- Get order number
		FETCH ordernumbers INTO o;
		
		-- Get the total for this order
		CALL ordertotal(o, 1, t);
		
		-- Insert order and total into ordertotals
		INSERT INTO ordertotals(order_num, total)
		VALUES(o, t);
		
	-- End of loop
	UNTIL done END REPEAT;
	
	-- Close the cursor
	CLOSE ordernumbers;
	
END;
```

循环统计对应订单号的总成本，插入到新表之中

```sql
SELECT *
FROM ordertotals;
```

查看结果



### 使用触发器

#### 触发器

触发器：MySQL响应以下任意语句而自动执行的一条MySQL语句

* DELETE
* INSERT
* UPDATE



#### 创建触发器

创建触发器需要的信息：

* 唯一的触发器名（数据库范围内的唯一）
* 触发器关联的表
* 触发器应该响应的活动（DELETE、INSERT或UPDATE）
* 触发器何时执行（处理之前或之后）



例：

```sql
CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'Product added' INTO @arg;
```

创建了名为newproduct的触发器，在插入products数据表之后执行，如果有插入行，则@arg变量变为Product added

> 只有表支持触发器，每个表的每个事件只能对应一个触发器，一个表最多支持6个触发器（INSERT、UPDATE、DELETE前后）。单一触发器不能和多个事件和表相关联

> 如果BEFORE触发器失败，MySQL将不执行请求的操作；若AFTER触发器或语句本身失败，MySQL将不执行AFTER触发器



#### 删除触发器

```sql
DROP TRIGGER newproduct;
```

删除newproduct触发器

触发器不能更新或覆盖，为修改一个触发器，必须先删除后重新创建。



#### 使用触发器

##### INSERT触发器

* 可引用名为NEW的虚拟表，访问被插入的行
* BEFORE INSERT触发器中可更新被插入行的值，通过更新NEW表的值
* 对于AUTO_INCREMENT列，NEW在INSERT执行之前包含0，执行之后包含新的自动生成值

```sql
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW SELECT NEW.order_num INTO @arg;
```



在执行插入操作后，arg的值会改变

```sql
INSERT INTO orders(order_date, cust_id)
VALUES(Now(), 10001);
SELECT @arg;
```

> BEFORE触发器通常用于数据验证和净化（保证插入表中数据是需要的数据）



##### DELETE触发器

* DELETE触发器内，可引用OLD虚拟表，访问被删除的行
* OLD的值只读，不能更新

```sql
CREATE TRIGGER deleteordere BEFORE DELETE ON orders
FOR EACH ROW 
BEGIN
	INSERT INTO archive_orders(order_num, order_date, cust_id)
	VALUES(OLD.order_num, OLD.order_date, OLD.cust_id);
END;
```

在订单删除前，将其保存到名为archive的存档表中。

使用BEFORE DELETE触发器的优点是，如果不能存档，则订单不能被删除。而AFTER DELETE则是无论能否存档，订单都会被删除。



##### UPDATE触发器

* 可引用OLD虚拟表访问以前的值，也可访问NEW虚拟表访问更新的值
* BEFORE UPDATE触发器中，NEW表中的值允许被更新
* OLD表中的值只读且不能更新

```sql
CREATE TRIGGER updatevendor BEFORE UPDATE ON vendors
FOR EACH ROW SET NEW.vend_state = UPPER(NEW.vend_state);
```



### 管理事务处理

#### 事务处理

> 并非所有引擎都支持事务处理，InnoDB是支持的

事务处理（transaction processing）：用来维护数据库的完整性，保证成批的MySQL操作要么完全执行，要么完全不执行

假设要给系统添加订单：

* 从customers表中检查是否有该客户，如果没有则添加
* 检索客户ID
* 添加一行到orders表，把它与客户ID关联
* 检索订单ID
* 添加一行到orderitems表，把它与订单ID关联

假设添加了客户后出现错误倒是订单和商品没有添加到各自的表中，但客户没有订单仍然合法，所以不会报错

但如果添加了订单后报错导致商品没有添加到表中，数据库出现了一个空订单，此时会报错。

为解决这种问题，引入事务处理，保证一组操作不会中途停止。如果没发生错误则完整执行，若发生错误，则回退到没有错误的状态。

* 从customers表中检查是否有该客户，如果没有则添加
* 检索客户ID
* 添加一行到orders表，把它与客户ID关联
* 如果在添加行到orders表中发生故障，回退
* 检索订单ID
* 添加一行到orderitems表，把它与订单ID关联
* 如果在添加行到orderitems表中发生故障，回退

**关键术语：**

* 事务（transaction）指一组SQL语句
* 回退（rollback）指撤销指定SQL语句的过程
* 提交（commit）指将未存储的SQL语句结果写入数据库表
* 保留点（savepoint）指事务处理中设置的临时占位符（place-holder），你可以对它发布回退（与回退整个事务处理不同）



#### 控制事务管理

```sql
START TRANSACTION;
```

标识事务开始



##### 使用ROLLBACK

```sql
SELECT * FROM ordertotals;
START TRANSACTION;
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK;
SELECT * FROM ordertotals;
```

ROLLBACK关键字表示回退到开始事务处，也就是说DELETE FROM和SELECT * FROM两句语句被撤销。

最后一句SELECT语句和第一句的结果是一致的

> 事务处理用来管理INSERT、UPDATE和DELETE语句，不能回退SELECT、CREATE和DROP



##### 使用COMMIT

由于MySQL语句直接对数据库表执行和编写，自动提交保存，这就是所谓的隐含提交（implicit commit）

在事务处理中，提交不会隐含地进行。为明确提交，使用COMMIT语句

```sql
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20010;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;
```

只有当两个表中删除操作都成功时，才会提交；否则，将会撤销删除操作

> 当COMMIT或ROLLBACK语句执行后，事务会自动关闭



##### 使用保留点

使用保留点的意义在于部分回退，而非全部回退，回退到指定占位符，占位符称为保留点。

```sql
SAVEPOINT delete1;
```

创建保留点，保留点的命名需要唯一

```sql
ROLLBACK TO delete1;
```

回退到delete1保留点

> 保留点越多越好
>
> 事务处理完成（ROLLBACK或COMMIT）自动释放



##### 更改默认的提交行为

```sql
SET autocommit = 0;
```

autocommit标志决定是否自动提交更改，为0则表示不自动提交更改。

> autocommit标志是针对连接而非服务器的



### 全球化和本地化

#### 字符集和校对顺序

* 字符集：字母和符号的集合
* 编码：某个字符集成员的内部表示
* 校对：规定字符如何比较的指令

> 校对可以帮助我们确认在排序、搜索时是否要区分大小写，在非拉丁文的字符集中，校对要处理的情况更加复杂

使用何种字符集和校对的决定在服务器、数据库和表级进行



#### 使用字符集和校对顺序

```sql
SHOW CHARACTER SET;
```

查看所支持的字符集完整列表

```sql
SHOW COLLATION;
```

查看所支持校对的完整列表



```sql
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';
```

确定当前数据库使用的字符集和校对

```sql
CREATE TABLE mytable
(
	columnn1 INT,
	columnn2 VARCHAR(10)
) DEFAULT CHARACTER SET hebrew
  COLLATE hebrew_general_ci;
```

指定字符集和校对顺序



* 如果指定CHARACTER SET和COLLATE两者，则使用这些值
* 如果只指定CHARACTER SET，则使用此字符集及其默认的校对
* 如果既不指定CHARACTER SET，也不指定COLLATE，则使用数据库默认



```sql
CREATE TABLE mytable
(
	columnn1 INT,
	columnn2 VARCHAR(10),
    column3  VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_general_ci
) DEFAULT CHARACTER SET hebrew
  COLLATE hebrew_general_ci;
```

对一个表和一个特定的列分别指定了字符集和校对



```sql
SELECT * FROM customers
ORDER BY lastname, firstname COLLATE latin1_general_cs;
```

按指定校对排序

> 临时区分大小写，如上述语句所示



### 安全管理

#### 访问控制

访问控制：给用户提供他们所需的访问权

不应该在日常的MySQL操作中使用root



#### 管理用户

```sql
USE mysql;
SELECT user FROM user;
```

获得所有账户列表



##### 创建用户账户

```sql
CREATE USER ben IDENTIFIED BY 'p@$$w0rd';
```



```sql
RENAME USER ben TO bforta;
```

重命名账户



##### 删除用户账号

```sql
DROP USER bforta;
```



##### 设置访问权限

```sql
SHOW GRANTS FOR bforta;
```

USAGE表示根本没有权限

> 用户定义为user@host，MySQL的权限用户名和主机名结合定义。如果不指定主机名，则使用默认的主机名%（授予用户访问权限而不管主机名）

```sql
GRANT SELECT ON crashcourse.* TO bforta;
```

允许用户在crashcourse数据库中的所有数据只读权限

```sql
SHOW GRANTS FOR bforta;
```

反映这个修改

```sql
REVOKE SELECT ON crashcourse.* FROM bforta;
```

撤销权限，被撤销的访问权限必须存在，否则出错。



GRANT和REVOKE可在几个层次上控制访问权限

* 整个服务器，GRANT ALL和REVOKE ALL
* 整个数据库，ON database.*
* 特定的表，ON database.table
* 特定的列
* 特定的存储过程



![image-20240213165625789](https://zephyr-chaser2.oss-cn-hangzhou.aliyuncs.com/img/image-20240213165625789.png)

![image-20240213165639929](https://zephyr-chaser2.oss-cn-hangzhou.aliyuncs.com/img/image-20240213165639929.png)

##### 简化多次授权

```sql
GRANT SELECT, INSERT ON crashcourse.* TO bforta;
```



##### 更改密码

```sql
SET PASSWORD FOR bforta = Password('n3w p@$$w0rd');
```

```sql
SET PASSWORD = Password('n3w p@$$w0rd');
```

在不指定用户名时，SET PASSWORD更新当前登录用户的口令



### 数据库维护

#### 备份数据

* 使用命令行实用程序mysqldump转储所有数据库内容到某个外部文件。在进行常规备份前这个实用程序应该正常运行，以便能正确地备份转储文件。
* 可用命令行实用程序mysqlhotcopy从一个数据库复制所有数据（并非所有数据库引擎都支持这个实用程序）
* 可以使用MySQL的BACKUP TABLE或SELECT INTO OUTFILE转储所有数据到某个外部文件。这两条语句都接受将要创建的系统文件名，此系统文件必须不存在，否则会出错。数据可以用RESTORE TABLE来复原。

> 为了保证所有数据被写到磁盘（包括索引数据），需要在进行备份前使用FLUSH TABLES语句



#### 进行数据库维护

```sql
ANALYZE TABLE orders;
```

检查表键是否正确。

```sql
CHECK TABLE orders, orderitems;
```

发现和修复问题。

CHANGED检查最后一次检查以来改动的表；EXTENDED执行最彻底的检查，FAST只检查未正常关闭的表，MEDIUM检查所有被删除的链接并进行键检验，QUICK只进行快速扫描。



* 如果MyISAM表访问产生不正确和不一致的结果，可能需要用REPAIR TABLE来修复响应的表，这条语句不应经常使用，可能会产生更大的问题。
* 如果从一个表中删除大量数据，应该用OPTIMIZE TABLE来收回所用的空间，从而优化表的性能。



#### 诊断启动问题

* --help显示帮助
* --safe-mode装载减去某些最佳配置的服务器
* --verbose显示全文本信息（为获得更详细的帮助信息与--help联合使用）
* --version显示版本信息然后退出



#### 查看日志文件

![image-20240213171819329](https://zephyr-chaser2.oss-cn-hangzhou.aliyuncs.com/img/image-20240213171819329.png)

![image-20240213171836729](https://zephyr-chaser2.oss-cn-hangzhou.aliyuncs.com/img/image-20240213171836729.png)

### 改善性能

