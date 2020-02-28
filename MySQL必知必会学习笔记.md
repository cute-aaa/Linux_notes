# MySQL必知必会学习笔记

```sh
systemctl start docker
docker start ff7
docker exec -it ff7 /usr/bin/mysql -uroot -p123456
```



## 查看数据库信息

​	查看所有数据库：

```
show databases;
```

​	使用某个数据库

```
use test;
```

​	查看数据库内的所有表：

```
show tables;
```

​	显示表内的所有列：

```
show columns from 表名;
可以简化为：
describe 表名;
```

​	显示广泛的服务器状态信息：

```
show status;
```

​	显示创建特定数据库

```
show create database;
```

​	显示创建特定表

```
show create table;
```

​	显示授予用户（所有用户或授权用户）的安全权限

```
show grants;
```

​	显示服务器错误

```
show errors;
```

​	显示服务器警告

```
show warnings;
```



​	使用 `help show` 来显示允许的show语句







## MySQL数据类型

如果数值是计算（求和、平均等）中使用的数值，则应该存储在数值数据类型列中。如果作为字符串（可能只包含数字）使用，则应该保存在串数据类型列中。



##### 字符串：

字符串必须扩在引号内。

| 数据类型  | 说明 |
| ------------ | ------ |
| CHAR |  1～255个字符的定长串。它的长度必须在创建时指定，否则MySQL假定为R(1) |
| ENUM |  接受最多64 K个串组成的一个预定义集合的某个串 |
| LONGTEXT |  与TEXT相同，但最大长度为4 GB  |
| MEDIUMTEXT |  与TEXT相同，但最大长度为16 K  |
| SET |  接受最多64个串组成的一个预定义集合的零个或多个串 |
| TEXT |  最大长度为64 K的变长文本 |
| TINYTEXT |  与TEXT相同，但最大长度为255字节 |
| VARCHAR |  长度可变，最多不超过255字节。如果在创建时指定为VARCHAR(n)，则可存储0到n个字符的变长串（其中n≤255） |

##### 数值：

数值不支持以0开头，比如以0开头的电话、邮编不能以数值类型存储。

所有数值类型（除bit和boolean）都可以有符号或无符号，默认为有符号，使用 unsigned 指定无符号。

| 数据类型  | 说 明  |
| ------------ | ------ |
| BIT |  位字段，1～64位。（在MySQL 5之前，BIT在功能上等价于TINYINT |
| BIGINT |  整数值，支持-9223372036854775808～9223372036854775807（如果是UNSIGNED，为0～18446744073709551615）的数 |
| BOOLEAN（BOOL） |  布尔标志，或者为0或者为1，主要用于开/关（on/off）标志 |
| DECIMAL（DEC）  | 精度可变的浮点值 |
| DOUBLE  | 双精度浮点值 |
| FLOAT  | 单精度浮点值 |
| INT（INTEGER） |  整数值，支持-2147483648～2147483647（如果是UNSIGNED，为0～4294967295）的数 |
| MEDIUMINT |  整数值，支持-8388608～8388607（如果是UNSIGNED，为0～16777215）的数 |
| REAL  | 4字节的浮点值 |
| SMALLINT  | 整数值，支持-32768～32767（如果是UNSIGNED，为0～65535）的数 |
| TINYINT  | 整数值，支持-128～127（如果为UNSIGNED，为0～255）的数 |

##### 日期和时间

| 数据类型  | 说 明 |
| ------------ | ------ |
| DATE  | 表示1000-01-01～9999-12-31的日期，格式为YYYY-MM-DD |
| DATETIME  | DATE和TIME的组合 |
| TIMESTAMP |  功能和DATETIME相同（但范围较小） |
| TIME  | 格式为HH:MM:SS |
| YEAR  | 用2位数字表示，范围是70（1970年）～69（2069年），用4位数字表示，范围是1901年～2155年 |

##### 二进制数据

二进制数据类型可存储任何数据（甚至包括二进制信息），如图像、多媒体、字处理文档等。

| 数据类型  | 说 明 |
| ------------ | ------ |
| BLOB |  Blob最大长度为64 KB  |
| MEDIUMBLOB  | Blob最大长度为16 MB  |
| LONGBLOB  | Blob最大长度为4 GB  |
| TINYBLOB  | Blob最大长度为255字节 |







## 创建和操作表



创建表：

```
create table customers
(
	cust_id		 int		not NULL	auto_increment,
	cust_name	 char(50)	not NULL,
	cust_address char(50)	NULL,
	cust_city	 char(50)	NULL,
	cust_state	 char(5)	NULL,
	cust_zip	 char(10)	NULL,
	cust_country char(50)	NULL,
	cust_contact char(50)	NULL,
	cust_email	 char(255)	NULL,
	primary key(cust_id)
) engine=innodb;
```

```
create table orderitems
(
	order_num	int		 NOT NULL,
	order_item	int		 NOT NULL,
	prod_id		char(10) NOT NULL,
	quantity	int		 NOT NULL	default 1,
	item_price	decimal(8, 2)	NOT NULL,
	primary key(order_num, order_item)
) engine=innodb;
```

如果想不存在时才创建，可以在表名后加上 `if not exists` 。

如果允许NULL，列名后可以不加NULL，默认允许空值。

每个表只能有一个自增列，并且必须被索引。如果手动指定了自增列的值，则后续开始从手动指定的值开始自增。

可以使用 `last_insert_id()` 获取最后一个自增的值。

```
select last_insert_id() ....
```

可以给列指定默认值，但mysql不允许函数作为默认值，只支持常量。

许多数据库开发人员设定列使用默认值而不是NULL，特别是对用于计算或数据分组的列更是如此。



### 引擎类型

MySQL有多个用于具体管理和处理数据的内部引擎，执行创建表、处理数据请求等操作。

可以不省略 `engine = ` 语句，会使用默认引擎

查看默认引擎：

```
show variables like '%storage_engine%';
或
show engines;
```

更改默认引擎：

（临时）

```
set default_storage_engine=引擎名
```

（永久）

```
在mysql配置文件（linux下为/etc/my.cnf），在mysqld后面增加default-storage-engine=INNODB即可。
```

更改表的引擎：

```
alter table 表名 engine=引擎名;
```

引擎可以混用，但要考虑另一个引擎是否也有需要的功能。



引擎比较：

| 功能         | MylSAM | MEMORY | InnoDB | Archive |
| ------------ | ------ | ------ | ------ | ------- |
| 存储限制     | 256TB  | RAM    | 64TB   | None    |
| 支持事务     | No     | No     | Yes    | No      |
| 支持全文索引 | Yes    | No     | No     | No      |
| 支持树索引   | Yes    | Yes    | Yes    | No      |
| 支持哈希索引 | No     | Yes    | No     | No      |
| 支持数据缓存 | No     | N/A    | Yes    | No      |
| 支持外键     | No     | No     | Yes    | No      |

- 如果要提供提交、回滚和恢复的事务安全（ACID 兼容）能力，并要求实现并发控制，InnoDB 是一个很好的选择。
- 如果数据表主要用来插入和查询记录，则 MyISAM 引擎提供较高的处理效率。
- 如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存的 MEMORY 引擎中，MySQL 中使用该引擎作为临时表，存放查询的中间结果。
- 如果只有 INSERT 和 SELECT 操作，可以选择Archive 引擎，Archive 存储引擎支持高并发的插入操作，但是本身并不是事务安全的。Archive 存储引擎非常适合存储归档数据，如记录日志信息可以使用 Archive 引擎。



引擎的具体信息参考https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html





## 更新表

- 更新表时，最好做一个完整的备份（模式和数据的备份），因为更改不能撤销。

使用 `alter table` 来更新表，但尽量设计表时多考虑，以便不对表进行大的改动。

添加与删除列：

```
alter table vendors add vend_phone char(20);
```

```
alter table vendors drop column vend_phone;
```

定义外键：

```
alter table orderitems 
add constraint fk_orderitems_orders
foreign key(order_num) references orders (order_num);
```

对单个表进行多个更改，可以使用单条alter table语句，每个更改用逗号分隔。

重命名表：

```
rename talbe 旧名 to 新名 [, 旧名 to 新名];
```



复杂的表结构更改一般需要手动删除过程，它涉及以下步骤：

- 用新的列布局创建一个新表；
- 使用INSERT SELECT语句从 旧表复制数据到新表。如果有必要，可使用转换函数和计算字段；
- 检验包含所需数据的新表；
- 重命名旧表（如果确定，可以删除它）；
- 用旧表原来的名字重命名新表；
- 根据需要，重新创建触发器、存储过程、索引和外键。



## 删除表

删除整个表而不是内容：

```
drop table 表名;
```

没有确认，也不能撤销。







## 检索数据

简单检索：

```
select 列名 from 表名;
```

```
select 列名, 列名, 列名 from 表名;
```

一般，除非你确实需要表中的每个列，否则最好别使用*通配符。虽然使用通配符可能会使你自己省事，不用明确列出所需列，但检索不需要的列通常会降低检索和应用程序的性能。

```
select * from 表名;
```

去掉重复数据：

```
select distinct 列名 from 表名;
```

返回指定行：

```
select 列名 from 表名 limit [开始行数, ]返回的行数;
如：
select prod_name from products limit 1,5;
返回从第二行(下标从0开始)开始往下数五个
行数不够时能只返回能返回的行数

另一种语法
select prod_name from products limit 5 offset 1;
从行1开始取5行，更直观
```

限定列名或表名

```
select 表名.列名 from 数据库名.表名;
select products.prod_name from test.products;
```







## 插入数据（insert）

---

### 更改语句的优先级

数据库的更新操作可能很耗时（特别是有很多索引需要更新时），并可能降低等待处理的select语句的性能。在请求较多时，可以降低语句的优先级：

```
insert low_priority into
```

```
update low_priority
```

```
delete low_priority
```

```
replace low_priority
```

---



插入可以用几种方式使用：

- 插入完整的行；

- 插入行的一部分；

- 插入多行；

- 插入某些查询的结果。

可针对每个表或每个用户，利用MySQL的安全机制禁止使用INSERT语句。

简单示例：

```mysql
insert into customers 
values(NULL, 
	'Pep E. LaPew', 
	'100 Main Street', 
	'Los Angeles', 
	'CA', 
	'90046', 
	'USA', 
	NULL, 
	NULL);
```

插入一行新值，没有值就指定NULL（如果允许NULL），各个列必须以在表的定义中的次序填充。

第一列也是NULL，因为设置了自增，设定的NULL会被mysql忽略。

如果不想依赖表定义的次序：

```
insert into customers(cust_name, 
	cust_address, 
	cust_city, 
	cust_state, 
	cust_zip, 
	cust_country, 
	cust_contact, 
	cust_email) 
values('Pep E. LaPew', 
	'100 Main Street', 
	'Los Angeles', 
	'CA', 
	'90046', 
	'USA', 
	NULL, 
	NULL);
```

自增的cust_id列和值可以省略不写了。

先明确给出要插入的列名，顺序随意，再用values给出对应值。

可以忽略一些列，省略列必须至少满足以下条件之一：

- 允许NULL（无值或空值）
- 有默认值

如果省略了不满足条件的列会报错。

一次插入多行：

```
insert into customers(cust_name, 
	cust_address, 
	cust_city, 
	cust_state, 
	cust_zip, 
	cust_country) 
values(
	'Pep E. LaPew',
	'100 Main Street',
	'Los Angeles',
	'CA',
	'90046',
	'USA'),
	(
	'M. Martian',
	'42 Galaxy Way',
	'New York',
	'NY',
	'11213',
	'USA');
```

这样比多个insert语句快。



**插入返回的数据**：

假如你想从另一表中合并客户列表到你的customers表。不需要每次读取一行，然后再将它用INSERT插入：

```
insert into customers(cust_id,
	cust_contact,
	cust_email,
	cust_name,
	cust_address,
	cust_city,
	cust_state,
	cust_zip,
	cust_country)
select cust_id,
	cust_contact,
	cust_email,
	cust_name,
	cust_address,
	cust_city,
	cust_state,
	cust_zip,
	cust_country
from custnew;
```





## 更新数据

注意update的where语句，不然会更新表中的所有行。

可以限制和控制update语句的使用。



```
update 表名 set 列名 = 新值 [, 列名 = 新值] where 条件
```

```
update customers set cust_email = 'elmer@fudd.com' where cust_id = 10005;
```

- update也可以使用select返回的子查询

如果用update更新多行，但更新某行时出现错误，则所有操作被回滚。如果即使发生错误也要继续更新，可以使用ignore：

```
update ignore 表名....
```

想用update来删除某个列的值，可以将它更新为NULL：

```
update customers set cust_email = NULL where cust_id = 10005;
```



## 删除数据

同样，注意where语句，不然会删除表中的所有行。

可以限制和控制delete语句的使用。



```
delete from 表名 where 条件
```

```
delete from customers where cust_id = 10006;
```

delete删除整行，update可以删除某个列。

delete不会删除表本身。但如果想从表中删除所有行，不要使用delete，使用 `truncate` ，速度更快（truncate实际上时删除原来的表并重新创建一个表，而delete是逐行删除数据）。



如果从一个表中删除大量数据，应该使用 OPTIMIZE TABLE 来回收所用的空间，优化性能。



**更新和删除的建议**：

