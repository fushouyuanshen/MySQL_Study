## MySQL day3 课堂笔记

### 用正则表达式进行搜索

#### 什么是正则表达式

已经学过的过滤方法：匹配、比较、通配符

正则表达式的作用：匹配文本中出现的特殊的串，而且串可以出现在文本的任何位置

用处：

1. 文本文件中提取电话号码
2. 提取文件名中有数字的文件
3. 文本块中找到重复的单词
4. 替换页面中的所有URL为实际的HTML链接



#### 使用MySQL正则表达式

##### 基本字符匹配

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '1000' ORDER BY prod_name;
```

返回prod_name中包含1000的数据

相当于

```sql
SELECT prod_name FROM products WHERE prod_name LIKE '%1000%' ORDER BY prod_name;
```



```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '.000' ORDER BY prod_name;
```

**. 是正则表达式语言中一个特殊的符号。它表示匹配任意一个字符**

也就是说，如果**一个任意字符以000结尾**，那么无论它出现在什么位置，都会被匹配到。

例如：JetPack 1000、J 1.000 etPack



##### 辨析LIKE与REGEXP

```sql
SELECT prod_name FROM products WHERE prod_name LIKE '1000' ORDER BY prod_name;
```

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '1000' ORDER BY prod_name;
```

通过执行上述语句，我们便可以知道LIKE与REGEXP之间的区别。

==在不使用通配符的前提下，LIKE匹配整个列。也就是说只有这个列名称是1000时，才会被匹配。==

==而REGEXP则不然，如果要匹配的文本出现在列中，那么相关的行就会被返回。也就是说列值为re100001，相关的行仍然会被返回==



在通配符的作用下，LIKE可以实现REGEXP的效果；同样，在定位符的帮助下，REGEXP同样可以实现LIKE的效果。



> 值得注意的是：MySQL中的正则表达式匹配不区分大小写，如果要区分大小写，则参考WHERE prod_name REGEXP BINARY 'JetPack .000'。同时，表格的排序顺序应改为UTFMP4.bin，使表格排序遵循大小写



##### 进行OR匹配

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '1000|2000' ORDER BY prod_name;
```

如果列值中出现1000或2000，则匹配并返回。

同样，也可以出现多个OR条件：

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '1000|3000|4000' ORDER BY prod_name;
```



##### 匹配几个字符之一

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[123] Ton' ORDER BY prod_name;
```

[123]可以匹配1或2或3，因此匹配文本可以是1 Ton、2 Ton、3 Ton中的一个

[123] Ton事实上是[1|2|3] Ton的缩写



**易错辨析：**

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '1|2|3 Ton' ORDER BY prod_name;
```

该语句匹配文本是'1'、'2'、'3 Ton'，也就是说如果你想要'1 Ton''2 Ton''3 Ton'，那么你必须将他们括起来[1|2|3]，否则OR会作用整个串。



```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[^123]' ORDER BY prod_name;
```

**该语句匹配除了1、2、3字符之外的任何东西，这是因为^的作用**



##### 匹配范围

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[1-5] Ton' ORDER BY prod_name;
```

这个语句则是匹配1到5的字符加上‘ Ton’，而[1-5]实际上相当于[12345]

同样也可以[1-5a-c]表示[12345abc]



##### 匹配特殊字符

```sql
SELECT vend_name FROM vendors WHERE vend_name REGEXP '.' ORDER BY vend_name;
```

由于.匹配任意字符，因此每个行都会被检索出来，而如果我们要匹配有'.'的字符串，那么应该参考如下代码

```sql
SELECT vend_name FROM vendors WHERE vend_name REGEXP '\\.' ORDER BY vend_name;
```

由此可以得到带有.的结果，如Furball Inc.

**因此，如果想要匹配的文本带有特殊符号，应该前导\\\\+特殊符号**

\\\\也用来引用元字符（具有特殊含义的字符）

| 元字符 |   说明   |
| :----: | :------: |
| \\\\f  |   换页   |
|  \\\n  |   换行   |
| \\\\r  |   回车   |
| \\\\t  |   制表   |
| \\\\v  | 纵向制表 |

> 多数正则表达式实现使用单个反斜杠转义特殊字符，MySQL要求两个反斜杠（MySQL自己解释一个，正则表达式解释另一个）



##### 匹配字符类

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[:alnum:]' ORDER BY prod_name;
```

