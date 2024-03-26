- DDL（Data Definition Language）语句： 数据定义语言，主要是进行定义/改变表的结构、数据类型、表之间的链接等操作。常用的语句关键字有 CREATE、DROP、ALTER 等。

- DML（Data Manipulation Language）语句: 数据操纵语言，主要是对数据进行增加、删除、修改操作。常用的语句关键字有 INSERT、UPDATE、DELETE 等。

- DQL（Data Query Language）语句：数据查询语言，主要是对数据进行查询操作。常用关键字有 SELECT、FROM、WHERE 等。
- DCL（Data Control Language）语句： 数据控制语言，主要是用来设置/更改数据库用户权限。常用关键字有 GRANT、REVOKE 等。

>> 一般人员很少用到DCL语句。

<br>

非关系型DBMS（NoSQL）无法读取SQL语言。<br>
<br>

SQL有两种读法：SQL和SQUEL<br>
要看你问的是谁，通常来说，非英语国家的人会叫它SQL。<br>
<br>

在SQL语言中要给日期带上引号，即使日期并不是字符串。<br>
<br>

LIKE '_y' 中的_表示一个单字符。也可以'_____y \'<br>
<br>

更改发票记录表中名字叫 Yadel 的记录，但该表只有 client_id，故先要从另一个顾客表中查询叫 Yadel 人的
client_id<br>
实际中这是很可能的情形，比如一个App是通过搜索名字来更改发票记录的<br>

```sql
USE sql_invoicing;
UPDATE invoices
SET payment_total = 567, payment_date = due_date
★WHERE client_id =
(SELECT client_id
FROM clients
WHERE name = 'Yadel');
-- 放入括号，确保先执行
-- 若子查询返回多个数据（一列多条数据）时就不能用等号而要用 IN 了：
WHERE client_id 【IN】
(SELECT client_id
FROM clients
WHERE state IN ('CA', 'NY'))
```

有些时候不能在Group by 前使用WHERE，因为只有分组之后才有分开的数据，而WHERE又不能放在 GROUP BY 后面，所以为了能对分组后的数据进行筛选，就要使用 Having。简单来说就是，**WHERE 用于分组前筛选，Having 用于分组后筛选**。<br>
<br>

不过，Having 后选取的列只能是 Select 中有的列，而Where可以选任何列，不论是否在SELECT中。<br>
<br>

在 WHERE IN (...) 中，如果子查询的返回结果很大，数据很多，那么效率就会较低，而换用 EXISTS 关键字效率会高一些，因为用 EXISTS 时，子查询并没有真的把结果集返回给外查询，只要知道子查询结果不为空，满足条件的行就行了。<br>
<br>

除了选择语句的 WHERE 子句中可以使用子查询外，在选择语句 SELECT 中也可以使用子查询。<br>
<br>

**不能在表达式中使用列的别名**。也可以这样解释：子查询可以直接调用新产生的列，这个新产生的列(即 invoice_average)不能直接调用，因为它并不在invoices里，是刚刚产生的column。为了调用，需要使用子查询，如下：

```sql
SELECT 
	invoice_id,
    invoice_total,
    (SELECT AVG(invoice_total)
    FROM invoices) AS invoice_average,
    invoice_total - (SELECT invoice_average) AS difference
FROM invoices
```

另外，AVG(invoice_total)只会直接产生一个数而不是一个列，放在 SELECT 里面后sql会直接将每一行都填充这个数，包括这一列的名字。这就像直接在 SELECT 中添加一个字符串 'ACTIVE' 一样。具体可以自己测试一下。<br>
<br>

下面的SUM这一列的别买total_sales既不能直接用，也不能 SELECT，似乎只能直接用 SUM：

```sql
SELECT 
	client_id,
    c.name,
    SUM(invoice_total) AS total_sales,
    (SELECT AVG(invoice_total)
    FROM invoices) AS average,
    -- 这样做会报错！！！Error Code: 1247. Reference 'total_sales' not supported (reference to group function)
    -- (SELECT total_sales- average) AS difference  
    -- 只能这样
    SUM(invoice_total) - (SELECT average) AS difference
FROM clients c
LEFT JOIN invoices i USING (client_id)
GROUP BY client_id
ORDER BY client_id
```

如果要把代码录入别的DBMS中，最好用EXTRACT，而不是 YEARNAME, MONTHNAME 这些。<br>
<br>

CONCAT() 可以不同的字符串连起来。<br>
<br>

IFNULL 里可以用其它值替代空值，而 COALESCE 函数里，我们提供一堆值，这个函数会返回这堆值中的第一个非空值。<br>
<br>