- 除非确实想更新和删除每一行，否则不要使用不带where的update和delete
- 保证每个表都有主键，尽可能在where中使用它，可以指定各逐渐、多个值或值的范围
- 在使用update和delete使用where之前，应该先用select进行测试，保证过滤的是正确的记录
- 使用强制实施引用完整性的数据库，避免删除具有与其他表相关联的数据的行





## 排序数据

排序

```
select 列名[, 列名...] from 表名 order by 列名[, 列名...]
```

倒序

```
select ........ 列名 desc;
desc只对它前面的列名生效，如果希望多个降序，需要都指定desc
```

示例：找出列中的最高值

```
select prod_price from products order by prod_price desc limit 1;
```



## 过滤数据

```
select 列名[, 列名] from 表名 where 条件[ and 条件] [ order by];
```



**算数运算符**：	= 		!=(或<>)		>		<		>=		<=		BETWEEN		 IS NULL

```
select prod_name, prod_price from products where prod_price = 2.50
```

```mysql
select prod_id, prod_price, prod_name from products where vend_id = 1003 and prod_price <=10;
```

```mysql
select prod_name, prod_price from products where vend_id = 1002 or vend_id = 1003;
```

```mysql
select prod_name, prod_price from products where (vend_id=1002 or vend_id=1003 ) and prod_price >= 10;
```

```
select prod_name, prod_price from products where prod_price between 5 and 10;
```



**逻辑运算符**：AND 		OR		IN		NOT

​	(AND优先级比OR高)

​	示例：

查找所有id=1002或id=1003的值。IN语句的整个清单必须在圆括号中，比如换成1002，1004则不会包含1003

IN一般比OR执行的快

IN可以包含其他select语句

```
select prod_name, prod_price from products where vend_id in (1002, 1003) order by prod_name;
```

NOT用来否定它之后的所跟的任何条件

列出出1002和1003之外的所有供应商：

```
select prod_name, prod_price from products where vend_id not in (1002, 1003) order by prod_name;
```



**通配符**：使用 LIKE 操作符：	%		_ 

- 不要过度使用通配符，因为它更耗时间。如果其他操作符能达到同样的目的，应该使用其他操作符
- 尽量不要将通配符用在搜索模式的开头，那样搜索起来是最慢的

%可以匹配[0, ∞)个字符，但不匹配空格。比如使用 '%anvil' 搜索时，不会匹配 'anvil '。一个简单的解决方法是在搜索模式最后加'%'，更好的办法是使用函数去掉首尾空格。

%不匹配NULL

```
select prod_id, prod_name from products where prod_name like 'jet%';
```

```
select prod_id, prod_name from products where prod_name like '%anvil%';
```

'_' 只匹配1个字符，0个和多个都不匹配

```
select prod_id, prod_name from products where prod_name like '_ ton anvil';
```



**正则表达式**：	REGEXP：	'.'		'|'		'[ ]'		'^'		'[-]'		'\\\\'	字符类	多次匹配

REGEXP将后面所跟的东西作为正则表达式处理

```
select prod_name from products where prod_name regexp '1000' order by prod_name;
输出JetPack 1000
```

'.' 匹配任意一个字符：

```
select prod_name from products where prod_name regexp '.000' order by prod_name;
输出
JetPack 1000
JetPack 2000
```

与LIKE区别：

LIKE匹配整个列的值

```
select prod_name from products where prod_name like '1000';
无数据
```

REGEXP可以在列值内匹配

```
select prod_name from products where prod_name regexp '1000';
返回JetPack 1000
select prod_name from products where prod_name regexp '000';
返回
JetPack 1000
JetPack 2000
```

REGEXP也可以匹配整个列的值：使用 '^' 和 '$'（在后面）

匹配默认不区分大小写，要区分：使用BINARY

```
select ...... where prod_name regexp binary 'JetPack .000';
```



'|'进行OR匹配

匹配1000或2000

```
select prod_name from products where prod_name regexp '1000|2000';
返回
JetPack 1000
JetPack 2000
```



'[]' 进行IN匹配

匹配1或2或3

```
select prod_name from products where prod_name regexp '[123] Ton';
('[1|2|3]'的缩写)
返回
1 ton anvil
2 ton anvil
```

匹配'1'或'2'或'3 Ton'

```mysql
select prod_name from products where prod_name regexp '1|2|3 Ton';
返回
1 ton anvil
2 ton anvil
JetPack 1000
JetPack 2000
TNT (1 stick)
```



'^' 进行NOT匹配

匹配除1、2、3之外的记录：

```
select ...... regexp '[^123]';
```



'[-]'进行范围匹配

```
select ...... regexp '[0-9]';
相当于
select ...... regexp '[0123456789]';
```



'\\\\' 转义字符

MySQL解释一个\，正则解释另一个\

匹配特殊字符：

```
select vend_name from vendors where vend_name regexp '\\.';
返回
Furball Inc.
```

还有：

| 元字符 |   说明   |
| :----: | :------: |
| \\\\f  |   换页   |
| \\\\n  |   换行   |
| \\\\r  |   回车   |
| \\\\t  |   制表   |
| \\\\v  | 纵向制表 |
| \\\\\  |    \     |



匹配字符类：
| 类 |   说明   |
| :----: | :------: |
| [:alnum:] |  任意字母和数字（同[a-zA-Z0-9]） |
| [:alpha:]  | 任意字符（同[a-zA-Z]） |
| [:blank:]  | 空格和制表（同[\\t]） |
| [:cntrl:]  | ASCII控制字符（ASCII 0到31和127） |
| [:digit:]  | 任意数字（同[0-9]） |
| [:graph:]  |  与[:print:]相同，但不包括空格 |
| [:lower:]  | 任意小写字母（同[a-z]） |
| [:print:]  | 任意可打印字符 |
| [:punct:]  | 既不在[:alnum:]又不在[:cntrl:]中的任意字符 |
| [:space:]  | 包括空格在内的任意空白字符（同[\\f\\n\\r\\t\\v]） |
| [:upper:]  | 任意大写字母（同[A-Z]） |
| [:xdigit:] | 任意十六进制数字（同[a-fA-F0-9]） |





匹配多次前面的搜索：
| 元 字 符 |  说 明 |
| :----: | :------: |
| *  | 0个或多个匹配 |
| +  | 1个或多个匹配（等于{1,}） |
| ?  | 0个或1个匹配（等于{0,1}） |
| {n}  | 指定数目的匹配 |
| {n,}  | 不少于指定数目的匹配 |
| {n,m}  | 匹配数目的范围（m不超过255） |

```
select prod_name from products where prod_name regexp '\\([0-9] sticks?\\)';
'\\('匹配'('，'[0-9]'匹配任意数字，'sticks?'匹配'stick'或'sticks'，因为'?'匹配了前面的s的0次或1次)
返回
TNT (1 stick)
TNT (5 sticks)
```

```
select prod_name from products where prod_name regexp '[[:digit:]]{4}';
'[:digit:]'匹配任意数字，'{4}'要求前面的匹配4次，所以'[[:digit:]]{4}'匹配连在一起的4位数字。
```



匹配特定位置：

| 元 字 符  | 说 明  |
| :----: | :------: |
| ^  | 文本的开始  |
| $  | 文本的结尾  |
| [[:<:]]  | 词的开始  |
| [[:>:]]  | 词的结尾  |

注意，'^'有两种用法，放在集合中('[^]')用来否定集合，否则指串的开头

```
select prod_name from products where prod_name regexp '^[0-9\\.]';
匹配以数字或'.'开头的数
返回
.5 ton anvil
1 ton anvil
2 ton anvil
```

```
select ...... regexp '^.....$';
相当于
select ...... like '.....';
即不再匹配子串，而是像like一样匹配整个串
```



简单的正则表达式测试：

可以在不使用数据库表的情况下测试表达式是否可用，测试返回0（不匹配）或1（匹配）

```
select 'hello' regexp '[0-9]';
返回0，因为文本'hello'中没有数字
```



## 计算字段

字段（field） 基本上与列（column）的意思相同，经常互换使用，不过数据库列一般称为列，术语字段通常用在计算字段的连接上。

**拼接（concatenate）**：将多个值联结到一起构成单个值，多数DBMS使用+或||来拼接，mysql使用Concat()函数拼接。

```
select concat(vend_name, ' (', vend_country, ')') from vendors order by vend_name;
```

上面的语句连接了"vend_name", " ' (' ", "vend_country", " ')' "四个元素。select 语句返回包含上面四个元素的单个列（计算字段）。

**删除空格**：Trim()		LTrim()		RTrim()

```
select concat(rtrim(vend_name), ' (', rtrim(vend_country), ')') from vendors order by vend_name;
```

三个分别是去掉两边空格、去掉左边空格、去掉右边空格

**使用别名**：前面的拼接函数返回的新计算列，并没有列名，只是一个值，未命名的列只能用于查看结果，但客户机没有办法引用。对此，可以使用别名（alias），作为字段或值的替换名。使用关键字AS来赋予别名。

别名还有其他用途。常见的用途包括在实际的表列名包含不符合规定的字符（如空格）时重新命名它，在原来的名字含混或容易误解时扩充它，等等。

别名有时也称为导出列（derived column）

```
select concat(rtrim(vend_name), ' (', rtrim(vend_country), ')') as vend_title from vendors order by vend_name;
```

上述语句指定sql创建一个包含指定计算的名为vend_title的计算字段，将值命名为vend_title，任何客户机都可以想其他实际表列一样按名来引用它。



**执行算数运算**：

orders表包含收到的所有订单，orderitems表包含每个订单中的各项物品。20005是订单号，item_price列包含订单中每项物品的单价。如下汇总物品的价格（单价乘以订购数量）：

```mysql
select prod_id, quantity, item_price, quantity * item_price as expanded_price from orderitems where order_num = 20005;
```

支持+-*/，圆括号可以区分优先级

测试计算：

```
select 3*2;
select trim('abc');
select Now();
```



## 使用函数处理数据

函数的可移植性没有SQL强

大多数SQL支持以下函数：

- 用于处理文本串，比如删除或填充值，转换大小写
- 执行算数运算，如返回绝对值，进行代数运算
- 处理日期和时间并从中提取特定成分，如求两日期之差，检查日期有效性
- 返回DBMS正使用的特殊信息，如返回用户登录信息，检查版本细节

**文本处理**

上面的concat()和Trim()就是一个文本处理函数，这次使用Upper()

```
select vend_name, upper(vend_name) as vend_name_upcase from vendors order by vend_name;
```

将vend_name列转化为大写，并重命名为vend_name_upcase

常用函数：

| 函数名称    | 作 用                                                        |
| ----------- | ------------------------------------------------------------ |
| LENGTH()    | 返回字符串的长度 |
| CONCAT()    | 合并字符串函数 |
| INSERT()    | 替换字符串函数 |
| LOWER()     | 将字符串中的字母转换为小写    |
| UPPER()     | 将字符串中的字母转换为大写   |
| LEFT()      | 截取串左边的字符(left('源串', 截取长度)) |
| RIGHT()     | 截取串右边的字符   |
| TRIM()      | 删除字符串左右两侧的空格  |
| LTrim()  | 去掉串左边的空格 |
| RTrim()  | 去掉串右边的空格 |
| Locate()  | 找出串的一个子串(locate('查找串', '源串')) |
| REPLACE()   | 字符串替换函数，返回替换后的新字符串 |
| SUBSTRING() | 截取字符串（substring('源串', '起始位置', '截取长度')） |
| REVERSE()   | 字符串反转（逆序） |
| Soundex()  | 返回串的SOUNDEX值 |

SOUNDEX是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。SOUNDEX考虑了类似的发音字符和音节，使得能对串进行发音比较而不是字母比较。

下面给出一个使用Soundex()函数的例子。customers表中有一个顾客Coyote Inc.，其联系名为Y.Lee。但如果这是输入错误，此联系名实际应该是Y.Lie，怎么办？显然，按正确的联系名搜索不会返回数据，如下所示：

```
Empty set (0.00 sec)
```

现在试一下使用Soundex()函数进行搜索，它匹配所有发音类似于Y.Lie的联系名：

```mysql
+-------------+--------------+
| cust_name   | cust_contact |
+-------------+--------------+
| Coyote Inc. | Y Lee        |
+-------------+--------------+
```

在这个例子中，WHERE子句使用Soundex()函数来转换cust_ contact列值和搜索串为它们的SOUNDEX值。因为Y.Lee和Y.Lie发音相似，所以它们的SOUNDEX值匹配，因此WHERE子句正确地过滤出了所需的数据。

**时间处理**

一般，应用程序不使用MYSQL中存储日期和时间的格式，因此日期和时间函数总是被用来读取、统计和处理这些值。

常用函数：

| 函 数 |  说 明 |
| ----------- | ------------------------------------------------------------ |
| CurDate()/current_date() | 返回当前日期 |
| CurTime()/current_time() | 返回当前时间 |
| Now()/sysdate()  | 返回当前日期和时间 |
| unix_timestamp()  | 获取unix时间戳（毫秒数） |
| from_unixtime() | 将unix时间戳转化为时间格式 |
| TIME_TO_SEC	| 将时间参数转换为秒数 |
| SEC_TO_TIME	| 将秒数转换为时间 |
| Second()  | 返回一个时间的秒部分 |
| Minute()  | 返回一个时间的分钟部分 |
| Hour()  | 返回一个时间的小时部分 |
| Day()  | 返回一个日期的天数部分 |
| Week() | 一年的第几周 |
| Month()  | 返回一个日期的月份部分 |
| Year()  | 返回一个日期的年份部分 |
| dayName() | 日期对应的星期名 |
| monthName()  | 日期对应的月份名 |
| DayOfWeek()  | 日期对应的星期几 |
| DayOfMonth()  | 日期对应的月的第几天(1-31) |
| DayOfYear() | 日期对应的年的第几天(1-366) |
| Date()  | 返回日期时间的日期部分 |
| AddTime()  | 增加一个时间（时、分等） |
| AddDate()  | 增加一个日期（天、周等） |
| Date_Add()  | 增加一个日期 |
| subtime() | 减去一个时间（时、分等） |
| DATE_SUB 和 SUBDATE | 两个函数功能相同，都是向日期减去指定的时间间隔 |
| DateDiff()  | 计算两个日期之差 |
| Date_Format()  | 返回一个格式化的日期或时间串 |
| Time() | 返回一个日期时间的时间部分 |
| WEEKDAY | 获取指定日期在一周内的对应的工作日索引 |

