### 事务四大特性

事务具有四大特性，被称为ACID：

- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败
- 一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
- 持久性（Durability）：事务一旦提交或滚回，它对数据库的改变就是永久的

### 并发事务问题

并发事务常常出现三种问题：

| 问题       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 脏读       | 一个事务读到另外一个事务还没有提交的数据                     |
| 不可重复读 | 一个事务先后读取同一条记录，但两次读取的数据不同，称为不可重复读 |
| 幻读       | 一个事务按照条件查询数据时，没有对应的数据行，但在插入数据时，又发现该数据已经存在 |

### 事务隔离级别

事务隔离级别分为四种：

| 隔离级别              | 脏读 | 不可重复读 | 幻读 |
| --------------------- | ---- | ---------- | ---- |
| Read uncommitted      | √    | √          | √    |
| Read committed        | ×    | √          | √    |
| Repeatable Read(默认) | ×    | ×          | √    |
| Serializable          | ×    | ×          | ×    |

# 用户数据管理

## 管理用户

~~~sql
/*查询用户*/
SELECT * FROM user

/*创建用户*/
CREATE USER '用户名'@ '主机名' IDENTIFUED BY '密码'

/*修改用户密码*/
ALTER USER '用户名'@ '主机名' IDENTIFUED WITH mysql_native_password BY '新密码'

/*删除用户*/
DROPUSER '用户名'@ '主机名'
~~~

主机名可以用%通配符，表示任意

## 权限控制

~~~sql
/*查询权限*/
SHOW GRANTS FOR '用户名'@ '主机名'

/*授予权限*/
GRANT 权限列表 ON 数据库名.表名 TO '用户名'@ '主机名'

/*撤销权限*/
REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@ '主机名'
~~~

> 多个权限之间，可以使用逗号分隔
>
> 授权时，数据库名和表名可以使用 * 进行统配，表示所有

# 函数

## 字符串函数

| 函数                     | 功能                                                       |
| ------------------------ | ---------------------------------------------------------- |
| CONCAT(S1,S2,...Sn)      | 字符串拼接，将S1，S2，... Sn拼接成一个字符串               |
| LOWER(str)               | 将字符串str全部转为小写                                    |
| UPPER(str)               | 将字符串str全部转为大写                                    |
| LPAD(str,n,pad)          | 左填充，用字符串pad对str的左边进行填充，达到n个字符 串长度 |
| RPAD(str,n,pad)          | 右填充，用字符串pad对str的右边进行填充，达到n个字符 串长度 |
| TRIM(str)                | 去掉字符串头部和尾部的空格                                 |
| SUBSTRING(str,start,len) | 返回从字符串str从start位置起的len个长度的字符串            |

## 数值函数

| 函数       | 功能                               |
| ---------- | ---------------------------------- |
| CEIL(x)    | 向上取整                           |
| FLOOR(x)   | 向下取整                           |
| MOD(x,y)   | 返回x/y的模                        |
| RAND()     | 返回0~1内的随机数                  |
| ROUND(x,y) | 求参数x的四舍五入的值，保留y位小数 |

## 日期函数

| 函数                               | 功能                                             |
| ---------------------------------- | ------------------------------------------------ |
| CURDATE()                          | 返回当前日期                                     |
| CURTIME()                          | 返回当前时间                                     |
| NOW()                              | 返回当前日期和时间                               |
| YEAR(date)                         | 获取指定date的年份                               |
| MONTH(date)                        | 获取指定date的月份                               |
| DAY(date)                          | 获取指定date的日期                               |
| DATE_ADD(date, INTERVAL expr type) | 返回一个日期/时间值加上一个时间间隔expr后的 时间 |
| DATEDIFF(date1,date2)              | 返回起始时间date1 和 结束时间date2之间的天数     |

## 流程控制函数

| 函数                                                         | 功能                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| IF(value,t,f)                                                | 如果value为true，返回 t ，否则返回 f                       |
| IFNULL(v1,v2)                                                | 如果 v1 不为空，返回 v1，否则返回 v2                       |
| CASE WHEN [ val1 ] THEN [res1] ... ELSE [ default ] END      | 如果val1为true，返回res1，... 否 则返回default默认值       |
| CASE [ expr ] WHEN [ val1 ] THEN [res1] ... ELSE [ default ] END | 如果expr的值等于val1，返回 res1，... 否则返回default默认值 |

