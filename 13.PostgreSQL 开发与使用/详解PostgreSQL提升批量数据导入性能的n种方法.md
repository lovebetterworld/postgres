# 详解PostgreSQL提升批量数据导入性能的n种方法

关键字：批量数据导入，数据加载，大量插入，加快，提升速度
 多元化选择时代，人生里很多事物都是如此，凡事都没有一成不变的方式和方法。不管白猫黑猫，能抓老鼠的就是好猫，适合自己的就是最好的。
 提升批量数据导入的方法亦是如此，没有何种方法是最优的，应用任何方法前根据自己的实际情况权衡利弊，做出选择。
 批量导入数据之前，无论采取何种方式，务必做好相应的备份。
 导入完成后亦需对相应对象进行ANALYZE操作，这样查询优化器才会按照最新的统计信息生成正确的执行计划。

下面正式介绍提升批量数据导入性能的n种方法。

# 方法1：禁用自动提交。

```plsql
psql
\set AUTOCOMMIT off

其他
BEGIN;
执行批量数据导入
COMMIT;
```

# 方法2：设置表为UNLOGGED。

导入数据之前先把表改成UNLOGGED模式，导入完成后改回LOGGED模式。

```plsql
ALTER TABLE tablename SET UNLOGGED;
执行批量数据导入
ALTER TABLE tablename LOGGED;
```

优点：
 导入信息不记录WAL日志，极大减少io，提升导入速度。
 缺点：
 1.在replication环境下，表无法设置为UNLOGGED模式。
 2.导入过程一旦出现停电死机等会导致数据库不能干净关库的情况，数据库中所有UNLOGGED表的数据将丢失。

# 方法3：重建索引。

导入数据之前先删除相关表上的索引，导入完成后重新创建之。

```plsql
DROP INDEX indexname;
执行批量数据导入
CREATE INDEX ...;
```

查询表上索引定义的方法

```plsql
select * from pg_indexes where tablename ='tablename' and schemaname = 'schemaname';
```

# 方法4：重建外键。

导入数据之前先删除相关表上的外键，导入完成后重新创建之。

```plsql
select * from pg_indexes where tablename ='tablename' and schemaname = 'schemaname';
```

相关信息可查询pg_constraint。

# 方法5：停用触发器

导入数据之前先DISABLE掉相关表上的触发器，导入完成后重新ENABLE之。

```plsql
ALTER TABLE tablename DISABLE TRIGGER ALL; 
执行批量数据导入
ALTER TABLE tablename ENABLE TRIGGER ALL;
```

相关信息可查询pg_trigger。

# 方法6：insert改copy

COPY针对批量数据加载进行了优化。

```plsql
COPY ... FROM 'xxx';
```

# 方法7：单值insert改多值insert

减少sql解析的时间。

# 方法8：insert改PREPARE

通过使用PREPARE预备语句，降低解析消耗。

```plsql
PREPARE fooplan (int, text, bool, numeric) AS
 INSERT INTO foo VALUES($1, $2, $3, $4);
EXECUTE fooplan(1, 'Hunter Valley', 't', 200.00);
```

方法9：修改参数

增大maintenance_work_mem，增大max_wal_size。

方法10：关闭归档模式，降低wal日志级别。

修改archive_mode参数控制归档开启和关闭。降低wal_level值为minimal来减少日志信息记录。
 此法需要重启数据库，需要规划停机时间。此外如有replication备库，还需考虑对其影响。