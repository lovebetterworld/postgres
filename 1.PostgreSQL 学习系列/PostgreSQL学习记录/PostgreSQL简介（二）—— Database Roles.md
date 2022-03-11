- [PostgreSQL简介（二）—— Database Roles_迷途的攻城狮-CSDN博客](https://blog.csdn.net/chenleiking/article/details/81170414)

### 1、角色（Role）

[PostgreSQL](https://so.csdn.net/so/search?q=PostgreSQL&spm=1001.2101.3001.7020)使用role这一概念来控制数据库的访问权限。一个role可以看作是一个数据的user，或者是一组数据库的user，取决于你如何设置role。Role可以作为数据库对象（for example, tables and functions）的拥有者，将这些数据库对象的权限分配给其他role来控制数据库对象的访问权限。

- 管理Role

```shell
--- role表 ---
postgres=# \d pg_roles 
                         View "pg_catalog.pg_roles"
     Column     |           Type           | Collation | Nullable | Default 
----------------+--------------------------+-----------+----------+---------
 rolname        | name                     |           |          | 
 rolsuper       | boolean                  |           |          | 
 rolinherit     | boolean                  |           |          | 
 rolcreaterole  | boolean                  |           |          | 
 rolcreatedb    | boolean                  |           |          | 
 rolcanlogin    | boolean                  |           |          | 
 rolreplication | boolean                  |           |          | 
 rolconnlimit   | integer                  |           |          | 
 rolpassword    | text                     |           |          | 
 rolvaliduntil  | timestamp with time zone |           |          | 
 rolbypassrls   | boolean                  |           |          | 
 rolconfig      | text[]                   |           |          | 
 oid            | oid                      |           |          | 

--- 创建role ---
postgres=# create role myrole;
CREATE ROLE
postgres=# select rolname from pg_roles;
       rolname        
----------------------
 postgres
 pg_monitor
 pg_read_all_settings
 pg_read_all_stats
 pg_stat_scan_tables
 pg_signal_backend
 myrole
(7 rows)
--- 删除role ---
postgres=# drop role myrole;
DROP ROLE123456789101112131415161718192021222324252627282930313233343536
```

> 如果直接在shell中创建或删除role，可以执行 `createuser name` 和 `dropuser name` 。
>
> 为了启动数据库，数据库在安装完之后，会自动创建超级用户角色，默认情况下，这个操作用户角色与操作系统运行PostgreSQL的用户同名，通常，这个角色被命名为***postgres\***，在创建其他角色之前，必须先连接这个角色。

- Role Attributes

在定义Role时，可以为其指定额外的属性，这些属性决定了Role的操作权限和认证权限， 例如：

```shell
--- 只有设置了login属性的role可以用来登陆 ---
postgres=# create role myrole login;
CREATE ROLE
postgres=# select rolname, rolcanlogin from pg_roles;
       rolname        | rolcanlogin 
----------------------+-------------
 postgres             | t
 pg_monitor           | f
 pg_read_all_settings | f
 pg_read_all_stats    | f
 pg_stat_scan_tables  | f
 pg_signal_backend    | f
 myrole               | t
(7 rows)1234567891011121314
```

> Create user 等同于 create role … login;

```shell
--- superuser角色会绕过除了登陆之外的所有权限检查，也就是登陆之后就畅通无阻了 ---
postgres=# create role myrole login superuser;
CREATE ROLE
postgres=# select rolname, rolcanlogin, rolsuper from pg_roles where rolname = 'myrole';
 rolname | rolcanlogin | rolsuper 
---------+-------------+----------
 myrole  | t           | t
(1 row)12345678
--- 创建除superuser之外的role必须要为其赋予创建database的权限 ---
postgres=# create role myrole createdb;
CREATE ROLE
postgres=# select rolname, rolcreatedb from pg_roles where rolname = 'myrole';
 rolname | rolcreatedb 
---------+-------------
 myrole  | t
(1 row)12345678
--- 创建除superuser之外的role必须要为其赋予创建role的权限 ---
postgres=# create role myrole createdb createrole;
CREATE ROLE
postgres=# select rolname, rolcreatedb, rolcreaterole from pg_roles where rolname = 'myrole';
 rolname | rolcreatedb | rolcreaterole 
---------+-------------+---------------
 myrole  | t           | t
(1 row)12345678
```

> 一个具备createrole属性的role可以修改和删除其他role，但是无法操作superuser

```shell
--- 创建除superuser之外的role必须要为其赋予流复制（initiate streaming replication）的权限 ---
postgres=# create role myrole replication login;
CREATE ROLE
postgres=# select rolname, rolcreatedb, rolcreaterole, rolreplication from pg_roles where rolname = 'myrole';;
 rolname | rolcreatedb | rolcreaterole | rolreplication 
---------+-------------+---------------+----------------
 myrole  | f           | f             | t
(1 row)12345678
```

> 在授予replication权限的同时，必须连同login权限一同授予

```shell
--- 创建role时，为其指定密码 ---
postgres=# create role myrole password 'awesomePass';
CREATE ROLE
postgres=# select rolname, rolpassword from pg_roles where rolname = 'myrole';
 rolname | rolpassword 
---------+-------------
 myrole  | ********
(1 row)12345678
```

- User（Role with login）

具备login属性的role可以认为是PostgreSQL的User，可以使用该User登陆PostgreSQL。

```shell
--- 修改认证方式 ---
-bash-4.2$ pwd
/var/lib/pgsql/10/data
-bash-4.2$ vi pg_hba.conf 
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
# local   all             all                                     peer
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
host    all             all             192.168.0.0/16          password
--- 重启服务 ---
[root@node-db ~]# systemctl restart postgresql-10
--- 创建User ---
postgres=# create role role_1 login password '888888';
CREATE ROLE
--- 使用新User登陆 ---
postgres=# \q
-bash-4.2$ psql -U role_1
Password for user role_1: 
psql: FATAL:  database "role_1" does not exist
-bash-4.2$ psql -U role_1 postgres
Password for user role_1: 
psql (10.4)
Type "help" for help.
123456789101112131415161718192021222324252627
```

> - PostgreSQL默认的本地（local）登陆验证方式为peer ，peer方式需要映射操作系统用户，也就是每一个peer方式的PostgreSQL用户需要一个同名的操作系统用户
> - `psql -U user` 默认会登陆一个与user同名的数据库，如果数据库不存在就会提示 `psql: FATAL: database "user" does not exist` ，使用 `psql -U user database` 指定数据库名称

- Group role（Role Membership）

为了方便权限管理，PostgreSQL的Role可以包含其他Role，上层Role称为Group，被包含的Role称之为Group的Membership。为了便于理解，可以将Group当作是一组权限集合，一旦Group被赋给某个Role，该Role就成为了Group的Membership，Group内的Membership可以使用Group所具备的权限。

```shell
--- 创建role_1和role_2两个角色，将role_1赋给role_2，role_2就是role_1的membership，role_1是group ---
db_1=# create role role_1 createdb ;
CREATE ROLE
db_1=# create role role_2;
CREATE ROLE
db_1=# grant role_1 to role_2 ;
GRANT ROLE
--- 创建role_3，将role_2赋给role_3，角色之间可以相互授权，但不能形成循环授权 ---
db_1=# grant role_2 to role_3;
GRANT ROLE
db_1=# grant role_3 to role_1;
ERROR:  role "role_3" is a member of role "role_1"123456789101112
```

Membership有两种方式使用Group的权限：一种是切换到Group role，这样就将当前数据库操作的session变成了Group role，所有的操作都是以Group role身份进行，比如新创建的对象的owner就是Group role，类似Linux系统中的 `su` 操作；另一种是继承，通过设置角色的 `inherit` 属性，使Membership在被赋予Group时获得Group的权限，这样Membership就可以以自己的身份进行相应的操作。

```shell
--- 设置测试数据 ---
postgres=# create role role_1;
CREATE ROLE
postgres=# set role role_1;
SET
postgres=> create table table_1(a int primary key);
CREATE TABLE
postgres=> reset role;
RESET
postgres=# create role role_2;
CREATE ROLE
postgres=# set role role_2;
SET
postgres=> create table table_2(a int primary key);
CREATE TABLE
postgres=> reset role;
RESET
postgres=# select * from pg_tables where tablename like 'table%';
 schemaname | tablename | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity 
------------+-----------+------------+------------+------------+----------+-------------+-------------
 public     | table_1   | role_1     |            | t          | f        | f           | f
 public     | table_2   | role_2     |            | t          | f        | f           | f
(2 rows)
--- 通过继承来获得权限 ---
postgres=# create role role_3;
CREATE ROLE
postgres=# set role role_3;
SET
postgres=> select * from table_1;
ERROR:  permission denied for relation table_1
postgres=> select * from table_2;
ERROR:  permission denied for relation table_2
postgres=> reset role;
RESET
postgres=# grant role_1 to role_3;
GRANT ROLE
postgres=# set role role_3;
SET
postgres=> select * from table_1;
 a 
---
(0 rows)

postgres=> select * from table_2;
ERROR:  permission denied for relation table_2
--- 通过切换来获得权限 ---
postgres=> reset role;
RESET
postgres=# create role role_4 login password '888888' noinherit;
CREATE ROLE
postgres=# grant role_2 to role_4;
GRANT ROLE
postgres=# \q
-bash-4.2$ psql -U role_4 -d postgres
Password for user role_4: 
psql (10.4)
Type "help" for help.

postgres=> select * from table_2;
ERROR:  permission denied for relation table_2
postgres=> create table table_3(a int primary key);
CREATE TABLE
postgres=> set role role_1;
ERROR:  permission denied to set role "role_1"
postgres=> set role role_2;
SET
postgres=> select * from table_2;
 a 
---
(0 rows)
postgres=> create table table_4(a int primary key);
CREATE TABLE
postgres=> select * from pg_tables where tablename like 'table%';
 schemaname | tablename | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity 
------------+-----------+------------+------------+------------+----------+-------------+-------------
 public     | table_1   | role_1     |            | t          | f        | f           | f
 public     | table_2   | role_2     |            | t          | f        | f           | f
 public     | table_3   | role_4     |            | t          | f        | f           | f
 public     | table_4   | role_2     |            | t          | f        | f           | f
(4 rows)
--- 查看角色信息 ---
postgres=> \du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 role_1    | Cannot login                                               | {}
 role_2    | Cannot login                                               | {}
 role_3    | Cannot login                                               | {role_1}
 role_4    | No inheritance                                             | {role_2}123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990
```

> - `LOGIN`、 `SUPERUSER`、 `CREATEDB` 和 `CREATEROLE` 属于特殊权限，不能继承，但是依然可以切换Role后使用；
> - 删除Group或自动收回所有权限，但不会对Membership照成此外的任何影响；
> - 所有Role默认含有 `inherit` 属性，会自动继承Group的权限 ;

- 删除Role

删除Role之前先删除与该role关联的其他对象，否则无法删除：

```shell
--- 无法直接删除非空角色 ---
postgres=# drop role role_1;
ERROR:  role "role_1" cannot be dropped because some objects depend on it
DETAIL:  owner of table table_1
--- 转移权限之后删除 ---
postgres=# drop role role_3;
DROP ROLE
postgres=# drop role role_4;
ERROR:  role "role_4" cannot be dropped because some objects depend on it
DETAIL:  owner of table table_3
postgres=# alter table table_3 owner to role_2;
ALTER TABLE
postgres=# drop role role_4;
DROP ROLE
--- 批量转移权限 ---
postgres=> reset role;
RESET
postgres=# select * from pg_tables where tablename like 'table%';
 schemaname | tablename | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity 
------------+-----------+------------+------------+------------+----------+-------------+-------------
 public     | table_1   | role_1     |            | t          | f        | f           | f
 public     | table_2   | role_2     |            | t          | f        | f           | f
 public     | table_3   | role_2     |            | t          | f        | f           | f
 public     | table_4   | role_2     |            | t          | f        | f           | f
(4 rows)

postgres=# reassign owned by role_2 to role_1;
REASSIGN OWNED
postgres=# select * from pg_tables where tablename like 'table%';
 schemaname | tablename | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity 
------------+-----------+------------+------------+------------+----------+-------------+-------------
 public     | table_1   | role_1     |            | t          | f        | f           | f
 public     | table_2   | role_1     |            | t          | f        | f           | f
 public     | table_3   | role_1     |            | t          | f        | f           | f
 public     | table_4   | role_1     |            | t          | f        | f           | f
(4 rows)123456789101112131415161718192021222324252627282930313233343536
```

- 系统内置Role

![这里写图片描述](https://img-blog.csdn.net/20180723164228430?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW5sZWlraW5n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)