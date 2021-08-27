- [PostgreSQL 表空间(TABLESPACE)](https://blog.csdn.net/neweastsun/article/details/113784173)

## 1. 表空间介绍

表空间即PostgreSQL存储数据文件的位置，其中包括数据库对象。如，索引、表等。
 PostgreSQL使用表空间映射逻辑名称和磁盘物理位置。默认提供了两个表空间：

- pg_default 表空间存储用户数据.
- pg_global 表空间存储全局数据.

利用表空间可以控制PostgreSQL的磁盘布局，它有两方面的优势：

首先，如果集群中的某个分区超出初始空间，可以在另一个分区上创建新的表空间并使用。后期可以重新配置系统。

其次，可以使用统计优化数据库性能。举例，可以把频繁访问的索引或表放在高性能的磁盘上，如固态硬盘；把归档数据放在较慢的设备上。

## 2. 创建表空间

下面介绍如何创建表空间，并通过示例进行学习。

### 2.1 创建表空间语句

使用`CREATE TABLESPACE`语句创建表空间，语法如下：

```plsql
CREATE TABLESPACE tablespace_name
OWNER user_name
LOCATION directory_path;
```

表空间名称不能以`pg`开头，因为这些名称为系统表空间保留。默认执行`CREATE TABLESPACE`语句的用户是表空间的拥有者。如果需要给其他用户赋权，可以值后面指定`owner`关键词。

`directory_path`是表空间使用空目录的绝对路径，PostgreSQL的用户必须拥有该目录的权限可以进行读写操作。

一旦创建好表空间，可以在CREATE DATABASE, CREATE TABLE 和 CREATE INDEX 语句中使用。

### 2.2 示例

下面语句创建新的表空间`ts_primary`：

```plsql
CREATE TABLESPACE ts_primary 
LOCATION 'e:\pg-data\primary';
```

上面示例使用unix风格斜杠作为目录路径，该目录必须要存在。列出所有表空间使用`\db`:

```plsql
postgres=# \db
                 表空间列表
    名称    |  拥有者  |       所在地
------------+----------+--------------------
 pg_default | postgres |
 pg_global  | postgres |
 ts_primary | postgres | E:\pg-data\primary
(3 行记录)
```

读者可以使用`\db+`查看更详细的表空间信息。

```plsql
postgres=# \db+
                                  表空间列表
    名称    |  拥有者  |       所在地       | 存取权限 | 选项 |  大小   | 描述
------------+----------+--------------------+----------+------+---------+------
 pg_default | postgres |                    |          |      | 6558 MB |
 pg_global  | postgres |                    |          |      | 575 kB  |
 ts_primary | postgres | E:\pg-data\primary |          |      | 7513 kB |
(3 行记录)
```

下面语句创建`logistics`数据库并使用`ts_primary`表空间：

```plsql
CREATE DATABASE logistics 
TABLESPACE ts_primary;
```

TABLESPACE子句指定要使用的表空间。

下面在logistics数据库中创建表并插入一行数据：

```plsql
CREATE TABLE deliveries (
    delivery_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, 
    order_date DATE, 
    customer_id INT
);

INSERT INTO deliveries(order_date, customer_id)
VALUES('2020-08-01',1);
```

既然`ts_primary`表空间上已经有了数据，我们进行查看并验证：

```plsql
\db+ ts_primary
```

输出结果：

```plsql
postgres=# \db+ ts_primary
                                  表空间列表
    名称    |  拥有者  |       所在地       | 存取权限 | 选项 |  大小   | 描述
------------+----------+--------------------+----------+------+---------+------
 ts_primary | postgres | E:\pg-data\primary |          |      | 7513 kB |
(1 行记录)
```

## 3. 总结

本文介绍了如何创建表空间。表空间是存储设备的具体位置，PostgreSQL用其实际存储数据文件。