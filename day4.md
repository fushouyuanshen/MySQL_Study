## MySQL day4 课堂笔记

### 分组数据

#### 数据分组

当我们试图计算一个供应商提供了多少产品时，可以采用下面的语句

```sql
SELECT COUNT(*) AS num_prods FROM products WHERE vend_id = 1003;
```

但如果要返回每个供应商分别提供了多少产品，我们就应对数据按供应商进行分组，之后聚焦函数计算。



#### 创建分组

```sql
SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id;
```

根据供应商分组并统计每个供应商提供了多少商品



**GROUP BY子句的重要规定：**

*  GROUP BY子句可以包含任意数目的列，这使得能对分组进行嵌套，即按照列进行组合分组

```sql
SELECT vend_id, prod_price, COUNT(*) AS num_prods FROM products GROUP BY vend_id, prod_price;
```

* 如果GTOUP BY子句嵌套了分组，数据将在最后规定的分组上进行汇总
* GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式，不能使用别名
* 除聚集计算语句外，SELECT语句中的每个列都必须在GTOUP BY子句中给出
* 如果分组列中具有NULL值，则将NULL作为一个分组返回，如果列中有多行NULL值，他们将为一组
* GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前



##### ROLLUP

```sql
SELECT region, category, SUM(sales_amount) AS total_sales
FROM orders
GROUP BY region, category WITH ROLLUP;
```

根据地区、产品类别分组并计算销售额，查询结果会是如下：

| Region    | Category    | Sales_Amount |
| --------- | ----------- | ------------ |
| East      | Electronics | 5000         |
| East      | Clothing    | 3000         |
| ==East==  | ==NULL==    | ==8000==     |
| West      | Electronics | 4000         |
| West      | Clothing    | 2000         |
| ==West==  | ==NULL==    | ==6000==     |
| North     | Electronics | 6000         |
| North     | Clothing    | 1000         |
| ==North== | ==NULL==    | ==7000==     |
| ==NULL==  | ==NULL==    | ==21000==    |

可以看到相较于原有的查询结果，WITH ROLLUP添加了如下查询结果，首先它汇总了每个地区的销售额，之后又汇总了所有地区的销售额

**因此：ROLLUP关键字的作用就是为每个分组提供汇总级别的值**



#### 过滤分组

> WHER和HAVING之间的区别：
>
> WHERE过滤的是行
>
> HAVING过滤的是组
>
> HAVING支持所有的WHERE操作符，同时可以取代之前的WHERE子句
>
> WHERE子句在GROUP BY之前
>
> HAVING子句在GROUP BY之后

```sql
SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >= 2;
```

过滤两个订单以下（不包括两个）的分组



同时运用WHERE子句和HAVING子句

```sql
SELECT vend_id, COUNT(*) AS num_prods FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >= 2;
```

先将产品价格大于等于10美元的过滤出来，之后再按供应商分组，返回有两个及以上产品的供应商



#### 分组和排序

分组后得出的数据并不一定是排序后的，所以如果想得到排序后的数据，就要加ORDER BY子句

```sql
SELECT order_num, SUM(quantity*item_price) AS ordertotal FROM orderitems GROUP BY order_num HAVING SUM(quantity*item_price) >= 50 ORDER BY ordertotal;
```

统计订单号与订单价格，返回排序后的大于50等于美元的订单



#### SELECT子句顺序

| 子句     | 说明               | 是否必须使用           |
| -------- | ------------------ | ---------------------- |
| SELECT   | 要返回的列或表达式 | 是                     |
| FROM     | 从中检索数据的表   | 仅在从表选择数据时使用 |
| WHERE    | 行级过滤           | 否                     |
| GROUP BY | 分组使用           | 仅在按组计算聚集时使用 |
| HAVING   | 组级过滤           | 否                     |
| ORDER BY | 输出排序顺序       | 否                     |
| LIMIT    | 要检索的行数       | 否                     |



### 使用子查询

#### 子查询（subquery）

**查询（query）：**任何SQL语句都是查询，但此术语一般指SELECT语句

>customers存储客户信息
>
>orders存储订单号、客户ID、订单日期
>
>orderitems存储各订单的物品
>
>假设要列出订购物品TNT2的所有客户，要怎么做？

* 不用子查询：

```sql
SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';
```

若输出结果为20005、20007，则下一条语句为

```sql
SELECT cust_id FROM orders WHERE order_num IN (20005, 20007);
```

若输出结果为10001、10004，则下一条语句为