# 存储引擎

存储结构 表空间、段、区（大小固定 1 M）、页（大小固定 16 k）、行

## MySQL的体系结构

- 第一层——连接层：接受客户端的链接，完成链接处理，认证授权，安全方案，检查最大链接数
- 第二层——服务层：SQL结构，解析器，查询优化器，缓存
- 第三层——存储引擎层：数据存储和提取的方式，索引在这一层实现
- 第四层——存储层：数据库的相关数据

## 存储引擎简介

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式 。存储引擎是基于表的，而不是 基于库的，所以存储引擎也可被称为表类型。

~~~mysql
/*查询建表语句*/
show create table 表名;

/*建表时指定存储引擎*/
CREATE TABLE 表名(
    字段1 字段1类型 [ COMMENT 字段1注释 ] ,
) ENGINE = 引擎名 [ COMMENT 表注释 ] ;

/*查询当前数据库支持的存储引擎*/
show engines;
~~~

### InnoDB 存储引擎

在 MySQL 5.5 之后，InnoDB是默认的 MySQL 存储引擎。

#### 特点 

- DML操作遵循ACID模型，支持事务；
-  行级锁，提高并发访问性能； 
- 支持外键FOREIGN KEY约束，保证数据的完整性和正确性；

#### 文件

xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结 构（frm-早期的 、sdi-新版的）、数据和索引。

#### 逻辑存储结构

1. 表空间 : InnoDB存储引擎逻辑结构的最高层，ibd文件其实就是表空间文件，在表空间中可以 包含多个Segment段。 
2. 段 : 表空间是由各个段组成的， 常见的段有数据段、索引段、回滚段等。InnoDB中对于段的管 理，都是引擎自身完成，不需要人为对其控制，一个段中包含多个区。 
3. 区 : 区是表空间的单元结构，每个区的大小为1M。 默认情况下， InnoDB存储引擎页大小为 16K， 即一个区中一共有64个连续的页。 
4. 页 : 页是组成区的最小单元，页也是InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默 认为 16KB。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区。 
5. 行 : InnoDB 存储引擎是面向行的，也就是说数据是按行进行存放的，在每一行中除了定义表时 所指定的字段以外，还包含两个隐藏字段

### MyISAM 存储引擎

是MySQL早期的默认存储引擎

#### 特点

- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快

#### 文件

- xxx.sdi：存储表结构信息
- xxx.MYD: 存储数据
- xxx.MYI: 存储索引

### Memory 存储引擎

Memory引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为 临时表或缓存使用。

#### 特点 

1. 内存存放
2. hash索引（默认）

#### 文件

- xxx.sdi：存储表结构信息

### 区别及特点



## 存储引擎选择

# 索引

## 分类

| 分类     | 含义                                                  | 特点                      | 关键字   |
| -------- | ----------------------------------------------------- | ------------------------- | -------- |
| 主键索引 | 针对于表中主键创建的索引                              | 默认自动创建, 只能 有一个 | PRIMARY  |
| 唯一索引 | 避免同一个表中某数据列中的值重复                      | 可以有多个                | UNIQUE   |
| 常规索引 | 快速定位特定数据                                      | 可以有多个                |          |
| 全文索引 | 全文索引查找的是文本中的关键词，而不是比 较索引中的值 | 可以有多个                | FULLTEXT |

**InnoDB中的索引形式**

| 分类     | 含义                                                        | 特点                 |
| -------- | ----------------------------------------------------------- | -------------------- |
| 聚集索引 | 将数据存储与索引放到了一块，索引结构的叶子 节点保存了行数据 | 必须有,而且只 有一个 |
| 二级索引 | 将数据与索引分开存储，索引结构的叶子节点关 联的是对应的主键 | 可以存在多个         |

聚集索引选取规则

> 1. 主键
> 2. 使用第一个 UNIQUE 作为聚集索引
> 3. 自动生成一个rowid作为隐藏的聚集索引

**回表查询：**先走二级索引找到主键值，在去聚集索引中拿到这一行的行数据

## sql性能分析

### sql执行频率

~~~sql
/*服务器状态信息*/
-- session 是查看当前会话 ; 
-- global 是查询全局数据 ;
show [session|global] status

