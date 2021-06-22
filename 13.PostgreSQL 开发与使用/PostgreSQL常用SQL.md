- [PostgreSQL常用SQL](https://blog.csdn.net/m0_38001814/article/details/114983502?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_v2~rank_aggregation-11-114983502.pc_agg_rank_aggregation&utm_term=postgresql%E6%98%BE%E7%A4%BA%E7%A9%BA%E9%97%B4%E6%95%B0%E6%8D%AE%E5%BA%93&spm=1000.2123.3001.4430)



# 授权

```plsql
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN schema public to "XXX_DMM_UPDATE";
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public to "XXX_DMM_UPDATE";
GRANT SELECT ON ALL TABLES IN schema public to "XXX_DMM_READONLY";
```

# 查看重复索引

```plsql
SELECT pg_size_pretty(SUM(pg_relation_size(idx))::BIGINT) AS SIZE,
 (array_agg(idx))[1] AS idx1, (array_agg(idx))[2] AS idx2,
 (array_agg(idx))[3] AS idx3, (array_agg(idx))[4] AS idx4
FROM (
  SELECT indexrelid::regclass AS idx, (indrelid::text ||E'\n'|| indclass::text ||E'\n'|| indkey::text ||E'\n'||
 	 COALESCE(indexprs::text,'')||E'\n' || COALESCE(indpred::text,'')) AS KEY
  FROM pg_index) sub
GROUP BY KEY HAVING COUNT(*)>1
ORDER BY SUM(pg_relation_size(idx)) DESC;
```

# 显示每张表上的索引

```plsql
SELECT
  t.tablename,
  indexname,
  c.reltuples AS num_rows,
  pg_size_pretty(pg_relation_size(quote_ident(t.tablename)::text)) AS table_size,
  pg_size_pretty(pg_relation_size(quote_ident(indexrelname)::text)) AS index_size,
  CASE WHEN indisunique THEN 'Y'
   ELSE 'N'
  END AS UNIQUE,
  idx_scan AS number_of_scans,
  idx_tup_read AS tuples_read,
  idx_tup_fetch AS tuples_fetched
FROM pg_tables t
LEFT OUTER JOIN pg_class c ON t.tablename=c.relname
LEFT OUTER JOIN
  ( SELECT c.relname AS ctablename, ipg.relname AS indexname, x.indnatts AS number_of_columns, idx_scan, idx_tup_read, idx_tup_fetch, indexrelname, indisunique FROM pg_index x
     JOIN pg_class c ON c.oid = x.indrelid
     JOIN pg_class ipg ON ipg.oid = x.indexrelid
     JOIN pg_stat_all_indexes psai ON x.indexrelid = psai.indexrelid )
  AS foo
  ON t.tablename = foo.ctablename
WHERE t.schemaname='public'
ORDER BY 1,2;
```

# 数据库中单个表的大小（不包含索引）

```plsql
select pg_size_pretty(pg_relation_size('表名'));
```

# 查出所有表（包含索引）并排序

```plsql
SELECT table_schema || '.' || table_name AS table_full_name, pg_size_pretty(pg_total_relation_size('"' || table_schema || '"."' || table_name || '"')) AS size
FROM information_schema.tables
ORDER BY
pg_total_relation_size('"' || table_schema || '"."' || table_name || '"') DESC limit 20
```

# 查出表大小按大小排序并分离data与index

```plsql
SELECT
table_name,
pg_size_pretty(table_size) AS table_size,
pg_size_pretty(indexes_size) AS indexes_size,
pg_size_pretty(total_size) AS total_size
FROM (
SELECT
table_name,
pg_table_size(table_name) AS table_size,
pg_indexes_size(table_name) AS indexes_size,
pg_total_relation_size(table_name) AS total_size
FROM (
SELECT ('"' || table_schema || '"."' || table_name || '"') AS table_name
FROM information_schema.tables
) AS all_tables
ORDER BY total_size DESC
) AS pretty_sizes
SELECT query, calls, total_time, total_time/calls, rows,
100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements ORDER BY total_time DESC LIMIT 5;
```

# 最耗IO SQL，单次调用最耗IO SQL TOP 5

```plsql
select userid, dbid, query from pg_stat_statements order by (blk_read_time+blk_write_time)/calls desc limit 5;  
```

# 总最耗IO SQL TOP 5

```plsql
select userid, dbid, query from pg_stat_statements order by (blk_read_time+blk_write_time) desc limit 5;  
```

# 最耗时 SQL，单次调用最耗时 SQL TOP 5

```plsql
select userid, dbid, query from pg_stat_statements order by mean_time desc limit 5;  
```

# 总最耗时 SQL TOP 5

```plsql
select userid, dbid, query from pg_stat_statements order by total_time desc limit 5;  
```

# 响应时间抖动最严重 SQL

```plsql
select userid, dbid, query from pg_stat_statements order by stddev_time desc limit 5;  
```

# 最耗共享内存 SQL

```plsql
select userid, dbid, query from pg_stat_statements order by (shared_blks_hit+shared_blks_dirtied) desc limit 5;  
```

# 最耗临时空间 SQL

```plsql
select userid, dbid, query from pg_stat_statements order by temp_blks_written desc limit 5;  
```

# 重置统计信息

pg_stat_statements是累积的统计，如果要查看某个时间段的统计，需要打快照

用户也可以定期清理历史的统计信息，通过调用如下SQL

```plsql
select pg_stat_statements_reset();
```

# 列出所有schema

```plsql
db_quhaizhou=# \dn
  List of schemas
  Name  |  Owner   
--------+----------
 public | postgres
(1 row)
```

# 列出所有的表空间

```plsql
db_quhaizhou=# \db
       List of tablespaces
    Name    |  Owner   | Location 
------------+----------+----------
 pg_default | postgres | 
 pg_global  | postgres | 
(2 rows)
```

备注：postgresql中的表空间就是对应一个目录，放在这个表空间的表，就是把表的数据文件放到这个表空间下

# 列出数据库中的所有角色或者用户

```plsql
db_quhaizhou=# \dg
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 quhaizhou |                                                            | {}
 
db_quhaizhou=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 quhaizhou |                                                            | {}
```

# 显示表的权限分配情况

```plsql
db_quhaizhou=# \z student
                              Access privileges
 Schema |  Name   | Type  | Access privileges | Column privileges | Policies 
--------+---------+-------+-------------------+-------------------+----------
 public | student | table |                   |                   | 
(1 row)
 
db_quhaizhou=# \dp student
                              Access privileges
 Schema |  Name   | Type  | Access privileges | Column privileges | Policies 
--------+---------+-------+-------------------+-------------------+----------
 public | student | table |                   |                   | 
(1 row)
```

# 切换字符集

```plsql
db_quhaizhou=# \encoding gbk;        
db_quhaizhou=# 
db_quhaizhou=# select * from student;
 id |                name                | number 
----+------------------------------------+--------
  2 |                                | 1023 
(1 row)
 
db_quhaizhou=# \encoding utf8;
db_quhaizhou=# 
db_quhaizhou=# select * from student;
 id |                name                | number 
----+------------------------------------+--------
  2 | 张三                               | 1023 
(1 row)
```