```sql
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN (10001,10004);
```



* 子查询

```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                 FROM orders
                 WHERE order_num IN (SELECT order_num
                                    FROM orderitems
                                    WHERE prod_id = 'TNT2'));
```

#### 作为计算字段使用子查询

> customers存储客户信息
>
> orders存储订单和对应的客户ID
>
> 要求显示每个客户的订单总数

```sql
SELECT cust_name,
	   cust_state,
	   (SELECT COUNT(*)
       FROM orders
       WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```

该语句使用了完全限定列名

```sql
WHERE orders.cust_id = customers.cust_id) AS orders
```

这种类型的查询被称为**相关子查询（correlated subquery）**，涉及外部查询的子查询



> **逐渐增加子查询来建立查询** 先建立内层查询，然后用硬编码数据建立和测试外层查询，并确认它正常后再嵌入子查询，再测试它。重复这些步骤，以保证正常工作。



### 联结表

#### 联结（join）

> 假定有供应商信息和产品信息，我们一般会将产品信息放在一个表里，供应商信息一个表。以避免表格内大量重复数据（相同的供应商信息），既浪费空间也很难改动

因此我们将两个表中共有的常用的信息互相关联，例如供应商表和产品表中都存放唯一标识供应商ID。在供应商表中将供应商ID称为主键，而产品表中称供应商ID为外键。

> 外键（foreign key） 外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系

关系数据库可伸缩性比非关系型数据要好

> 可伸缩性（scale） 能够适应不断增加的工作量而不失败。设计良好的数据库或应用程序称之为可伸缩性好（scale well）。



#### 为什么要使用联结

当数据存储在多个表中，用单条SELECT语句检索出数据的方法便是联结。

> 联结不是物理实体，它在实际的数据库中不存在，只有在查询的执行当中存在



#### 创建联结

```sql
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name;
```

根据供应商id返回供应商姓名、产品名称、产品价格



**注意：**WHERE子句很重要，它是联结的条件

如果没有WHERE子句这样的联结条件，就会返回笛卡尔积。即vendors表的每一行与products表的每一行进行组合，而不管这两行之间是否有关系。检索出来的行的数目也就成了**vendors行数 * products行数**

例如：

```sql
SELECT vend_name, prod_name, prod_price
FROM vendors, products
ORDER BY vend_name, prod_name;
```

假设vendors表为

| vend_id | vend_name |
| ------- | --------- |
| 1       | alice     |
| 2       | ex        |
| 3       | zom       |

products表为

| vend_id | prod_name | prod_price |
| ------- | --------- | ---------- |
| 1       | elec      | 10.00      |
| 4       | esc       | 4.50       |
| 5       | paper     | 3.00       |

返回的结果为

| vend_name | prod_name | prod_price |
| --------- | --------- | ---------- |
| alice     | elec      | 10.00      |
| alice     | esc       | 4.50       |
| alice     | paper     | 3.00       |
| ex        | elec      | 10.00      |
| ex        | esc       | 4.50       |
| ex        | paper     | 3.00       |
| zom       | elec      | 10.00      |
| zom       | esc       | 4.50       |
| zom       | paper     | 3.00       |

上表就是笛卡尔积，有些供应商没有产品，却被匹配了产品。为避免笛卡尔积的出现，WHERE子句应当存在，

> 笛卡尔积有时也被称为叉联结（cross join）



#### 内部联结

之前的联结被称为**等值联结（equijoin）**，它基于两个表之间的相等测试，也被称为内部联结，可以通过稍微不同的语法明确指定联结类型并得到相同的结果。

```sql
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id;
```

> ANSI SQL规范首选INNER JOIN语法
>
> 此外明确的连接语法能够确保不会忘记联结条件，有时也可能影响性能



#### 联结多个表

> orderitems表存放产品ID，订单信息
>
> products表存放产品信息
>
> vendors表存放供应商信息
>
> 找到订单编号为20005的产品

```sql
SELECT prod_name, vend_name, prod_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id
  AND orderitems.prod_id = products.prod_id
  AND order_num = 20005;
```



在子查询时的示例中，有以下语句

```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                 FROM orders
                 WHERE order_num IN (SELECT order_num
                                    FROM orderitems
                                    WHERE prod_id = 'TNT2'));
```

而联结的解法如下

```sql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE customer.cust_id = orders.cust_id
  AND orders.order_num = orderitems.order_num
  AND prod_id = 'TNT2';
```

