- [全方位比较PostgreSQL和MySQL](https://cloud.tencent.com/developer/article/1576728)

## 一、原文

https://www.enterprisedb.com/blog/postgresql-vs-mysql-360-degree-comparison

## 二、摘要

本文对MySQL和[PostgreSQL](https://cloud.tencent.com/product/postgresql?from=10680)进行详细的比较，方便选择。

1、为什么使用PostgreSQL

2、为什么使用MySQL

3、易用性

4、语法

5、数据类型

6、复制与集群

7、视图

8、触发器

9、存储过程

10、查询

11、分区

12、表的可伸缩性

13、NoSQL能力

14、安全

15、分析函数

16、GUI工具

17、性能

18、Adoption

19、最佳环境

## 三、PG vs MySQL：选择哪个？

PostgreSQL和MySQL都是最流行的开源[数据库](https://cloud.tencent.com/solution/database?from=10680)。MySQL被认为是世界上最流行的数据库，而PostgreSQL被认为是世界上最先进的数据库。MySQL并不完全符合SQL标准，并且很多PG上的特性并不支持。这就是为什么PG受到大量开发者喜欢的原因，并且现在PG越来越流行。

前几年，Oracle收购了MySQL，导致MySQL的出现两个版本：商业版和社区版。对于后者，由于Oracle控制了MySQL的开发，受到了广大使用者的批评。

PostgreSQL是世界上最受欢迎的数据库：他支持大量企业级特性和功能。PG由postgresql全球社区开发，该社区由一批优秀的开发人员组成，几十年来一直努力确保PG具有丰富的功能，并与其他开源、商业数据库竞争。社区也从世界各地的公司得到巨大贡献。

### 1、为什么使用PG

PG作为开源、功能丰富的数据库，可与Oracle展开竞争。开发者也会将PG当做NoSQL数据库来使用。在云中和本地部署使用PG非常简单，也可以在docker容器等各个平台使用。

PG完全支持ACID，对开发人员和DBA非常友好，是跨任何域的高并发事务、复杂应用程序最佳选择，可以满足基于WEB和移动的各种应用程序服务。PG也是一个非常好的数据仓库，用于大数据上运行复杂的报告查询。

### 2、为什么使用MySQL

MySQL具有社区版和商业版。商业版由Oracle管理。作为[关系型数据库](https://cloud.tencent.com/product/cdb-overview?from=10680)，部署和使用非常简单。但是对于SQL标准要求很高的应用不太合适。MySQL的集成能力也有限，很难成为异构数据库环境的一部分。

MySQL适用于简单web应用程序或者需要简单schema、SQL执行数据库操作的应用。对于处理大量数据的复杂应用来说，MySQL并不是一个很好的选择。

### 3、易用性

PG能够处理结构化和非结构化的数据、具备关系型数据库所有的特性。MySQL在SQL和特性方面的局限性可能会为其构建高效的RDBMS应用程序带来挑战。

### 4、语法

大部分数据库的SQL语法都比较相似。然而，MySQL并不支持所有的SQL。对于支持的SQL和其他数据库都比较相似。例如查询，PG和MySQL都是：

SELECT  FROM employees;

### 5、数据类型

MySQL和PG都支持许多数据类型，从传统的数据类型（integer、date、timestamp）到复杂类型（json、xml、text）。然而，在复杂实时数据查询下又有所不同。

PG不止支持传统数据类型：numeric、strings、date、decimal等，还支持非结构的数据类型：json、xml、hstore等以及网络数据类型、bit字符串，还有ARRAYS，地理数据类型。

MySQL不支持地理数据类型。

从9.2开始，PG支持json数据类型。相对于MySQL来说，PG对json的支持比较先进。他有一些json指定的操作符和函数，是的搜索json文本非常高效。9.4开始，可以以二进制的格式存储json数据，支持在该列上进行全文索引（GIN索引），从而在json文档中进行快速搜索。

从5.7开始，MySQL支持json数据类型，比PG晚。也可以在json列上建立索引。然而对json相关的函数的支持比较有限。不支持在json列上全文索引。由于MySQL对SQL支持的限制，在存储和处理json数据方面，MySQL不是一个很好的选择。

### 6、复制和集群

MySQL和PG都具有复制和集群的能力，能够确保数据操作水平分布。

MySQL支持主-备、一主多备的复制机制，通过SQLs即binlog保证将所有的数据传输到备机上。这也是复制只能是异步、半同步的原因。

优点：备机可以写。这就意味着一旦master崩溃了，slave可以马上接管，确保应用正常工作。DBAs需要确保slave变成主了，并且新的binlog复制到原主。当有很多长SQL时，复制会变得慢。

MySQL也支持NDB集群，即多主的复制机制。这种类型的复制对要求水平扩展的事务有利。

PG的复制和MySQL不同，他是基于WAL文件，使复制更加可靠、更快、更有利于管理。他也支持主备和一主多从的模式，包括级联复制形式。PG的复制成为流复制或物理复制，可以异步也可以同步。

默认情况下，复制时异步，Slave能够满足读请求。如果要求在备机上读到的数据和主机上一样，就需要设置同步复制。但是缺点是一旦备机上事务没有提交，主机就会hang住。

可以使用第三方工具Slony、Bucardo、Londiste、RubyRep等对表级别的复制进行归档。这些工具都是基于触发器的复制。PG也支持逻辑复制。最初通过pglogical扩展支持逻辑复制，从10开始内核支持逻辑复制。

### 7、视图

MySQL支持视图，视图下面通过SQL使用的表的个数限制为61。视图不存储物理数据，也不支持物化视图。简单SQL语句创建的视图可以更新，复杂SQL创建的视图不可以更新。

PG和MySQL类似。简单SQL创建的视图可更新，复杂的不行。但是可以通过RULES更新复杂的视图。PG支持物化视图和REFRESHED。

### 8、触发器

MySQL支持INSERT、UPDATE、DELETE上AFTER和BEFORE事件的触发器。触发器不同执行动态SQL语句和存储过程。

PG的触发器比较先进。支持AFTER、BEFORE、INSTEAD OF事件的触发器。如果在触发器唤醒时执行一个复杂的SQL，可以通过函数来完成。PG中的触发器可以动态执行函数：

```sql
CREATE TRIGGER audit
AFTER INSERT OR UPDATE OR DELETE ON employee
  FOR EACH ROW EXECUTE FUNCTION employee_audit_func();
```

### 9、存储过程

MySQL和PG都支持存储过程，但MySQL仅支持标准的SQL语法，而PG支持非常先进的存储过程。PG以带RETURN VOID子句的函数形式完成存储过程。PG支持的语言有很多：Ruby、Perl、Python、TCL、PL/pgSQL、SQL和JavaScript。而MySQL则没有这么多。

### 10、查询

使用MySQL时需要考虑的限制：

l 某些UPDATE SQL的返回值不符合SQL标准

```sql
mysql> select  from test;
+------+------+
| c | c1  |
+------+------+
|  10 |  100 |
+------+------+
1 row in set (0.01 sec)
mysql> update test set c=c+1, c1=c;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql>  select  from test;
+------+------+
| c | c1  |
+------+------+
|  11 |  11 |
+------+------+
1 row in set (0.00 sec)
```

预期的标准形式：

```sql
mysql>  select  from test;
+------+------+
| c | c1  |
+------+------+
|  11 |  10 |
+------+------+
```

l 不能执行的UPDATE或DELETE语句：

```sql
mysql> delete from test where c in (select t1.c from test t1, test t2 where t1.c=t2.c);
ERROR 1093 (HY000): 
```

l 子查询中不能使用LIMIT子句

```sql
mysql> select  from test where c in (select c from test2 where c<3 limit 1);
ERROR 1235 (42000): 
```

MySQL也不支持“LIMIT & IN/ALL/ANY/SOME子句”。同样也不支持FULL OUTER JOINS、INTERSECT、EXCEPT等。也不支持Partial索引、bitmap索引、表达式索引等。PG支持所有SQL标准的特性。对于需要写复杂SQL的开发者来说，PG是一个很好的选择。

### 11、分区

MySQL和PG都支持表分区，然而双方都有一些限制。

MySQL支持的分区类型有RANGE、LIST、HASH、KEY和COLUMNS（RANGE和LIST），也支持SUBPARTITIONING。然而DBA在使用时可能不太易用。

l MySQL8.0，只有innodb和NDB存储引擎支持表分区，其他存储引擎不支持。

l 如果分区key的列不是主键或者唯一键的一部分，那么就不可能对表进行分区。

l 从5.7.24开始，逐步取消支持将表分区放在表空间上，这意味着DBA无法平衡表分区和磁盘IO。

```sql
mysql> create table emp (id int not null, fname varchar (30), lname varchar(30), store_id int not null ) partition by range (store_id) ( partition p0 values less than (6) tablespace tbs, partition p1 values less than(20) tablespace tbs1, partition p2 values less than (40) tablespace tbs2);
ERROR 1478 (HY000): InnoDB : A partitioned table is not allowed in a shared tablespace.
mysql>
```

PG支持表分区继承和声明表分区。声明表分区在10引入，和MySQL类似，而表分区继承通过使用触发器和规则来完成。分区类型支持RANGE、LIST、HASH。限制：

l 和MySQL类似，声明表分区只能在主键和唯一键上

l 继承表分区，子表不能继承主键和唯一键。

l INSERT和UPDATE不能自动恒信到字表。

### 12、表的扩展性

表段变得越来越大时会造成性能问题，在这个表上的查询会占用更多资源，花费更多时间。MySQL和PG需考虑不同因素。

MySQL支持B+tree索引和分区，这些可以对大表提升性能。然而，由于不支持bitmap、partial和函数索引，DBA不能更好的进行调优。而且分区表不能放到不同表空间上，这也造成IO不能更好平衡。

PG的表达式索引、partial索引、bitmap索引和全文索引都可以提升大表的性能。PG的表分区和索引可以放到不同的磁盘上，能够更好提升表的扩展性。为实现水平表级别的扩展，可以使用citusdb、Greenplum、Netezza等。开源的PG不支持水平表分区，PostgresXC支持，但是他的性能不好。

### 13、存储

数据存储是数据库的一个关键能力。PG和MySQL都提供多种选项存储数据。

PG有一个通用的存储特性：表空间能够容纳表、索引、物化视图等物理对象。通过表空间，可以将对象进行分组并存储到不同物理位置，可以提升IO能力。PG12之前版本，不支持可拔插存储，12只支持可拔插架构。

MySQL和PG类似，未来具有表空间特性。他支持可拔插存储引擎。这是MySQL的一个优点。

### 14、支持的数据模型

关系型数据库的NoSQL能力能够帮助处理非结构化的数据，例如json、xml、text等。

MySQL的NoSQL能力比较有限。5.7引入了json数据类型，需要很长时间才能变得更加成熟。

PG具有丰富的json能力，未来3年内是需要NoSQL能力的开发者的一个很好的选择。Json和jsonb数据类型，使得PG对json操作更快更有效。同样可以在json数据列上建立B-tree索引和GIN索引。XML和HSTORE数据类型可以处理XML格式以及其他复杂text格式的数据。对空间数据类型的支持，使得PG是一个完整的多模型数据库。

### 15、安全性

数据库安全在未认证即可访问的数据库中扮演者很重要的角色。安全包括对象级别和连接级别。

MySQL通过ROLES和PRIVILEGES将访问权限付给数据库、对象和连接。每个用户都需要赋予连接权限。

```sql
GRANT ALL PRIVILEGES ON testdb. TO 'testuser@'192.168.1.1’ IDENTIFIED BY 'newpassword';
GRANT ALL PRIVILEGES ON testdb. TO 'testuser@'192.168.1.’ IDENTIFIED BY 'newpassword';
```

每次赋权时都需要指定密码，否则用户将不能连接。

MySQL同样支持SSL连接。可以和外部认证系统LDAP和PAM集成。是其企业版一部分。

PG使用GRANT命令通过ROLES和PRIVILEGES提供访问权限。连接认证比较简单，通过pg_hba.conf认证文件设置：

```sql
host  database  user  address  auth-method  [md5 or trust or reject]
```

PG开源版本同样支持SSL连接，可以和外部认证系统集成。

解析函数对一组行数据进行聚合。有两种类型的解析函数：窗口函数和聚合函数。聚合函数执行聚合并返回记录集合的一个聚合值（sum,avg,min,max等）；而解析函数返回每个记录的聚合值。MySQL和PG都支持多种聚合函数。MySQL8.0才支持窗口函数，PG很早就已经支持了。

PG支持的窗口函数：

| 函数名       | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| CUME_DIST    | Return the relative rank of the current row.                 |
| DENSE_RANK   | Rank the current row within its partition without gaps.      |
| FIRST_VALUE  | Return a value evaluated against the first row within its partition. |
| LAG          | Return a value evaluated at the row that is at a specified physical offset row before the current row within the partition. |
| LAST_VALUE   | Return a value evaluated against the last row within its partition. |
| LEAD         | Return a value evaluated at the row that is offset rows after the current row within the partition. |
| NTILE        | Divide rows in a partition as equally as possible and assign each row an integer starting from 1 to the argument value. |
| NTH_VALUE    | Return a value evaluated against the nth row in an ordered partition. |
| PERCENT_RANK | Return the relative rank of the current row (rank-1) / (total rows-1) |
| RANK         | Rank the current row within its partition with gaps.         |
| ROW_NUMBER   | Number the current row within its partition starting from 1. |

MySQL支持PG所有的窗口函数，除了以下限制：

l 窗口函数不能出现在UPDATE和DELETE中

l 窗口函数不支持DISTINCT

l 窗口函数不支持NESTED

### 16、图形界面工具

MySQL有Oracle的SQL Developer、MySQL workbench、dbeaver、omnidb等，监控工具有nagios、cacti、zabbix等。PG也可以使用Oracle的SQL Developer、pgAdmin、omnidb、dbeaver。监控工具有Nagios, Zabbix, and Cacti。

### 17、性能

MySQL数据库性能调优选项比较有限，很多索引类型都不支持。写一个高效的SQL语句具有挑战性。对于大规模数据，MySQL也不是个很好的选择。表空间仅支持innodb，并且无法容纳表分区。

PG非常适合任何类型的负载：OLTP，OLAP，数据仓库等。由于支持的索引类型比较多，可以更好的提升性能。PG也有选项采集数据库内存使用，分区表可以放到不同表空间平衡IO。

### 18、Adoption

PG是世界上最先进的开源数据库。 EnterpriseDB 和2ndQuadrant公司能够保证PG在世界范围上被更多用户使用。

MySQL表示RDBMS和ORDBMS应用的最佳选择。因为自从Oracle收购MySQL依赖，MySQL的采用率明显下降，开源领域的开发进度也受到冲击，招致MySQL用户的批评。

### 19、最佳环境

MySQL流行于LAMP栈，PG流行于LAPP栈。LAPP栈代表Linux、Apache、Postgres和Php/Python，并且越来越流行。LAMP栈代表 Linux Apache MySQL/MongoDB and Php/Python。