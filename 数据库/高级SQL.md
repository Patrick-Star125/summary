## 高级（中级）SQL

SQL（Structured Query Language）并不是全是对数据库的基础操作，高级的SQL语句可以让我们只需要关系需要怎样的数据，而不需要关系怎样构造查询路径。高端的数据库系统一般都会有一个极其复杂的查询优化器，其性能也是数据库系统的差距所在。

SQL本身的版本也在不断的迭代，其中SQL-92是最重要的版本，从逻辑上SQL分成三类，分别是Data Manipulation Language (DML)Data Definition Language (DDL)Data Control Language (DCL)，但目前主流版本的SQL一般是从以下方面来对SQL语句和特性进行分类的

* Aggregations + Group By
* String / Date / Time Operations
* Output Control + Redirection
* Nested Queries
* Common Table Expressions
* Window Functions

### Aggregations ＆ Group By

Aggregations函数接收一组tuples作为其输入，然后生成一个标量值作为其输出。Aggregation函数（几乎）只能在SELECT输出列表中使用。包括

* AVG(COL): The average of the values in coL
* MIN(COL): The minimum value in coL
* MAX(COL): The maximum value in coL
* COUNT (COL): The number of tuples in the relation

这些函数在菜鸟教程上都有详细的说明[SQL 函数 | 菜鸟教程 (runoob.com)](https://www.runoob.com/sql/sql-function.html)，因为这些部分都是有很多很好的资料，而且几乎都要记，所以这部分的笔记我就课程笔记的截图。

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222113711.png" style="zoom:80%;" />

这一部分表示如果是全选的话，星号、表名和1都是一样的意思，并且LIKE中%表示匹配任意数字，但是尽量不要用在最前面，因为这样DBMS不知道要匹配的是什么数字，依然会将整个表遍历一遍，很难优化。

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222114436.png"  />

作业题中也有Aggregation专题

~~~sql
SELECT CategoryName
     , COUNT(*) AS CategoryCount
     , ROUND(AVG(UnitPrice), 2) AS AvgUnitPrice
     , MIN(UnitPrice) AS MinUnitPrice
     , MAX(UnitPrice) AS MaxUnitPrice
     , SUM(UnitsOnOrder) AS TotalUnitsOnOrder
FROM Product INNER JOIN Category on CategoryId = Category.Id
GROUP BY CategoryId
HAVING CategoryCount > 10
ORDER BY CategoryId;
~~~

### **String / Date / Time Operations**

在不同的数据库中字符串的操作是有不同规定的，以下是一些数据库中字符串大小写和单双引号的使用规范

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222131859.png" style="zoom:80%;" />

Like用于字符串匹配，%和_是常见的字符串匹配运算符

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222132102.png" style="zoom:67%;" />

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222132426.png" style="zoom: 80%;" />

**Date ＆ Time**

关于Date ＆ Time的操作在不同系统中差异过大，这里只列举我知道的部分

**SQLite**

julianday函数：

### Output Control ＆ Redirection

**输出重定向**

将查询的结果重新到已有的一张表里或者创建一个新表，而不是直接输出到客户，这种操作就叫做输出重定向，有如下图所示的两种方式

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222133137.png" style="zoom:80%;" />

**输出控制**

就是说**ODER BY**和**LIMIT**

ODER BY在上一篇笔记说的比较清楚了

LIMIT就是切片输出，没什么好记的

除非我们使用带有限制的ORDER BY子句，否则DBMS可能会在每次调用查询时在结果中生成不同的元组，因为关系模型不会施加顺序。

### Nested Queries

子查询的意思就是将查询语句嵌套在查询语句中，在结构上和输出重定向很像。外部查询的范围包含在内部查询中（即，内部查询可以从外部查询访问属性），但不能反过来。内部查询几乎可以出现在查询的**任何部分**：

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222143534.png" style="zoom:80%;" />

子查询的结果表达式如下：

* ALL: Must satisfy expression for all rows in sub-query.
* ANY: Must satisfy expression for at least one row in sub-query.
* IN: Equivalent to =ANY().
* EXISTS: At least one row is returned.

### Window Functions

窗口函数类似于GROUP BY，其基本结构就是窗口函数名FUNC-NAME()和OVER语句。

~~~sql
SELECT ... FUNC-NAME(...) OVER (...) FROM tableName
~~~

窗口函数往往是聚合函数或者系统独有的函数，OVER表示怎么切分数据

~~~sql
SELECT cid,sid,
	ROW_NUMBER() OVER (PARTITION BY cid)
 FROM enrolled
  ORDER BY cid;
~~~

PARTITION BY表示基于什么col去切分，除此之外，OVER中还可以是ODER BY

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222144732.png" style="zoom:80%;" />

具体可以看这个例子，这段SQL的功能是找到每门课程GPA第二高的学生。

**IMPORTANT**：The DBMS computes RANK after the window function sorting, whereas it computes ROW NUMBER
before the sorting.

来看一个课后作业的例子

~~~sql
SELECT Id, OrderDate, PrevOrderDate, ROUND(julianday(OrderDate) - julianday(PrevOrderDate), 2)
    FROM (
           SELECT Id, OrderDate, LAG(OrderDate, 1, OrderDate) OVER (ORDER BY OrderDate ASC) AS PrevOrderDate
           FROM 'Order'
           WHERE CustomerId = 'BLONP'
           ORDER BY OrderDate ASC
           LIMIT 10
    );
~~~

[通俗易懂的学会：SQL窗口函数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/92654574)

### Common Table Expressions

在编写更复杂的查询时，公共表表达式（CTE）可以替代windows或嵌套查询。它们为用户在更大的查询中编写辅助语句提供了一种方法。CTE可以被视为一个临时表，其作用空间为单个查询，超出存在的SQL即失效。

~~~sql
WITH cteName (col1, col2) AS (
SELECT 1, 2
)
SELECT col1 + col2 FROM cteName;
~~~

用WITH ... AS ...语句来定义临时表，上面SQL的功能就是实现计算1+2

具体的用法可以看下面的这个博客[SQL With As 用法 - Niko12230 - 博客园 (cnblogs.com)](https://www.cnblogs.com/Niko12230/p/5945133.html)

这里有一个临时表的例子

~~~sql
WITH expenditures AS (
    SELECT
        IFNULL(c.CompanyName, 'MISSING_NAME') AS CompanyName,
        o.CustomerId,
        ROUND(SUM(od.Quantity * od.UnitPrice), 2) AS TotalCost
    FROM 'Order' AS o
    INNER JOIN OrderDetail od on od.OrderId = o.Id
    LEFT JOIN Customer c on c.Id = o.CustomerId
    GROUP BY o.CustomerId
),
quartiles AS (
    SELECT *, NTILE(4) OVER (ORDER BY TotalCost ASC) AS ExpenditureQuartile
    FROM expenditures
)
SELECT CompanyName, CustomerId, TotalCost
FROM quartiles
WHERE ExpenditureQuartile = 1
ORDER BY TotalCost ASC;
~~~

另外，CTEs的一个显著的特征是CTE是可以递归的，一个具体的例子有如下的SQL语句

~~~sql
#其实这一段我s
with p as (
    	select Product.Id, Product.ProductName as name
    	from Product
            inner join OrderDetail on Product.id = OrderDetail.ProductId
            inner join 'Order' on 'Order'.Id = OrderDetail.OrderId
            inner join Customer on CustomerId = Customer.Id
    	where DATE(OrderDate) = '2014-12-25' and CompanyName = 'Queen Cozinha'
    	group by Product.id
    ),
    c as (
            select row_number() over (order by p.id asc) as seqnum, p.name as name
            from p
    ),
    flattened as (
            select seqnum, name as name
            from c
            where seqnum = 1
            union all
            select c.seqnum, f.name || ', ' || c.name
        	#递归调用
            from c join
                    flattened f
                    on c.seqnum = f.seqnum + 1
    )
    select name from flattened
    order by seqnum desc limit 1;
~~~

