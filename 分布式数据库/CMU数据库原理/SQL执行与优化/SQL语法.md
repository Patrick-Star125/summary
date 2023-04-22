## SQL

这一节并不是CMU-15445的课程，但是很多人包括我在学习CMU-15445的时候完全没有数据库的基础，因此学习基础SQL的使用对于学习Advanced SQL课程是必要的。

一个数据库通常包含一个或多个表。每个表有一个名字标识（例如:"Websites"）,表包含带有数据的记录（行）。

* SQL 对大小写不敏感：SELECT 与 select 是相同的。

大部分数据库要求在每条 SQL 语句的末端使用分号。分号是在数据库系统中分隔每条 SQL 语句的标准方法，这样就可以在对服务器的相同请求中执行一条以上的 SQL 语句。在本教程中，我们将在每条 SQL 语句的末端使用分号。

**一些最重要的 SQL 命令**

- **SELECT** - 从数据库中提取数据
- **UPDATE** - 更新数据库中的数据
- **DELETE** - 从数据库中删除数据
- **INSERT INTO** - 向数据库中插入新数据
- **CREATE DATABASE** - 创建新数据库
- **ALTER DATABASE** - 修改数据库
- **CREATE TABLE** - 创建新表
- **ALTER TABLE** - 变更（改变）数据库表
- **DROP TABLE** - 删除表
- **CREATE INDEX** - 创建索引（搜索键）
- **DROP INDEX** - 删除索引

### SELECT语句

用于提取数据库中的数据

SELECT使用

~~~sql
SELECT column_name,column_name FROM table_name;
SELECT * FROM table_name;
~~~

### SELECT DISTINCT 语句

在表中，一个列可能会包含多个重复值，有时您也许希望仅仅列出不同（distinct）的值。DISTINCT 关键词用于返回唯一不同的值。

SELECT DISTINCT使用

~~~sql
SELECT DISTINCT column_name,column_name FROM table_name;
~~~

### WHERE 子句

WHERE 子句用于提取那些满足指定条件的记录。

WHERE使用

~~~sql
SELECT column_name,column_name
FROM table_name
WHERE column_name operator value;
~~~

其中operator有以下几种

| 运算符  | 描述                                                       |
| :------ | :--------------------------------------------------------- |
| =       | 等于                                                       |
| <>      | 不等于。**注释：**在 SQL 的一些版本中，该操作符可被写成 != |
| >       | 大于                                                       |
| <       | 小于                                                       |
| >=      | 大于等于                                                   |
| <=      | 小于等于                                                   |
| BETWEEN | 在某个范围内                                               |
| LIKE    | 搜索某种模式                                               |
| IN      | 指定针对某个列的多个可能值                                 |

### AND & OR 运算符

AND & OR 运算符用于基于一个以上的条件对记录进行过滤。

如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

用AND ＆ OR联合实现嵌套operator

使用实例

~~~sql
SELECT * FROM Websites
WHERE alexa > 15
AND (country='CN' OR country='USA');
~~~

### ORDER BY 关键字

ORDER BY 关键字用于对结果集按照一个列或者多个列进行排序。

ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 **DESC** 关键字。

使用实例

~~~sql
SELECT * FROM Websites
ORDER BY country,alexa DESC;
~~~

在CMU课上也有例子

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222133531.png" style="zoom:80%;" />

### INSERT INTO 语句

INSERT INTO 语句用于向表中插入新记录。可以有两种编写形式。

第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：

~~~sql
INSERT INTO table_name
VALUES (value1,value2,value3,...);
~~~

第二种形式需要指定列名及被插入的值：

~~~sql
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);
~~~

使用实例

~~~sql
INSERT INTO Websites (name, url, alexa, country)
VALUES ('百度','https://www.baidu.com/','4','CN');
~~~

### UPDATE 语句

UPDATE 语句用于更新表中已存在的记录。

~~~sql
UPDATE table_name
SET column1=value1,column2=value2,...  //设定更改哪一页
WHERE some_column=some_value;  //定位到某一行
~~~

在更新记录时要格外小心！如果我们省略了WHERE子句，那么会将表中所有数据的column更改，执行没有 WHERE 子句的 UPDATE 要慎重，再慎重。

### DELETE 语句

DELETE 语句用于删除表中的行。

~~~sql
DELETE FROM table_name
WHERE some_column=some_value;
~~~

