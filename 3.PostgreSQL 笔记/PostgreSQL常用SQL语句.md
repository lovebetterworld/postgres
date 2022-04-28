- [30个实用SQL语句，玩转PostgreSQL - 掘金 (juejin.cn)](https://juejin.cn/post/7087007261154869278)

## 一、数据库连接

**1、获取数据库实例连接数**

```sql
select count(*) from pg_stat_activity;
```

**2、获取数据库最大连接数**

```sql
show max_connections
```

**3、查询当前连接数详细信息**

```sql
select * from pg_stat_activity;
```

**4、查询数据库中各个用户名对应的数据库连接数**

```sql
select usename, count(*) from pg_stat_activity group by usename; 
```

## 二、赋权操作

**1、为指定用户赋予指定表的select权限**

```sql
GRANT SELECT ON table_name TO username;
```

**2、修改数据库表所属的ownner**

```sql
alter table table_name owner to username;
```

**3、授予指定用户指定表的所有权限**

```sql
grant all privileges on table product to username
```

**4、授予指定用户所有表的所有权限**

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO username;
```

## 三、数据库表或者索引

**1、获取数据库表中的索引**

```sql
select * from pg_indexes where tablename = 'product'; 
```

**2、获取当前db中所有表信息**

```sql
 select * from pg_tables;
```

**3、查询数据库安装了哪些扩展**

```sql
select * from pg_extension; 
```

**4、查询数据库中的所有表及其描述**

```sql
select relname as TABLE_NAME ,col_description(c.oid, 0) as COMMENTS from pg_class c where relkind = 'r' and relname not like 'pg_%' and relname not like 'sql_%';
```

## 四、获取数据大小

**1、查询执行数据库大小**

```sql
select pg_size_pretty (pg_database_size('db_product'));
```

**2、查询数据库实例当中各个数据库大小**

```sql
select datname, pg_size_pretty (pg_database_size(datname)) AS size from pg_database;
```

**3、查询单表数据大小**

```sql
select pg_size_pretty(pg_relation_size('table_name')) as size;
```

**4、查询数据库表包括索引的**

```sql
select pg_size_pretty(pg_total_relation_size('table_name')) as size;
```

**5、查看表中索引大小**

```sql
select pg_size_pretty(pg_indexes_size('table_name'));
```

**6、获取各个表中的数据记录数**

```sql
select relname as TABLE_NAME, reltuples as rowCounts from pg_class where relkind = 'r' order by rowCounts desc
```

**7、查看数据库表对应的数据文件**

```sql
select pg_relation_filepath('product');
```

## 五、数据库分析

**1、查看数据库实例的版本**

```sql
select version();
```

**2、查看最新加载配置的时间**

```sql
select pg_conf_load_time();
```

**3、查看当前wal的buffer中有多少字节没有写入到磁盘中**

```sql
select pg_xlog_location_diff(pg_current_xlog_insert_location(),pg_current_xlog_location());
```

**4、查询最耗时的5个sql**

```sql
select * from pg_stat_statements order by total_time desc limit 5;
```

备注：需要开启pg_stat_statements

**5、获取执行时间最慢的3条SQL，并给出CPU占用比例**

```sql
SELECT substring(query, 1, 1000) AS short_query,
round(total_time::numeric, 2) AS total_time,
calls,
round((100 * total_time / sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 3;
```

**6、分析评估SQL执行情况**

```sql
EXPLAIN ANALYZE SELECT * FROM product
```

**7、查看当前长时间执行却不结束的SQL**

```
select datname, usename, client_addr, application_name, state, backend_start, xact_start, xact_stay, query_start, query_stay, replace(query, chr(10), ' ') as query from (select pgsa.datname as datname, pgsa.usename as usename, pgsa.client_addr client_addr, pgsa.application_name as application_name, pgsa.state as state, pgsa.backend_start as backend_start, pgsa.xact_start as xact_start, extract(epoch from (now() - pgsa.xact_start)) as xact_stay, pgsa.query_start as query_start, extract(epoch from (now() - pgsa.query_start)) as query_stay , pgsa.query as query from pg_stat_activity as pgsa where pgsa.state != 'idle' and pgsa.state != 'idle in transaction' and pgsa.state != 'idle in transaction (aborted)') idleconnections order by query_stay desc limit 5;
```

**8、查出使用表扫描最多的表**

```sql
select * from pg_stat_user_tables where n_live_tup > 100000 and seq_scan > 0 order by seq_tup_read desc limit 10;
```

**9、查询读取buffer最多的5个SQL**

```sql
select * from pg_stat_statements order by shared_blks_hit+shared_blks_read desc limit 5;
```

**10、获取数据库当前的回滚事务数以及死锁数**

```sql
select datname,xact_rollback,deadlocks from pg_stat_database
```

**11、查询指定表的慢查询**

```sql
select * from pg_stat_activity where query ilike '%<table_name>%' and query_start - now() > interval '10 seconds';
```

## 六、数据库备份

**1、备份postgres库并tar打包**

```sql
pg_dump -h 127.0.0.1 -p 5432 -U postgres -f postgres.sql.tar -Ft
```

**2、备份postgres库，转储数据为带列名的INSERT命令**

```sql
pg_dumpall -d postgres -U postgres -f postgres.sql --column-inserts
```

