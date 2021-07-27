- [系统表详解](https://www.cnblogs.com/jianyungsun/p/6634861.html)

# 一、pg_class:

  该系统表记录了数据表、索引(仍然需要参阅pg_index)、序列、视图、复合类型和一些特殊关系类型的元数据。注意：不是所有字段对所有对象类型都有意义。

| **名字**       | **类型**  | **引用**          | **描述**                                                     |
| -------------- | --------- | ----------------- | ------------------------------------------------------------ |
| relname        | name      |                   | 数据类型名字。                                               |
| relnamespace   | oid       | pg_namespace.oid  | 包含这个对象的名字空间(模式)的OI。                           |
| reltype        | oid       | pg_type.oid       | 对应这个表的行类型的数据类型。                               |
| relowner       | oid       | pg_authid.oid     | 对象的所有者。                                               |
| relam          | oid       | pg_am.oid         | 对于索引对象，表示该索引的类型(B-tree，hash)。               |
| relfilenode    | oid       |                   | 对象存储在磁盘上的文件名，如果没有则为0。                    |
| reltablespace  | oid       | pg_tablespace.oid | 对象所在的表空间。如果为零，则表示使用该数据库的缺省表空间。(如果对象在磁盘上没有文件，这个字段就没有什么意义) |
| relpages       | int4      |                   | 该数据表或索引所占用的磁盘页面数量，查询规划器会借助该值选择最优路径。 |
| reltuples      | float4    |                   | 表中行的数量，该值只是被规划器使用的一个估计值。             |
| reltoastrelid  | oid       | pg_class.oid      | 与此表关联的TOAST表的OID，如果没有为0。TOAST表在一个从属表里"离线"存储大字段。 |
| reltoastidxid  | oid       | pg_class.oid      | 如果是TOAST表，该字段为它索引的OID，如果不是TOAST表则为0。   |
| relhasindex    | bool      |                   | 如果这是一个数据表而且至少有(或者最近有过)一个索引，则为真。它是由CREATE INDEX设置的，但DROP INDEX不会立即将它清除。如果VACUUM发现一个表没有索引，那么它清理 relhasindex。 |
| relisshared    | bool      |                   | 如果该表在整个集群中由所有数据库共享，则为真。               |
| relkind        | char      |                   | r = 普通表，i = 索引，S = 序列，v = 视图， c = 复合类型，s = 特殊，t = TOAST表 |
| relnatts       | int2      |                   | 数据表中用户字段的数量(除了系统字段以外，如oid)。在pg_attribute里肯定有相同数目的数据行。见pg_attribute.attnum. |
| relchecks      | int2      |                   | 表中检查约束的数量，参阅pg_constraint表。                    |
| reltriggers    | int2      |                   | 表中触发器的数量；参阅pg_trigger表。                         |
| relhasoids     | bool      |                   | 如果我们为对象中的每行都生成一个OID，则为真。                |
| relhaspkey     | bool      |                   | 如果该表存在主键，则为真。                                   |
| relhasrules    | bool      |                   | 如表有规则就为真；参阅pg_rewrite表。                         |
| relhassubclass | bool      |                   | 如果该表有子表，则为真。                                     |
| relacl         | aclitem[] |                   | 访问权限。                                                   |

 见如下应用示例：
 代码如下:

```plsql
  \#查看指定表对象testtable的模式
   postgres=# SELECT relname,relnamespace,nspname FROM pg_class  c,pg_namespace n WHERE relname = 'testtable' AND relnamespace = n.oid;
   relname  | relnamespace | nspname
  -------------+--------------+---------
   testtable  |     2200  | public
  (1 row)
  \#查看指定表对象testtable的owner(即role)。
  postgres=# select relname,rolname from pg_class c,pg_authid au where relname = 'testtable' and relowner = au.oid;
   relname  | rolname
  -------------+----------
   testtable  | postgres
  (1 row)
```

# 二、pg_attribute:

  该系统表存储所有表(包括系统表，如pg_class)的字段信息。数据库中的每个表的每个字段在pg_attribute表中都有一行记录。

| **名字**      | **类型** | **引用**     | **描述**                                                     |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| attrelid      | oid      | pg_class.oid | 此字段所属的表。                                             |
| attname       | name     |              | 字段名。                                                     |
| atttypid      | oid      | pg_type.oid  | 字段的数据类型。                                             |
| attstattarget | int4     |              | attstattarget控制ANALYZE为这个字段设置的统计细节的级别。零值表示不收集统计信息，负数表示使用系统缺省的统计对象。正数值的确切信息是和数据类型相关的。 |
| attlen        | int2     |              | 该字段所属类型的长度。(pg_type.typlen的拷贝)                 |
| attnum        | int2     |              | 字段的编号，普通字段是从1开始计数的。系统字段，如oid，是任意的负数。 |
| attndims      | int4     |              | 如果该字段是数组，该值表示数组的维数，否则是0。              |
| attcacheoff   | int4     |              | 在磁盘上总是-1，但是如果装载入内存中的行描述器中， 它可能会被更新为缓冲在行中字段的偏移量。 |
| atttypmod     | int4     |              | 表示数据表在创建时提供的类型相关的数据(比如，varchar字段的最大长度)。其值对那些不需要atttypmod的类型而言通常为-1。 |
| attbyval      | bool     |              | pg_type.typbyval字段值的拷贝。                               |
| attstorage    | char     |              | pg_type.typstorage字段值的拷贝。                             |
| attalign      | char     |              | pg_type.typalign字段值的拷贝。                               |
| attnotnull    | bool     |              | 如果该字段带有非空约束，则为真，否则为假。                   |
| atthasdef     | bool     |              | 该字段是否存在缺省值，此时它对应pg_attrdef表里实际定义此值的记录。 |
| attisdropped  | bool     |              | 该字段是否已经被删除。如果被删除，该字段在物理上仍然存在表中，但会被分析器忽略，因此不能再通过SQL访问。 |
| attislocal    | bool     |              | 该字段是否局部定义在对象中的。                               |
| attinhcount   | int4     |              | 该字段所拥有的直接祖先的个数。如果一个字段的祖先个数非零，那么它就不能被删除或重命名。 |

 见如应用示例：
 代码如下:

```plsql
  \#查看指定表中包含的字段名和字段编号。
   postgres=# SELECT relname, attname,attnum FROM pg_class c,pg_attribute  attr WHERE relname = 'testtable' AND c.oid = attr.attrelid;
   relname  | attname | attnum
  -------------+----------+--------
   testtable  | tableoid  |   -7
   testtable  | cmax    |   -6
   testtable  | xmax   |   -5
   testtable  | cmin    |   -4
   testtable  | xmin    |   -3
   testtable  | ctid     |   -1
   testtable  | i       |   1
  (7 rows)
  \#只查看用户自定义字段的类型
   postgres=# SELECT relname,attname,typname FROM pg_class c,pg_attribute  a,pg_type t WHERE c.relname = 'testtable' AND c.oid = attrelid AND  atttypid = t.oid AND attnum > 0;
   relname  | attname | typname
  -------------+----------+---------
   testtable  | i       | int4
  (7 rows)
```

# 三、pg_attrdef:

  该系统表主要存储字段缺省值，字段中的主要信息存放在pg_attribute系统表中。注意：只有明确声明了缺省值的字段在该表中才会有记录。

| **名字** | **类型** | **引用**            | **描述**                                  |
| -------- | -------- | ------------------- | ----------------------------------------- |
| adrelid  | oid      | pg_class.oid        | 这个字段所属的表                          |
| adnum    | int2     | pg_attribute.attnum | 字段编号，其规则等同于pg_attribute.attnum |
| adbin    | text     |                     | 字段缺省值的内部表现形式。                |
| adsrc    | text     |                     | 缺省值的人可读的表现形式。                |

 见如下应用示例：
代码如下:

```plsql
  \#查看指定表有哪些字段存在缺省值，同时显示出字段名和缺省值的定义方式
  postgres=# CREATE TABLE testtable2 (i integer DEFAULT 100);
  CREATE TABLE     
   postgres=# SELECT c.relname, a.attname, ad.adnum, ad.adsrc FROM  pg_class c, pg_attribute a, pg_attrdef ad WHERE relname = 'testtable2'  AND ad.adrelid = c.oid AND adnum = a.attnum AND attrelid = c.oid;
   relname  | attname | adnum | adsrc
  -------------+----------+---------+-------
   testtable2 | i      |     1 | 100
  (1 row)
```

# 四、pg_authid:

   该系统表存储有关数据库认证的角色信息，在PostgreSQL中角色可以表现为用户和组两种形式。对于用户而言只是设置了rolcanlogin标志的角色。由于该表包含口令数据，所以它不是公共可读的。PostgreSQL中提供了另外一个建立在该表之上的系统视图pg_roles，该视图将口令字段填成空白。

| **名字**      | **类型**    | **引用** | **描述**                                                     |
| ------------- | ----------- | -------- | ------------------------------------------------------------ |
| rolname       | name        |          | 角色名称。                                                   |
| rolsuper      | bool        |          | 角色是否拥有超级用户权限。                                   |
| rolcreaterole | bool        |          | 角色是否可以创建其它角色。                                   |
| rolcreatedb   | bool        |          | 角色是否可以创建数据库。                                     |
| rolcatupdate  | bool        |          | 角色是否可以直接更新系统表(如果该设置为假，即使超级用户也不能更新系统表)。 |
| rolcanlogin   | bool        |          | 角色是否可以登录，换句话说，这个角色是否可以给予会话认证标识符。 |
| rolpassword   | text        |          | 口令(可能是加密的)；如果没有则为NULL。                       |
| rolvaliduntil | timestamptz |          | 口令失效时间(只用于口令认证)；如果没有失效期，则为NULL。     |
| rolconfig     | text[]      |          | 运行时配置变量的会话缺省。                                   |

见如下应用示例：

代码如下:

```plsql
  \#从输出结果可以看出口令字段已经被加密。
  postgres=# SELECT rolname,rolpassword FROM pg_authid;
   rolname |       rolpassword
  -----------+-------------------------------------
   postgres | md5a3556571e93b0d20722ba62be61e8c2d
```



# 五、pg_auth_members:

  该系统表存储角色之间的成员关系。

| **名字**     | **类型** | **引用**      | **描述**                                         |
| ------------ | -------- | ------------- | ------------------------------------------------ |
| roleid       | oid      | pg_authid.oid | 组角色的ID。                                     |
| member       | oid      | pg_authid.oid | 属于组角色roleid的成员角色的ID。                 |
| grantor      | oid      | pg_authid.oid | 赋予此成员关系的角色的ID。                       |
| admin_option | bool     |               | 如果具有把其它成员角色加入组角色的权限，则为真。 |

 见如下应用示例：
代码如下:

```plsql
  \#1. 先查看角色成员表中有哪些角色之间的隶属关系，在当前结果集中只有一个成员角色隶属于一个组角色，
  \#  如果有多个成员角色隶属于同一个组角色，这样将会有多条记录。
  postgres=# SELECT * FROM pg_auth_members ;
   roleid | member | grantor | admin_option
  --------+--------+---------+--------------
   16446 | 16445 |   10  | f
  (1 row)
  \#2. 查看组角色的名字。
  postgres=# SELECT rolname FROM pg_authid a,pg_auth_members am WHERE a.oid = am.roleid;
   rolname
  \---------
   mygroup
  (1 row)
  \#3. 查看成员角色的名字。
  \#4. 如果需要用一个结果集获取角色之间的隶属关系，可以将这两个结果集作为子查询后再进行关联。
  postgres=# SELECT rolname FROM pg_authid a,pg_auth_members am WHERE a.oid = am.member;
   rolname
  \---------
   myuser
  (1 row)
```



# 六、pg_constraint:

  该系统表存储PostgreSQL中表对象的检查约束、主键、唯一约束和外键约束。

| **名字**      | **类型** | **引用**            | **描述**                                                  |
| ------------- | -------- | ------------------- | --------------------------------------------------------- |
| conname       | name     |                     | 约束名字(不一定是唯一的)。                                |
| connamespace  | oid      | pg_namespace.oid    | 包含这个约束的名字空间(模式)的OID。                       |
| contype       | char     |                     | c = 检查约束， f = 外键约束， p = 主键约束， u = 唯一约束 |
| condeferrable | bool     |                     | 该约束是否可以推迟。                                      |
| condeferred   | bool     |                     | 缺省时这个约束是否是推迟的？                              |
| conrelid      | oid      | pg_class.oid        | 该约束所在的表，如果不是表约束则为0。                     |
| contypid      | oid      | pg_type.oid         | 该约束所在的域，如果不是域约束则为0。                     |
| confrelid     | oid      | pg_class.oid        | 如果为外键，则指向参照的表，否则为0。                     |
| confupdtype   | char     |                     | 外键更新动作代码。                                        |
| confdeltype   | char     |                     | 外键删除动作代码。                                        |
| confmatchtype | char     |                     | 外键匹配类型。                                            |
| conkey        | int2[]   | pg_attribute.attnum | 如果是表约束，则是约束控制的字段列表。                    |
| confkey       | int2[]   | pg_attribute.attnum | 如果是外键，则是参照字段的列表。                          |
| conbin        | text     |                     | 如果是检查约束，则表示表达式的内部形式。                  |
| consrc        | text     |                     | 如果是检查约束，则是表达式的人可读的形式。                |

# 七、pg_tablespace:

  该系统表存储表空间的信息。注意：表可以放在特定的表空间里，以帮助管理磁盘布局和解决IO瓶颈。

| **名字**    | **类型**  | **引用**      | **描述**                             |
| ----------- | --------- | ------------- | ------------------------------------ |
| spcname     | name      |               | 表空间名称。                         |
| spcowner    | oid       | pg_authid.oid | 表空间的所有者，通常是创建它的角色。 |
| spclocation | text      |               | 表空间的位置(目录路径)。             |
| spcacl      | aclitem[] |               | 访问权限。                           |

 见下应用示例：

代码如下:

```plsql
 \#1. 创建表空间。
  postgres=# CREATE TABLESPACE my_tablespace LOCATION '/opt/PostgreSQL/9.1/mydata';
  CREATE TABLESPACE
  \#2. 将新建表空间的CREATE权限赋予public。
  postgres=# GRANT CREATE ON TABLESPACE my_tablespace TO public;
  GRANT
  \#3. 查看系统内用户自定义表空间的名字、文件位置和创建它的角色名称。
  \#4. 系统创建时自动创建的两个表空间(pg_default和pg_global)的文件位置为空(不是NULL)。
   postgres=# SELECT spcname,rolname,spclocation FROM pg_tablespace  ts,pg_authid a WHERE ts.spcowner = a.oid AND spclocation <> '';
    spcname  | rolname |    spclocation
  ---------------+----------+----------------------------
   my_tablespace | postgres | /opt/PostgreSQL/9.1/mydata
  (1 row)
```



# 八、pg_namespace:

  该系统表存储名字空间(模式)。

| **名字** | **类型**  | **引用**      | **描述**               |
| -------- | --------- | ------------- | ---------------------- |
| nspname  | name      |               | 名字空间(模式)的名称。 |
| nspowner | oid       | pg_authid.oid | 名字空间(模式)的所有者 |
| nspacl   | aclitem[] |               | 访问权限。             |

见如下应用示例：  

代码如下:

```plsql
  \#查看当前数据库public模式的创建者的名称。
  postgres=# SELECT nspname,rolname FROM pg_namespace n, pg_authid a WHERE nspname = 'public' AND nspowner = a.oid;
   nspname | rolname
  ----------+----------
   public  | postgres
  (1 row)
```



# 九、pg_database:

  该系统表存储数据库的信息。和大多数系统表不同的是，在一个集群里该表是所有数据库共享的，即每个集群只有一份pg_database拷贝，而不是每个数据库一份。

| **名字**      | **类型**  | **引用**          | **描述**                                                     |
| ------------- | --------- | ----------------- | ------------------------------------------------------------ |
| datname       | name      |                   | 数据库名称。                                                 |
| datdba        | oid       | pg_authid.oid     | 数据库所有者，通常为创建该数据库的角色。                     |
| encoding      | int4      |                   | 数据库的字符编码方式。                                       |
| datistemplate | bool      |                   | 如果为真，此数据库可以用于CREATE DATABASE TEMPLATE子句，把新数据库创建为此数据库的克隆。 |
| datallowconn  | bool      |                   | 如果为假，则没有人可以联接到这个数据库。                     |
| datlastsysoid | oid       |                   | 数据库里最后一个系统OID，此值对pg_dump特别有用。             |
| datvacuumxid  | xid       |                   |                                                              |
| datfrozenxid  | xid       |                   |                                                              |
| dattablespace | text      | pg_tablespace.oid | 该数据库的缺省表空间。在这个数据库里，所有pg_class.reltablespace为零的表都将保存在这个表空间里，特别要指出的是，所有非共享的系统表也都存放在这里。 |
| datconfig     | text[]    |                   | 运行时配置变量的会话缺省值。                                 |
| datacl        | aclitem[] |                   | 访问权限。                                                   |

# 十、pg_index:

  该系统表存储关于索引的一部分信息。其它的信息大多数存储在pg_class。

| **名字**       | **类型**   | **引用**            | **描述**                                                     |
| -------------- | ---------- | ------------------- | ------------------------------------------------------------ |
| indexrelid     | oid        | pg_class.oid        | 该索引在pg_class里的记录的OID。                              |
| indrelid       | oid        | pg_class.oid        | 索引所在表在pg_class里的记录的OID。                          |
| indnatts       | int2       |                     | 索引中的字段数量(拷贝的pg_class.relnatts)。                  |
| indisunique    | bool       |                     | 如果为真，该索引是唯一索引。                                 |
| indisprimary   | bool       |                     | 如果为真，该索引为该表的主键。                               |
| indisclustered | bool       |                     | 如果为真，那么该表在这个索引上建了簇。                       |
| indkey         | int2vector | pg_attribute.attnum | 该数组的元素数量为indnatts，数组元素值表示建立这个索引时所依赖的字段编号，如1 3，表示第一个字段和第三个字段构成这个索引的键值。如果为0，则表示是表达式索引，而不是基于简单字段的索引。 |
| indclass       | oidvector  | pg_opclass.oid      | 对于构成索引键值的每个字段，这个字段都包含一个指向所使用的操作符表的OID。 |
| indexprs       | text       |                     | 表达式树用于那些非简单字段引用的索引属性。它是一个列表，在indkey里面的每个零条目一个元素。如果所有索引属性都是简单的引用，则为空。 |
| indpred        | text       |                     | 部分索引断言的表达式树。如果不是部分索引， 则是空字串。      |

见如下应用示例：

代码如下:

```plsql
  \#查看该索引所在表的名称，以及构成该索引的键值数量和具体键值的字段编号。 
  postgres=# SELECT indnatts,indkey,relname FROM pg_index i, pg_class c WHERE c.relname = 'testtable2' AND indrelid = c.oid;
   indnatts | indkey | relname
  ----------+--------+------------
      2 | 1 3  | testtable2
  (1 row)
  \#查看指定表包含的索引，同时列出索引的名称。
   postgres=# SELECT t.relname AS table_name, c.relname AS index_name FROM (SELECT relname,indexrelid FROM pg_index i, pg_class c WHERE c.relname = 'testtable2' AND indrelid = c.oid) t, pg_index i,pg_class c WHERE  t.indexrelid = i.indexrelid AND i.indexrelid = c.oid;
   table_name |  index_name
  ------------+----------------
   testtable2 | testtable2_idx
  (1 row)
```