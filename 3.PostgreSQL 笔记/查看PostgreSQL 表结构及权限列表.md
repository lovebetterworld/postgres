- [查看PostgreSQL 表结构及权限列表](https://blog.csdn.net/neweastsun/article/details/119669540)

> 本文我们解释如何查看PostgreSQL 表结构以及权限列表，即哪些用户对表拥有哪些权限。

## 查看表结构

### 使用命令查看

```sql
\d+ table_name;
```

示例：

```sql
test=# \d+ books
                                          数据表 "public.books"
  栏位  |  类型   | Collation | Nullable |              Default              |   存储   | 统计目标 | 描述
--------+---------+-----------+----------+-----------------------------------+----------+----------+------
 id     | integer |           | not null | nextval('books_id_seq'::regclass) | plain    |          |
 client | text    |           | not null |                                   | extended |          |
 data   | jsonb   |           | not null |                                   | extended |          |
索引：
    "books_pkey" PRIMARY KEY, btree (id)
```

### 使用SQL查看

首先通过PostgreSQL的元数据查看，另外也可以直接查看数据表。

1. 通过元数据 `information_schema.columns`

示例：

```sql
SELECT *
FROM information_schema.columns
where table_name ='books'
```

| table_catalog | table_schema | table_name | column_name | ordinal_position | column_default                    | is_nullable | data_type | character_maximum_length | character_octet_length | numeric_precision | numeric_precision_radix | numeric_scale | datetime_precision | interval_type | interval_precision | character_set_catalog | character_set_schema | character_set_name | collation_catalog | collation_schema | collation_name | domain_catalog | domain_schema | domain_name | udt_catalog | udt_schema | udt_name | scope_catalog | scope_schema | scope_name | maximum_cardinality | dtd_identifier | is_self_referencing | is_identity | identity_generation | identity_start | identity_increment | identity_maximum | identity_minimum | identity_cycle | is_generated | generation_expression | is_updatable |
| ------------- | ------------ | ---------- | ----------- | ---------------- | --------------------------------- | ----------- | --------- | ------------------------ | ---------------------- | ----------------- | ----------------------- | ------------- | ------------------ | ------------- | ------------------ | --------------------- | -------------------- | ------------------ | ----------------- | ---------------- | -------------- | -------------- | ------------- | ----------- | ----------- | ---------- | -------- | ------------- | ------------ | ---------- | ------------------- | -------------- | ------------------- | ----------- | ------------------- | -------------- | ------------------ | ---------------- | ---------------- | -------------- | ------------ | --------------------- | ------------ |
| test          | public       | books      | id          | 1                | nextval(‘books_id_seq’::regclass) | NO          | integer   |                          |                        | 32                | 2                       | 0             |                    |               |                    |                       |                      |                    |                   |                  |                |                |               |             | test        | pg_catalog | int4     |               |              |            |                     | 1              | NO                  | NO          |                     |                |                    |                  |                  | NO             | NEVER        |                       | YES          |
| test          | public       | books      | client      | 2                |                                   | NO          | text      |                          | 1073741824             |                   |                         |               |                    |               |                    |                       |                      |                    |                   |                  |                |                |               |             | test        | pg_catalog | text     |               |              |            |                     | 2              | NO                  | NO          |                     |                |                    |                  |                  | NO             | NEVER        |                       | YES          |
| test          | public       | books      | data        | 3                |                                   | NO          | jsonb     |                          |                        |                   |                         |               |                    |               |                    |                       |                      |                    |                   |                  |                |                |               |             | test        | pg_catalog | jsonb    |               |              |            |                     | 3              | NO                  | NO          |                     |                |                    |                  |                  | NO             | NEVER        |                       | YES          |

1. 直接查表，传输条件为false

```sql
test=# select * from books where 1=2;
 id | client | data
----+--------+------
(0 行记录)
```

上面介绍了几种方法查看表结构，下面我们查看哪些用户拥有表的权限列表。

## 查看表权限

### 使用命令查看

使用 `\z` 命令，后面跟上需要查看的表名称：

```sql
test=# \z books
                               存取权限
 架构模式 | 名称  |  类型  |         存取权限          | 列特权 | 策略
----------+-------+--------+---------------------------+--------+------
 public   | books | 数据表 | postgres=arwdDxt/postgres+|        |
          |       |        | alice=rw/postgres         |        |
(1 行记录)
```

列出两个用户的权限情况。

### 使用SQL查看

我们通过元数据视图 `information_schema.role_table_grants`查看，同时使用 `string_agg` 函数对用户进行分组：

```sql
SELECT grantee, string_agg( privilege_type,', ' ) as privilege_type
FROM information_schema.role_table_grants 
WHERE table_name='books'
group by grantee
```

返回结果：

| grantee  | privilege_type                                               |
| -------- | ------------------------------------------------------------ |
| alice    | SELECT, UPDATE                                               |
| postgres | UPDATE, DELETE, INSERT, REFERENCES, TRIGGER, TRUNCATE, SELECT |

列存两个用户不同的权限列表。

## 总结

本文介绍使用命令方式和SQL方式查看PostgreSQL表结构和权限。