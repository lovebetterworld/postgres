- [PostgreSQL 内存参数调优](https://blog.csdn.net/neweastsun/article/details/111768840)

本文讨论PostgreSQL中一些管理内存参数，并对每个参数提供参考值建议。

## 1. 概述

GUC: Grand Unified Configuration  是postgreSQL对数据库参数进行管理的机制。通常理解是对postgresql.conf文件中变量进行修改，或通过set命令对参数进行设置。本文对GUC参数中内存管理相关参数进行说明，用于提升数据库服务器的性能。所有这些参数位于数据库服务器配置管理文件postgresql.conf中($PDATA目录中)。主要包括下面四个参数。

- Shared_buffers (integer)
- Work_mem (integer)
- Maintenance_work_mem (integer)
- Effective_cache_size (integer)

## 2. 内存参数说明

### 2.1 shared_buffers (integer)

shared_buffers 参数决定多少内存用于数据库服务器缓存数据。在postgresql.conf的缺省值是128M。

```
#shared_buffers = 128MB
```

该值应该被设为整个机器内存的 15% ~ 25%。举例：如果机器RAM为32 GB，那么建议设置shared_buffers 为 8 GB。注意，调整该参数需要重启数据库服务器。

### 2.2 Work_mem (integer)

work_mem 参数设置提供内部排序和写入临时磁盘文件的hash操作的内存数。排序操作用在 order by,  distinct以及merge join 场景. Hash表操作用于hash joins 和 hash 聚集。  postgresql.conf中参数缺省值：

```
#work_mem = 4MB
```

正确设置work_mem参数值可避免磁盘交换从而加快查询速度。可以通过下面公式计算最优值：

```
Total RAM * 0.25 / max_connections
```

max_connections是一个GUC 参数，用于指定数据库服务器的最大并发连接数，缺省为 100 。

也可以直接给角色设置work_mem参数值:

```
postgres=# alter user test set work_mem='4GB';

ALTER ROLE
```

### 2.3 Maintenance_work_mem (integer)

maintenance_work_mem 参数设置维护操作的最大内存数，如 vacuum, create index, alter table add foreign key 操作。
 在postgresql.conf缺省值为:

```
#maintenance_work_mem = 64MB
```

建议设置值比work_mem值大，可以提升vacuum性能。通常设置为：

```
Total RAM * 0.05
```

### 2.4 Effective_cache_size (integer)

effective_cache_size参数有操作系统和数据库评估多少内存可用磁盘缓存，PostgreSQL查询计划决定它是否固定在RAM中。索引扫描最有可能用于较高的值;如果该值为低将使用顺序扫描。建议将effecve_cache_size设置为机器总RAM的50%。

跟多其他参数请参考 PostgreSQL 官方文档: https://www.postgresql.org/docs/12/runtime-config-resource.html。

## 3. 总结

本文介绍了PostgreSQL常用内存参数，并提供参考计算公式。