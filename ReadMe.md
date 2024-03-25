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
如果一个索引只关联了一个字段，那么这个索引称为**单列索引**；如果一个索引关联了多个字段，则称为**联合索引**或**组合索引**。