/*查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次*/
SHOW GLOBAL STATUS LIKE 'Com_______';
~~~

### 慢查询日志

*慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有 SQL语句的日志*

~~~sql
/*慢查询日志默认没有开启，需要在配置文件（/etc/my.cnf）中配置*/
使用vim添加配置信息
# 开启MySQL慢日志查询开关
slow_query_log=1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
~~~

### profile详情

先使用 `SELECT @@have_profiling;` 查看是否支持 profile 功能

开启 profiling ：`SET profiling = 1;` （可以指定在 session/global级别开启）

~~~sql
-- 查看每一条SQL的耗时基本情况
show profiles;

-- 查看指定query_id的SQL语句各个阶段的耗时情况
show profile for query query_id;

-- 查看指定query_id的SQL语句CPU的使用情况
show profile cpu for query query_id;
~~~

### explain执行计划

EXPLAIN 或者 DESC命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行 过程中表如何连接和连接的顺序

~~~sql
-- 直接在select语句之前加上关键字 explain / desc
EXPLAIN SELECT 字段列表 FROM 表名 WHERE 条件;
~~~

**Explain 执行计划中各个字段的含义:**

|              |                                                              |
| ------------ | ------------------------------------------------------------ |
| id           | select查询的序列号，表示查询中执行select子句或者是操作表的顺序 (id相同，执行顺序从上到下；id不同，值越大，越先执行)。 |
| select_type  | 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接 或者子查询）、PRIMARY（主查询，即外层的查询）、 UNION（UNION 中的第二个或者后面的查询语句）、 SUBQUERY（SELECT/WHERE之后包含了子查询）等 |
| type         | 表示连接类型，性能由好到差的连接类型为NULL、system、const、 eq_ref、ref、range、 index、all 。 |
| possible_key | 显示可能应用在这张表上的索引，一个或多个。                   |
| key          | 实际使用的索引，如果为NULL，则没有使用索引。                 |
| key_len      | 表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长 度，在不损失精确性的前提下， 长度越短越好 。 |
| rows         | MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值， 可能并不总是准确的。 |
| filtered     | 表示返回结果的行数占需读取行数的百分比， filtered 的值越大越好。 |

## 索引使用

### 最左前缀法则

如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的最左前缀法则指的是，查询时，最左变的列，必须存在，否则索引全部失效。 而且中间不能跳过某一列，否则该列后面的字段索引将失效。（与顺序无关）

### 索引失效情况

范围查询中(>,<)，范围查询右侧的列索引失效。在业务允许的情况下，尽可能的使用类似于 >= 或 <= 这类的范围查询，而避免使用 > 或 <。

在索引列上进行运算操作， 索引将失效。

字符类型数据，不加字符串会导致索引失效

在模糊匹配中，尾部模糊匹配，索引不会失效。头部模糊匹配，索引失效。

当or连接的条件，其中一侧没有索引，索引失效，左右两侧字段都有索引时，索引才会生效。

数据分布，如果MySQL评估使用索引比全表更慢，则不使用索引。

### SQL提示

- use index ： 建议MySQL使用哪一个索引完成此次查询（仅仅是建议，mysql内部还会再次进 行评估）
- ignore index ： 忽略指定的索引。
- force index ： 强制使用索引。

~~~sql
select * from 表名 sql提示(索引名) where 条件;
~~~

### 覆盖索引

尽量使用覆盖索引（覆盖索引是指 查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到）

### 前缀索引

当字段类型为字符串，有时候需要索引很长的字符串，者会让索引变得很大，此时可以将字符串的一部分前缀建立索引，可以节约空间

~~~sql
create index idx_xxxx on table_name(column(n)) ;
~~~

**前缀长度**

以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值， 索引选择性越高则查询效率越高， 唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。前缀索引主要时解决长字符串数据查询产生了大量的磁盘 IO 情况

~~~sql
-- 不设置索引时的索引选择性
select count(distinct email) / count(*) from tb_user ;
-- 前5个字符的索引选择性
select count(distinct substring(email,1,5)) / count(*) from tb_user ;
~~~

##  索引设计原则

1. 在数据量大，且查询比较频繁的表建立索引
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引。
3. 选择区分度高的列，建立唯一索引，区分度越高，索引的效率越高。
4. 字符串较长的列建立前缀索引
5. 尽量采用联合索引，避免回表，提高查询效率
6. 控制索引的数据量，索引越多，维护索引结构的代价就越大
7. 在创建表的时候尽量使用 NOT NULL 约束，方便优化器更好判断索引查询效率

