## MySQL day5 课堂笔记

### 全文本搜索

#### 理解全文本搜索

为什么要使用**全文本搜索**？

* 相较于通配符和正则表达式而言，更为高效

* 可以明确控制必须包含什么或必须不包含什么

* 可以区分单个匹配和包含多个匹配的行



#### 使用全文本搜索

##### 启用全文本搜索支持

```sql
CREATE TABLE productnotes
(
  note_id	int				NOT NULL AUTO_INCREMENT,
  prod_id	char(10)		NOT NULL,
  note_date datetime		NOT NULL,
  note_text text			NULL,
  PRIMARY KEY(note_id),
  FULLTEXT(note_text)
) ENGINE=MyISAM;
```

为了进行全文本搜索，MySQL根据子句FULLTEXT(note_text)的指示对它进行索引。这里的FULLTEXT可以索引单个列，也可以指定多个列。

在定义后，MySQL自动维护该索引。再增加、更新或删除行时，索引随之自动更新。

> 不要在导入数据时使用FULLTEXT，在导入全部数据后再使用FULLTEXT



##### 进行全文本搜索

```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');
```

此SELECT语句检索指定列note_text，并指定rabbit为搜索文本

> 传递给MATCH()的值必须与FULLTEXT()定义中的相同

> 除非使用BINARY方式，否则全文本搜索不区分大小写



上述语句等价于

```sql
SELECT note_text
FROM productnotes
WHERE note_text LIKE '%rabbit%';
```



但这两个语句返回的结果却不一定相同，原因是全文本搜索会对结果进行排序，具有较高等级的行会优先返回

> 包含词汇数量的多少，词汇在文本中处于较前或较后的位置都会影响等级



##### 使用查询扩展

查询扩展用来设法扩宽所返回的全文本搜索结果的范围

MySQL在进行查询扩展时步骤如下：

* 首先，进行一个基本的全文本搜索，找出与搜索条件匹配的行
* 其次，在已经匹配的行中找到有用的词
* 最后，根据有用的词再次进行全文本搜索



没有查询扩展的语句如下，结果只能返回匹配anvils的行

```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils');
```



有查询扩展的语句如下，除了返回匹配anvils的行，还会返回和匹配anvils的行相近的语句，根据语句等级进行排序

```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils' WITH QUERY EXPANSION)
```



##### 布尔文本搜索

可以提供以下内容的细节：

* 要匹配的词
* 要排斥的词
* 排列提示
* 表达式分组
* 另外一些内容

即使没有FULLTEXT索引也可以使用布尔表达式，但这是一种非常缓慢的操作



```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy' IN BOOLEAN MODE);
```

该语句返回包含heavy的语句，与没有使用布尔表达式的结果相同



```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE);
```

匹配包含heavy同时不包含任何以rope开始的词



布尔操作符：

| 布尔操作符 |                             说明                             |
| :--------: | :----------------------------------------------------------: |
|     +      |                       包含，词必须存在                       |
|     -      |                      排除，词必须不存在                      |
|     >      |                     包含，而且增加等级值                     |
|     <      |                     包含，而且减少等级值                     |
|     ()     | 把词组成子表达式（允许这些子表达式作为一个组被包含、排除、排列等） |
|     ~      |                      取消一个词的排序值                      |
|     *      |                         词尾的通配符                         |
|     ""     | 定义一个短语（与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语） |



```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+rabbit +bait' IN BOOLEAN MODE);
```

匹配包含词rabbit和bait的行



```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);
```

匹配包含rabbit和bait中的至少一个词的行



```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('"rabbit bait"' IN BOOLEAN MODE);
```

匹配短语rabbit bait而不是匹配两个词rabbit和bait



```sql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('>rabbit <carrot' IN BOOLEAN MODE);
```

匹配rabbit和carrot，增加前者的等级，降低后者的等级



```sql
SELECT note_text
FORM productnotes
WHERE Match(note_text) Against('+safe +(<combination)' IN BOOLEAN MODE);
```

匹配包含safe和combination的句子，降低combination的等级



##### 全文本搜索的使用说明

* 在索引全文本数据时，短词被忽略且从索引中删除，短词定义为那些具有3个或3个以下字符的词（如有需要，数目可更改）
* MySQL带有一个内建的非用词(stopword)列表，这些词在索引全文本数据时总是被忽略，如有需要，可以覆盖这个列表
* 许多次出现频率很高，搜索它们没有用处（返回太多结果）。因此，MySQL规定了一条50%规则，若一个词出现在50%以上的行中，则将它作为一个非用词忽略。50%规则不用于IN BOOLEAN MODE
* 如果表中的行数少于3行，则全文本搜索不返回结果（因为每个词或者不出现，或者至少出现在50%的行中）。
* 忽略词中的单引号。例如，don't索引为dont。
* 不具有词分隔符（包括日语和汉语）的语言不能恰当地返回全文本搜索结果
* 如前所述，仅在MyISAM数据库引擎中支持全文本搜索