如果再 SELECT 语句中定义了新的列和它的别名，在接下来的 WHERE 中不能使用这个别名，也不能用 SELECT ，原因应该是一个查询在具体执行时，顺序是先 from，再 where，再到 SELECT，所以 where 是无法感知到这个新的列的，因为还没执行到 SELECT。<br>
<br>

DROP PROCEDURE IF EXISTS ...; 后一定要有一个 ";" ，否则会报错。<br>
<br>


在触发器中我们可以修改任何表中的数据，除了触发器所在的表。<br>
<br>

在MySQL中，默认的事务隔离级别是“可重复读取”，但无法阻止幻读。<br>
<br>

关系型数据库中没有“多对多”关系，只有“一对一”和“一对多”，每当出现“多对多”的关系，都要把它拆分成2组“一对多”的关系。<br>
<br>

全文索引有两种方式，自然语言模式、布林模式<br>
<br>

如果在表达式中利用了列，比如 points + 10 > 2010，那么 MySQL 就无法以最优的方式利用索引了。为此，可以将这个列单独放一边，比如改为 points > 2000.<br>
<br>

**当在列上添加索引时，MySQL会获取该列中的所有值，对其排序，并将其存储在索引中。** <br>
<br>

使用没有索引的列来排序（ORDER BY）耗费的资源几乎是用有索引的列来排序的10倍。使用如下语句可以看到上一次查询的耗费：

```sql
SHOW STATUS LIKE 'last_query_cost';
```

有时会因为查询语句较为复杂等原因，导致MySQL不一定就用我们的索引来排序，基本规则是，ORDER BY 子句中的列的顺序，应该与索引中的列的顺序相同，猜测这句话的意思是，比如idx_state_points 这个索引，排序时可以用 ORDER BY state, points，但不能用ORDER BY state, first_name, points，因为first_state并不再索引中。<br>
<br>

还有就是不同列的升降序要搞清楚，要保持一致，比如不能写ORDER BY state, points DESC。因为基于这两列的复合索引，是先按州再按积分升序排序的，如果使用降序，那么MySQL就无法利用这个索引来完成查询，因为排序方向不一致。但ORDER BY state DESC, points DESC可以。（这部分内容在p-144集）<br>
<br>

也不把索引中的第二列放到前面来排序，比如 ORDER BY points，否则仍然是外部排序(filesort)。一个特例是，在一个固定的 state 里，可以用 points 来排，比如加一个 WHERE state = 'CA'。<br>
<br>

每当创建一个二级索引时，MySQL都会自动将主键包含在第二索引中，比如在 idx_state_points 索引中，还保存了 customer_id 这一列。<br>
<br>

覆盖索引是最快的，因为它可以在不读取表的情况下就执行查询，比如，SELECT 值选中了 customer_id 和 state，那么 idx_state_points 就是一个覆盖索引，执行这个查询时不用读取表，直接使用索引就行。<br>
<br>

在新建索引之前，先查看现有索引。<br>
<br>



# 高级部分

如果再命令行出来的表不方便观看，可以在命令行末尾（分号之前）添加“\G”，转换成一行一行的格式。



## 存储引擎
逻辑存储结构：
- 表空间
- 段
- 区
- 页
- 行

每个页大小为16k，每个区为1M，每个区有64个页，这些都是固定的。


## 索引
索引在引擎层，所以不同的引擎索引的结构也不一样。<br>
<br>

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式 。存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型。<br>
<br>

ACID是衡量事务的四个特性：
- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）

InnoDB的特点：

- DML操作遵循ACID模型，支持事务；
- 行级锁，提高并发访问性能；
- 支持外键FOREIGN KEY约束，保证数据的完整性和正确性；

三点：事务、行级锁、外键<br>
<br>

xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm-早期的 、sdi-新版的）、数据和索引。（**注意，是ibd，不是idb**）<br>
<br>

索引（index）是帮助MySQL高效获取数据的数据结构(有序)。是一种数据结构，而且是有序的，是用来高效获取数据的。<br>
<br>

索引的两个劣势一般可以忽略，第一、磁盘很便宜，不怕索引多占用点空间；第二、一般的业务大部分操作都是查询，而不是增删改，所以维护索引的成本并不算太高。<br>
<br>


### 索引的结构

索引的数据结构默认为B+tree结构<br>
<br>

树的度数指的是一个节点的子节点个数。<br>
<br>

以一颗最大度数（max-degree）为5(5阶)的b-tree为例，那这个B树每个节点最多存储4个key，5个指针。这5个指针分别为：<20的指针，20-30之间的指针....

