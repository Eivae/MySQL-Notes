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