# SQL优化

## insert优化

1. 尽量使用批量插入
2. 手动提交事务
3. 采用数据插入

## 大批量插入数据

主键顺序插入效率大于乱序插入

使用 load 指令插入

~~~sql
-- 客户端连接服务端时，加上参数 -–local-infile
mysql –-local-infile -u root -p

-- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1;

-- 执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table tb_user fields terminated by ',' lines terminated by '\n' ;
~~~

## 主键优化

主键乱序插入会产生页分裂现象

删除一行记录时，该记录只是被标记为删除，并没有进行物理删除，并且它的空间变得允许被其他记录声明使用

### 主键设计原则

- 尽量降低主键的长度
- 插入数据的时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键
- 尽量不使用UUID作为主键或其他主键，比如身份证号

## order by优化

### MySQL的排序，有两种方式：

- Using filesort : 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。 
- Using index : 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要 额外排序，操作效率高。 对于以上的两种排序方式，Using index的性能高，而Using filesort的性能低，我们在优化排序 操作时，尽量要优化为 Using index。

### 优化注意事项

- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则
- 尽量使用覆盖索引
- 多字段排序，一个升序一个降序，注意联合索引在创建时的规则
- 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小 sort_buffer_size(默认256k)。

## group by优化

Using temporary 使用到临时表

在分组操作时，也是需要满足最左前缀法则

## limit优化

通过覆盖索引加子查询的方式进行优化

## count优化

无优化方案，自己做一个计数器，计算数据插入删除

在计数的时候不计数 null 值

尽量使用 count(*) 对这个进行了优化

## update数据优化

InnoDB的行锁是针对索引加的锁，不是针对记录加的锁 ,并且该索引不能失效，否则会从行锁 升级为表锁 。

# 视图

视图（View）是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

~~~sql
-- 创建
CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [WITH [CASCADED | LOCAL ] CHECK OPTION ]

-- 查询
查看创建视图语句：SHOW CREATE VIEW 视图名称;
查看视图数据：SELECT * FROM 视图名称 ...... ;

-- 修改
方式一：CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH
[ CASCADED | LOCAL ] CHECK OPTION ]
方式二：ALTER VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH [ CASCADED |
LOCAL ] CHECK OPTION ]

-- 删除
1 DROP VIEW [IF EXISTS] 视图名称 [,视图名称] ..
~~~

### 视图检查选项

**WITH [ CASCADED | LOCAL ]  CHECK OPTION**：在创建视图的时候添加，在对视图进行数据操作时，对该条件进行检查

- CASCADED ：级联，检查所有父视图的检查选项，当父视图只有条件没有指定with check ，也会强制检查
- LOCAL ：本地，只检查当前视图的检查选项，当相关视图只有条件没有指定with check ，也会强制检查

###  视图更新及作用

更新注意

在更新时，视图中的数据必须与基表数据保持一对一关系

# 存储过程

## 基本语法

### 创建

~~~sql
CREATE PROCEDURE 存储过程名称 ([ 参数列表 ])
BEGIN
-- SQL语句
END ;
~~~

### 调用

~~~sql
CALL 名称 ([ 参数 ]);
~~~

### 查看

~~~sql
SELECT * FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'xxx'; -- 查询指
定数据库的存储过程及状态信息
SHOW CREATE PROCEDURE 存储过程名称 ; -- 查询某个存储过程的定义
~~~

### 删除

~~~sql
DROP PROCEDURE [ IF EXISTS ] 存储过程名称 ;
~~~

## 变量

变量分为三种类型: 系统变量、用户定义变量、局部变量。

### 系统变量

MySQL服务器提供，不是用户定义的，属于服务器层面。分为全局变量（GLOBAL）、会话变量（SESSION）

#### 基本使用语法

~~~sql
/*查看系统变量*/
SHOW [ SESSION | GLOBAL ] VARIABLES ; -- 查看所有系统变量
SHOW [ SESSION | GLOBAL ] VARIABLES LIKE '......'; -- 可以通过LIKE模糊匹配方式查找变量
SELECT @@[SESSION | GLOBAL] 系统变量名; -- 查看指定变量的值