### 插入数据

#### 数据插入

插入有四种方式：

* 插入完整的行
* 插入行的一部分
* 插入多行
* 插入某些查询的结果

> 可针对每个表或每个用户，利用MySQL的安全机制禁止使用INSERT语句



#### 插入完整的行

```sql
INSERT INTO customers
VALUES(NULL,
      'Pep E. LaPew',
      '100 Main Street',
      'Los Angeles',
      'CA',
      '90046',
      'USA',
      NULL,
      NULL);
```

此例子插入一个新客户到customers表。存储到每个表列中的数据在VALUES子句中给出，**对每个列必须提供一个值**。如果某个列没有值，则应该用NULL值。各个列必须以它们在表定义中出现的次序填充。如果有列是自动填充的，则也用NULL值填充。



> INSERT语句一般不会产生输出



上述语句虽然简单，却不安全，因为表的结构有可能会改变（列名的次序改变），运行上述语句可能会报错。

更安全（也更繁琐）的语句如下：

```sql
INSERT INTO customers(cust_name,
                     cust_address,
                     cust_city,
                     cust_state,
                     cust_zip,
                     cust_country,
                     cust_contact,
                     cust_email)
                  VALUES('Pep E. LaPew',
                        '100 Main Street',
                        'Los Angeles',
                        'CA',
                        '90046',
                        'USA',
                        NULL,
                        NULL);
```

此语句和上个语句等价，但这个语句制定了列名，即使表的结构改变，该语句仍能正常工作。而如果是自动增量的列，则可以省略。



> 如果表的定义允许，则可以在INSERT操作中省略某些列。省略的列必须满足以下某个条件。
>
> * 该列定义为允许NULL值（无值或空值）
> * 在表定义中给出默认值。
>
> 如果对表中不允许NULL值且没有默认值的列不给出值，则MySQL将产生一条错误信息，并且相应的行插入不成功。

> 提高整体性能
>
> INSERT操作很耗时，可能降低等待处理的SELECT语句的性能，如果数据检索最重要，可以这样写插入语句INSERT LOW_PRIORITY INTO，降低INSERT语句的优先级
>
> 适用UPDATE和DELETE语句



#### 插入多个行

多个语句：

```sql
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES('Pep E. LaPew',
      '100 Main Street',
      'Los Angeles',
      'CA',
      '90046',
      'USA');
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES('M. Martian',
      '42 Galaxy Way',
      'New York',
      'NY',
      '11213',
      'USA');
```

如果INSERT语句中的列名和次序相同，也可以用如下语句：

```sql
INSERT INTO customers(cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country)
VALUES(
      'Pep E. LaPew',
      '100 Main Street',
      'Los Angeles',
      'CA',
      '90046',
      'USA'
),
(
      'M. Martian',
      '42 Galaxy Way',
      'New York',
      'NY',
      '11213',
      'USA'
);
```

> 此技术可以提高数据库处理性能，因为MySQL用单条INSERT语句处理多个插入比使用多条INSERT语句快。



#### 插入检索出的数据

```sql
INSERT INTO customers(cust_id,
                     cust_contact,
                     cust_email,
                     cust_name,
                     cust_address,
                     cust_city,
                     cust_state,
                     cust_zip,
                     cust_country)
                 SELECT cust_id,
                     cust_contact,
                     cust_email,
                     cust_name,
                     cust_address,
                     cust_city,
                     cust_state,
                     cust_zip,
                     cust_country
                 FROM custnews;
```

将custnews表中的数据导入到customers表中。

INSERT SELECT中SELECT语句可包含WHERE子句以过滤插入的数据。



### 更新和删除数据

#### 更新数据

两种用途：

* 更新表中特定行
* 更新表中所有行

基本结构：

* 要更新的表
* 列名和它们的新值
* 要更新行的过滤条件

> 不要省略WHERE子句，以免更新所有行



```sql
UPDATE customers
SET cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```

更新customer表中cust_id为10005的那一行的cust_email



更新多个列：

```sql
UPDATE customers
SET cust_name = 'The Fudds',
    cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```

> UPDATE语句中可以使用子查询

> 如果用UPDATE语句更新多行时出现了错误，则整个UPDATE操作取消，已经更新的行也会恢复到原来的值，为了即使发生错误，也继续更新。可使用IGNORE关键字，如下所示：UPDATE IGNORE customers



为了删除某个列的值，可设置它为NULL(假如表定义允许NULL值)

```sql
UPDATE customers
SET cust_email = NULL
WHERE cust_id = 10005;
```

其中NULL用来去除cust_email的值



#### 删除数据

两种用途：

* 从表中删除特定行
* 从表中删除所有行

> 省略WHERE子句将导致删掉表中所有行



```sql
DELETE FROM customers
WHERE cust_id = 10006;
```

删除客户10006



**DELETE不需要列名或通配符。DELETE删除整行而非列，UPDATE删除列**



