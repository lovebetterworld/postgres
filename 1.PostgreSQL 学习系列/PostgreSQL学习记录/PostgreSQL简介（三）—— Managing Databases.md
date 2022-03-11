- [PostgreSQL简介（三）—— Managing Databases_迷途的攻城狮-CSDN博客](https://blog.csdn.net/chenleiking/article/details/81170589)

### 1、数据库（Databbase）

Database是SQL对象（table、function等）的集合，通常情况下，一个SQL对象仅属于一个数据库。也存在多个数据库共享同一个SQL对象的特殊情况，比如：***pg_database\***，整个PostgreSQL中的所有database都可以访问。更确切的说，database时schema的集合，然后schema包含table、function等SQL对象，所以PostgreSQL的完整层次结构：server、database、schema、table（或者其他类似的SQL对象）。

客户端程序在连接PostgreSQL服务时，必须指定需要访问的数据库名称。PostgreSQL不允许在一个连接中访问多个database，但是并不限制打开的connention数量。database是物理隔离的，访问权限控制在connection级别。

- 创建数据库

```shell
--- 创建数据库 ---
postgres=# create database db_1;
CREATE DATABASE
--- 创建数据库并为其指定DBA ---
postgres=# create database db_2 owner role_2;12345
```

> - 客户端连接PostgreSQL需要指定database，但是没有database就无法连接数据库，陷入死循环。所以在PostgreSQL部署好之后 `initdb` 命令会创建一个名为***postgres\***的初始数据库，通过连接这个数据库来为其他用户创建其他数据库；
> - 在不指定owner的情况下所创建的database，其owner为当前操作用户；
> - Owner可以删除自己的database，即使database中的SQL对象并不全部属于自己；
> - Owner无法查询自己的database中的不属于自己的table，也就是没有操作其他用户SQL对象的权限；

- 数据库模版（Template Databases）

创建数据库实际上是复制了一个已经存在的数据库。默认情况下，会复制标准的系统模版数据库：***template1\***。因此，你在template1上做的任何改变，都将传播到之后创建的所有以此为模版的数据库上。这种行为允许你定制自己的数据库模版，例如，在template1上创建一个存储过程，那么后续创建的所有数据都将可以执行这个存储过程。

除template1之外的另一个模版数据库：***template0\***，初始情况下与template1完全一致，其目的是为了保证原始模版不被修改，因此template0是不允许连接与修改的。

```shell
--- 根据指定模版创建数据库 ---
postgres=# create database db_1 template template0;
CREATE DATABASE
postgres=# create database db_3 template db_1;
CREATE DATABASE
--- pg_database的结构 ---
postgres=# \d pg_database;
               Table "pg_catalog.pg_database"
    Column     |   Type    | Collation | Nullable | Default 
---------------+-----------+-----------+----------+---------
 datname       | name      |           | not null | 
 datdba        | oid       |           | not null | 
 encoding      | integer   |           | not null | 
 datcollate    | name      |           | not null | 
 datctype      | name      |           | not null | 
 datistemplate | boolean   |           | not null | 
 datallowconn  | boolean   |           | not null | 
 datconnlimit  | integer   |           | not null | 
 datlastsysoid | oid       |           | not null | 
 datfrozenxid  | xid       |           | not null | 
 datminmxid    | xid       |           | not null | 
 dattablespace | oid       |           | not null | 
 datacl        | aclitem[] |           |          | 
Indexes:
    "pg_database_datname_index" UNIQUE, btree (datname), tablespace "pg_global"
    "pg_database_oid_index" UNIQUE, btree (oid), tablespace "pg_global"
Tablespace: "pg_global"123456789101112131415161718192021222324252627
```

> - 事实上，可以以任何已经存在的数据为模版创建新数据库，并非一定是template1或者template0；
> - pg_database中有两个重要的字段：
>   - datistemplate：表示当前database是一个模版数据库，任何人都可以以此为模版进行克隆，如果设置为false，仅superuser和owner可以进行克隆操作；
>   - datallowconn：设置为false，表示该数据库无法创建新的connection，已经存在的connection不受影响，template0通常被设置为false，以防止被修改。

- 删除数据库

```shell
--- 删除数据库 ---
db_1=# drop database db_3;
DROP DATABASE123
```

> 你无法删除当前处于连接状态的数据库，你可以连接其他任何数据库之后再执行删除命令

- 表空间（Tablespaces）

PostgreSQL的tablespace代表一个操作系统目录（directory），在创建SQL对象时，可以为其指定tablespace，表示将该对象存放在tablespace对应的目录下的文件中。tablespace可以用来规划PostgreSQL的磁盘布局，比如：软件安装的磁盘分区容量有限且无法扩展，tablespace可以创建在指定的另一块可用分区；或者是在建立索引时，将索引存放在一块SSD磁盘表空间，以加快检索速度；

```shell
--- 创建tablespace ---
-bash-4.2$ pwd
/var/lib/pgsql
-bash-4.2$ mkdir tbs_01
-bash-4.2$ ll
drwx------. 4 postgres postgres 51 Jul 18 17:17 10
drwxr-xr-x. 2 postgres postgres  6 Jul 23 12:55 tbs_01
postgres=# create tablespace tbs_01 location '/var/lib/pgsql/tbs_01';
CREATE TABLESPACE123456789
```

> - 创建tablespace必须为其指定一个已经存在，且owner为PostgreSQL操作系统用户的目录
> - 不要将tablespace指定到临时文件系统，tablespace损坏将影响整个PostgreSQL集群的可用性

```shell
--- 使用tablespace ---
postgres=# create table table_tbs_01(a int primary key) tablespace tbs_01;
CREATE TABLE
--- 设置默认表空间 ---
postgres=# set default_tablespace = tbs_01;
SET
postgres=# create table table_tbs_02(a int primary key);
CREATE TABLE
postgres=# create table table_tbs_03(a int primary key);
CREATE TABLE
postgres=# select * from pg_tables where tablename like 'table%';
 schemaname |  tablename   | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity 
------------+--------------+------------+------------+------------+----------+-------------+-------------
 public     | table_tbs_01 | postgres   | tbs_01     | t          | f        | f           | f
 public     | table_tbs_02 | postgres   | tbs_01     | t          | f        | f           | f
 public     | table_tbs_03 | postgres   | tbs_01     | t          | f        | f           | f
(3 rows)1234567891011121314151617
```

> - 通过*temp_tablespaces*可以设置默认临时表空间
> - 表空间一旦创建，可以在任何database中使用
> - 无法删除非空的表空间，删除前必须要删除所有数据库在此表空间内人所有对象

```shell
--- 删除表空间 ---
postgres=# drop tablespace tbs_01 ;
ERROR:  tablespace "tbs_01" is not empty
postgres=# drop table table_tbs_01;
DROP TABLE
postgres=# drop table table_tbs_02;
DROP TABLE
postgres=# drop table table_tbs_03;
DROP TABLE
postgres=# drop tablespace tbs_01;
DROP TABLESPACE
```