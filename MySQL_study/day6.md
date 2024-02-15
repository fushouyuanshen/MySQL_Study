## MySQL day6 课堂笔记

### 使用视图

#### 视图

视图是虚拟的表，与包含数据的表不一样，视图只包含使用时动态检索数据的查询。



```sql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = orders.order_num
	AND prod_id = 'TNT2';
```

该语句检索购买了TNT2的客户姓名及联系方式，而我们可以把这个查询语句包装成名为productcustomers的视图，并通过下述语句返回同样结果。

```sql
SELECT cust_name, cust_contact
FROM productcustomers
WHERE prod_id = 'TNT2';
```



#### 为什么使用视图

* 重用SQL语句
* 简化复杂SQL操作
* 使用表的组成部分而不是整个表
* 保护数据。可以给用户授予表特定部分的访问权限而不是整个表的访问权限
* 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。



> 因为视图不包含数据，所以每次使用视图时，都必须处理查询执行时所需的任一个检索。创建复杂的视图或嵌套视图会导致性能严重下降



#### 视图的规则和限制

* 与表一样，视图必须唯一命名
* 对于可以创建的视图数目没有限制
* 为了创建视图，必须有足够的访问权限。这些限制通常由数据库管理人员授予
* 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图
* ORDER BY可以用在视图中，但如果视图对应的SELECT语句含有ORDER BY，那么该视图中的ORDER BY将被覆盖
* 视图不能索引，也不能有关联的触发器或默认值
* 视图可以和表一起使用。例如，编写一条联结表和视图的SELECT语句。



#### 使用视图

* 创建视图CREATE VIEW
* 查看视图SHOW CREATE VIEW viewname;
* 删除视图DROP VIEW viewname;
* 更新视图，可以先DROP后CREATE，也可以CREATE OR REPLACE VIEW



##### 利用视图简化复杂的联结

```sql
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = orders.order_num;
```

创建一个检索客户姓名、客户联系方式、产品型号的视图



```sql
SELECT cust_name, cust_contact
FROM productcustomers
WHERE prod_id = 'TNT2';
```

检索订购了产品TNT2的客户



##### 用视图重新格式化检索出的数据

```sql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')')
		AS vend_title
FROM vendors
ORDER BY vend_name;
```

格式化数据，但可以把此语句转换为视图



```sql
CREATE VIEW vendorlocations AS
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')')
		AS vend_title
FROM vendors
ORDER BY vend_name;
```

```sql
SELECT *
FROM vendorlocations;
```



##### 用视图过滤不想要的数据

```sql
CREATE VIEW customeremaillist AS
SELECT cust_id, cust_name, cust_email
FROM customers
WHERE cust_email IS NOT NULL;
```

排除没有邮箱地址的用户

```sql
SELECT *
FORM customeremaillist;
```



> 如果视图检索数据时使用了WHERE子句，则两组子句（一组在视图中，另一组是传递给视图的）自动结合



##### 使用视图与计算字段

```sql
SELECT prod_id,
	   quantity,
	   item_price,
	   quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```

等价于以下语句

```sql
CREATE VIEW orderitemsexpanded AS 
SELECT order_num,
	   prod_id,
	   quantity,
	   item_price,
	   quantity*item_price AS expanded_price
FROM orderitems;
SELECT *
FROM orderitemsexpanded
WHERE order_num = 20005;
```



##### 更新视图

视图定义中有以下操作不能更新：

* 分组（使用GROUP BY和HAVING）
* 联结
* 子查询
* 并
* 聚集函数（Min()、Count()、Sum()等）
* DISTINCT
* 导出（计算）列



> 视图应当用于检索而非更新



### 使用存储过程

#### 存储过程

情景：

* 为了处理订单，需要核对以保证库存中有相应的物品
* 如果库存有物品，这些物品需要预定以便不将它们再卖给别的人，并且要减少可用的物品数量以反映正确的库存量
* 库存中没有的物品需要预购，这需要与供应商进行某种交互。
* 关于哪些物品入库（并且可以立即发货）和哪些物品退订，需要通知相应的客户

以上情景可以用存储过程解决。

**存储过程：为以后使用而保存的一条或多条MySQL语句的集合。**



#### 为什么要使用存储过程

* 封装处理到容易使用的单元，简化复杂操作。
* 由于不要求反复建立一系列处理步骤，保证了数据的完整性。通过使执行某一项操作的代码相同，从而能防止错误，保证数据一致性。
* 简化对变动的管理



#### 使用存储过程

##### 执行存储过程

```sql
CALL productpricing(@pricelow,
                   @pricehigh,
                   @priceaverage);
```