**因此，子查询并不总是执行复杂SELECT操作的最有效的方法**

> 处理联结有可能很耗费资源，因此应注意不要关联不必要的表。
>
> 联结的表越多，性能下降越厉害



### 创建高级联结

#### 使用表别名

```sql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS vend_title
FROM vendors
ORDER BY vend_name;
```

这是之前为计算字段取别名的语句，同样的，也可以为表取别名。

```sql
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
  AND oi.order_num = o.order_num
  AND prod_id = 'TNT2';
```

为表取别名可以缩短SQL语句、允许在单条SQL语句中多次使用相同的表。

**注意：表别名可以在查询执行中重复使用，而列别名则可以返回到客户机上**



#### 使用不同类型的联结

##### 自联结

> products表中存储了供应商名、产品名、产品ID
>
> 假设一个产品出现问题，现要求返回该供应商的所有产品

```sql
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id
                FROM products
                WHERE prod_id = 'DTNTR');
```

上述解决方案利用子查询的方法

下面给出使用联结的相同查询

```sql
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
  AND p2.prod_id = 'DTNTR';
```

尽管p1、p2是同一个表，但这样是合法的，这就是**自联结**

自联结和子查询得到的结果相同，但有时处理速度快于子查询



##### 自然联结

如果不同的表中除了主键之外，仍然有相同的列出现，比如两个表中都有供应商id和产品名称

自然联结就是用来避免重复的列出现的，因此在SELECT后跟的列名一定要小心



##### 外部联结

* 对每个客户订单数进行统计，包括未下单客户
* 列出产品订购数量包括未被订购的产品

以上场景均可用到外部联结，简而言之就是要返回没有关联的行

> 检索客户及订单规模

```sql
SELECT customers.cust_id, orders.order_num
FROM customers INNER JOIN orders
ON customers.cust_id = orders.cust_id;
```

以上是内部联结的结果，而为了包含没有订购的客户，就要用到如下的外部联结

```sql
SELECT customers.cust_is, orders.order_num
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id;
```

```sql
LEFT OUTER JOIN
```

LEFT表示左边的表选择所有行

```sql
RIGHT OUTER JOIN
```

RIGHT表示右边的表选择所有行



#### 使用带聚集函数的联结

```sql
SELECT customers.cust_name,
	   customers.cust_id,
	   COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```

先根据顾客ID内部联结，之后按照顾客ID分组返回顾客姓名、顾客ID以及订单数量



```sql
SELECT customers.cust_name,
	   customers.cust_id,
	   COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = ordres.cust_id
GROUP BY customers.cust_id;
```

这次包含了没有订购的顾客



#### 使用联结和联结条件

* 注意所使用的联结类型，比如内部联结和外部联结
* 保证使用正确的联结条件
* 提供联结条件以避免笛卡尔积
* 可以联结多个表，也可以对每个联结采取不同的联结类型，但要分别测试每个联结

```sql
SELECT *
FROM table1, table2, table3, table4, table5
LEFT JOIN table2 ON table1.id = table2.table1_id
RIGHT JOIN table3 ON table2.id = table3.table2_id
INNER JOIN table4 ON table3.id = table4.table3_id
CROSS JOIN table5
WHERE some_conditions;
```



### 组合查询

#### 组合查询

组合查询：执行多个查询（多条SELECT）语句，并将结果作为单个查询结果集返回。这些组合查询通常称为并（union）或复合查询（compound query)

使用情况：

* 在单个查询中从不同表返回类似结构的数据
* 对单个表执行多个查询，按单个查询返回数据



> 多数情况下，组合相同表的两个查询完成的工作与具有多个WHERE子句条件的单条查询完成的工作相同。



#### 使用UNION

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002);
```

返回价格小于等于5美元或制造商编号为1001、1002的产品

等价于

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
  OR vend_id IN (1001, 1002);
```



#### UNION规则

* UNION必须由两条或两条以上的SELECT语句组合，语句之间用UNION分隔
* UNION中的每个查询必须包含相同的列、表达式或集聚函数
* 列数据类型必须兼容



#### 包含或取消重复的行

UNION关键字返回的是不重复的行，重复的行会被删掉

如果需要重复的行，则UNION关键字改为UNION ALL

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION ALL
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002);
```



#### 对组合查询结果排序

UNION组合查询时，只能用一条ORDER BY子句且必须出现在最后一条SELECT语句之后。不允许使用多条ORDER BY子句

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION 
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002)
ORDER BY vend_id, prod_price;
```