使用实例

~~~sql
DELETE FROM Websites
WHERE name='Facebook' AND country='USA';
~~~

可以在不删除表的情况下，删除表中所有的行。这意味着表结构、属性、索引将保持不变：

~~~sql
DELETE FROM table_name;
或
DELETE * FROM table_name;
~~~

**注释：**在删除记录时要格外小心！因为不能重来！	

### JOIN语句

引用并合并两张表可以说是在构造SQL语句中最常见的逻辑之一了，JOIN...ON语句用于合并两个表，其中ON后面加上合并条件

JOIN有四种方式，INNER、LEFT、RIGHT、FULL

**INNER JOIN**：内部连接就是只生成ON条件成立的行，效果上有些像交集

**LEFT JOIN**：左连接会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行

**RIGHT JOIN**：右连接与左连接类似，只返回右表

**FULL JOIN**：全连接保存两个表所有行的信息，

以MySQL为例，在MySQL中JOIN有

**AS语句**

as的作用就是更改别名，很多时候可以极大的提升代码可读性和表可读性

<img src="http://1.14.100.228:8002/images/2022/02/22/20220222202527.png" style="zoom: 67%;" />

### UNION语句

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

**UNION操作符**

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。

语法1 UNION

~~~sql
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
~~~

默认地，UNION 操作符选取不同的值。如果允许重复的值，使用 UNION ALL。

~~~sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
~~~

**INTERSECT操作符**



**EXCEPT操作符**



### FROM联合句式

**INNER-JOIN-ON句式**

先上例子

~~~sql
SELECT CompanyName, round(delayCnt * 100.0 / cnt, 2) AS pct
FROM (
      SELECT ShipVia, COUNT(*) AS cnt 
      FROM 'Order'
      GROUP BY ShipVia
     ) AS totalCnt
INNER JOIN (
            SELECT ShipVia, COUNT(*) AS delaycnt 
            FROM 'Order'
            WHERE ShippedDate > RequiredDate 
            GROUP BY ShipVia
           ) AS delayCnt
          ON totalCnt.ShipVia = delayCnt.ShipVia
INNER JOIN Shipper on totalCnt.ShipVia = Shipper.Id
ORDER BY pct DESC;
~~~



### SELECT联合句式

