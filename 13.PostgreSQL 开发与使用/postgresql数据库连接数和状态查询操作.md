# postgresql数据库连接数和状态查询操作

### 查看数据库的当前连接数和状态的几种方式：

只是能看出数据库服务是否正在运行和启动路径

```plsql
pg_ctl status
```

统计当前postgresql相关进程数，在大体上可以估算数据库的连接数，非精准，但是目前最常用的

```plsql
ps -ef |grep postgres |wc -l
```

包含本窗口的所有数据库连接数

```plsql
SELECT count(*) FROM pg_stat_activity；
```

不包含本窗口的所有数据库连接数，其中pg_backend_pid()函数的意思是当前进程相关的后台进程ID

```plsql
SELECT count(*) FROM pg_stat_activity WHERE NOT pid=pg_backend_pid();
```

数据库状态查询（类似于

Oracle 的 select open_mode from v$database;

```plsql
 select state from pg_stat_activity where datname = 'highgo';
```

**补充：postgres数据库最大连接数**

–当前总共正在使用的连接数

```plsql
postgres=# select count(1) from pg_stat_activity;
```

–显示系统允许的最大连接数

```plsql
postgres=# show max_connections;
```

–显示系统保留的用户数

```plsql
postgres=# show superuser_reserved_connections ;
```

–按照用户分组查看

```plsql
select usename, count(*) from pg_stat_activity group by usename order by count(*) desc;
```