```
select cust_id, order_num from orders where order_date = '2005-09-01';
```

上述语句只匹配日期，不匹配时间，所以假设order_date值为 2005-09-01，也不会被匹配

指示mysql只匹配日期：

```
select cust_id, order_num from orders where date(order_date) = '2005-09-01';
```

使用time()只匹配时间。

匹配日期范围：

指定具体范围，可以使用BETWEEN

```
select cust_id, order_num from orders where date(order_date) between '2005-09-01' and '2005-09-30';
```

如果想返回整月范围内的，可以使用YEAR和MONTH

```
select cust_id, order_num from orders where year(order_date) = 2005 and month(order_date) = 9;
```



**数值处理函数**：

| 函数名称                                                     | 作 用                                                      |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| [ABS](http://c.biancheng.net/mysql/abc.html)                 | 求绝对值                                                   |
| [SQRT](http://c.biancheng.net/mysql/sqrt.html)               | 求二次方根                                                 |
| [MOD](http://c.biancheng.net/mysql/mod.html)                 | 求余数                                                     |
| [CEIL 和 CEILING](http://c.biancheng.net/mysql/ceil_celing.html) | 两个函数功能相同，都是返回不小于参数的最小整数，即向上取整 |
| [FLOOR](http://c.biancheng.net/mysql/floor.html)             | 向下取整，返回值转化为一个BIGINT                           |
| [RAND](http://c.biancheng.net/mysql/rand.html)               | 生成一个0~1之间的随机数，传入整数参数是，用来产生重复序列  |
| [ROUND](http://c.biancheng.net/mysql/round.html)             | 对所传参数进行四舍五入                                     |
| [SIGN](http://c.biancheng.net/mysql/sign.html)               | 返回参数的符号                                             |
| [POW 和 POWER](http://c.biancheng.net/mysql/pow_power.html)  | 两个函数的功能相同，都是所传参数的次方的结果值             |
| [SIN](http://c.biancheng.net/mysql/sin.html)                 | 求正弦值                                                   |
| [ASIN](http://c.biancheng.net/mysql/asin.html)               | 求反正弦值，与函数 SIN 互为反函数                          |
| [COS](http://c.biancheng.net/mysql/cos.html)                 | 求余弦值                                                   |
| [ACOS](http://c.biancheng.net/mysql/acos.html)               | 求反余弦值，与函数 COS 互为反函数                          |
| [TAN](http://c.biancheng.net/mysql/tan.html)                 | 求正切值                                                   |
| [ATAN](http://c.biancheng.net/mysql/atan.html)               | 求反正切值，与函数 TAN 互为反函数                          |
| [COT](http://c.biancheng.net/mysql/cot.html)                 | 求余切值                                                   |
| exp                                                          | 球一个数的指数                                             |
| pi                                                           | 圆周率                                                     |




| 函数名称                                           | 作用           |
| -------------------------------------------------- | -------------- |
| [IF](http://c.biancheng.net/mysql/if.html)         | 判断，流程控制 |
| [IFNULL](http://c.biancheng.net/mysql/ifnull.html) | 判断是否为空   |
| [CASE](http://c.biancheng.net/mysql/case.html)     | 搜索语句       |





**数据统计**：


| 函数名称                                         | 作用                             |
| ------------------------------------------------ | -------------------------------- |
| [MAX](http://c.biancheng.net/mysql/max.html)     | 查询指定列的最大值               |
| [MIN](http://c.biancheng.net/mysql/min.html)     | 查询指定列的最小值               |
| [COUNT](http://c.biancheng.net/mysql/count.html) | 统计查询结果的行数               |
| [SUM](http://c.biancheng.net/mysql/sum.html)     | 求和，返回指定列的总和           |
| [AVG](http://c.biancheng.net/mysql/avg.html)     | 求平均值，返回指定列数据的平均值 |

```
select avg(prod_price) as avg_price from products;
select avg(prod_price) as avg_price from products where vend_id = 1003;
```

注意：avg只作用于参数列，如果要获取多个列，要用多个avg。avg()忽略值为NULL的行

COUNT：

count(*)多所有行进行计数，包括NULL

count(column)对指定列计数，忽略NULL，即指定列名时忽略NULL

```
select count(*) as num_cust from customers;
select count(cust_email) as num_cust from customers;
```

Max：

max忽略NULL

max可以返回文本、数值、日期的最大值。用于文本时，如果数据按相应的列排序，则max返回最后一行

```
select max(prod_price) as max_price from produsts;
```

min：

与max相反

sum：

返回指定列值的和

忽略NULL

```
select sum(quantity) as items_ordered from orderitems where order_num = 20005;
也可以合计：
select sum(item_price*quantity) as total_price from orderitems where order_num = 20005;
```



以上的5个聚集函数都支持过滤重复值：

```
select avg(distinct prod_price) as avg_price from products where vend_id = 1003;
```

如果不指定distinct，则使用默认值all

distinct必须使用在列名上，不能用于计算或表达式，如不能用于count(*)，但可以count(distinct 列名)

distinct 用于min和max可行，但没有意义，列的最值不管是否包含不同值结果都是相同的



**组合聚合**：

select可以包含多个聚集函数：

```
select count(*) as num_items, min(prod_price) as price_min, max(prod_price) as price_max, avg(prod_price) as price_avg from products;
```



## 数据分组

将数据分为多个逻辑组，以便对每个组进行聚集计算

```
select vend_id, count(*) as num_prods from products group by vend_id;
```

num_prods为计算字段（用count(*)建立），group by 表示按vend_id排序并分组数据，根据vend_id的分组而不是整个表执行count(\*)

- group by 可以包含任意数目的列，可以对分组进行嵌套
- 如果在group by中嵌套了分组，数据将在最后规定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算（所以不能从个别的列取回数据）
- GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
- 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。
- 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。
- GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

使用WITH ROLLUP关键字，可以得到每个分组以及每个分组汇总级别（针对每个分组）的值，如下所示：

```
select vend_id, count(*) as num_prods from products group by vend_id with rollup;
```



**使用HAVING分组**：

having类似于where，但where过滤行，having过滤分组。到目前为止，学过的所有类型的where都可以用having代替。

也可以理解为，where在分组前进行过滤，having在分组后进行过滤

having支持所有where的操作符

having和where可以同时使用

过滤分组：

```
select cust_id, count(*) as orders from orders group by cust_id having count(*) >= 2;
```

过滤出有两个以上订单的分组。换成where就不行，因为这个过滤基于分组而不基于行

```
select vend_id, count(*) as num_prods from products where prod_price >= 10 group by vend_id having count(*) >= 2;
```

返回数量在2个以上、价格在10个以上的产品。where过滤pord_price在10以上的行，having过滤数量2以上的分组

虽然group by的输出经常是有序的，但有序输出并未在SQL规范中要求。如果有排序需求，还是要用order by

```
select order_num, sum(quantity*item_price) as ordertotal from orderitems group by order_num having sum(quantity*item_price) >= 50;
```

如果想对价格进行排序，就添加order by

```
select order_num, sum(quantity*item_price) as ordertotal from orderitems group by order_num having sum(quantity*item_price) >= 50 order by ordertotal;
```

----

到目前位置的select子语句的顺序：

| 子 句 | 说 明  | 是否必须使用 |
| ------- | ----- | -------------- |
| SELECT  | 要返回的列或表达式  | 是 |
| FROM  | 从中检索数据的表  | 仅在从表选择数据时使用 |
| WHERE  | 行级过滤 |  否 |
| GROUP BY  | 分组说明  | 仅在按组计算聚集时使用 |
| HAVING  | 组级过滤  | 否 |
| ORDER BY  | 输出排序顺序 |  否 |
| LIMIT  | 要检索的行数  | 否 |





## 子查询

假如需要列出订购物品TNT2的所有客户，会有以下步骤：

(1) 检索包含物品TNT2的所有订单的编号。
(2) 检索具有前一步骤列出的订单编号的所有客户的ID。
(3) 检索前一步骤返回的所有客户ID的客户信息。

```
select order_num from orderitems where prod_id = 'TNT2';
```

```
select cust_id from orders where order_num in (20005, 20007);
```

现在，把第一个查询（返回订单号的那一个）变为子查询组合两个查询：

```
select cust_id from orders where order_num in (select order_num from orderitems where prod_id = 'TNT2');
```

在select语句中，子查询总是由内向外处理，首先执行括号内的语句，返回两个订单号20005和20007，然后这两个值以IN操作符要求的逗号分隔的格式传递给外部查询的where子句，查询变成了没有组合的查询的第二段。

可以增加缩进来提升可读性：

```
select cust_name, cust_contact
from customers
where cust_id in (10001, 10004);
```

```
select cust_name, cust_contact 
from customers 
where cust_id in (select cust_id 
				  from orders
				  where order_num in (select order_num
				  					  from orderitems
				  					  where prod_id = 'TNT2'));
```

- 在WHERE子句中使用子查询（如这里所示），应该保证SELECT语句具有与WHERE子句中相同数目的列。可以使用多个列。

- 使用子查询并不总是执行这种类型的数据检索的最有效的方法。



### 创建计算字段

假如需要显示customers表中每个客户的订单总数。订单与相应的客户ID存储在orders表中。查询步骤为：

(1) 从customers表中检索客户列表。
(2) 对于检索出的每个客户，统计其在orders表中的订单数目。

可使用SELECT COUNT(*)对表中的行进行计数，并且通过提供一条WHERE子句来过滤某个特定的客户ID，可仅对该客户的订单进行计数。例如，下面的代码对客户10001的订单进行计数：

```
select count(*) as orders from orders where cust_id = 10001;
```

为了对每个客户执行count(*)计算，应该将count(\*)作为一个子查询：

```
select cust_name, 
	   cust_state, 
	   (select count(*) 
	    from orders 
	    where orders.cust_id = customers.cust_id) as orders 
from customers
order by cust_name;
```

![image-20191104175535413](D:/Typora/imgs/MySQL必知必会学习笔记/image-20191104175535413.png)

这 条 SELECT 语句对 customers 表中每个客户返回 3 列 ：cust_name、cust_state和orders。orders是一个计算字段，它是由圆括号中的子查询建立的。该子查询对检索出的每个客户执行一次。在此例子中，该子查询执行了5次，因为检索出了5个客户。

子查询中的WHERE子句告诉SQL比较orders表中的cust_id与当前正从customers表中检索的cust_id.

**可以理解流程为**：在customers表中，选择一条记录，选中cust_name, cust_state, 然后执行嵌套的select计算字段：统计order表中与当前记录的cust_id相同的记录数，作为orders表，然后通过cust_name排序。

不使用完全限定列名：

![image-20191104175619927](D:/Typora/imgs/MySQL必知必会学习笔记/image-20191104175619927.png)

如果不完全限定列名，MySQL将假定你是对orders表中的cust_id进行自身比较。而SELECT COUNT(*) FROM orders WHERE cust_id = cust_id;总是返回orders表中的订单总数（因为MySQL查看每个订单的cust_id是否与本身匹配，当然，它们总是匹配的）。

同样，虽然这里给出的样例代码运行良好，但它并不是解决这种数据检索的最有效的方法。









## 联结表

**等值联结（内部联结）**：

规定要连接的所有表以及如何关联即可：

```
select vend_name, prod_name, prod_price 
from vendors, products 
where vendors.vend_id = products.vend_id 
order by vend_name, prod_name;
```

SELECT语句与前面所有语句一样指定要检索的列。这里，最大的差别是所指定的两个列（prod_name和prod_price）在一个表中，而另一个列（vend_name）在另一个表中。

FROM子句列出了两个表，分别是vendors和products。就是这条SELECT语句联结的两个表的名字。这两个表用WHERE子句正确联结，WHERE子句指示MySQL匹配vendors表中的vend_id和products表中的vend_id。

在一条SELECT语句中联结几个表时，相应的关系是在运行中构造的。。在联结两个表时，你实际上做的是将第一个表中的每一行与第二个表中的每一行配对。WHERE子句作为过滤条件，它只包含那些匹配给定条件（这里是联结条件）的行。没有WHERE子句，第一个表中的每个行将与第二个表中的每个行配对，而不管它们逻辑上是否可以配在一起。由没有联结条件的表关系返回的结果为笛卡儿积。检索出的行的数目将是第一个表中的行数乘以第二个表中的行数。可以去掉where看一下结果，返回的数据用每个供应商匹配了每个产品，它包括了供应商不正确的产品。实际上有的供应商根本就没有产品。

语句可以直接替换为：

```
select vend_name, prod_name, prod_price 
from vendors inner join products on vendors.vend_id = products.vend_id;
```

这里的from子句不同，两个表之间的关系是from子句的组成部分，以inner join指定。联结条件使用on子句，传递给on的实际条件与传递给where的相同。



可以连接多个表

```
select prod_name, vend_name, prod_price, quantity 
from products, orderitems, vendors 
where products.vend_id = vendors.vend_id 
    and orderitems.prod_id = products.prod_id 
    and order_num = 20005;
```

此例子显示编号为20005的订单中的物品。订单物品存储在orderitems表中。每个产品按其产品ID存储，它引用products表中的产品。这些产品通过供应商ID联结到vendors表中相应的供应商，供应商ID存储在每个产品的记录中。这里的FROM子句列出了3个表，而WHERE子句定义了这两个联结条件，而第三个联结条件用来过滤出订单20005中的物品。

之前的例子：

![image-20191105234048687](D:/Typora/imgs/MySQL必知必会学习笔记/image-20191105234048687.png)

就可以换成：

```
select cust_name, cust_contact 
from customers, orders, orderitems 
where customers.cust_id = orders.cust_id 
    and orderitems.order_num = orders.order_num 
    and prod_id = 'tnt2';
```



不要联结不必要的表。联结的表越多，性能下降越厉害。



**自联结**：

如果想通过一个物品，找到与它出自同一个供应商的其他物品，可以使用子查询：

```
select prod_id, prod_name 
from products 
where vend_id = (select vend_id 
				  from products 
				  where prod_id = 'DTNTR');
```

或者，使用自联结：

```
select p1.prod_id, p1.prod_name 
from products as p1, products as p2 
where p1.vend_id = p2.vend_id 
	and p2.prod_id = 'dtntr';
```

意为：找到p2.prod_id = 'DTNTR'的物品，然后找出他的vend_id，再用p1里的vend_id匹配找出的p2的vend_id，然后列出p1.prod_id, p1,prod_name。

这里使用的都是products表，但使用了两个别名，用来区分两条记录中的字段。



**自然联结**：

无论何时对表进行联结，应该至少有一个列出现在不止一个表中（被联结的列）。标准的联结（前一章中介绍的内部联结）返回所有数据，甚至相同的列多次出现。自然联结排除多次出现，使每个列只返回一次。

可以手动指定需要出现的列：

```
select c.*, o.order_num, o.order_date, oi.prod_id, oi.quantity, oi.item_price 
from customers as c, orders as o, orderitems as oi 
where c.cust_id = o.cust_id 
	and oi.order_num = o.order_num 
	and prod_id = 'FB';
```

事实上，迄今为止我们建立的每个内部联结都是自然联结。



**外部联结**：

使用内部联结检查所有客户及订单：

```
select customers.cust_id , orders.order_num 
from customers inner join orders 
  on customers.cust_id = orders.cust_id ;
```

没有下过订单的客户不会返回。

使用外部联结：

```
select customers.cust_id , orders.order_num 
from customers left outer join orders 
on customers.cust_id = orders.cust_id ;
```

会返回一个order_num为NULL的用户。

外联必须指定左外联还是右外联，上面使用了左外联，即从outer join语句左边的表（customers）中选择所有行，customers表里包含没有订单的用户，其cust_id为NULL，无法匹配外联条件，所以返回NULL。

外联一定要注意顺序，看清楚要全选哪个表。



**带计算的联结**：

```
select 
	customers.cust_name ,
    customers.cust_id , 
    count(orders.order_num ) as num_ord 
from customers left outer join orders 
on customers.cust_id = orders.cust_id 
group by customers.cust_id ;
```

使用左外联将表连接，group by 按客户分组，count对每个用户的订单计数。









**表的别名**：

前面介绍了列的别名：

![image-20191106150256282](D:/Typora/imgs/MySQL必知必会学习笔记/image-20191106150256282.png)

还可以给表起别名：

```
select cust_name, cust_contact 
from customers as c, orders as o, orderitems as oi 
where c.cust_id = o.cust_id 
    and oi.order_num = o.order_num 
    and prod_id = 'tnt2';
```

表别名只在查询执行中使用。与列别名不一样，表别名不返回到客户机。





## 组合查询

有两种基本情况，其中需要使用组合查询：
 在单个查询中从不同的表返回类似结构的数据；
 对单个表执行多个查询，按单个查询返回数据。

多数情况下，组合相同表的两个查询完成的工作与具有多个WHERE子句条件的单条查询完成的工作相同。

假如需要价格小于等于5的所有物品的一个列表，而且还想包括供应商1001和1002生产的所有物品（不考虑价格）。

多个select：

```
select vend_id, prod_id, prod_price from products where prod_price <= 5;
select vend_id, prod_id, prod_price from products where vend_id in (1001, 1002);
```

多个where：

```
select vend_id, prod_id, prod_price 
from products 
where prod_price <= 5 
  or vend_id in (1001, 1002);
```

组合：

```mysql
select vend_id, prod_id, prod_price 
from products 
where prod_price <= 5
union
select vend_id, prod_id, prod_price 
from products 
where vend_id in (1001, 1002);
```

union的结果默认自动去除了重复的行，如果想显示所有行，可以使用union all。

多个where的结果也默认去重，但无法改变这个行为。



- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔（因此，如果组合4条SELECT语句，将要使用3个UNION关键字）。

-  UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。

- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）。

- 虽然使用union能联结多个select，但只能有一个排序语句，不能为每个select指定排序方式。

  - ```
    select vend_id, prod_id, prod_price 
    from products 
    where prod_price <= 5 
    union 
    select vend_id, prod_id, prod_price 
    from products 
    where vend_id in (1001, 1002) 
    order by vend_id, prod_price;
    ```

    





## 索引

 索引建立在一个或几个字段上，建立了索引后,表中的数据就按照索引的一定规则排列。这样可以提高查询速度。 

MySQL中，所有的数据类型都可以被索引。MySQL的索引包括普通索引、惟一性索引、全文索引、单列索引、多列索引和空间索引等。
 索引的优点是可以提高检索数据的速度，这是创建索引的最主要的原因；索引的缺点是创建和维护索引需要耗费时间，耗费时间的数量随着数据量的增加而增加。



###### 创建索引

如果一个简单的where子句返回结果很长，则其中使用的列（或几个列）就是需要索引的对象。

- 创建表时创建索引

```csharp
CREATE TABLE 表名 (属性名 数据类型 [完整性约束条件],
属性名 数据类型 [完整性约束条件],...属性名 数据类型
[UNIQUE | FULLTEXT | SPATIAL] INDEX | KEY
[别名](属性名1 [(长度)] [ASC | DESC]));
```

- 在已经存在的表上创建索引

```css
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX 索引名
ON 表名 (属性名[(长度)] [ASC | DESC]);
```

- 用ALTER TABLE语句来创建索引

```css
ALTER TABLE 表名 ADD [ UNIQUE | FULLTEXT | SPATIAL ] INDEX
索引名(属性名 [(长度)] [ASC | DESC]);
```

###### 删除索引

```mysql
DROP INDEX 索引名 ON 表名 ;
```



## 全文本搜索

---

注意，InnoDB不支持全文本搜索

---

两个最常使用的引擎为MyISAM和InnoDB，前者支持全文本搜索，而后者不支持。



正则表达式的限制：

   性能——通配符和正则表达式匹配通常要求MySQL尝试匹配表中所有行（而且这些搜索极少使用表索引）。因此，由于被搜索行数不断增加，这些搜索可能非常耗时。
   明确控制——使用通配符和正则表达式匹配，很难（而且并不总是能）明确地控制匹配什么和不匹配什么。例如，指定一个词必须匹配，一个词必须不匹配，而一个词仅在第一个词确实匹配的情况下才可以匹配或者才可以不匹配。
   智能化的结果——虽然基于通配符和正则表达式的搜索提供了非常灵活的搜索，但它们都不能提供一种智能化的选择结果的方法。例如，一个特殊词的搜索将会返回包含该词的所有行，而不区分包含单个匹配的行和包含多个匹配的行（按照可能是更好的匹配来排列它们）。类似，一个特殊词的搜索将不会找出不包含该词但包含其他相关词的行

全文本搜索时，MySQL不需要分别查看每个行，不需要分别分析和处理每个词。MySQL创建指定列中各词的一个索引，搜索可以针对这些词进行。这样，MySQL可以快速有效地决定哪些词匹配（哪些行包含它们），哪些词不匹配，它们匹配的频率，等等。

为了进行全文本搜索，必须索引被搜索的列，而且要随着数据的改变不断地重新索引。在对表列进行适当设计后，MySQL会自动进行所有的索引和重新索引。

在索引之后，SELECT可与Match()和Against()一起使用以实际执行搜索。



启用全文本搜索：

```
create table productnotes(
    note_id 	int not 	null auto_increment,
    prod_id 	char(10) 	not null,
    note_date 	datetime 	not null,
    note_text 	text 		null,
    primary key(note_id),
    fulltext(note_text)
    ) engine = myisam;
```

FULLTEXT(note_text)的指示对note_text列进行索引。定义之后，mysql自动维护该索引，在增加、更新或删除行时，索引会自动更新。

也可以在创建之后指定fulltext，但最好将所有数据导入完成之后再指定fulltext，有助于更快的导入数据，而且使索引数据的总时间小于在导入每行时分别进行索引所需的总时间。



索引之后，使用 `match()` 和 `against()` 执行全文本搜索，其中match指定被搜索的列，against指定要使用的表达式：

```
select note_text from productnotes where match(note_text) against('rabbit');
```

传递给match的值必须与fulltext()中定义的相同，并且次序正确。

返回note_text列中包含'rabbit' 的行，默认不区分大小写，可以通过binary指定区分。

上面的语句换成like：

```
select note_text from productnotes where note_text like '%rabbit%';
```

区别：

like返回乱序，fulltext返回以文本匹配的良好成都排序的数据：两个行都包含词rabbit，但包含词rabbit作为第3个词的行的等级比作为第20个词的行高，所以在第一行显示，而like不会。

全文本的数据时索引的，所以也更快。



演示排序如何工作：

```
mysql5:
select note_text, match(note_text) against('rabbit') as rank from productnotes;
mysql8:不能使用别名了
select note_text, match(note_text) against('rabbit') from productnotes;
```

返回的rank列为文本的匹配度。有助于说明全文本搜索如何排除行（排除那些等级为0的行），如何排序结果（按等级以降序排序）。

**扩展查询**：

用来放宽返回的全文本搜索结果的范围。

比如你想找出所有提到anvils的注释。但你还想找出可能与你的搜索有关的所有其他行，即使它们不包含词anvils。

在使用查询扩展时，MySQL对数据和索引进行两遍扫描来完成搜索：

- 首先，进行一个基本的全文本搜索，找出与搜索条件匹配的所有行；
- 其次，MySQL检查这些匹配行并选择所有有用的词（我们将会简要地解释MySQL如何断定什么有用，什么无用）。
- 再其次，MySQL再次进行全文本搜索，这次不仅使用原来的条件，而且还使用所有有用的词。

普通全文本：

```
select note_text from productnotes where match(note_text) against('anvils');
只返回一行
```

带扩展的全文本：

```
select note_text from productnotes where match(note_text) against('anvils' with query expansion);
返回7行
```

返回了7行。第一行包含词anvils，因此等级最高。第二行与anvils无关，但因为它包含第一行中的两个词（customer和recommend），所以也被检索出来。第3行也包含这两个相同的词，但它们在文本中的位置更靠后且分开得更远，因此也包含这一行，但等级为第三。第三行确实也没有涉及anvils（按它们的产品名）。

表中的行越多（这些行中的文本就越多），使用查询扩展返回的结果越好。



**布尔文本搜索**：

即使没有fulltext索引也可以使用，但这个东西比较慢，性能随数据量的增加而降低。

以布尔值提供以下内容：

- 要匹配的词；
- 要排斥的词（如果某行包含这个词，则不返回该行，即使它包含其他指定的词也是如此）；
- 排列提示（指定某些词比其他词更重要，更重要的词等级更高）；
- 表达式分组；
- 另外一些内容
- 布尔文本搜索不提供等级值排序功能！！

```
select note_text 
from productnotes 
where match(note_text) against('heavy' in boolean mode);
```

```mysql
#匹配heavy，排除rope*
select note_text 
from productnotes 
where match(note_text) against('heavy -rope*' in boolean mode);
```

其他操作符：
| 函数名称                              | 作用     |
| -------------------------------------- | -------------- |
| -  | 排除，词必须不出现 |
| >  | 包含，而且增加等级值 |
| <  | 包含，且减少等级值 |
| () | 把词组成子表达式（允许这些子表达式作为一个组被包含、排除、排列等） |
| ~  | 取消一个词的排序值 |
| *  | 词尾的通配符 |
| "" | 定义一个短语（与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语）|

```
# 匹配rabbit和bait
select note_text from productnotes where match(note_text) against('+rabbit +bait' in boolean mode);
```

```
# 匹配rabbit和/或bait
select note_text from productnotes where match(note_text) against('rabbit bait' in boolean mode);
```
```
# 匹配"rabbit bait"这一个词
select note_text from productnotes where match(note_text) against('"rabbit bait"' in boolean mode);
```
```
# 增加rabbit等级，降低carrot等级
select note_text from productnotes where match(note_text) against('>rabbit <carrot' in boolean mode);
```
```
# 匹配safe和combination，降低后者等级
select note_text from productnotes where match(note_text) against('+safe +(<combination)' in boolean mode);
```


**全文本搜索注意点**：

- 在索引全文本数据时，短词被忽略且动索引中排除，短词定义为3个或以下字符的词（可改）
- mysql有一个内建的非用词列表(stopword)，在索引时也被忽略，如有需要可以覆盖这个列表，参考mysql文档。
- 如果一个词出现在50%以上的行中，则被作为非用词忽略。但此规则不用于布尔搜索。
- 表中的行数少于三行，则全文本搜索不返回结果。
- 词中的单引号会被忽略，如don't索引为dont。
- 没有词分隔符的语言（包括汉语）不能恰当地返回全文本搜索结果。
- 不支持邻近操作符





## 视图

视图是虚拟的表，只包含使用时动态检索数据的查询。

比如，要查询订购了某个产品的用户:

```
select cust_name, cust_contact 
from customers, orders, orderitems 
where customers.cust_id = orders.cust_id 
    and orderitems.order_num = orders.order_num 
    and prod_id = 'tnt2';
```

可以把查询包装成一个名为productcustomers的视图：

```
select cust_name, cust_contact
from productcustomers
where prod_id = 'tnt2';
```

productcustomers是一个视图，不包含任何列或数据，而是包含一个SQL查询。

视图的好处与特点：

- 重用SQL语句。
- 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道它的基本查询细节。
- 使用表的组成部分而不是整个表。
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据
- 视图可以嵌套
- 对视图的order by会被视图内部的order by覆盖（如果有）
- 视图不能索引，也不能有关键的触发器或默认值

视图的使用方式与表基本相同，支持过滤、排序、联结等，甚至可以添加和更新数据（存在限制）。

每次使用视图时，该执行的查询一个也不会少，所以复杂或嵌套的视图可能会导致性能下降严重。

**对视图的操作**：

创建：

```
create view ....
```

查看视图：

```
show create view 视图名;
```

删除：

```
drop view 视图名;
```

更新：

```
方法1：先drop再create
方法2：create or replace view
```



**使用视图**：

视图中的where会与使用视图时的where自动组合。

常用于隐藏复杂的SQL，通常都是联结：

创建productcustomers视图，联结三个表，返回订购过产品的所有用户的列表：

```
create view productcustomers as
select cust_name, cust_contact, prod_id
from customers, orders, orderitems
where customers.cust_id = orders.cust_id
  and orderitems.order_num = orders.order_num;
```

列出所有订购过任意产品的客户：

```
select * from productcustomers;
```

检索订购了指定产品的用户：

```
select cust_name, cust_contact
from productcustomers
where prod_id = 'tnt2';
```

在MySQL处理此查询时，它将指定的WHERE子句添加到视图查询中的已有WHERE子句中，以便正确过滤数据。



使用视图简化格式化操作：

如果想格式化数据：

```
select concat(rtrim(vend_name), '(', rtrim(vend_country), ')') 
	as vend_title 
from vendors 
order by vend_name;
```

转化为视图：

```
create view vendorlocations as
select concat(rtrim(vend_name), '(', rtrim(vend_country), ')') 
	as vend_title 
from vendors 
order by vend_name;
```

```
select * from vendorlocations;
```



使用视图过滤数据：

过滤掉email为空的用户：

```
create view customeremaillist as
select cust_id, cust_name, cust_email
from customers
where cust_email is not NULL;
```

```
select * from customeremaillist;
```



使用视图简化计算字段：

```
select prod_id, quantity, item_price, quantity * item_price 
	as expanded_price 
from orderitems 
where order_num = 20005;
```

使用视图：

```
create view orderitemsexpanded as
select order_num, prod_id, quantity, item_price, quantity * item_price 
	as expanded_price 
from orderitems 
where order_num = 20005;
```

```
select * from orderitemsexpanded where order_num = 20005;s
```



更新视图中的数据：

能否更新，视情况而定。对视图的增加行、删除行操作实际就是对基表的操作，如果MySQL不能正确地确定被更新的基数据，则不允许更新（包括插入删除），比如视图定义中有以下操作则不能更新：

- 分组（使用GROUP BY和HAVING）
- 联结；
- 子查询；
- 并；
- 聚集函数（Min()、Count()、Sum()等）
- DISTINCT；
- 导出（计算）列。

听起来有很多限制，但视图主要用于检索数据，更新操作并不多。







## 存储过程

存储过程简单来说，就是为以后的使用二保存的一条或多条sql语句的集合。可以看作是批文件，不过他们的作用不仅限于批处理。

（相当于函数）

存储过程的好处：

- 简化复杂的操作。

- 不要求反复建立一系列处理步骤，保证了数据的完整性。如果所有开发人员和应用程序都使用同一（试验和测试）存储过程，则所使用的代码都是相同的。防止错误，保证了数据的一致性。

- 简化对变动的管理。如果表名、列名或业务逻辑等有变化，只需要更改存储过程。这一点的延伸就是安全性。通过存储过程限制对基础数据的访问减少了数据讹误的机会。

- 提高性能。因为使用存储过程比使用单独的SQL语句要快。

- 有些只能用在单个请求中的MySQL元素和特性，存储过程可以使用它们来编写功能更强更灵活的代码。

使用存储过程有3个主要的好处，即简单、安全、高性能。但也有缺点：

- 一般存储过程的编写比基本SQL语句复杂。
- 你可能没有创建存储过程的安全访问权限。许多数据库管理员限制存储过程的创建权限，允许用户使用存储过程，但不允许他们创建存储过程

**创建存储过程**：

```
CREATE PROCEDURE productpricing()
BEGIN
	select avg(prod_price) as priceaverage
	from products;
END;
```

---

注意，如果用的是mysql命令行，那么存储过程体中的语句分隔符 ";" 会被认为是创建语句的结束，导致报错，解决方法是使用 `delimiter` 语句临时更改语句分隔符：

```
delimiter //
CREATE PROCEDURE productpricing()
BEGIN
	select avg(prod_price) as priceaverage
	from products;
END //
delimiter ;
```

除了 '\\' 之外的任何字符都可以作为语句分隔符。



**查看存储过程**：

某个存储过程的信息：

```
show create procedure ordertotal;
```

存储过程列表：

```
show procedure status;
```

过滤结果：

```
show procedure status like 'ordertotal';
```





**调用存储过程**：

```
call productpricing();
```



**删除存储过程**：

```
drop procedure [if exists] productpricing;
```

注意没有括号了。

加入 if exists可以在不存在的情况下删除时不报错



**传参**：

```
delimiter //
CREATE PROCEDURE productpricing(
	OUT pl DECIMAL(8,2), 
	OUT ph DECIMAL(8,2), 
	OUT pa DECIMAL(8,2)) 
BEGIN 
	select min(prod_price) 
	INTO pl 
	from products; 
	select max(prod_price) 
	INTO ph 
	from products; 
	select avg(prod_price) 
	INTO pa 
	from products; 
END //
delimiter ;
```

这里定义三个参数，pl、ph、pa，变量名后面是数据类型，OUT指定参数从函数中传出

- IN：传递给函数
- OUT：从函数传出
- INOUT：传入和传出

上面函数体的意思是select之后把结果存到变量里。

OUT：

在调用时给出3个变量名，返回的值存到变量里。变量名必须以@开头。

```
call productpricing(@pricelow, @pricehigh, @priceaverage);
```

查看变量值：

```
select @pricelow [, @pricehigh];
```

IN：

```
delimiter //
CREATE PROCEDURE ordertotal(
	IN onumber INT, 
	OUT ototal DECIMAL(8,2))
BEGIN
	select sum(item_price*quantity)
	from orderitems
	where order_num = onumber
	into ototal;
END //
delimiter ;
```

```
call ordertotal(20005, @total);
select @total;
```



**建立智能存储过程**：

假如，需要合计订单，但需要针对某些顾客增加营业税，那么需要：

获得合计-》把营业税有条件地添加到合计-》返回合计（带或不带税）。

```
delimiter //
-- Name: ordertotal
-- Parameters: onumber = order number
-- 		taxable = 0 if not taxable, 1 if taxable
-- 		ototal = order total variable

CREATE PROCEDURE ordertotal(
	IN onumber INT, 
	IN taxable BOOLEAN,
	OUT ototal DECIMAL(8,2)
) COMMENT 'Obtain order total, optinally adding tax'
BEGIN

	-- Declare variable for total
	DECLARE total DECIMAL(8,2);
	-- Declare tax percentage
	DECLARE taxrate INT DEFAULT 6;
	
	-- Get the order total
	select sum(item_price * quantity)
	from orderitems
	where order_num = onumber
	INTO total;
	
	-- Is this taxable?
	IF taxable THEN
		-- add taxable to the total
		select total + (total / 100 * taxrate) INTO total;
	END IF;
	
	-- save to out variable
	select total into ototal;
END //
delimiter ;
```

--是注释

COMMENT设置了函数的说明，将在show procedure status中显示

DECLARE用来声明变量，并可以设置初始值。

```
call ordertotal(20005, 0, @total);
select @total;
call ordertotal(20005, 1, @total);
select @total;
```







## 游标

游标是存储在MySQL服务器上的数据库查询的结果集，在存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。

（相当于每次指向一行数据的指针）

- MySQL的游标只能用于存储过程

使用游标的步骤：

先声明（此过程不检索数据，而是定义要使用的select语句）-》

打开游标（用前面定义的语句检索数据）-》

从游标中检索各行-》

关闭游标

**创建游标**：

```
delimiter //
CREATE PROCEDURE processorders()
BEGIN
	-- 声明游标
	DECLARE ordernumbers CURSOR
	FOR
	-- 表名，也可以是返回的集合
	select order_num from orders;
END //
delimiter ;
```

存储过程处理完成后，游标就消失（它局限于存储过程）

**打开与关闭游标**：

下面的语句是放在存储过程里的，不是单独执行的：

```
OPEN ordernumbers;
```

```
CLOSE ordernumbers;
```

如果不指定关闭，则在END语句会自动关闭

上面的函数就修改为：

```mysql
delimiter //
CREATE PROCEDURE processorders()
BEGIN
	-- 声明游标
	DECLARE ordernumbers CURSOR
	FOR
	-- 表名，也可以是返回的集合
	select order_num from orders;
	
	OPEN ordernumbers;
	CLOSE ordernumbers;
END //
delimiter ;
```

**使用游标数据**：

使用FETCH遍历每一行，并可以指定检索什么数据（所需的列），检索出来的数据存储在什么地方。并且会自动向前移动行指针。

（在存储过程中使用）：

检索第一行：

```mysql
DECLARE o INT;
...-- 创建和打开
FETCH ordernumbers INTO o;
```

检索每一行：

```
DECLARE done BOOLEAN DEFAULT 0;
DECLARE o INT;
...-- 创建
DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
...-- 打开
REPEAT
	FETCH ordernumbers INTO o;
UNTIL done END REPEAT;
CLOSE ordernumbers;
```

以下是MySQL支持的循环语句

```mysql
WHILE……DO……END WHILE
REPEAT……UNTIL END REPEAT
LOOP……END LOOP
GOTO
```

CONTINUE HANDLER 是在条件出现时被执行的代码，这里指定当出现 `SQLSTATE '02000'` 时， `SET done = 1` 。SQLSTATE '02000' 是一个错误代码，在REPEAT没有更多的行供循环时就会出现 SQLSTATE '02000' 的错误代码。

关于错误代码详见https://dev.mysql.com/doc/refman/8.0/en/error-handling.html

---

用DECLARE语句定义的局部变量必须在游标之前，句柄必须定义在游标之后

---

简单示例：


```
delimiter //
CREATE PROCEDURE processorders()
BEGIN
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	DECLARE t DECIMAL(8,2);
	
	-- 声明游标
	DECLARE ordernumbers CURSOR
	FOR
	select order_num from orders;	-- 表名，也可以是返回的集合
	
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
	
	CREATE TABLE IF NOT EXISTS ordertotals(
		order_num INT, total DECIMAL(8,2));
	
	OPEN ordernumbers;
	
	REPEAT
		FETCH ordernumbers INTO o;
		CALL ordertotal(o, 1, t);
		INSERT INTO ordertotals(order_num, total) VALUES(o, t);
	UNTIL done END REPEAT;
	
	CLOSE ordernumbers;
END //
delimiter ;
```

调用另一个函数，将结果放到新创建的ordertotals表中。

```
CALL processorders();
select * from ordertotals;
```





## 触发器

类似于监听

我们会在需要时执行语句和函数，但如果想要语句在时间发生时自动执行呢？比如：

每增加一个顾客到数据库时，都检查电话格式、地址格式（这些其实可以由前端完成，这里仅做例子）

每订购一个产品，都从库存中减去订购数量

删除行时，在存档表中保存副本

这些可以在某个表发生更改时自动处理（执行语句或存储过程，但不能调用CALL），就是使用触发器。

- 触发器只响应以下语句：
  - DELETE
  - INSERT
  - UPDATE
- 每个表每个事件只允许一个触发器，所以一个表最多有6个触发器（每个动作的之前和之后）。
- 只有表支持触发器，视图和临时表都不支持。
- 触发器和事件只能一一对应。
- 如果触发器或语句执行出错，则后面的语句和触发器都不再执行
- 创建触发器需要权限，但对应语句能执行，那触发器就能执行

**创建触发器**：

创建触发器时，给出：唯一的触发器名、要关联的表、要响应的语句、何时执行（之前或之后）。

触发器在表中唯一，数据库中的不同表可以相同（其他DBMS不允许，所以最好是数据库内唯一）。

```
CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'Product added' into @insertinfo;
```

创建了一个名为newproduct的触发器，在products表执行插入操作之后触发，FOR EACH ROW表示对每个插入行执行，操作为 SELECT 'Product added' ，也就是输出'Product added'；

不加后面的 into @insertinfo 会报错，因为MySQL已经不支持使用触发器返回结果集了，所以把它放在一个变量中。

测试：

```
insert into products(prod_id, vend_id, prod_name, prod_price, prod_desc) values('test', 1001, 'test', 11.11, 'test');
select @insertinfo;
```



**使用触发器**：

INSERT触发器：

在INSERT触发器中，可以引用一个名为NEW的虚拟表，表示被插入的行，在BEFORE INSERT触发器中，NEW的值可以更改（插入时使用更改后的值），对于自增列，NEW在INSERT执行前（比如BEFORE阶段）值为0，INSERT执行后（AFTER阶段）值为自动生成的值。

BEFORE通常用于数据验证和净化。

比如要获取自增的值：

```
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW SELECT NEW.order_num INTO @last_num;
```

触发器在插入之后从NEW.order_num取得值并保存到变量中。

测试：

```
INSERT INTO orders(order_date, cust_id) values(Now(), 10001);
select @last_num;
```



DELETE触发器：

DELETE触发器可以引用一个名为OLD的虚拟表，访问被删除的行，但OLD的值是只读的，不能更新。

比如备份删除的行：

```
delimiter //
CREATE TRIGGER deleteorder BEFORE DELETE ON orders
FOR EACH ROW
BEGIN
	INSERT INTO archive_orders(order_num, order_date, cust_id)
	VALUES(OLD.order_num, OLD.order_date, OLD.cust_id);
END //
delimiter ;
```

如果不能备份，则会删除失败。



**UPDATE触发器**：

UPDATE触发器可以引用OLD访问更新前的值，也可以引用NEW访问新值。可以在BEFOFE UPDATE 中更改要插入的值（更改NEW），OLD只读。

比如要保证州名大写：

```
CREATE TRIGGER updatevendor BEFORE UPDATE ON vendors
FOR EACH ROW SET NEW.vend_state = Upper(NEW.vend_state);
```



**删除触发器**：

```
DROP TRIGGER newproduct;
```

触发器不能覆盖，要修改只能先删除再重新创建。







## 事务

使用InnoDB引擎来进行事务管理

事务保证SQL操作成功就提交，失败就回滚或恢复到保留点

事务处理不支持CREATE和DROP回退，可以在事务中使用，但操作不能回滚。

**开始事务**

标识事务的开始：

```
START TRANSACTION;
```

回滚：

```
select * from ordertotals;
start transaction;
delete from ordertotals;
select * from ordertotals;
ROLLBACK;
select * from ordertotals;
```

上面的语句，先查看表，表非空，然后开始事务，清空表，查看表，空了，回滚事务，查看表，非空，说明事务里的语句成功回滚。



**提交事务**：

执行完COMMIT或ROLLBACK之后，事务会自动关闭，之后的语句不再属于事务。

在非事务中，语句是隐含提交的，即提交（写或保存）操作是自动进行的。但在事务中，需要使用 `commit` 语句手动提交。

```
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20010;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;
```

如果语句执行出错则自动回滚。



**使用保留点**：

如果不想回滚整个事务，可以使用 `SAVEPOINT` 语句添加一个保留点

```
SAVEPOINT delete1;
ROLLBACK TO delete1;
```

保留点在事务关闭之后会自动释放，也可以使用 `RELEASE SAVE POINT` 明确地释放保留点。



**不自动提交**：

如果不想使用事务，还不想让语句自动提交，可以设置

```
SET autocommit = 0;
```

这样即使使用了 COMMIT 语句也不会提交，知道autocommit设置为真。

autocommit针对每个连接而不是服务器。







## 全球化和本地化

重要术语：

- 字符集
- 编码
- 校对（就是字符如何比较）

查看MySQL支持的字符集信息：

```
SHOW CHARACTER SET;
```

查看校对列表及适用的字符集：

```
SHOW COLLATION;
```

有的字符集不止一种校对，并且有些校对出现两次，一种区分大小写（由_cs表示），一种不区分大小写（由\_ci表示）。

通常在安装时定义一个默认的字符集和校对，也可以在创建数据库、表、列等时指定默认的字符集和校对。

查看字符集和校对：

```
SHOW VARIABLES LIKE 'character%';
SHOW VARIBALES LIKE 'collation%';
```

为表指定字符集和校对：

```
CREATE TABLE mytable(
	columnn1 INT,
	columnn2 VARCHAR(10)
) DEFAULT CHARACTER SET hebrew
  COLLATE hebrew_general_ci;
```

为列指定字符集和校对：

```
CREATE TABLE mytable(
	columnn1 INT,
	columnn2 VARCHAR(10),
	column3 VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_general_ci
) DEFAULT CHARACTER SET hebrew
  COLLATE hebrew_general_ci;
```

不指定则使用数据库默认的字符集和校对。

如果想使用临时指定的字符集排序，可以在SELECT中进行：

```
SELECT * FROM customers
ORDER BY lastname, firstname COLLATE latin1_general_cs;
```

COLLATE还可以用于 GROUP BY, HAVING, 聚集函数，别名等。

如果绝对需要，串还可以转换字符集，使用 Cast()或Convert()函数。









## 访问控制

用户应该对他们需要的数据具有适当的访问权，既不能多也不能少。换句话说，用户不能对过多的数据具有过多的访问权。

比如，名为mysql的数据库一般不需要访问，里面都是MySQL系统文件

```
use mysql;
select user from user;
```

上面的语句可以查看所有的用户账号。



### 创建用户

```
CREATE USER  ben IDENTIFIED BY 'mima';
```

创建用户并设置密码。密码在保存到user表之前会被加密。如果想加密密码，可以参考 https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html#function_password 的各种函数。

```
select md5('passwd123456');
```

使用GRANT语句也可以创建用户，还可以直接向user表插入行，不过并不建议这样做。

### 更改密码

更改自己的密码

```
set password = md5('123456');
```

更改其他用户密码

```
SET PASSWORD FOR bforta = md5('123456');
```



### 重命名用户

```
RENAME USER ben TO bforta;
```

### 删除用户

```
DROP USER bforta;
```

在MySQL5之前，需要先用 REVOKE 删除与账号相关的权限，再删除账号。

### 设置访问权限

创建用户之后需要分配权限，新用户没有任何访问权限，他们可以登录MySQL，但不能执行任何数据库操作。

**查看权限**：

```
show grants for bforta;
```

USAGE表示没有任何权限。

权限用 user@host 格式定义，如果不指定host，则默认使用%

**设置权限**：

```
GRANT 权限[, 权限] ON 数据库或表 TO 用户名;
```

```
GRANT SELECT ON crashcourse.* TO bforta;
```

**撤销权限**：

```
REVOKE SELECT ON crashcourse.* FROM bforta;
```

被撤销的权限不存在的话会报错。

**权限作用范围**：

- 整个服务器：GRANT ALL
- 整个数据库：ON database.*
- 特定表： ON database.table;
- 特定列
- 特定存储过程

**权限列表**：

| 权 限  | 说 明 |
| ---------------------- | -------------- |
| ALL  | 除GRANT OPTION外的所有权限 |
| ALTER  | 使用ALTER TABLE |
| ALTER ROUTINE  | 使用ALTER PROCEDURE和DROP PROCEDURE |
| CREATE  | 使用CREATE TABLE |
| CREATE ROUTINE  | 使用CREATE PROCEDURE |
| CREATE TEMPORARY TABLES  | 使用CREATE TEMPORARY TABLE |
| CREATE USER  | 使用CREATE USER、DROP USER、RENAME USER和REVOKE ALL PRIVILEGES |
| CREATE VIEW  | 使用CREATE VIEW |
| DELETE  | 使用DELETE |
| DROP  | 使用DROP TABLE |
| EXECUTE  | 使用CALL和存储过程 |
| FILE  | 使用SELECT INTO OUTFILE和LOAD DATA INFILE |
| GRANT OPTION  | 使用GRANT和REVOKE |
| INDEX  | 使用CREATE INDEX和DROP INDEX |
| INSERT  | 使用INSERT |
| LOCK TABLES  | 使用LOCK TABLES |
| PROCESS  | 使用SHOW FULL PROCESSLIST |
| RELOAD  | 使用FLUSH |
| REPLICATION CLIENT  | 服务器位置的访问 |
| REPLICATION SLAVE | 由复制从属使用 |
| SELECT  | 使用SELECT |
| SHOW DATABASES  | 使用SHOW DATABASES |
| SHOW VIEW  | 使用SHOW CREATE VIEW |
| SHUTDOWN  | 使用mysqladmin shutdown（用来关闭MySQL） |
| SUPER  | 使用CHANGE MASTER、KILL、LOGS、PURGE、MASTER 和SET GLOBAL。还允许mysqladmin调试登录 |
| UPDATE  | 使用UPDATE |
| USAGE  | 无访问权限 |

可以一次授予多个权限

```
grant select, insert on crashcourse.* to bforta;
```

**一直存在的权限**：

注意，在使用grant和revoke时，用户必须存在，但操作的数据库和表等可以不存在，这样可以在创建数据库和表之前设置权限。

这样的副作用是，当某个数据库或表被删除时，权限依然存在，并且如果将来重新创建该数据库或表，权限依然起作用。







## 数据库维护

MySQL数据库是基于磁盘的文件，普通的备份系统和例程就能备份MySQL的数据，但是，由于这些文件总是处于打开和使用状态，普通的文件副本备份不一定总是有效。

**备份方法**：

- 使用命令行程序mysqldump转储数据库内容到外部文件

- 使用mysqlhotcopy从数据库复制所有数据（并非所有引擎都支持）

- 使用自带的 backup table 或 select into outfile 转储到外部文件，文件必须已存在，使用restore table 来还原。

在将数据写入到磁盘时，最好先刷新一下未写数据：

```
flush tables;
```



**检测数据库**：

检测表键是否正确

```
analyze table 表名[, 表名];
```

```
check table 表名[, 表名];
```

check table还有一系列用于MyISAM表的方式：CHANGED检查自最后一次检查以来改动过的表。EXTENDED执行最彻底的检查，FAST只检查未正常关闭的表，MEDIUM检查所有被删除的链接并进行键检验，QUICK只进行快速扫描。



**修复表**：

```
repair table 表名[, 表名];
```

这条语句不应该经常使用，如果需要经常使用，意味着数据库可能存在问题。

如果从一个表中删除大量数据，应该使用 OPTIMIZE TABLE 来回收所用的空间，优化性能。



**诊断启动问题**：

诊断启动问题时，应尽量手动启动服务器（执行mysqld），下面是几个重要的mysqld选项：

- --help 显示帮助
- --safe-mode 装在减去某些最佳配置的服务器
- --verbose 显示全文本消息（为获得更详细的帮助消息与--help联合使用）
- --version显示版本信息然后退出



**查看日志**：

- 错误日志：baohh按启动和关闭问题以及任意关键错误的细节，通常为data/hostname.err，日志名可通过 --log-error选项更改
- 查询日志：记录所有mysql活动，通常为data/hostname.log，可以使用 --log进行更改
- 二进制日志：记录更新过（或可能更新过）数据的所有语句，通常为data/hostname-bin，可以使用 --log-bin 进行更改
- 缓慢查询日志：记录执行缓慢的查询，可以用来确定哪里需要优化，通常为data/hostname-slow.log，可以使用 --log-slow-queries 选项进行更改

在查看日志前，可以使用 `flush logs` 来刷新和重新开始所有日志文件（关闭当前日志文件，打开一个新的日志文件，文件序号加1，旧的不删除）







## 改善性能

下面的内容并不能完全决定MySQL的性能。我们只是想回顾一下前面各章的重点，提供进行性能优化探讨和分析的一个出发点。

- MySQL有特定的硬件建议，用于生产的服务器应坚持遵循这些硬件建议
- 关键的生产DBMS应该运行在专用服务器上
- 可能需要调整内存分配、缓冲区大小等。可使用 show variables 和show status 查看设置
- MySQL是多用户多线程的，如果其中一个执行缓慢，则所有请求都会执行缓慢，如果遇到显著的性能不良，可以使用 show processlist 显示所有活动进程，使用 kill 来结束进程（管理员）
- 尝试多种select方法，比如联结、并、子查询等，找出最佳方法
- 使用 explain 语句让MySQL解释它如何执行一条select语句
- 一般存储过程执行得比多个单条语句快
- 应该总是使用正确的数据类型
- 不要检索比需求还要多的数据，换言之，不要使用select *，除非真正需要每个列
- 有的操作（包括INSERT）支持delayed关键字，可以让客户端无需等待操作执行完成而直接返回，语句会被排入队列，当表没有其他线程使用时执行。多个客户端也会进入一个队列一起执行。
- 导入数据时，应该关闭自动提交，如果还想删除索引（包括fulltext索引），应该在导入完成后再重建它们。
- 索引是加快查询速度的重要方式，确定要索引什么很重要，需要分析使用的select语句以找出重复的where和order by子句。如果一个简单的where子句返回结果很长，则其中使用的列（或几个列）就是需要索引的对象。
- 如果select中有复杂的 or 条件，可以使用多条select并使用union连接，可以极大改善性能。
- 索引改善检索数据的性能，但损害更新数据的功能，如果有表不经常被检索但频繁更新，则有必要删除索引
- like 很慢，最好使用 fulltext 而不是 like
- 由于表的使用和内容的更改，理想的优化和配置也会改变
- 每条规则在某些条件下都会被打破







## 保留字

 https://dev.mysql.com/doc/refman/8.0/en/keywords.html 

- `ACCESSIBLE` (R)
- `ACCOUNT`
- `ACTION`
- `ACTIVE`; added in 8.0.14 (nonreserved)
- `ADD` (R)
- `ADMIN`; became nonreserved in 8.0.12
- `AFTER`
- `AGAINST`
- `AGGREGATE`
- `ALGORITHM`
- `ALL` (R)
- `ALTER` (R)
- `ALWAYS`
- `ANALYSE`; removed in 8.0.1
- `ANALYZE` (R)
- `AND` (R)
- `ANY`
- `ARRAY`; added in 8.0.17 (reserved); became nonreserved in 8.0.19
- `AS` (R)
- `ASC` (R)
- `ASCII`
- `ASENSITIVE` (R)
- `AT`
- `AUTOEXTEND_SIZE`
- `AUTO_INCREMENT`
- `AVG`
- `AVG_ROW_LENGTH`

B

- `BACKUP`
- `BEFORE` (R)
- `BEGIN`
- `BETWEEN` (R)
- `BIGINT` (R)
- `BINARY` (R)
- `BINLOG`
- `BIT`
- `BLOB` (R)
- `BLOCK`
- `BOOL`
- `BOOLEAN`
- `BOTH` (R)
- `BTREE`
- `BUCKETS`; added in 8.0.2 (nonreserved)
- `BY` (R)
- `BYTE`

C

- `CACHE`
- `CALL` (R)
- `CASCADE` (R)
- `CASCADED`
- `CASE` (R)
- `CATALOG_NAME`
- `CHAIN`
- `CHANGE` (R)
- `CHANGED`
- `CHANNEL`
- `CHAR` (R)
- `CHARACTER` (R)
- `CHARSET`
- `CHECK` (R)
- `CHECKSUM`
- `CIPHER`
- `CLASS_ORIGIN`
- `CLIENT`
- `CLONE`; added in 8.0.3 (nonreserved)
- `CLOSE`
- `COALESCE`
- `CODE`
- `COLLATE` (R)
- `COLLATION`
- `COLUMN` (R)
- `COLUMNS`
- `COLUMN_FORMAT`
- `COLUMN_NAME`
- `COMMENT`
- `COMMIT`
- `COMMITTED`
- `COMPACT`
- `COMPLETION`
- `COMPONENT`
- `COMPRESSED`
- `COMPRESSION`
- `CONCURRENT`
- `CONDITION` (R)
- `CONNECTION`
- `CONSISTENT`
- `CONSTRAINT` (R)
- `CONSTRAINT_CATALOG`
- `CONSTRAINT_NAME`
- `CONSTRAINT_SCHEMA`
- `CONTAINS`
- `CONTEXT`
- `CONTINUE` (R)
- `CONVERT` (R)
- `CPU`
- `CREATE` (R)
- `CROSS` (R)
- `CUBE` (R); became reserved in 8.0.1
- `CUME_DIST` (R); added in 8.0.2 (reserved)
- `CURRENT`
- `CURRENT_DATE` (R)
- `CURRENT_TIME` (R)
- `CURRENT_TIMESTAMP` (R)
- `CURRENT_USER` (R)
- `CURSOR` (R)
- `CURSOR_NAME`

D

- `DATA`
- `DATABASE` (R)
- `DATABASES` (R)
- `DATAFILE`
- `DATE`
- `DATETIME`
- `DAY`
- `DAY_HOUR` (R)
- `DAY_MICROSECOND` (R)
- `DAY_MINUTE` (R)
- `DAY_SECOND` (R)
- `DEALLOCATE`
- `DEC` (R)
- `DECIMAL` (R)
- `DECLARE` (R)
- `DEFAULT` (R)
- `DEFAULT_AUTH`
- `DEFINER`
- `DEFINITION`; added in 8.0.4 (nonreserved)
- `DELAYED` (R)
- `DELAY_KEY_WRITE`
- `DELETE` (R)
- `DENSE_RANK` (R); added in 8.0.2 (reserved)
- `DESC` (R)
- `DESCRIBE` (R)
- `DESCRIPTION`; added in 8.0.4 (nonreserved)
- `DES_KEY_FILE`; removed in 8.0.3
- `DETERMINISTIC` (R)
- `DIAGNOSTICS`
- `DIRECTORY`
- `DISABLE`
- `DISCARD`
- `DISK`
- `DISTINCT` (R)
- `DISTINCTROW` (R)
- `DIV` (R)
- `DO`
- `DOUBLE` (R)
- `DROP` (R)
- `DUAL` (R)
- `DUMPFILE`
- `DUPLICATE`
- `DYNAMIC`

E

- `EACH` (R)
- `ELSE` (R)
- `ELSEIF` (R)
- `EMPTY` (R); added in 8.0.4 (reserved)
- `ENABLE`
- `ENCLOSED` (R)
- `ENCRYPTION`
- `END`
- `ENDS`
- `ENFORCED`; added in 8.0.16 (nonreserved)
- `ENGINE`
- `ENGINES`
- `ENUM`
- `ERROR`
- `ERRORS`
- `ESCAPE`
- `ESCAPED` (R)
- `EVENT`
- `EVENTS`
- `EVERY`
- `EXCEPT` (R)
- `EXCHANGE`
- `EXCLUDE`; added in 8.0.2 (nonreserved)
- `EXECUTE`
- `EXISTS` (R)
- `EXIT` (R)
- `EXPANSION`
- `EXPIRE`
- `EXPLAIN` (R)
- `EXPORT`
- `EXTENDED`
- `EXTENT_SIZE`

F

- `FAILED_LOGIN_ATTEMPTS`; added in 8.0.19 (nonreserved)
- `FALSE` (R)
- `FAST`
- `FAULTS`
- `FETCH` (R)
- `FIELDS`
- `FILE`
- `FILE_BLOCK_SIZE`
- `FILTER`
- `FIRST`
- `FIRST_VALUE` (R); added in 8.0.2 (reserved)
- `FIXED`
- `FLOAT` (R)
- `FLOAT4` (R)
- `FLOAT8` (R)
- `FLUSH`
- `FOLLOWING`; added in 8.0.2 (nonreserved)
- `FOLLOWS`
- `FOR` (R)
- `FORCE` (R)
- `FOREIGN` (R)
- `FORMAT`
- `FOUND`
- `FROM` (R)
- `FULL`
- `FULLTEXT` (R)
- `FUNCTION` (R); became reserved in 8.0.1

G

- `GENERAL`
- `GENERATED` (R)
- `GEOMCOLLECTION`; added in 8.0.11 (nonreserved)
- `GEOMETRY`
- `GEOMETRYCOLLECTION`
- `GET` (R)
- `GET_FORMAT`
- `GET_MASTER_PUBLIC_KEY`; added in 8.0.4 (reserved); became nonreserved in 8.0.11
- `GLOBAL`
- `GRANT` (R)
- `GRANTS`
- `GROUP` (R)
- `GROUPING` (R); added in 8.0.1 (reserved)
- `GROUPS` (R); added in 8.0.2 (reserved)
- `GROUP_REPLICATION`

H

- `HANDLER`
- `HASH`
- `HAVING` (R)
- `HELP`
- `HIGH_PRIORITY` (R)
- `HISTOGRAM`; added in 8.0.2 (nonreserved)
- `HISTORY`; added in 8.0.3 (nonreserved)
- `HOST`
- `HOSTS`
- `HOUR`
- `HOUR_MICROSECOND` (R)
- `HOUR_MINUTE` (R)
- `HOUR_SECOND` (R)

I

- `IDENTIFIED`
- `IF` (R)
- `IGNORE` (R)
- `IGNORE_SERVER_IDS`
- `IMPORT`
- `IN` (R)
- `INACTIVE`; added in 8.0.14 (nonreserved)
- `INDEX` (R)
- `INDEXES`
- `INFILE` (R)
- `INITIAL_SIZE`
- `INNER` (R)
- `INOUT` (R)
- `INSENSITIVE` (R)
- `INSERT` (R)
- `INSERT_METHOD`
- `INSTALL`
- `INSTANCE`
- `INT` (R)
- `INT1` (R)
- `INT2` (R)
- `INT3` (R)
- `INT4` (R)
- `INT8` (R)
- `INTEGER` (R)
- `INTERVAL` (R)
- `INTO` (R)
- `INVISIBLE`
- `INVOKER`
- `IO`
- `IO_AFTER_GTIDS` (R)
- `IO_BEFORE_GTIDS` (R)
- `IO_THREAD`
- `IPC`
- `IS` (R)
- `ISOLATION`
- `ISSUER`
- `ITERATE` (R)

J

- `JOIN` (R)
- `JSON`
- `JSON_TABLE` (R); added in 8.0.4 (reserved)

K

- `KEY` (R)
- `KEYS` (R)
- `KEY_BLOCK_SIZE`
- `KILL` (R)

L

- `LAG` (R); added in 8.0.2 (reserved)
- `LANGUAGE`
- `LAST`
- `LAST_VALUE` (R); added in 8.0.2 (reserved)
- `LATERAL` (R); added in 8.0.14 (reserved)
- `LEAD` (R); added in 8.0.2 (reserved)
- `LEADING` (R)
- `LEAVE` (R)
- `LEAVES`
- `LEFT` (R)
- `LESS`
- `LEVEL`
- `LIKE` (R)
- `LIMIT` (R)
- `LINEAR` (R)
- `LINES` (R)
- `LINESTRING`
- `LIST`
- `LOAD` (R)
- `LOCAL`
- `LOCALTIME` (R)
- `LOCALTIMESTAMP` (R)
- `LOCK` (R)
- `LOCKED`; added in 8.0.1 (nonreserved)
- `LOCKS`
- `LOGFILE`
- `LOGS`
- `LONG` (R)
- `LONGBLOB` (R)
- `LONGTEXT` (R)
- `LOOP` (R)
- `LOW_PRIORITY` (R)

M

- `MASTER`
- `MASTER_AUTO_POSITION`
- `MASTER_BIND` (R)
- `MASTER_COMPRESSION_ALGORITHMS`; added in 8.0.18 (nonreserved)
- `MASTER_CONNECT_RETRY`
- `MASTER_DELAY`
- `MASTER_HEARTBEAT_PERIOD`
- `MASTER_HOST`
- `MASTER_LOG_FILE`
- `MASTER_LOG_POS`
- `MASTER_PASSWORD`
- `MASTER_PORT`
- `MASTER_PUBLIC_KEY_PATH`; added in 8.0.4 (nonreserved)
- `MASTER_RETRY_COUNT`
- `MASTER_SERVER_ID`
- `MASTER_SSL`
- `MASTER_SSL_CA`
- `MASTER_SSL_CAPATH`
- `MASTER_SSL_CERT`
- `MASTER_SSL_CIPHER`
- `MASTER_SSL_CRL`
- `MASTER_SSL_CRLPATH`
- `MASTER_SSL_KEY`
- `MASTER_SSL_VERIFY_SERVER_CERT` (R)
- `MASTER_TLS_CIPHERSUITES`; added in 8.0.19 (nonreserved)
- `MASTER_TLS_VERSION`
- `MASTER_USER`
- `MASTER_ZSTD_COMPRESSION_LEVEL`; added in 8.0.18 (nonreserved)
- `MATCH` (R)
- `MAXVALUE` (R)
- `MAX_CONNECTIONS_PER_HOUR`
- `MAX_QUERIES_PER_HOUR`
- `MAX_ROWS`
- `MAX_SIZE`
- `MAX_UPDATES_PER_HOUR`
- `MAX_USER_CONNECTIONS`
- `MEDIUM`
- `MEDIUMBLOB` (R)
- `MEDIUMINT` (R)
- `MEDIUMTEXT` (R)
- `MEMBER`; added in 8.0.17 (reserved); became nonreserved in 8.0.19
- `MEMORY`
- `MERGE`
- `MESSAGE_TEXT`
- `MICROSECOND`
- `MIDDLEINT` (R)
- `MIGRATE`
- `MINUTE`
- `MINUTE_MICROSECOND` (R)
- `MINUTE_SECOND` (R)
- `MIN_ROWS`
- `MOD` (R)
- `MODE`
- `MODIFIES` (R)
- `MODIFY`
- `MONTH`
- `MULTILINESTRING`
- `MULTIPOINT`
- `MULTIPOLYGON`
- `MUTEX`
- `MYSQL_ERRNO`

N

- `NAME`
- `NAMES`
- `NATIONAL`
- `NATURAL` (R)
- `NCHAR`
- `NDB`
- `NDBCLUSTER`
- `NESTED`; added in 8.0.4 (nonreserved)
- `NETWORK_NAMESPACE`; added in 8.0.16 (nonreserved)
- `NEVER`
- `NEW`
- `NEXT`
- `NO`
- `NODEGROUP`
- `NONE`
- `NOT` (R)
- `NOWAIT`; added in 8.0.1 (nonreserved)
- `NO_WAIT`
- `NO_WRITE_TO_BINLOG` (R)
- `NTH_VALUE` (R); added in 8.0.2 (reserved)
- `NTILE` (R); added in 8.0.2 (reserved)
- `NULL` (R)
- `NULLS`; added in 8.0.2 (nonreserved)
- `NUMBER`
- `NUMERIC` (R)
- `NVARCHAR`

O

- `OF` (R); added in 8.0.1 (reserved)
- `OFFSET`
- `OJ`; added in 8.0.16 (nonreserved)
- `OLD`; added in 8.0.14 (nonreserved)
- `ON` (R)
- `ONE`
- `ONLY`
- `OPEN`
- `OPTIMIZE` (R)
- `OPTIMIZER_COSTS` (R)
- `OPTION` (R)
- `OPTIONAL`; added in 8.0.13 (nonreserved)
- `OPTIONALLY` (R)
- `OPTIONS`
- `OR` (R)
- `ORDER` (R)
- `ORDINALITY`; added in 8.0.4 (nonreserved)
- `ORGANIZATION`; added in 8.0.4 (nonreserved)
- `OTHERS`; added in 8.0.2 (nonreserved)
- `OUT` (R)
- `OUTER` (R)
- `OUTFILE` (R)
- `OVER` (R); added in 8.0.2 (reserved)
- `OWNER`

P

- `PACK_KEYS`
- `PAGE`
- `PARSER`
- `PARTIAL`
- `PARTITION` (R)
- `PARTITIONING`
- `PARTITIONS`
- `PASSWORD`
- `PASSWORD_LOCK_TIME`; added in 8.0.19 (nonreserved)
- `PATH`; added in 8.0.4 (nonreserved)
- `PERCENT_RANK` (R); added in 8.0.2 (reserved)
- `PERSIST`; became nonreserved in 8.0.16
- `PERSIST_ONLY`; added in 8.0.2 (reserved); became nonreserved in 8.0.16
- `PHASE`
- `PLUGIN`
- `PLUGINS`
- `PLUGIN_DIR`
- `POINT`
- `POLYGON`
- `PORT`
- `PRECEDES`
- `PRECEDING`; added in 8.0.2 (nonreserved)
- `PRECISION` (R)
- `PREPARE`
- `PRESERVE`
- `PREV`
- `PRIMARY` (R)
- `PRIVILEGES`
- `PRIVILEGE_CHECKS_USER`; added in 8.0.18 (nonreserved)
- `PROCEDURE` (R)
- `PROCESS`; added in 8.0.11 (nonreserved)
- `PROCESSLIST`
- `PROFILE`
- `PROFILES`
- `PROXY`
- `PURGE` (R)

Q

- `QUARTER`
- `QUERY`
- `QUICK`

R

- `RANDOM`; added in 8.0.18 (nonreserved)
- `RANGE` (R)
- `RANK` (R); added in 8.0.2 (reserved)
- `READ` (R)
- `READS` (R)
- `READ_ONLY`
- `READ_WRITE` (R)
- `REAL` (R)
- `REBUILD`
- `RECOVER`
- `RECURSIVE` (R); added in 8.0.1 (reserved)
- `REDOFILE`; removed in 8.0.3
- `REDO_BUFFER_SIZE`
- `REDUNDANT`
- `REFERENCE`; added in 8.0.4 (nonreserved)
- `REFERENCES` (R)
- `REGEXP` (R)
- `RELAY`
- `RELAYLOG`
- `RELAY_LOG_FILE`
- `RELAY_LOG_POS`
- `RELAY_THREAD`
- `RELEASE` (R)
- `RELOAD`
- `REMOTE`; added in 8.0.3 (nonreserved); removed in 8.0.14
- `REMOVE`
- `RENAME` (R)
- `REORGANIZE`
- `REPAIR`
- `REPEAT` (R)
- `REPEATABLE`
- `REPLACE` (R)
- `REPLICATE_DO_DB`
- `REPLICATE_DO_TABLE`
- `REPLICATE_IGNORE_DB`
- `REPLICATE_IGNORE_TABLE`
- `REPLICATE_REWRITE_DB`
- `REPLICATE_WILD_DO_TABLE`
- `REPLICATE_WILD_IGNORE_TABLE`
- `REPLICATION`
- `REQUIRE` (R)
- `REQUIRE_ROW_FORMAT`; added in 8.0.19 (nonreserved)
- `RESET`
- `RESIGNAL` (R)
- `RESOURCE`; added in 8.0.3 (nonreserved)
- `RESPECT`; added in 8.0.2 (nonreserved)
- `RESTART`; added in 8.0.4 (nonreserved)
- `RESTORE`
- `RESTRICT` (R)
- `RESUME`
- `RETAIN`; added in 8.0.14 (nonreserved)
- `RETURN` (R)
- `RETURNED_SQLSTATE`
- `RETURNS`
- `REUSE`; added in 8.0.3 (nonreserved)
- `REVERSE`
- `REVOKE` (R)
- `RIGHT` (R)
- `RLIKE` (R)
- `ROLE`; became nonreserved in 8.0.1
- `ROLLBACK`
- `ROLLUP`
- `ROTATE`
- `ROUTINE`
- `ROW` (R); became reserved in 8.0.2
- `ROWS` (R); became reserved in 8.0.2
- `ROW_COUNT`
- `ROW_FORMAT`
- `ROW_NUMBER` (R); added in 8.0.2 (reserved)
- `RTREE`

S

- `SAVEPOINT`
- `SCHEDULE`
- `SCHEMA` (R)
- `SCHEMAS` (R)
- `SCHEMA_NAME`
- `SECOND`
- `SECONDARY`; added in 8.0.16 (nonreserved)
- `SECONDARY_ENGINE`; added in 8.0.13 (nonreserved)
- `SECONDARY_LOAD`; added in 8.0.13 (nonreserved)
- `SECONDARY_UNLOAD`; added in 8.0.13 (nonreserved)
- `SECOND_MICROSECOND` (R)
- `SECURITY`
- `SELECT` (R)
- `SENSITIVE` (R)
- `SEPARATOR` (R)
- `SERIAL`
- `SERIALIZABLE`
- `SERVER`
- `SESSION`
- `SET` (R)
- `SHARE`
- `SHOW` (R)
- `SHUTDOWN`
- `SIGNAL` (R)
- `SIGNED`
- `SIMPLE`
- `SKIP`; added in 8.0.1 (nonreserved)
- `SLAVE`
- `SLOW`
- `SMALLINT` (R)
- `SNAPSHOT`
- `SOCKET`
- `SOME`
- `SONAME`
- `SOUNDS`
- `SOURCE`
- `SPATIAL` (R)
- `SPECIFIC` (R)
- `SQL` (R)
- `SQLEXCEPTION` (R)
- `SQLSTATE` (R)
- `SQLWARNING` (R)
- `SQL_AFTER_GTIDS`
- `SQL_AFTER_MTS_GAPS`
- `SQL_BEFORE_GTIDS`
- `SQL_BIG_RESULT` (R)
- `SQL_BUFFER_RESULT`
- `SQL_CACHE`; removed in 8.0.3
- `SQL_CALC_FOUND_ROWS` (R)
- `SQL_NO_CACHE`
- `SQL_SMALL_RESULT` (R)
- `SQL_THREAD`
- `SQL_TSI_DAY`
- `SQL_TSI_HOUR`
- `SQL_TSI_MINUTE`
- `SQL_TSI_MONTH`
- `SQL_TSI_QUARTER`
- `SQL_TSI_SECOND`
- `SQL_TSI_WEEK`
- `SQL_TSI_YEAR`
- `SRID`; added in 8.0.3 (nonreserved)
- `SSL` (R)
- `STACKED`
- `START`
- `STARTING` (R)
- `STARTS`
- `STATS_AUTO_RECALC`
- `STATS_PERSISTENT`
- `STATS_SAMPLE_PAGES`
- `STATUS`
- `STOP`
- `STORAGE`
- `STORED` (R)
- `STRAIGHT_JOIN` (R)
- `STRING`
- `SUBCLASS_ORIGIN`
- `SUBJECT`
- `SUBPARTITION`
- `SUBPARTITIONS`
- `SUPER`
- `SUSPEND`
- `SWAPS`
- `SWITCHES`
- `SYSTEM` (R); added in 8.0.3 (reserved)

T

- `TABLE` (R)
- `TABLES`
- `TABLESPACE`
- `TABLE_CHECKSUM`
- `TABLE_NAME`
- `TEMPORARY`
- `TEMPTABLE`
- `TERMINATED` (R)
- `TEXT`
- `THAN`
- `THEN` (R)
- `THREAD_PRIORITY`; added in 8.0.3 (nonreserved)
- `TIES`; added in 8.0.2 (nonreserved)
- `TIME`
- `TIMESTAMP`
- `TIMESTAMPADD`
- `TIMESTAMPDIFF`
- `TINYBLOB` (R)
- `TINYINT` (R)
- `TINYTEXT` (R)
- `TO` (R)
- `TRAILING` (R)
- `TRANSACTION`
- `TRIGGER` (R)
- `TRIGGERS`
- `TRUE` (R)
- `TRUNCATE`
- `TYPE`
- `TYPES`

U

- `UNBOUNDED`; added in 8.0.2 (nonreserved)
- `UNCOMMITTED`
- `UNDEFINED`
- `UNDO` (R)
- `UNDOFILE`
- `UNDO_BUFFER_SIZE`
- `UNICODE`
- `UNINSTALL`
- `UNION` (R)
- `UNIQUE` (R)
- `UNKNOWN`
- `UNLOCK` (R)
- `UNSIGNED` (R)
- `UNTIL`
- `UPDATE` (R)
- `UPGRADE`
- `USAGE` (R)
- `USE` (R)
- `USER`
- `USER_RESOURCES`
- `USE_FRM`
- `USING` (R)
- `UTC_DATE` (R)
- `UTC_TIME` (R)
- `UTC_TIMESTAMP` (R)

V

- `VALIDATION`
- `VALUE`
- `VALUES` (R)
- `VARBINARY` (R)
- `VARCHAR` (R)
- `VARCHARACTER` (R)
- `VARIABLES`
- `VARYING` (R)
- `VCPU`; added in 8.0.3 (nonreserved)
- `VIEW`
- `VIRTUAL` (R)
- `VISIBLE`

W

- `WAIT`
- `WARNINGS`
- `WEEK`
- `WEIGHT_STRING`
- `WHEN` (R)
- `WHERE` (R)
- `WHILE` (R)
- `WINDOW` (R); added in 8.0.2 (reserved)
- `WITH` (R)
- `WITHOUT`
- `WORK`
- `WRAPPER`
- `WRITE` (R)

X

- `X509`
- `XA`
- `XID`
- `XML`
- `XOR` (R)

Y

- `YEAR`
- `YEAR_MONTH` (R)

Z

- `ZEROFILL` (R)

### 新增保留字：

A

- `ACTIVE`
- `ADMIN`
- `ARRAY`

B

- `BUCKETS`

C

- `CLONE`
- `COMPONENT`
- `CUME_DIST` (R)

D

- `DEFINITION`
- `DENSE_RANK` (R)
- `DESCRIPTION`

E

- `EMPTY` (R)
- `ENFORCED`
- `EXCEPT` (R)
- `EXCLUDE`

F

- `FAILED_LOGIN_ATTEMPTS`
- `FIRST_VALUE` (R)
- `FOLLOWING`

G

- `GEOMCOLLECTION`
- `GET_MASTER_PUBLIC_KEY`
- `GROUPING` (R)
- `GROUPS` (R)

H

- `HISTOGRAM`
- `HISTORY`

I

- `INACTIVE`
- `INVISIBLE`

J

- `JSON_TABLE` (R)

L

- `LAG` (R)
- `LAST_VALUE` (R)
- `LATERAL` (R)
- `LEAD` (R)
- `LOCKED`

M

- `MASTER_COMPRESSION_ALGORITHMS`
- `MASTER_PUBLIC_KEY_PATH`
- `MASTER_TLS_CIPHERSUITES`
- `MASTER_ZSTD_COMPRESSION_LEVEL`
- `MEMBER`

N

- `NESTED`
- `NETWORK_NAMESPACE`
- `NOWAIT`
- `NTH_VALUE` (R)
- `NTILE` (R)
- `NULLS`

O

- `OF` (R)
- `OJ`
- `OLD`
- `OPTIONAL`
- `ORDINALITY`
- `ORGANIZATION`
- `OTHERS`
- `OVER` (R)

P

- `PASSWORD_LOCK_TIME`
- `PATH`
- `PERCENT_RANK` (R)
- `PERSIST`
- `PERSIST_ONLY`
- `PRECEDING`
- `PRIVILEGE_CHECKS_USER`
- `PROCESS`

R

- `RANDOM`
- `RANK` (R)
- `RECURSIVE` (R)
- `REFERENCE`
- `REQUIRE_ROW_FORMAT`
- `RESOURCE`
- `RESPECT`
- `RESTART`
- `RETAIN`
- `REUSE`
- `ROLE`
- `ROW_NUMBER` (R)

S

- `SECONDARY`
- `SECONDARY_ENGINE`
- `SECONDARY_LOAD`
- `SECONDARY_UNLOAD`
- `SKIP`
- `SRID`
- `SYSTEM` (R)

T

- `THREAD_PRIORITY`
- `TIES`

U

- `UNBOUNDED`

V

- `VCPU`
- `VISIBLE`

W

- `WINDOW` (R)

### 已删除的保留字：

- `ANALYSE`
- `DES_KEY_FILE`
- `PARSE_GCOL_EXPR`
- `REDOFILE`
- `SQL_CACHE`