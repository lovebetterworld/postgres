# PostgreSQL数据库中如何保证LIKE语句的效率(推荐)

在任何数据库中使用LIKE语句往往都是令人头疼的一件事，因为不少用户发现LIKE语句效率极低，查看执行计划后发现原来没有走索引，那么在Postgresql数据中LIKE语句的执行效率又是怎样的呢？我们又该如何提高LIKE语句的执行效率呢？

##  **实验环境**

数据库环境： PostgreSQL 12.3 X86_64 

创建虚拟环境：

```plsql
postgres=# create database testdb01 owner highgo;
CREATE DATABASE
postgres=# \c testdb01 highgo
 
 
testdb01=# create table testliketb01 (userid int primary key,username varchar(20),password varchar(60),description text);
CREATE TABLE
```

为何保证测试效果更直观，我们使用随机数据填充一下该表

```plsql
testdb01=# insert into testliketb01 select generate_series(1,500000),split_part('张三,李四,王五,小明,小红',',',(random()*(5-1)+1)::int),md5((random()*(5-1)+1)::varchar),split_part('highgo,highgo02,highgo03',',',(random()*(3-1)+1)::int);
```

 至此，虚拟数据创建完毕。

```plsql
testdb01=# select * from testliketb01 limit 10;
userid | username |             password             | description
--------+----------+----------------------------------+-------------
      1 | 王五     | 4f2bca371b42abd1403d5c20c4542dff | highgo
      2 | 李四     | 2a978c605188770c5ed162889fff189e | highgo02
      3 | 李四     | f5d129ab728b72ac6f663fe544bc7c16 | highgo
      4 | 小明     | 53134fa1022c58e65168b6aa1fbe5e39 | highgo02
      5 | 王五     | 2cf9abb2a8b676a626fa2c317d401ed8 | highgo02
      6 | 王五     | 2247a0cfda1f2819554d6e8e454622eb | highgo02
      7 | 张三     | 59dfdc680c17533dfba1c72c9ce0bf76 | highgo02
      8 | 王五     | 87db4258236a3826259dcc3e7cb5fc63 | highgo02
      9 | 王五     | baaf7a2f7027df9aaeb665121432b6e2 | highgo02
     10 | 王五     | 2f8fb36b3227c795b111b9bd5b031a76 | highgo02
(10 rows)
此时数据库的状态：
testdb01=# \l+ testdb01
                                                List of databases
   Name   | Owner  | Encoding |   Collate   |    Ctype    | Access privileges | Size  | Tablespace | Description
----------+--------+----------+-------------+-------------+-------------------+-------+------------+-------------
testdb01 | highgo | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                   | 59 MB | pg_default |
(1 row)
```

 简单LIKE语句查询：

```plsql
testdb01=# explain analyze select * from testliketb01 where username like '王%';
                                                      QUERY PLAN                                                       
-----------------------------------------------------------------------------------------------------------------------
Seq Scan on testliketb01  (cost=0.00..11405.00 rows=125350 width=52) (actual time=0.014..177.571 rows=124952 loops=1)
   Filter: ((username)::text ~~ '王%'::text)
   Rows Removed by Filter: 375048
Planning Time: 0.121 ms
Execution Time: 190.554 ms
(5 rows)
```

结论：LIKE查询没有走索引  创建普通索引： testdb01=# create index  idx_testliketb01_username on testliketb01(username); CREATE INDEX  执行三遍：analyze testliketb01 ; 重新执行LIKE语句，发现还是没有走索引   创建包含operator  class的索引： testdb01=# create index idx_testliketb01_username on  testliketb01(username varchar_pattern_ops); CREATE INDEX 执行三遍：analyze  testliketb01 ;   

```plsql
testdb01=# explain analyze select * from testliketb01 where username like '王%';
                                                                   QUERY PLAN                                                                    
-------------------------------------------------------------------------------------------------------------------------------------------------
Bitmap Heap Scan on testliketb01  (cost=2665.26..9387.14 rows=125350 width=52) (actual time=31.383..94.745 rows=124952 loops=1)
   Filter: ((username)::text ~~ '王%'::text)
   Heap Blocks: exact=5155
   ->  Bitmap Index Scan on idx_testliketb01_username  (cost=0.00..2633.92 rows=125350 width=0) (actual time=29.730..29.730 rows=124952 loops=1)
         Index Cond: (((username)::text ~>=~ '王'::text) AND ((username)::text ~<~ '玌'::text))
Planning Time: 0.111 ms
Execution Time: 107.030 ms
(7 rows)
```

结论：在创建完普通索引并收集统计信息后数据库在执行LIKE语句时有可能仍然无法使用索引。在创建完带有操作类的索引收集完统计信息后，执行LIKE语句可以看到正常使用索引，且执行效率有了不小提升。 

**PS：operator class是Postgresql新版中创建索引的新选项，旨在通过制定索引的操作类可以更精准的收集统计信息。** 

 为了更精准的收集统计信息，我们也可以在初始化或者创建数据库时将Collate设置为"C"，这也是Postgresql数据中常用的优化手段。  我们来测试一下将Collate设置为"C"的效果：

```plsql
testdb01=# create database testdb02 with TEMPLATE template0  LC_COLLATE='C'  LC_CTYPE ='C' owner highgo;
CREATE DATABASE
 
 
testdb02=# \l+ testdb02
                                           List of databases
   Name   | Owner  | Encoding | Collate | Ctype | Access privileges | Size  | Tablespace | Description
----------+--------+----------+---------+-------+-------------------+-------+------------+-------------
testdb02 | highgo | UTF8     | C       | C     |                   | 59 MB | pg_default |
(1 row)
 
 
testdb02=# create index idx_testliketb01_username on testliketb01(username);
CREATE INDEX
testdb02=# analyze testliketb01 ;
ANALYZE
testdb02=# analyze testliketb01 ;
ANALYZE
testdb02=# analyze testliketb01 ;
ANALYZE
testdb02=#  explain analyze select * from testliketb01 where username like '王%';
                                                                   QUERY PLAN                                                                    
-------------------------------------------------------------------------------------------------------------------------------------------------
Bitmap Heap Scan on testliketb01  (cost=2680.26..9410.67 rows=126033 width=52) (actual time=35.262..99.052 rows=124992 loops=1)
   Filter: ((username)::text ~~ '王%'::text)
   Heap Blocks: exact=5155
   ->  Bitmap Index Scan on idx_testliketb01_username  (cost=0.00..2648.75 rows=126033 width=0) (actual time=33.920..33.920 rows=124992 loops=1)
         Index Cond: (((username)::text >= '王'::text) AND ((username)::text < '玌'::text))
Planning Time: 0.276 ms
Execution Time: 111.578 ms
(7 rows)
```

结论：创建数据库时将Collate设置为"C"，即便索引为普通索引，LIKE语句也可以使用索引提升查询效率。  

## 优化建议：

1、初始化数据库或者创建数据库时将Collate设置为"C"。

2、创建索引时指定索引的操作类。（text_pattern_ops、varchar_pattern_ops和 bpchar_pattern_ops分别支持类型text、varchar和 char上的B-tree索引）

3、优化思路，对于%X的列无法使用索引，可以新增一列 反存储列，将%X改为X%。

4、创建覆盖索引，保证复杂SQL中可以尽可能调用该索引。

5、调整业务逻辑，尽量不用LIKE语句或者调整LIKE语句在WHERE中的位置。