只要文本中有任意字母或数字就可以被匹配返回。

上述代码中采用了字符类[:alnum:]：预定义的字符集

|     类     |                           说明                            |
| :--------: | :-------------------------------------------------------: |
| [:alnum:]  |               任意字母和数字(同[a-zA-Z0-9])               |
| [:alpha:]  |                   任意字符(同[a-zA-Z])                    |
| [:blank:]  |                   空格和制表(同[\\\\t])                   |
| [:cntrl:]  |             ASCII控制字符(ASCII值0-31以及127)             |
| [:digit:]  |                    任意数字(同[0 - 9])                    |
| [:graph:]  |               与[:print:]相同，但不包括空格               |
| [:lower:]  |                   任意小写字符(同[a-z])                   |
| [:print:]  |                      任意可打印字符                       |
| [:punct:]  |        既不在[:alnum:]又不在[:cntrl:]中的任意字符         |
| [:space:]  | 包括空格在内的任意空白字符(同[\\\\f\\\\n\\\\r\\\\t\\\\v]) |
| [:upper:]  |                   任意大写字母(同[A-Z])                   |
| [:xdigit:] |              任意十六进制数字(同[a-fA-F0-9])              |



##### 匹配多个实例

目前讲过的正则表达式都试图匹配单次出现，即只是检测文本是否出现。

但如果我们对文本出现的次数有要求，那么我么们需要**重复元字符**来完成

| 元字符 |             说明             |
| :----: | :--------------------------: |
|   *    |        0个或多个匹配         |
|   +    |  1个或多个匹配（等于{1,}）   |
|   ?    |  0个或1个匹配（等于{0, 1}）  |
|  {n}   |        指定数目的匹配        |
| {n, }  |     不少于指定数目的匹配     |
| {n, m} | 匹配数目的范围（m不超过255） |

例子：

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '\\([0-9] sticks?\\)' ORDER BY prod_name;
```

正则表达式\\\\([0-9] sticks?\\\\)

\\\\(匹配(    [0-9]匹配0到9中的任意数字     sticks?匹配stick或sticks       \\\\)匹配)

输出示例：TNT (1 stick)、TNT (5 sticks)



```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[[:digit:]]{4}' ORDER BY prod_name;
```

[:digit:]匹配任何数字，而{4}要求它前面的字符出现4次，因此[[:digit:]]{4}匹配连在一起的任意4位数字



但需要注意的是：

正则表达式中某个特殊的表达式一般都有多种写法，比如上述代码等价于

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '[0-9][0-9][0-9][0-9]' ORDER BY prod_name;
```



##### 定位符

目前为止所有的例子都是匹配一个串中任意位置的文本，而定位符可以帮助匹配特定位置的文本。

| 元字符  |    说明    |
| :-----: | :--------: |
|    ^    | 文本的开始 |
|    $    | 文本的结尾 |
| [[:<:]] |  词的开始  |
| [[:>:]] |  词的结尾  |

```sql
SELECT prod_name FROM products WHERE prod_name REGEXP '^[0-9\\.]' ORDER BY prod_name;
```

匹配以数字或小数点开头的文本

> ^的双重用途：^有两种用法，用它来否定该集合或指串的开始处

[^123]即表示匹配不包含1、2、3的文本

^[123]表示匹配以1、2、3开头的文本

^123表示匹配以123开头的文本



> 简单的正则表达式测试
>
> 在不使用数据库的情况下，用以下语句
>
> SELECT 'hello' REGEXP '[0-9]';
>
> 这个例子显然将返回0，因为文本"hello"中没有数字



### 创建计算字段

#### 计算字段

不存在于数据库表中，而是运行时SELECT语句创建的

作用：格式化数据，包括拼接字段、计算数据等

字段(field)基本与列(column)意思相同，数据库列一般称列，字段一般用于计算字段的连接上。

