- [备份 PostgreSQL特定schema或table数据](https://blog.csdn.net/neweastsun/article/details/119887298)

除了创建独立数据库，PostgreSQL DBA 通常建议创建schema，因为PostgreSQL不支持跨库进行查询。在当前数据库中，你不能选择当前数据库服务器上的任何其他数据库的数据，如果要实现需要配置DB link。

因此大多数数据库用户会为不同应用场景创建不同的schema，本文针对这种应用场景介绍如何备份特定schema及表。

## 备份特定范围数据

### 备份schema

```sql
pg_dump -U postgres -d postgres --schema=public > back1.sql
```

### 备份指定表

```sql
pg_dump -U postgres -d postgres -t public.t_oil >t_oil.sql
```

另外还有其他几个常用参数：

```sql
pg_dump -U postgres -W -F t dvdrental > c:\pgbackup\dvdrental.tar
-U postgres` 指定用户连接数据库服务器，这里是 `postgres
```

`-W` 强制 pg_dump 在连接数据库服务器之前提示密码

`-F` 指定输入文件格式：

- `c`: 自定义归档文件格式
- `d`: 目录方式归档，创建目录包括每个表对应一个文件
- `t`: tar压缩文件格式
- `p`: 普通SQL文本

## 备份其他数据库对象

我们知道可以通过 pg_dumpall 命令备份全部数据库对象：

```sql
pg_dumpall -U postgres > c:\pgbackup\all.sql
```

我们也可以备份其他数据库特定对象，如角色、表空间、数据库、schema、table、索引等。下面给几个典型示例。

仅备份scheam定义：

```
pg_dumpall --schema-only > c:\pgdump\definitiononly.sql
```

仅备份角色定义：

```sql
pg_dumpall --roles-only > c:\pgdump\allroles.sql
```

仅备份表空间定义：

```sql
pg_dumpall --tablespaces-only > c:\pgdump\allroles.sql
```