叶子结点是指下面那一排

在B-tree（也就是B树）中：
- 非叶子节点和叶子节点都会存放数据。
- 裂变时中间元素向上分裂。
<br>

在B+tree的特点：
- 所有的元素都会出现在叶子结点，非叶子节点主要起到索引作用（不存储数据）。
- 叶子结点形成了一个单向链表，每一个节点都会通过一个指针指向下一个节点，最终形成单向链表

b+树的非叶子节点仅仅记录地址，不记录具体内容，所有具体内容集中在叶子节点，这和索引的思想几乎一致<br>
<br>

索引中每一个节点都是储存在数据块当中，也叫页。<br>
<br>

自适应哈希功能指的是，mysql会根据我们的查询条件，在指定的条件下，自动地将B+树索引构建为哈希。<br>
<br>

为什么InnoDB存储引擎选择使用B+tree索引结构?
- 相对于二叉树，层级更少，搜索效率高；
- 对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
- 相对Hash索引，B+tree支持范围匹配及排序操作；


Meomery引擎的索引结构是Hash。<br>
<br>


### 索引分类

四种索引：
- 主键索引
- 唯一索引
- 常规索引
- 全文索引

当在某个字段上加了唯一约束的时候，它会自动的为这个字段去创建一个唯一索引。

在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种：
- 聚集索引（Clustered Index）：必须有且只有一个
- 二级索引(Secondary Index)

聚集索引选取规则:
- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。


特点：
- 聚集索引的叶子节点下挂的是这一行的数据 。
- 二级索引的叶子节点下挂的是该字段值对应的主键值。



### 索引语法
如果一个索引只关联了一个字段，那么这个索引称为**单列索引**；如果一个索引关联了多个字段，则称为**联合索引**或**组合索引**。<br>
<br>




### SQL性能分析

MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息。通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次：

```sql
-- session 是查看当前会话 ;
-- global 是查询全局数据 ;
SHOW GLOBAL STATUS LIKE 'Com_______';
```


加入设置慢查询日志的时间为2秒，那对于某些查询时间为1.8s、1.95秒之类的表，其效率也是比较低的，那我们也需要对这类的sql进行优化，那怎么定位到这类sql呢？慢查询日志满足不了，我们可以用profile详情。<br>
<br>

- desc 关键字：在MySQL中第一个是describe的缩写，desc 表名/查询语句，**相当于explain？**
- 第二个是descend的缩写，select * from 表 order by 字段 desc


多对多的关系需要用中间表来维护。<br>
<br>

explain后得到的表id不是自增的？如果id相同表示执行顺序是从上到下（相对于表），如果id值不同，则id值越大，优先级越高。

type：表示连接类型，性能由好到差的连接类型为NULL、system、const、eq_ref、ref、range、 index、all 
- 当查询不访问任何表时，type就为null，性能最高，一般的查询都不会是null
- system时相当于访问系统表
- 根据**主键**或者**唯一索引**进行访问，一般会出现const（唯一索引不是主键索引，唯一索引可以有多个）
- eq_ref在主键索引或者唯一性索引用于做被驱动表的连接字段时就会出现
- ref是指当我们使用非唯一性的索引进行查询时就会出现ref
- range 是使用了非唯一索引, 但是范围匹配


### 索引使用规则


**联合索引可以解决回表查询**（面试会问）<br>
<br>



#### 索引失效

- 联合索引中，出现范围查询(>,<)，范围查询右侧的列索引失效：这个右侧是指索引中的右侧，而不是做查询时写的条件之间的位置，查询时条件顺序与查询结果无关。改成>= 或 <= 时，就不会出现索引失效的情况了。
- 不要在索引列上进行运算操作，索引将失效。
- 字符串类型字段使用时，不加引号，索引将失效。
- 如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。
- 用or分割开的条件， 如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会
被用到。

<br>

在索引列上进行运算操作：

```sql
-- 失效
explain select * from tb_user where substring(phone,10,2) = '15';
```

这个查询用like不加%的话不会失效？<br>
<br>


如果MySQL评估使用索引比全表更慢，则不使用索引。<br>
<br>

只有要筛选的数据只占总数据的一小部分时才使用索引，如果用了索引还要查出大部分数据，则还不如全表查询更快，因为使用索引有可能会有回标的情况，所以即使是查询大部分数据也比不使用索引查询全表慢。<br>
<br>



#### SQL提示
SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

