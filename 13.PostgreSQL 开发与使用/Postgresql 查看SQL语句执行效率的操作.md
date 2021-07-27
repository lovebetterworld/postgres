# Postgresql 查看SQL语句执行效率的操作

Explain命令在解决数据库性能上是第一推荐使用命令，大部分的性能问题可以通过此命令来简单的解决，Explain可以用来查看 SQL 语句的执行效 果，可以帮助选择更好的索引和优化查询语句，写出更好的优化语句。

## Explain语法：

```plsql
explain select … from … [where ...]
```

### 例如：

```plsql
explain select * from dual;
```

## 这里有一个简单的例子，如下：

```plsql
EXPLAIN SELECT * FROM tenk1;
               QUERY PLAN
----------------------------------------------------------------
   Seq Scan on tenk1 (cost=0.00..458.00 rows=10000 width=244)
```

### EXPLAIN引用的数据是：

1). 预计的启动开销(在输出扫描开始之前消耗的时间，比如在一个排序节点里做排续的时间)。

2). 预计的总开销。

3). 预计的该规划节点输出的行数。

4). 预计的该规划节点的行平均宽度(单位：字节)。

这里开销(cost)的计算单位是磁盘页面的存取数量，如1.0将表示一次顺序的磁盘页面读取。其中上层节点的开销将包括其所有子节点的开销。这里的输出行数(rows)并不是规划节点处理/扫描的行数，通常会更少一些。一般而言，顶层的行预计数量会更接近于查询实际返回的行数。

### 现在我们执行下面基于系统表的查询：

```
SELECT relpages, reltuples FROM pg_class WHERE relname = 'tenk1';
```

从查询结果中可以看出tenk1表占有358个磁盘页面和10000条记录，然而为了计算cost的值，我们仍然需要知道另外一个系统参数值。

```plsql
postgres=# show cpu_tuple_cost;
   cpu_tuple_cost
  ----------------
   0.01
  (1 row)
cost = 458(磁盘页面数) + 10000(行数) * 0.01(cpu_tuple_cost系统参数值
```

**补充：postgresql SQL COUNT(DISTNCT FIELD) 优化**

## 背景

统计某时段关键词的所有总数，也包含null （statistics 有400w+的数据，表大小为 600M），故

写出sql：

```plsql
select count(distinct keyword) +1 as count from statistics;
```

## 问题

虽然是后台查询，但是太慢了，执行时间为为 38.6s，那怎么优化呢？

## 解决

### 方法1（治标）

把这个定时执行，然后把sql结果缓存下，然后程序访问缓存结果，页面访问是快了些，但是本质上还没有解决sql执行慢的问题。

### 方法2（治本）

优化sql，首先说说 count( distinct FIELD) 为啥这么慢，此处不再赘述了，请看这篇：https://www.jb51.net/article/65680.htm

优化内容：

```plsql
select count( distinct FIELD ) from table
```

修改为

```plsql
select count(1) from (select distinct FIELD from table) as foo;
```

### 比较

执行过程比对，可以使用 explian anaylze sql语句 查看