- [PostgreSQL简介（一）—— Getting Started_迷途的攻城狮-CSDN博客](https://blog.csdn.net/chenleiking/article/details/81170082)

## 1、What is PostgreSQL?

PostgreSQL是一个开源的对象关系型数据库，开发于加州伯克利计算机科学实验室。支持大部分标志SQL语法，并且提供很多现代功能：

- 复杂查询
- 外键
- 触发器
- 视图更新
- 事务完整性
- 多版本并发控制

此外，PostgreSQL还提供很多方式让用户自定义扩展，例如增加如下对象：

- 数据类型（data types）
- 函数（functions）
- 操作（operators）
- 统计函数（aggregate functions）
- 索引方法（index methods）
- 过程语言（procedural languages）

And because of the liberal license, PostgreSQL can be used, modified, and distributed by anyone free of charge for any purpose, be it private, commercial, or academic.

## 2、Installation

```shell
--- 安装yum.repo ---
[root@node-db ~]# yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm

--- 安装pg server ---
[root@node-db ~]# yum install postgresql10-server12345
```

> 安装完之后，会自动创建一个操作系统用户：postgres

```shell
--- 初始化数据库（Optionally） ---
[root@node-db ~]# /usr/pgsql-10/bin/postgresql-10-setup initdb
--- 启动数据库（Optionally） ---
[root@node-db ~]# systemctl start postgresql-10
--- 设置开机自启动（Optionally） ---
[root@node-db ~]# systemctl enable postgresql-10123456
```

> 安装参考：https://www.postgresql.org/download/linux/redhat/

## 3、Getting Started

### 3.1、创建数据库

```shell
[root@node-db ~]# createdb mydb
createdb: could not connect to database template1: FATAL:  role "root" does not exist12
```

> 原因：当前操作系统用户（root）并非pg用户

```shell
--- 切换用户 ---
[root@node-db ~]# su - postgres
Last login: Thu Jul 19 10:25:07 CST 2018 on pts/0
--- 创建数据库 ---
-bash-4.2$ createdb mydb
--- 删除数据库 ---
-bash-4.2$ dropdb mydb1234567
```

### 3.2、访问数据库

访问数据库可以使用交互式命令行工具psql，或者使用图形界面工具pgAdmin，在或者是自己开发一个自定义的管理应用。

```shell
--- 使用psql连接数据库 ---
-bash-4.2$ psql mydb
psql (10.4)
Type "help" for help.

mydb=# 
--- 查询数据库版本 ---
mydb=# select version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 10.4 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-28), 64-bit
(1 row)123456789101112
```

PostgreSQL有一些内在的命令，这些命令并非SQL命令，它们通常以”\”开头，例如：

```shell
--- 打印帮助信息 ---
mydb=# \h
--- 退出psql ---
mydb=# \q1234
```

### 3.3、SQL语法

PostgreSQL是对象关系型数据库管理系统，本质上还是关系型数据库，依然使用table来存储数据，数据在table中以row的形式存在，row有一组column组成，column的顺序是固定的，查询时，SQL不保证row的顺序；

- 建表

```shell
postgres=# CREATE TABLE weather (
postgres(#     city            varchar(80),
postgres(#     temp_lo         int,           -- low temperature
postgres(#     temp_hi         int,           -- high temperature
postgres(#     prcp            real,          -- precipitation
postgres(#     date            date
postgres(# );
CREATE TABLE
postgres=# 
postgres=# CREATE TABLE cities (
postgres(#     name            varchar(80),
postgres(#     location        point
postgres(# );
CREATE TABLE1234567891011121314
```

> PostgreSQL支持标准的SQL类型：`int`, `smallint`, `real`, `double precision`, `char(*N*)`, `varchar(*N*)`, `date`, `time`, `timestamp`, 和 `interval`，此外，用户还可以创建自定义类型。

- 删除表

```shell
postgres=# drop table weather;1
```

> PostgreSQL以分号结尾，且在SQL中除双引号外不区分大小写，回车并不代表输入结束。

- 插入数据

```shell
postgres=# INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
INSERT 0 1
postgres=# INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');
INSERT 0 1
postgres=# INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
postgres-#     VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
INSERT 0 1
postgres=# INSERT INTO weather (date, city, temp_hi, temp_lo)
postgres-#     VALUES ('1994-11-29', 'Hayward', 54, 37);
INSERT 0 112345678910
```

- 将数据写入文件

```shell
--- 将数据写入文件 ---
postgres=# copy weather to '/tmp/weather.txt';
COPY 3
postgres-# \q
-bash-4.2$ cat /tmp/weather.txt 
San Francisco   46  50  0.25    1994-11-27
San Francisco   43  57  0   1994-11-29
Hayward 37  54  \N  1994-11-29
--- 将数据打印在控制台 --- 
postgres=# copy weather to stdout;
San Francisco   46  50  0.25    1994-11-27
San Francisco   43  57  0   1994-11-29
Hayward 37  54  \N  1994-11-2912345678910111213
```

> copy写入文件时，必须指定绝对路径

- 从文件中加载数据

```shell
postgres=# copy weather from '/tmp/weather.txt';
COPY 312
```

- 简单查询

```shell
--- 使用*查询所有列 ---
postgres=# select * from weather;
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29
(6 rows)
--- 指定查询的列 ---
postgres=# SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
--- 四则运算 ---
postgres=# SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
--- 条件过滤 ---
postgres=# SELECT * FROM weather
postgres-#     WHERE city = 'San Francisco' AND prcp > 0.0;
--- 排序 ---
postgres=# SELECT * FROM weather
postgres-#     ORDER BY city;
--- 多字段排序 ---
postgres=# SELECT * FROM weather
postgres-#     ORDER BY city, temp_lo;
--- 去掉重复数据 ---
postgres=# SELECT DISTINCT city
postgres-#     FROM weather;
--- 去掉重复数据并排序 ---
postgres=# SELECT DISTINCT city
postgres-#     FROM weather
postgres-#     ORDER BY city;12345678910111213141516171819202122232425262728293031
```

> 用法与普通SQL大体一致
>
> tips：在psql中使用tab可以快速补全！

- 联合查询

```shell
postgres=# SELECT *
postgres-#     FROM weather, cities
postgres-#     WHERE city = name;
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location  
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(2 rows)
--- 指定查询列 ---
postgres=# SELECT city, temp_lo, temp_hi, prcp, date, location
postgres-#     FROM weather, cities
postgres-#     WHERE city = name;
--- 明确列所属表 ---
postgres=# SELECT weather.city, weather.temp_lo, weather.temp_hi,
postgres-#        weather.prcp, weather.date, cities.location
postgres-#     FROM weather, cities
postgres-#     WHERE cities.name = weather.city;
--- 内连接 ---
postgres=# SELECT *
postgres-#     FROM weather INNER JOIN cities ON (weather.city = cities.name);
--- 左外连接 ---
postgres=# SELECT *
postgres-#     FROM weather LEFT OUTER JOIN cities ON (weather.city = cities.name);
--- 使用别名 ---
postgres=# SELECT W1.city, W1.temp_lo AS low, W1.temp_hi AS high,
postgres-#     W2.city, W2.temp_lo AS low, W2.temp_hi AS high
postgres-#     FROM weather W1, weather W2
postgres-#     WHERE W1.temp_lo < W2.temp_lo
postgres-#     AND W1.temp_hi > W2.temp_hi;1234567891011121314151617181920212223242526272829
```

- 统计函数

```shell
postgres=# SELECT max(temp_lo) FROM weather;
 max 
-----
  46
(1 row)12345
```

> 和标准SQL一样，统计函数无法在where条件中使用

```shell
--- 分组函数 ---
postgres=# SELECT city, max(temp_lo)
postgres-#     FROM weather
postgres-#     GROUP BY city;
     city      | max 
---------------+-----
 Hayward       |  37
 San Francisco |  46
(2 rows)
--- 分组过滤 ---
postgres=# SELECT city, max(temp_lo)
postgres-#     FROM weather
postgres-#     GROUP BY city
postgres-#     HAVING max(temp_lo) < 40;1234567891011121314
```

- 更新数据

```shell
postgres=# UPDATE weather
postgres-#     SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
postgres-#     WHERE date > '1994-11-28';
UPDATE 21234
```

- 删除数据

```shell
postgres=# DELETE FROM weather WHERE city = 'Hayward';
DELETE 1
--- 删除指定表中所有数据 ---
postgres=# delete from weather ;
DELETE 212345
```

### 3.4、高级功能

- 视图

```shell
postgres=# CREATE VIEW myview AS
postgres-#     SELECT city, temp_lo, temp_hi, prcp, date, location
postgres-#         FROM weather, cities
postgres-#         WHERE city = name;
CREATE VIEW
postgres=# SELECT * FROM myview;
     city      | temp_lo | temp_hi | prcp |    date    | location  
---------------+---------+---------+------+------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | (-194,53)
(2 rows)1234567891011
```

- 外键

```shell
postgres=# drop view myview ;
DROP VIEW
postgres=# drop table weather ;
DROP TABLE
postgres=# drop table cities ;
DROP TABLE
postgres=# CREATE TABLE cities (
postgres(#         city     varchar(80) primary key,
postgres(#         location point
postgres(# );
CREATE TABLE
postgres=# 
postgres=# CREATE TABLE weather (
postgres(#         city      varchar(80) references cities(city),
postgres(#         temp_lo   int,
postgres(#         temp_hi   int,
postgres(#         prcp      real,
postgres(#         date      date
postgres(# );
CREATE TABLE
postgres=# INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28');
ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Berkeley) is not present in table "cities".1234567891011121314151617181920212223
```

- 事务

```shell
postgres=# select * from weather ;              
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
(2 rows)

--- 回滚 ---
postgres=# begin;
BEGIN
postgres=# update weather set prcp = prcp + 1;
UPDATE 2
postgres=# rollback;
ROLLBACK
postgres=# select * from weather ;
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
(2 rows)
--- 提交 ---
postgres=# begin;
BEGIN
postgres=# update weather set prcp = prcp + 1;
UPDATE 2
postgres=# commit;
COMMIT
postgres=# select * from weather;
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 1.25 | 1994-11-27
 San Francisco |      43 |      57 |    1 | 1994-11-29
(2 rows)
--- savepoint ---
postgres=# begin;
BEGIN
postgres=# update weather set prcp = prcp - 1;
UPDATE 2
postgres=# savepoint s1;
SAVEPOINT
postgres=# update weather set prcp = prcp + 100;
UPDATE 2
postgres=# rollback to s1;
ROLLBACK
postgres=# update weather set prcp = prcp + 2;
UPDATE 2
postgres=# commit;
COMMIT
postgres=# select * from weather;
     city      | temp_lo | temp_hi | prcp |    date    
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 2.25 | 1994-11-27
 San Francisco |      43 |      57 |    2 | 1994-11-29
(2 rows)123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354
```

- 分析函数（Window Functions）

```shell
--- 准备数据 ---
postgres=# CREATE TEMPORARY TABLE empsalary (
postgres(#     depname varchar,
postgres(#     empno bigint,
postgres(#     salary int,
postgres(#     enroll_date date
postgres(# );
CREATE TABLE
postgres=# INSERT INTO empsalary VALUES
postgres-# ('develop', 10, 5200, '2007-08-01'),
postgres-# ('sales', 1, 5000, '2006-10-01'),
postgres-# ('personnel', 5, 3500, '2007-12-10'),
postgres-# ('sales', 4, 4800, '2007-08-08'),
postgres-# ('personnel', 2, 3900, '2006-12-23'),
postgres-# ('develop', 7, 4200, '2008-01-01'),
postgres-# ('develop', 9, 4500, '2008-01-01'),
postgres-# ('sales', 3, 4800, '2007-08-01'),
postgres-# ('develop', 8, 6000, '2006-10-01'),
postgres-# ('develop', 11, 5200, '2007-08-15');
INSERT 0 10
--- 计算部门平均值，不分组 ---
postgres=# SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;
  depname  | empno | salary |          avg          
-----------+-------+--------+-----------------------
 develop   |    11 |   5200 | 5020.0000000000000000
 develop   |     7 |   4200 | 5020.0000000000000000
 develop   |     9 |   4500 | 5020.0000000000000000
 develop   |     8 |   6000 | 5020.0000000000000000
 develop   |    10 |   5200 | 5020.0000000000000000
 personnel |     5 |   3500 | 3700.0000000000000000
 personnel |     2 |   3900 | 3700.0000000000000000
 sales     |     3 |   4800 | 4866.6666666666666667
 sales     |     1 |   5000 | 4866.6666666666666667
 sales     |     4 |   4800 | 4866.6666666666666667
(10 rows)
--- 组内排序 ---
postgres=# SELECT depname, empno, salary,
postgres-#        rank() OVER (PARTITION BY depname ORDER BY salary DESC)
postgres-# FROM empsalary;
  depname  | empno | salary | rank 
-----------+-------+--------+------
 develop   |     8 |   6000 |    1
 develop   |    10 |   5200 |    2
 develop   |    11 |   5200 |    2
 develop   |     9 |   4500 |    4
 develop   |     7 |   4200 |    5
 personnel |     2 |   3900 |    1
 personnel |     5 |   3500 |    2
 sales     |     1 |   5000 |    1
 sales     |     4 |   4800 |    2
 sales     |     3 |   4800 |    2
(10 rows)
--- 求和、不分组 ---
postgres=# SELECT salary, sum(salary) OVER () FROM empsalary;
--- 组内求和 ---
postgres=# SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary;
--- 组内top2 ---
postgres=# SELECT depname, empno, salary, enroll_date
postgres-# FROM
postgres-#   (SELECT depname, empno, salary, enroll_date,
postgres(#           rank() OVER (PARTITION BY depname ORDER BY salary DESC, empno) AS pos
postgres(#      FROM empsalary
postgres(#   ) AS ss
postgres-# WHERE pos < 3;
--- 提取并复用公共部分 ---
postgres=# SELECT sum(salary) OVER w, avg(salary) OVER w
postgres-#   FROM empsalary
postgres-#   WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768
```

> WINDOW … AS … 这种用法有点意识，之前没用过～

- 继承（Inheritance）

继承的概念来自面向对象的数据库，这一概念开启了一种有趣的数据库设计的可能性。

```shell
--- 创建表 ---
postgres=# CREATE TABLE cities (
postgres(#   name       text,
postgres(#   population real,
postgres(#   altitude   int     -- (in ft)
postgres(# );
CREATE TABLE
--- 创建表，继承自另一张表 ---
postgres=# CREATE TABLE capitals (
postgres(#   state      char(2)
postgres(# ) INHERITS (cities);
CREATE TABLE
--- 插入数据 ---
postgres=# insert into cities values ('San Francisco', 7.24E+5, 63);
INSERT 0 1
postgres=# insert into cities values ('Las Vegas', 2.583E+5, 2174);
INSERT 0 1
postgres=# insert into cities values ('Mariposa', 1200, 1953);
INSERT 0 1
postgres=# insert into capitals values ('Sacramento', 3.694E+5, 30, 'CA');
INSERT 0 1
postgres=# insert into capitals values ('Madison', 1.913E+5, 845, 'WI');
INSERT 0 1
--- 查询 ---
postgres=# SELECT name, altitude FROM cities WHERE altitude > 500;
   name    | altitude 
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
 Madison   |      845
(3 rows)
postgres=# SELECT name, altitude FROM ONLY cities WHERE altitude > 500;
   name    | altitude 
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
(2 rows)
postgres=# select * from capitals;
    name    | population | altitude | state 
------------+------------+----------+-------
 Sacramento |     369400 |       30 | CA
 Madison    |     191300 |      845 | WI
(2 rows)12345678910111213141516171819202122232425262728293031323334353637383940414243
```

> 子表继承了父表的字段，父表保存子表的数据，Interesting～