- use index ：建议MySQL使用哪一个索引完成此次查询（仅仅是建议，mysql内部还会再次进行评估）。
- ignore index ：忽略指定的索引。
- force index ：强制使用索引。


#### 覆盖索引
覆盖索引是指查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到。<br>
<br>

尽量使用覆盖索引，减少select *。<br>
<br>

使用覆盖索引后就不会回表了，所以效率更高。<br>
<br>


#### 前缀索引
当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。<br>
<br>



### 索引设计原则
- 针对于数据量较大，且查询比较频繁的表建立索引。
- 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引。
- 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
- 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
- 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
- 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
- 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。

<br>
<br>


## SQL优化

在MySQL中，每执行完一条语句就会进行事务的提交，即使后面还有其他的insert语句，所以建议手动提交事务。<br>
<br>

批量插入数据用load

```sql
-- 客户端连接服务端时，加上参数 -–local-infile
mysql –-local-infile -u root -p

-- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1;

-- 执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table tb_user fields
terminated by ',' lines terminated by '\n' ;
```

>> 在load时，主键顺序插入性能高于乱序插入


#### 主键优化
在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表(index organized table **IOT**)。<br>
<br>


1. 页分裂
页可以为空，也可以填充一半，也可以填充100%。每个页包含了2-N行数据(如果一行数据过大，会行溢出)，根据主键排列。<br>
<br>
为什么一定要最少2行数据？因为只有一行就是一个链表。<br>
<br>

2. 页合并

>> MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引时指定。


3. 索引设计原则
- 满足业务需求的情况下，尽量降低主键的长度：因为二级索引下面挂着主键，如果主键太长，在二级索引较多的情况下，既浪费磁盘空间，又要更多地磁盘IO时间
- 插入数据时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键：乱序插入会导致页分裂，消耗性能，效率较低
- 尽量不要使用UUID做主键或者是其他自然主键，如身份证号：因为这些主键不一定有序，也会导致页分裂，而且长度可能也较长
- 业务操作时，避免对主键的修改：因为修改了主键还需要修改其他索引，代价比较大



#### order by优化
反向排序后在extra项中会出现反向扫描索引，因为B+tree是升序构建的，叶子结点是升序的，倒序排就要反着扫叶子结点？<br>
<br>

**注意：在order by语句中，有先后顺序，一定要按着索引里面的先后顺序来写语句，如果反着，好像也能用索引（using index），但还会using filesort，效率要低些。** <br>
<br>

如果用 order by age asc, phone desc的话，也会出现using index，using filesort都有的情况，如果要解决的话，可以创建一个一升序一降序的索引，如下：

```sql
create index idx_user_age_phone_ad on tb_user(age asc ,phone desc);
```

**注意：上面说的只使using index出现的方法的前提是使用的是覆盖索引，如果是select \*，就必然变为using filesort，无论什么索引，也无论order by后面怎么写的**



rder by优化原则:
- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
- 尽量使用覆盖索引。
- 多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。
- 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小

如果在排序的时候缓冲区满了，就会在磁盘文件里进行排序，这时候性能就比较低了。所以，如果是在大数据量排序时，可以增大排序缓冲区。<br>
<br>

show variables like 'sort_buffer_size' (默认256k)。



#### group by优化
Using temporary，用到了临时表，性能相对较低。<br>
<br>

对于联合索引 idx_user_prof_age_sta ，当使用`select age, count(*) group by age`时走覆盖索引了，虽然用不了树结构的特点但是遍历叶节点比遍历聚集索引的叶节点快，也就是说如果没有联合索引，那么只会使用聚集索引来得到每一行（聚集索引下面挂的是每一行的数据），再在行中找到age数据，这样就比较慢。如果直接遍历联合索引的叶节点，则能直接得到age，少了从行中再得到age这一步，所以变快了，因此mysql会选择使用联合索引。<br>
<br>

**in()的括号里不能有limit** <br>
<br>



#### limit优化
优化思路: 一般分页查询时，通过创建 覆盖索引 能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。<br>
<br>


```sql
select count(1) from tb_user;
```

#### count优化

count(1)是什么意思？它表示我们查询返回的每一条记录它都会放一个1进去，然后在我们的服务层对数据进行累加<br>
<br>

按照效率排序的话，count(字段) < count(主键 id) < count(1) ≈ count(*)，所以尽量使用 count(*)。<br>
<br>


#### update优化
begin=start transaction


在执行update的时候，我们一定要根据索引字段进行更新，否则就会进行**行锁**升级为**表锁**，从而锁住整张表，一旦锁表了，并发性能就降低了。

索引字段是行锁，非索引字段是表锁

