> 格式化工作一般都在DBMS内通过SQL语句完成，因为DBMS是专门设计来快速有效完成这些操作的



#### 拼接字段

```sql
SELECT Concat(vend_name, ' (', vend_country, ')') FROM vendors ORDER BY vend_name;
```

将四个数据拼接起来，最后的效果会类似ACME (USA)

> MySQL的不同之处
>
> 多数DBMS使用+或||来实现拼接，MySQL则使用Concat()函数来实现。



#### 删除空格

删除右边的空格RTrim()	删除左边的空格LTrim()	删除两边的空格Trim()

```sql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') FROM vendors ORDER BY vend_name;
```



#### 使用别名

通过拼接后得到的新计算列是没有名字的，为了赋给它们一个名字方便客户机引用，采用**别名(alias)**

```sql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS vend_title FROM vendors ORDER BY vend_name;
```

> 别名的其他用途：原有表列名包含不符合规定的字符、原有表列名易弄混或误解
>
> 别名有时也称导出列（derived column）
>
> 应当注意的是，别名也好，计算字段也好，都是SELECT语句后创建的，并非实际存在的表列名



#### 执行算术计算

```sql
SELECT prod_id, quantity, item_price, quantity*item_price AS expanded_price FROM orderitems WHERE order_num = 20005;
```



#### MySQL算术操作符

| 操作符 | 说明 |
| :----: | :--: |
|   +    |  加  |
|   -    |  减  |
|   *    |  乘  |
|   /    |  除  |



> 测试运算：
>
> SELECT省略FROM子句就可以测试运算
>
> SELECT 3*2；
>
> SELECT Trim('abc');
>
> SELECT Now();返回当前日期和时间



### 使用数据处理函数

#### 函数

函数的可移植性不强，因为SQL之间实现功能的函数名可能不一致，如果要采用函数，应该保证做好代码注释



#### 使用函数

* 处理文本串的函数
* 数值数据的算术操作
* 日期和时间函数
* 特殊信息（用户登录信息，检查版本细节）的系统函数



#### 文本处理函数

```sql
SELECT vend_name, Upper(vend_name) AS vend_name_upcase FROM vendors ORDER BY vend_name;
```

将所有结果转换为大写



常用的文本处理函数

|    函数     |        说明         |
| :---------: | :-----------------: |
|   Left()    |  返回串左边的字符   |
|  Length()   |    返回串的长度     |
|  Locate()   |  找出串的一个子串   |
|   Lower()   |   将串转换为小写    |
|   LTrim()   |  去掉串左边的空格   |
|   Right()   |  返回串右边的字符   |
|   RTrim()   |  去掉串右边的空格   |
|  Soundex()  | 返回串的SOUNDEX的值 |
| SubString() |   返回子串的字符    |
|   Upper()   |   将串转换为大写    |

在实际情况中，我们根据顾客姓名输入时，可能会出现输错的现象；如果采用Soundex()函数，就可以根据姓名发音进行比较，通过发音匹配数据

```sql
SELECT cust_name, cust_contact FROM customers WHERE Soundex(cust_contact) = Soundex('Y Lie');
```

输出结果可能会是：Coyote Inc.	Y Lee

由此可见，通过发音正确过滤出了想要的数据



#### 日期和时间处理函数

常用日期和时间处理函数

|     函数      |             说明             |
| :-----------: | :--------------------------: |
|   AddDate()   |   增加一个日期（天、周等）   |
|   AddTime()   |   增加一个时间（时、分等）   |
|   CurDate()   |         返回当前日期         |
|   CurTime()   |         返回当前时间         |
|    Date()     |    返回日期时间的日期部分    |
|  DateDiff()   |       计算两个日期之差       |
|  Date_Add()   |    高度灵活的日期运算函数    |
| Date_Format() | 返回一个格式化的日期或时间串 |

基本的日期比较

```sql
SELECT cust_id, order_num FROM orders WHERE order_date = '2005-09-01';
```

该语句可以检索出一个order_date为2005-09-01的订单记录

但如果order_date包括了时间，那么下列语句更可靠