/*设置系统变量*/
-- 默认是SESSION，会话变量。
SET [ SESSION | GLOBAL ] 系统变量名 = 值 ;
SET @@[SESSION | GLOBAL]系统变量名 = 值 ;

~~~

### 用户定义变量

#### 基本使用语法

~~~sql
-- 赋值
SET @var_name = expr [, @var_name = expr] ... ;
SET @var_name := expr [, @var_name := expr] ... ;
SELECT @var_name := expr [, @var_name := expr] ... ;
SELECT 字段名 INTO @var_name FROM 表名;

-- 使用
SELECT @var_name ;
~~~

*用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NULL。*

### 局部变量

在局部生效的变量，访问之前，需要DECLARE声明。可用作存储过程内的 局部变量和输入参数，局部变量的范围是在其内声明的BEGIN ... END块。

#### 基本使用语法

~~~sql
-- 声明
DECLARE 变量名 变量类型 [DEFAULT ... ] ;

-- 赋值
SET 变量名 = 值 ;
SET 变量名 := 值 ;
SELECT 字段名 INTO 变量名 FROM 表名 ... ;
~~~

## if 判断

~~~sql
IF 条件1 THEN
.....
ELSEIF 条件2 THEN -- 可选
.....
ELSE -- 可选
.....
END IF;
-- ELSE IF 结构可以有多个，也可以没有。 ELSE结构可以有，也可以没有。
~~~

## 参数

主要分为以下三种：IN、OUT、INOUT。 

- IN 该类参数作为输入，也就是需要调用时传入值 默认
- OUT 该类参数作为输出，也就是该参数可以作为返回值
- INOUT 既可以作为输入参数，也可以作为输出参数

## case

~~~sql
-- 方式一
CASE case_value
WHEN when_value1 THEN statement_list1
[ WHEN when_value2 THEN statement_list2] ...
[ ELSE statement_list ]
END CASE;

-- 方式二
CASE
WHEN search_condition1 THEN statement_list1
[WHEN search_condition2 THEN statement_list2] ...
[ELSE statement_list]
END CASE
~~~

## while

~~~sql
WHILE 条件 DO
	SQL逻辑...
END WHILE;
~~~

## repeat

~~~sql
REPEAT
    SQL逻辑...
    UNTIL 条件
END REPEAT;
~~~

## loop

LOOP 实现简单的循环，如果不在SQL逻辑中增加退出循环的条件，可以用其来实现简单的死循环。 LOOP可以配合一下两个语句使用： 

- LEAVE ：配合循环使用，退出循环。 
- ITERATE：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

~~~sql
[begin_label:] LOOP
	SQL逻辑...
END LOOP [end_label];

LEAVE label; -- 退出指定标记的循环体
ITERATE label; -- 直接进入下一次循环
~~~

## 游标

游标（CURSOR）是用来存储查询结果集的数据类型

先声明普通变量，在声明游标

~~~sql
-- 声明
DECLARE 游标名称 CURSOR FOR 查询语句 ;

-- 打开
OPEN 游标名称 ;

-- 获取
FETCH 游标名称 INTO 变量 [, 变量 ] ;

-- 关闭
CLOSE 游标名称 ;
~~~

## 条件处理程序

条件处理程序（Handler）可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。

~~~sql
DECLARE handler_action HANDLER FOR condition_value [, condition_value]... statement ;

-- handler_action 的取值：
CONTINUE: 继续执行当前程序
EXIT: 终止执行当前程序

-- condition_value 的取值：
SQLSTATE sqlstate_value: 状态码，如 02000
SQLWARNING: 所有以01开头的SQLSTATE代码的简写
NOT FOUND: 所有以02开头的SQLSTATE代码的简写
SQLEXCEPTION: 所有没有被SQLWARNING 或 NOT FOUND捕获的SQLSTATE代码的简写
~~~

## 存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是IN类型的。

~~~sql
CREATE FUNCTION 存储函数名称 ([ 参数列表 ])
RETURNS type [characteristic ...]
BEGIN
-- SQL语句
RETURN ...;
END ;
~~~

characteristic说明： 

- DETERMINISTIC：相同的输入参数总是产生相同的结果
- NO SQL ：不包含 SQL 语句
- READS SQL DATA：包含读取数据的语句，但不包含写入数据的语句。

