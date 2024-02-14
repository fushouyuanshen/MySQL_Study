## MySQL day2 课堂笔记

### 排序检索数据

#### 排序数据

```sql
SELECT prod_name FROM products ORDER BY prod_name;
```

按照prod_name列字母排序并输出prod_name列的值



也可以通过**非选择列**

```sql
SELECT prod_name FROM products ORDER BY prod_id;
```



#### 按多个列排序

```sql
SELECT prod_id, prod_price, prod_name FROM products ORDER BY prod_price, prod_name;
```

先按照prod_price排序，在prod_price相同时按照prod_name排序



#### 指定排序方向

默认排序方向为升序，如果我们需要使用降序时，则在要降序的列后加上DESC

```sql
SELECT prod_id, prod_price, prod_name FROM products ORDER BY prod_price DESC, prod_name;
```

DESC关键字只作用其前面的列，也就是说：这行语句先对prod_price进行降序排序，再对prod_name进行升序排序。



#### 查找最值

```sql
SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 1;
```

查找最昂贵的商品



### 过滤数据

#### 使用WHERE子句

检索所需数据需要指定搜索条件（过滤条件）

```sql
SELECT prod_name, prod_price FROM products WHERE prod_price = 2.50;
```

输出所有prod_price值为2.50的数据



#### WHERE子句操作符

| 操作符  |        说明        |
| :-----: | :----------------: |
|    =    |        等于        |
|   <>    |       不等于       |
|   !=    |       不等于       |
|    <    |        小于        |
|   <=    |      小于等于      |
|    >    |        大于        |
|   >=    |      大于等于      |
| BETWEEN | 在指定的两个值之间 |



#### 检查单个值

```sql
SELECT prod_name, prod_price FROM products WHERE prod_name = 'fuses';
```

输出prod_name值为fuses的数据



```sql
SELECT prod_name, prod_price FROM products WHERE prod_price < 10;
```

输出prod_price < 10的数据



#### 不匹配检查

```sql
SELECT vend_id, prod_name FROM products WHERE vend_id <> 1003;
```

输出vend_id不为1003的数据

等价于

```sql
SELECT vend_id, prod_name FROM products WHERE vend_id != 1003;
```



#### 范围值检查

```sql
SELECT prod_name, prod_price FROM products WHERE prod_price BETWEEN 5 AND 10;
```

检索价格在5美元到10美元之间的商品



#### 空值检查

一个列不包含值时，称为其包含空值NULL

```sql
SELECT prod_name FROM products WHERE prod_price IS NULL;
```

返回没有价格的所有商品



**在返回不等于某值的结果中，NULL值也是不会返回的。也就是说：如果要返回价格不等于5美元的商品，如果有个产品价格为空值，那么这个产品也不会被返回**



**同时需要注意的是，NULL不是0，而是没有值**



**ORDER语句应在WHERE语句的后面**



### 数据过滤

#### 组合WHERE子句

操作符：连接WHERE子句的关键字，也称逻辑操作符



#### AND操作符

```sql
SELECT prod_id, prod_price, prod_name FROM products WHERE vend_id = 1003 AND prod_price <= 10;
```

返回供应商为1003且价格小于10美元的产品



#### OR操作符

```sql
SELECT prod_name, prod_price, FROM products WHERE vend_id = 1002 OR vend_id = 1003;
```

返回供应商为1002或1003的产品



#### 计算次序

先AND后OR，可用括号括起来优先处理

```sql
SELECT prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;
```

返回供应商为1002或者价格大于等于10美元的1003供应商的产品



```sql
SELECT prod_name, prod_price FROM products WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;
```

返回价格大于等于10美元的供应商为1002或1003的产品



#### IN操作符

```sql
SELECT prod_name, prod_price FROM products WHERE vend_id IN (1002, 1004) ORDER BY prod_name;
```

返回供应商为1002或1004的按照名称排序的产品

上述语句等价于

```sql
SELECT prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1004 ORDER BY prod_name;
```



为什么要使用IN操作符？其优点如下：

* IN操作符的语法清楚直观
* 计算次序容易管理（因为操作符数量更少）
* 相较OR操作符清单执行更快
* 可以包含其他SELECT语句，从而动态建立WHERE语句

**IN在WHERE子句中与OR功能相当**



#### NOT操作符

否定它之后所跟的任何条件

```sql
SELECT prod_name, prod_price FROM products WHERE vend_id NOT IN (1002, 1003) ORDER BY prod_name;
```

返回供应商不是1002或1003的产品



### 用通配符进行过滤



#### LIKE操作符

**通配符(wildcard)：**用来匹配值的一部分的特殊字符。例如，利用通配符可以搜索产品名中包含文本anvil的产品。

**搜索模式(search pattern)：**由字面值、通配符或两者组合构成的搜索条件。



**问题：谓词和操作符的区别是什么？**



#### 百分号(%)通配符

在搜索串中，%表示任何字符出现的任意次数，0次、1次或多次。



```sql
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE 'jet%';
```

返回以jet开头的字符串



```sql
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE '%anvil%';
```

返回任何位置包含文本anvil的值，而不论它之前或之后出现什么字符



```sql
SELECT prod_name FROM products WHERE prod_name LIKE 's%e';
```

返回以s开头e结尾的字符串



> **注意尾空格** 尾空格可能干扰通配符匹配，例如子句WHERE prod_name LIKE '%anvil'时将不会匹配"anvil "，因为该字符串后面有空格，一个简单方法是把%anvil后面加一个%，更好的方法是利用函数去掉首尾空格

> **注意NULL** WHERE prod_name LIKE '%'也不能匹配值为NULL的产品

> **注意单引号** LIKE后面一定要跟单引号，不然会报错



#### 下划线(_)通配符

下划线只匹配单个字符而不是多个字符

```sql
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE '_ ton anvil';
```



#### 使用通配符的技巧

* 不要过度使用通配符，如果其他操作符可以达到相同的目的，应该使用其他操作符
* 在确实需要使用通配符时，除非绝对有必要，否则不要把它们用在搜索模式的开始处。把通配符置于搜索模式的开始处，搜索起来是最慢的
* 仔细注意通配符的位置。如果放错位置，可能不会返回想要的数据。