```sql
SELECT cust_id, order_num FROM orders WHERE Date(order_date) = '2005-09-01';
```

> 使用Date()和Time()函数去匹配日期和时间是好习惯



```sql
SELECT cust_id, order_num FROM orders WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';
```

检索2005年9月下的所有订单



```sql
SELECT cust_id, order_num FROM orders WHERE Year(order_date) = 2005 AND Month(order_date) = 9;
```

同样可以检索2005年9月的所有订单，但相较于上面的代码，则可以不用记住每月有多少天或闰年2月的问题



#### 数值处理函数

常见的数值处理函数

|  函数  |        说明        |
| :----: | :----------------: |
| Abs()  | 返回一个数的绝对值 |
| Cos()  | 返回一个角度的余弦 |
| Exp()  | 返回一个数的指数值 |
| Mod()  |  返回除操作的余数  |
|  Pi()  |     返回圆周率     |
| Rand() |   返回一个随机数   |
| Sin()  | 返回一个角度的正弦 |
| Sqrt() | 返回一个数的平方根 |
| Tan()  | 返回一个角度的正切 |



### 汇总数据

#### 聚集函数

有时候检索数据是为了它们的汇总信息，而不需要返回实际数据，因此需要聚焦函数。

**聚焦函数（aggregate function）：**运行在行组上，运算和返回单个值的函数。

|  函数   |       说明       |
| :-----: | :--------------: |
|  AVG()  | 返回某列的平均值 |
| COUNT() |  返回某列的行数  |
|  MAX()  | 返回某列的最大值 |
|  MIN()  | 返回某列的最小值 |
|  SUM()  |  返回某列值的和  |



#### AVG()函数

```sql
SELECT AVG(prod_price) AS avg_price FROM products;
```

返回商品的平均价格

```sql
SELECT AVG(prod_price) AS avg_price FROM products WHERE vend_id = 1003;
```

返回特定商品的平均价格

> AVG必须有且只有一个参数表示列
>
> 如果要获得多列平均值，则使用多个AVG函数
>
> AVG()函数忽略列值为NULL的行



#### COUNT()函数

```sql
SELECT COUNT(*) AS num_cust FROM customers;
```

对所有行计数，即使列值是NULL也会计数

```sql
SELECT COUNT(cust_email) AS num_cust FROM customers;
```

只对电子邮件不为NULL的客户计数



#### MAX()函数

```sql
SELECT MAX(prod_price) AS max_price FROM products;
```

返回最贵商品的价格

> MAX函数也可以用于文本数据，会按照字典顺序，找出ASCII值或UNICODE值最大的文本。
>
> MAX()函数忽略列值为NULL的行



#### MIN()函数

```sql
SELECT MIN(prod_price) AS max_price FROM products;
```

>MIN函数也可以用于文本数据，会按照字典顺序，找出ASCII值或UNICODE值最小的文本。
>
>MIN()函数忽略列值为NULL的行



#### SUM()函数

```sql
SELECT SUM(quantity) AS items_ordered FROM orderitems WHERE order_num = 20005;
```

返回特定商品的订购总数

```sql
SELECT SUM(item_price*quantity) AS total_price FROM orderitems WHERE order_num = 20005;
```

合计计算值

> SUM()函数忽略列值为NULL的行



#### 聚集不同值

DISTINCT + 列表示只包含不同的值

```sql
SELECT AVG(DISTINCT prod_price) AS avg_price FROM products WHERE vend_id = 1003;
```

把所有商品的价格列出，每种价格只加一次，然后平均

比如商品价格分别为45、45、34、23、51，则平均值的公式是(45 + 34 + 23 + 51) / 4

> 将DISTINCT用于MIN()和MAX()是没有意义的，因为最值本就唯一



#### 组合聚集函数

```sql
SELECT COUNT(*) AS num_items, MIN(prod_price) AS price_min, MAX(prod_price) AS price_max, AVG(prod_price) AS price_avg FROM products;
```



> 建议取别名时与表列名不一致，唯一的别名会使SQL易于理解使用
>
> 本章函数很高效，建议使用