> DELETE语句从表中删除行甚至删除所有行，但DELETE不删除表本身

> 如果想从表中删除所有行，可以使用TRUNCATE TABLE，它完成相同工作同时更为高效，因为它是重新创建一个表来实现删除原来的表，而不是逐行删除表中数据



#### 更新和删除的指导原则

* 除非确实打算更新和删除每一行，否则绝对不要使用不带WHERE子句的UPDATE或DELETE语句
* 保证每个表都有主键，尽可能像WHERE子句那样使用它
* 对UPDATE和DELETE中WHERE子句进行SELECT测试
* 使用强制实施引用完整性的数据库，这样MySQL将不允许删除其它与其它库相关联数据的行

> MySQL没有撤销按钮，应非常小心使用UPDATE和DELETE



### 创建和操纵表

#### 创建表

##### 表创建基础

```sql
CREATE TABLE customers
(
	cust_id			int			NOT NULL AUTO_INCREMENT,
    cust_name		char(50) 	NOT NULL,
    cust_address	char(50)	NULL,
    cust_city		char(50)	NULL,
    cust_state		char(5)		NULL,
    cust_zip		char(10)	NULL,
    cust_country	char(50)	NULL,
    cust_contact	char(50)	NULL,
    cust_email		char(255)	NULL,
    PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
```

> 处理现有的表：在创建新表时，指定的表名必须不存在，否则将出错。如果仅想在一个表不存在时创建它，应该在表名后加上IF NOT EXISTS



#####  使用NULL值

在创建表时，规定NOT NULL的列不接受NULL值



##### 主键再介绍

多个列组成的主键

```sql
CREATE TABLE orderitems
(
	order_num		int				NOT NULL,
    order_item		int				NOT NULL,
    prod_id			char(10)		NOT NULL,
    quantity		int				NOT NULL,
    item_price		decimal(8,2)	NOT NULL,
    PRIMARY KEY(order_num, order_item)
)ENGINE=InnoDB;
```

> 主键只能使用不允许NULL值的列



##### 使用AUTO_INCREMENT

自动增量，每个表只允许一个AUTO_INCREMENT，而且它必须被索引（如，通过使它成为主键）

> 如果一个列被指定为AUTO_INCREMENT，若想给一行赋给指定值，需保证该值至今从未使用过，而指定值将取代自动生成的值，其后的行都将在指定值的基础上依次加一。

> SELECT last_insert_id()函数可以帮助返回最后一个AUTO_INCREAMENT值



##### 指定默认值

```sql
CREATE TABLE orderitems
(
	order_num	int			NOT NULL,
    order_item	int			NOT NULL,
    prod_id		char(10)	NOT NULL,
    quantity	int			NOT NULL DEFAULT 1,
    item_price	decimal(8,2)NOT NULL,
    PRIMARY KEY(order_num, order_item)
)ENGINE=InnoDB;
```

> MySQL不允许使用函数作为默认值，它只支持常量



##### 引擎类型

如果省略ENGINE=语句，则使用默认引擎（很可能是MyISAM）。

需要知道的引擎：

* InnoDB：可靠的事务处理引擎，不支持全文本搜索
* MEMORY：功能等同MyISAM，由于数据存储在内存（不是磁盘）中，速度很快（适合临时表）

* MyISAM是一个性能极高的引擎，支持全文本搜索，不支持事务处理



#### 更新表

```sql
ALTER TABLE vendors
ADD vend_phone CHAR(20);
```

给表增加一个列



```sql
ALTER TABLE vendors
DROP COLUMN vend_phone;
```

删除刚添加的列



```sql
ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);

ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_products FOREIGN KEY (prod_id)
REFERENCES products (prod_id);

ALTER TABLE orders
ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id)
REFERENCES customers (cust_id);

ALTER TABLE products
ADD CONSTRAINT fk_products_vendors
FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);
```

定义外键



由于更改4个不同的表，所以用了4条ALTER TABLE语句。为了对单个表进行多个更改，可以使用单条ALTER TABLE语句，每个更改用逗号分隔。



复杂的表结构更改一般需要手动删除过程，它涉及以下步骤：

* 用新的列布局创建一个新表
* 使用INSERT SELECT语句，从旧表复制数据到新表。如果有必要，可使用转换函数和计算字段。
* 检验包含所需数据的新表。
* 重命名旧表（如果确定，可以删除它）
* 用旧表原来的名字重命名新表
* 根据需要，重新创建触发器、存储过程、索引和外键



> 小心使用ALTER TABLE，因为数据库的更改不能撤销



#### 删除表

```sql
DROP TABLE customers;
```

删除customers2表（假设它存在），没有确认，不能撤销，永久删除。



#### 重命名表

```sql
RENAME TABLE customers2 TO customers;
```

重命名一个表，也可以和下面的语句一样重命名多个表：

```sql
RENAME TABLE backup_customers TO customer,
			 backup_vendores TO vendors,
			 backup_products TO products;
```