[(32条消息) sql中的 IF 条件语句的用法_霓虹深处-CSDN博客_sql中if语句的用法](https://blog.csdn.net/qq_36850813/article/details/80449860)

利用联合句式实现IF-ELSE逻辑

**CASE-WHEN-THEN-ELSE-END句式**

先上例子

```sql
SELECT Id, ShipCountry,
       CASE
              WHEN ShipCountry IN ('USA', 'Mexico','Canada')
              THEN 'NorthAmerica'
              ELSE 'OtherPlace'
       END
FROM 'Order'
WHERE Id >= 15445
ORDER BY Id ASC
LIMIT 20;
```

CASE语句有两种形式：第一种评估一个或多个条件，并返回第一个符合条件的结果。 如果没有条件是符合的，则返回ELSE子句部分的结果，如果没有ELSE部分，则返回NULL：

~~~sql
CASE
  WHEN condition1 THEN result1
  WHEN condition2 THEN result2
  WHEN conditionN THEN resultN
  ELSE result
END;
~~~

第二种CASE句法返回第一个value = compare_value比较结果为真的结果。 如果没有比较结果符合，则返回ELSE后的结果，如果没有ELSE部分，则返回NULL：

~~~sql
CASE compare_value
  WHEN condition1 THEN result1
  WHEN condition2 THEN result2
  WHEN conditionN THEN resultN
  ELSE result
END;
~~~

那么假如SELECT中所选的col有多个需要进行CASE-WHEN-THEN-ELSE-END句式来筛选呢

### 其它语句

**ALTER语句—修改表**

RENAME：修改表名`ALTER TABLE stu RENAME student`

CHANGE：修改属性名和数据类型`ALTER TABLE 表名 CHANGE 旧属性名 新属性名 新数据类型`

MODIFY：修改数据类型`ALTER TABLE 表名 MODIFY 字段名 数据类型`

ADD：添加属性`ALTER TABLE 表名 ADD 新属性名 数据类型 [约束条件] [FIRST|AFTER] 已存在属性名`

DROP：删除列`ALTER TABLE 表名 DROP 字段名`

**DESCRIBE**

描述表的结构：`DESCRIBE TABLE`

**UPDATE**

数据更新

~~~sql
UPDATE  表名
SET 字段名1 = 内容1,  字段名2 =  内容2, 字段名3 =  内容3
WHERE 过滤条件; 
~~~

**外键链接**

`CONSTRAINT 外键名 FOREIGN KEY (字段名) REFERENCES 主表名(主键名)`

### 其它函数

这里记录我遇到的一些其它函数

**substr函数**

用于截取字符串

用法：substr(string string,num start,num length);

select  substr(参数1，参数2，参数3)  from  表名

string为字符串；start为起始位置；length为长度。例如

~~~sql
select kename,substr(kename,1,locate('.',kename)) as subkename from  web_dev_api where 1;
~~~

**instr函数**

instr函数返回字符串str中子字符串substr第一次出现的位置，在sql中第一字符的位置是1,如果 str不含substr返回0。

用法：INSTR(C1,C2,I,J)

C1  被搜索的字符串，C2  希望搜索的字符串，I   搜索的开始位置,默认为1，J   出现的位置,默认为1

**round函数**

ROUND 函数用于把数值字段舍入为指定的小数位数

语法如下：

~~~sql
SELECT ROUND(column_name,decimals) FROM table_name  #column_name是要舍入的字段  decimals是规定要返回的小数位数
~~~

**NTILE函数**

是一个窗口函数，用于分桶，语法如下

~~~sql
NTILE(buckets) OVER (
    [PARTITION BY partition_expression, ... ]
    ORDER BY sort_expression [ASC | DESC], ...
)
~~~

`buckets` - 行划分的桶数。 存储桶可以是表达式或子查询，其计算结果为正整数。 它不能是一个窗口功能。

`PARTITION BY`子句将结果集的行分配到应用了`NTILE()`函数的分区中。

`ORDER BY`子句指定应用`NTILE()`的每个分区中行的逻辑顺序。

**Date函数**

专门用于提取日期的函数，用法是DATE(col)

### SQL语句设计方法

以下面这句SQL为例，该语句的功能需求为

对于数据库中的8种停产产品中的每一种，哪个客户首先订购了该产品？输出客户的CompanyName和ContactName，输出表格式要求为

产品名称|用户公司|用户联系人

~~~sql
SELECT pname, CompanyName, ContactName
FROM (					#第一层
      SELECT pname, min(OrderDate), CompanyName, ContactName
      FROM (		#第二层
            SELECT Id AS pid, ProductName AS pname
            FROM Product			#第三层
            WHERE Discontinued != 0
           ) as discontinued
      INNER JOIN OrderDetail on ProductId = pid			#这些JOIN都是第二层
      INNER JOIN 'Order' on 'Order'.Id = OrderDetail.OrderId
      INNER JOIN Customer on CustomerId = Customer.Id
      GROUP BY pid
    )
ORDER BY pname ASC;
~~~

得到语句需求后，我们从数据库层级来设计SQL语句，每一层在语句上要有所分层。总体设计依照SELECT-FROM结构，计算在SELECT后，而构建表在FROM后，操作（排序、分组）表在SELECT-FROM之后。

作业第九题也是体现了表架构为重点的逻辑

~~~sql
SELECT RegionDescription, FirstName, LastName, bday
FROM
(
  SELECT RegionId AS rid, MAX(Employee.Birthdate) AS bday
  FROM Employee
    INNER JOIN EmployeeTerritory ON Employee.Id = EmployeeTerritory.EmployeeId
    INNER JOIN Territory ON TerritoryId = Territory.Id
  GROUP BY RegionId
)
INNER JOIN (
            SELECT FirstName, LastName, Birthdate, RegionId, EmployeeId
            FROM Employee
              INNER JOIN EmployeeTerritory ON Employee.Id = EmployeeTerritory.EmployeeId
              INNER JOIN Territory ON TerritoryId = Territory.Id
           )
           ON Birthdate = bday
INNER JOIN Region ON Region.Id = RegionId
GROUP BY EmployeeId
ORDER BY rid;
~~~

事实上，SQL语句的查询路径即是语句设计思路，也是实际上的底层查询思路，例如在CTE递归中

~~~sql
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
~~~

明显能够发现from是比select要先执行的，因此会进入一种递归的状态
