执行名为productpricing的存储过程，计算并返回产品的最低、最高和平均价格



##### 创建存储过程

```sql
CREATE PROCEDURE productpricing()
BEGIN
	SELECT Avg(prod_price) AS priceaverage
	FROM products;
END;
```

返回平均商品价格



**注意：**

如果是在MySQL的命令行程序中，FROM products;会导致程序结束输入，从而导致报错。因此我们会采用以下方式输入

```sql
DELIMITER //

CREATE PROCEDURE productpricing()
BEGIN
	SELECT Avg(prod_price) AS priceaverage
	FROM products;
END //

DELIMITER ;
```



DELIMITER //告诉命令程序行以//作为语句分隔符，因此END //语句输入并回车后，语句就结束了。

DELIMITER ;告诉命令程序以;作为语句分隔符。

因此，我们可以明白DELIMITER后跟随的符号都可以作为语句分隔符，任何字符均可。

> 应当仔细观察示例程序，DELIMITER后是一个空格加字符，空格是不可以省略的！



```sql
CALL productpricing();
```

执行productpricing存储过程



##### 删除存储过程

```sql
DROP PROCEDURE productpricing;
```

删除productpricing存储过程，注意语句中不带()

> 如果不存在该存储过程则会报错，如果想消去该错误可使用DROP PROCEDURE IF EXISTS



##### 使用参数

```sql
CREATE PROCEDURE productpricing(
	OUT pl DECIMAL(8,2),
    OUT ph DECIMAL(8,2),
    OUT pa DECIMAL(8,2)
)
BEGIN
	SELECT Min(prod_price)
	INTO pl
	FROM products;
	INTO ph
	FROM products;
	SELECT Avg(prod_price)
	INTO pa
	FROM products;
END;
```

该存储过程接受3个参数：pl存储产品最低价格，ph存储产品最高价格，pa存储产品平均价格，关键字OUT指出相应参数从存储过程返回给调用者，而与之对应的IN（传递给存储过程），INOUT（对存储过程传入和传出）



当调用此时的productpricing存储过程时，语句如下

```sql
CALL productpricing(@pricelow,
                   @pricehigh,
                   @priceavereage);
```

> 所有MySQL变量必须以@开始



而执行该存储过程后，我们可以获得三个参数并检索其值

```sql
SELECT @priceaverage;
```

```sql
SELECT @pricehigh, @pricelow, @priceaverage;
```



```sql
CREATE PROCEDURE ordertotal(
	IN onnumber INT,
	OUT ototal DECIMAL(8,2)
)
BEGIN
	SELECT Sum(item_price*quantity)
	FROM orderitems
	WHERE order_num = onumber
	INTO ototal;
END;
```

输入订单编号，返回订单商品总价



调用ordertotal存储过程

```sql
CALL ordertotal(20005, @total);
```



显示该合计

```sql
SELECT @total;
```



##### 建立智能存储过程

通过IF语句添加判断条件 、-- 后加语句表注释，DECLARE要求指定变量名和数据类型作临时变量

```sql
-- Name: ordertotal
-- Parameters: onumber = order number
-- 			   taxable = 0 if not taxable, 1 if taxable
-- 		   	   ototal = order total variable

CREATE PROCEDURE ordertotal(
	IN onumber INT,
	IN taxable BOOLEAN,
	OUT ototal DECIMAL(8,2)
) COMMENT 'Obtain order total, optionally adding tax'
BEGIN
	-- Declare variable for total
	DECLARE total DECIMAL(8,2);
	-- Declare tax percentage
    DECLARE taxrate INT DEFAULT 6;
    
    -- Get the order total
    SELECT Sum(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO total;
    -- Is this taxabel?
    IF taxable THEN
    	-- Yes, so add taxrate to the total
    	SELECT total+(total/100*taxrate) INTO total;
    END IF;
    -- And finally, save to out variable
    SELECT total INTO ototal;
    
    END;
```

创建ordertotal存储过程，输入订单编号和是否收营业税从而得出订单总值



> COMMENT关键字是不必须的，但若给出，可在SHOW PROCEDURE STATUS的结果中显示



执行该存储过程

```sql
CALL ordertotal(20005, 0, @total);
SELECT @total;
```

```sql
CALL ordertotal(20005, 1, @total);
SELECT @total;
```



##### 检查存储过程

```sql
SHOW CREATE PROCEDURE ordertotal;
```

获得包括何时、由谁创建等详细信息的存储过程列表。



```sql
SHOW PROCEDURE STATUS LIKE 'ordertotal';
```

限制过程状态结果