# 触发器

mysql只支持行级触发，不支持语句级触发。

触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作

## 基本语法

~~~sql
-- 创建
CREATE TRIGGER trigger_name
BEFORE/AFTER INSERT/UPDATE/DELETE
ON tbl_name FOR EACH ROW -- 行级触发器
BEGIN
	trigger_stmt ;
END;

-- 查看
SHOW TRIGGERS ;

-- 删除
DROP TRIGGER [schema_name.]trigger_name ; -- 如果没有指定 schema_name，默认为当前数据库 。
~~~

# 锁

## 全局锁

对整个数据库实例加锁，加锁后整个实例就处于只读状态

主要用于对数据库进行备份

~~~sql
-- 加锁
flush tables with read lock ;

-- 解锁
unlock tables ;

-- 数据备份
$ mysqldump -uroot –p1234 itcast > itcast.sql
~~~

## 表级锁

表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。

表级锁，有三种分类：表锁、元数据锁、意向锁

### 表锁

~~~sql
-- 加锁：
lock tables 表名... read/write。

-- 释放锁：
unlock tables 
~~~

读锁：别的客户端不能写入，但可以读

写锁：别的客户端不能写，也不能读

### 元数据锁

加锁过程系统自动控制，在访问一张表的时候会自动加上。

MDL锁主要作用：

- 维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作
- 为了避免DML与DDL冲突，保证读写的正确性。
- 某一张表涉及到未提交的事务时，是不能够修改这张表的表结构的。

MySQL5.5中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁(共享)；当对表结构进行变 更操作的时候，加MDL写锁(排他)。

查看锁

~~~sql
# 查看所有锁
select object_type,object_schema,object_name,lock_type,lock_duration fromperformance_schema.metadata_locks ;

~~~



### 意向锁

为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

分类：

- 意向共享锁(IS): 由语句select ... lock in share mode添加 。 与 表锁共享锁 (read)兼容，与表锁排他锁(write)互斥。 
- 意向排他锁(IX): 由insert、update、delete、select...for update添加 。与表锁共 享锁(read)及排他锁(write)都互斥，意向锁之间不会互斥。

### 行级锁

锁对应的行数据，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在 InnoDB存储引擎中。

- 对于行级锁，主要分为以下三类：
  1. 行锁：
     - 锁定单个行记录的锁，防止其他事务对此行进行update和delete。
     - 在RC、RR隔离级别下都支持。
  2. 间隙锁（Gap Lock）：
     - 锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。
     - 在RR隔离级别下都支持。
  3. 临键锁（Next-Key Lock）：
     - 行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。
     - 在RR隔离级别下支持。

#### 行锁

InnoDB实现了以下两种类型的行锁 ：

- 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。
- 排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁。

注意：

- 仅当共享锁和共享锁共存时兼容
- 其他情况兼不兼容

下面我们给出不同SQL语句相对应的行锁级别：

| SQL                           | 行锁类型 | 说明                                     |
| ----------------------------- | -------- | ---------------------------------------- |
| INSERT                        | 排他锁   | 自动加锁                                 |
| UPDATE                        | 排他锁   | 自动加锁                                 |
| DELETE                        | 排他锁   | 自动加锁                                 |
| SELECT                        | 不加锁   |                                          |
| SELECT ... LOCK IN SHARE MOOE | 共享锁   | 需要手动在SELECT之后加LOCK IN SHARE MODE |
| SELECT ... FOR UPDATE         | 排他锁   | 需要手动在SELECT之后加FOR UPDATE         |

行锁特点：

- 默认情况下，InnoDB在 REPEATABLE READ事务隔离级别运行，InnoDB使用 next-key 锁进行搜索和索引扫描，以防止幻读。
- 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
- InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，此时就会升级为表锁。

#### 间隙锁&临键锁

默认情况下，InnoDB在 REPEATABLE READ事务隔离级别运行，InnoDB使用 next-key 锁进行搜索和索引扫描，以防止幻读。

一般出现上述锁有以下三种情况：

- 索引上的等值查询(唯一索引)，给不存在的记录加锁时, 优化为间隙锁 。
- 索引上的等值查询(非唯一普通索引)，向右遍历时最后一个值不满足查询需求时，next-keylock 退化为间隙锁。
- 索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止。

注意：

- 间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。
