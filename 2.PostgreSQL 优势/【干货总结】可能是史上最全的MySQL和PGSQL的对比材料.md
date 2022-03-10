- [【干货总结】:可能是史上最全的MySQL和PGSQL的对比材料](https://www.cnblogs.com/lyhabc/p/11628042.html)

运维了MySQL和PGSQL已经有一段时间了，最近接到一个数据库选型需求，于是便开始收集资料整理了一下，然后就有了下面的对比表

**关键词**：PostgreSQL 11、MySQL5.7

比较版本：PostgreSQL 11  **VS**   MySQL5.7（innodb引擎） Oracle官方社区版

版权情况：PostgreSQL 11（免费开源）、MySQL5.7 Oracle官方社区版（免费开源）

------

**1. CPU限制**

PGSQL

没有CPU核心数限制，有多少CPU核就用多少

MySQL

能用128核CPU，超过128核用不上

------

 **2. 配置文件参数**

PGSQL

一共有255个参数，用到的大概是80个，参数比较稳定，用上个大版本配置文件也可以启动当前大版本数据库

MySQL

一共有707个参数，用到的大概是180个，参数不断增加，就算小版本也会增加参数，大版本之间会有部分参数不兼容情况

------

**3. 第三方工具依赖情况**

PGSQL

只有高可用集群需要依靠第三方中间件，例如：patroni+etcd、repmgr

MySQL

大部分操作都要依靠percona公司的第三方工具（percona-toolkit，XtraBackup），工具命令太多，学习成本高，高可用集群也需要第三方中间件，官方MGR集群还没成熟

------

**4. 高可用主从复制\**底层\**原理**

PGSQL

物理流复制，属于物理复制，跟SQL Server镜像/AlwaysOn一样，严格一致，没有任何可能导致不一致，性能和可靠性上，物理复制完胜逻辑复制，维护简单  

MySQL

主从复制，属于逻辑复制，（sql_log_bin、binlog_format等参数设置不正确都会导致主从不一致）
大事务并行复制效率低，对于重要业务，需要依赖 percona-toolkit的pt-table-checksum和pt-table-sync工具定期比较和修复主从一致
主从复制出错严重时候需要重搭主从
MySQL的逻辑复制并不阻止两个不一致的数据库建立复制关系

------

**5. 从库只读状态**

PGSQL

系统自动设置从库默认只读，不需要人工介入，维护简单  

MySQL

从库需要手动设置参数super_read_only=on，让从库设置为只读，super_read_only参数有bug，链接：https://baijiahao.baidu.com/s?id=1636644783594388753&wfr=spider&for=pc

------

**6. 版本分支**

PGSQL

只有社区版，没有其他任何分支版本，PGSQL官方统一开发，统一维护，社区版有所有功能，不像SQL Server和MySQL有标准版、企业版、经典版、社区版、开发版、web版之分
国内外还有一些基于PGSQL做二次开发的数据库厂商，例如：Enterprise DB、瀚高数据库等等，当然这些只是二次开发并不算独立分支

MySQL

由于历史原因，分裂为三个分支版本，MariaDB分支、Percona分支 、Oracle官方分支，发展到目前为止各个分支基本互相不兼容
Oracle官方分支还有版本之分，分为标准版、企业版、经典版、社区版

------

**7. SQL特性支持**

PGSQL

SQL特性支持情况支持94种，SQL语法支持最完善，例如：支持公用表表达式（WITH查询）

MySQL

SQL特性支持情况支持36种，SQL语法支持比较弱，例如：不支持公用表表达式（WITH查询）

关于SQL特性支持情况的对比，可以参考：http://www.sql-workbench.net/dbms_comparison.html

------

**8. 主从复制安全性**

PGSQL
同步流复制、强同步（remote apply)、高安全，不会丢数据
PGSQL同步流复制：所有从库宕机，主库会罢工，主库无法自动切换为异步流复制（异步模式），需要通过增加从库数量来解决，一般生产环境至少有两个从库
手动解决：在PG主库修改参数synchronous_standby_names =''，并执行命令： pgctl reload ，把主库切换为异步模式

主从数据完全一致是高可用切换的第一前提，所以PGSQL选择主库罢工也是可以理解

MySQL
增强半同步复制 ，mysql5.7版本增强半同步才能保证主从复制时候不丢数据
mysql5.7半同步复制相关参数：
参数rpl_semi_sync_master_wait_for_slave_count 等待至少多少个从库接收到binlog，主库才提交事务，一般设置为1，性能最高
参数rpl_semi_sync_master_timeout 等待多少毫秒，从库无回应自动切换为异步模式，一般设置为无限大，不让主库自动切换为异步模式
所有从库宕机，主库会罢工，因为无法收到任何从库的应答包

手动解决：在MySQL主库修改参数rpl_semi_sync_master_wait_for_slave_count=0

------

**9. 多字段统计信息**

PGSQL

支持多字段统计信息

MySQL

不支持多字段统计信息

------

**10. 索引类型**

PGSQL

多种索引类型（btree , hash , gin , gist , sp-gist , brin , bloom , rum , zombodb , bitmap，部分索引，表达式索引）

MySQL

btree 索引，全文索引（低效），表达式索引(需要建虚拟列)，hash 索引只在内存表

------

**11. 物理表连接算法**

PGSQL

支持 nested-loop join 、hash join 、merge join  

MySQL

只支持 nested-loop join

------

**12. 子查询和视图性能**

PGSQL

子查询，视图优化，性能比较高

MySQL

视图谓词条件下推限制多，子查询上拉限制多

------

**13. 执行计划即时编译**

PGSQL

支持 **JIT**  执行计划即时编译，使用LLVM编译器

MySQL

不支持执行计划即时编译

------

**14. 并行查询**

PGSQL

并行查询（多种并行查询优化方法），并行查询一般多见于商业数据库，是重量级功能

MySQL

有限，只支持主键并行查询

------

**15. 物化视图**

PGSQL

支持物化视图

MySQL

不支持物化视图

------

**16. 插件功能**

PGSQL

支持插件功能，可以丰富PGSQL的功能，GIS地理插件，时序数据库插件， 向量化执行插件等等

MySQL

不支持插件功能

------

**17. check约束**

PGSQL

支持check约束

MySQL

不支持check约束，可以写check约束，但存储引擎会忽略它的作用，因此check约束并不起作用（mariadb 支持）

------

**18. gpu 加速SQL**

PGSQL

可以使用gpu 加速SQL的执行速度  

MySQL

不支持gpu 加速SQL 的执行速度  

------

**19. 数据类型**

PGSQL

数据类型丰富，如 ltree，hstore，数组类型，ip类型，text类型，有了text类型不再需要varchar，text类型字段最大存储1GB

MySQL

数据类型不够丰富

------

**20. 跨库查询**

PGSQL

不支持跨库查询，这个跟Oracle 12C以前一样

MySQL

可以跨库查询

------

**21. 备份还原**

PGSQL

备份还原非常简单，时点还原操作比SQL Server还要简单，完整备份+wal归档备份（增量）
假如有一个三节点的PGSQL主从集群，可以随便在其中一个节点做完整备份和wal归档备份

MySQL

备份还原相对不太简单，完整备份+binlog备份（增量）
完整备份需要percona的XtraBackup工具做物理备份，MySQL本身不支持物理备份
时点还原操作步骤繁琐复杂

------

**22. 性能视图**

PGSQL

需要安装pg_stat_statements插件，pg_stat_statements插件提供了丰富的性能视图：如：等待事件，系统统计信息等
不好的地方是，安装插件需要重启数据库，并且需要收集性能信息的数据库需要执行一个命令：create extension pg_stat_statements命令
否则不会收集任何性能信息，比较麻烦

MySQL

自带PS库，默认很多功能没有打开，而且打开PS库的性能视图功能对性能有影响（如：内存占用导致OOM bug）

------

**23. 安装方式**

PGSQL

有各个平台的包rpm包，deb包等等，相比MySQL缺少了二进制包，一般用源码编译安装，安装时间会长一些，执行命令多一些

MySQL

有各个平台的包rpm包，deb包等等，源码编译安装、二进制包安装，一般用二进制包安装，方便快捷

------

**24. DDL操作**

PGSQL

加字段、可变长字段类型长度改大不会锁表，所有的DDL操作都不需要借助第三方工具，并且跟商业数据库一样，DDL操作可以回滚，保证事务一致性

MySQL

由于大部分DDL操作都会锁表，例如加字段、可变长字段类型长度改大，所以需要借助percona-toolkit里面的pt-online-schema-change工具去完成操作
将影响减少到最低，特别是对大表进行DDL操作
DDL操作不能回滚

------

**25. 大版本发布速度**

PGSQL

PGSQL每年一个大版本发布，大版本发布的第二年就可以上生产环境，版本迭代速度很快

PGSQL 9.6正式版推出时间：2016年

PGSQL 10 正式版推出时间：2017年
PGSQL 11 正式版推出时间：2018年
PGSQL 12 正式版推出时间：2019年

MySQL
MySQL的大版本发布一般是2年~3年，一般大版本发布后的第二年才可以上生产环境，避免有坑，版本发布速度比较慢
MySQL5.5正式版推出时间：2010年
MySQL5.6正式版推出时间：2013年
MySQL5.7正式版推出时间：2015年
MySQL8.0正式版推出时间：2018年

------

**26. returning语法**

PGSQL

支持returning语法，returning clause 支持 DML 返回 Resultset，减少一次 Client <-> DB Server 交互

MySQL

不支持returning语法

------

**27. 内部架构**

PGSQL

多进程架构，并发连接数不能太多，跟Oracle一样，既然跟Oracle一样，那么很多优化方法也是相通的，例如：开启大页内存

MySQL

多线程架构，虽然多线程架构，但是官方有限制连接数，原因是系统的并发度是有限的，线程数太多，反而系统的处理能力下降，随着连接数上升，反而性能下降
一般同时只能处理200 ~300个数据库连接

------

**28. 聚集索引**

PGSQL

不支持聚集索引，PGSQL本身的MVCC的实现机制所导致

MySQL

支持聚集索引

------

**29. 空闲事务终结功能**

PGSQL

通过设置 **idle_in_transaction_session_timeout** 参数来终止空闲事务，比如：应用代码中忘记关闭已开启的事务，PGSQL会自动查杀这种类型的会话事务

MySQL

不支持终止空闲事务功能

------

**30. 应付超大数据量**

PGSQL

不能应付超大数据量，由于PGSQL本身的MVCC设计问题，需要垃圾回收，只能期待后面的大版本做优化  

MySQL

不能应付超大数据量，MySQL自身架构的问题

------

**31. 分布式演进**

PGSQL

HTAP数据库：cockroachDB、腾讯Tbase

分片集群： Postgres-XC、Postgres-XL

MySQL
HTAP数据库：TiDB
分片集群： 各种各样的中间件，不一一列举

------

**32. 数据库的文件名和命名规律**

PGSQL

PGSQL在这方面做的比较不好，DBA不能在操作系统层面（停库状态下）看清楚数据库的文件名和命名规律，文件的数量，文件的大小

一旦操作系统发生文件丢失或硬盘损坏，非常不利于恢复，因为连名字都不知道

PGSQL表数据物理文件的命名/存放规律是： 在一个表空间下面，如果没有建表空间默认在默认表空间也就是base文件夹下，例如：/data/base/16454/3599

base：默认表空间pg_default所在的物理文件夹
16454：表所在数据库的oid
3599：就是表对象的oid，当然，一个表的大小超出1GB之后会再生成多个物理文件，还有表的fsm文件和vm文件，所以一个大表实际会有多个物理文件

由于PGSQL的数据文件布局内容太多，大家可以查阅相关资料

当然这也不能全怪PGSQL，作为一个DBA，时刻做好数据库备份和容灾才是正道，做介质恢复一般是万不得已的情况下才会做

MySQL

数据库名就是文件夹名，数据库文件夹下就是表数据文件，但是要注意表名和数据库名不能有特殊字符或使用中文名，每个表都有对应的frm文件和ibd文件，存储元数据和表/索引数据，清晰明了，做介质恢复或者表空间传输都很方便

------

**33. 权限设计**

PGSQL

PGSQL在权限设计这块是比较坑爹，抛开实例权限和表空间权限，PGSQL的权限层次有点像SQL Server，db=》schema=》object

要说权限，这里要说一下Oracle，用Oracle来类比

在ORACLE 12C之前，实例与数据库是一对一，也就是说一个实例只能有一个数据库，不像MySQL和SQL Server一个实例可以有多个数据库，并且可以随意跨库查询

而PGSQL不能跨库查询的原因也是这样，PGSQL允许建多个数据库，跟ORACLE类比就是有多个实例（之前说的实例与数据库是一对一）

一个数据库相当于一个实例，因为PGSQL允许有多个实例，所以PGSQL单实例不叫一个实例，叫集簇（cluster），集簇这个概念可以查阅PGSQL的相关资料

PGSQL里面一个实例/数据库下面的schema相当于数据库，所以这个schema的概念对应MySQL的database

**注意点：**正因为是一个数据库相当于一个实例，PGSQL允许有多个实例/数据库，所以数据库之间是互相逻辑隔离的，导致的问题是，不能一次对一个PGSQL集簇下面的所有数据库做操作

必须要逐个逐个数据库去操作，例如上面说到的安装pg_stat_statements插件，如果您需要在PGSQL集簇下面的所有数据库都做性能收集的话，需要逐个数据库去执行加载命令

又例如跨库查询需要dblink插件或fdw插件，两个数据库之间做查询相当于两个实例之间做查询，已经跨越了实例了，所以需要dblink插件或fdw插件，所以道理非常简单

权限操作也是一样逐个数据库去操作，还有一个就是PGSQL虽然像SQL Server的权限层次结构db=》schema=》object，但是实际会比SQL Server要复杂一些，还有就是新建的表还要另外授权

在PGSQL里面，角色和用户是一样的，对新手用户来说有时候会傻傻分不清，也不知道怎么去用角色，所以PGSQL在权限设计这一块确实比较坑爹

MySQL

使用mysql库下面的5个权限表去做权限映射，简单清晰，唯一问题是缺少权限角色

user表
db表
host表
tables_priv表
columns_priv表

------

**34. 发展历史**

PGSQL
在1995年，开发人员Andrew Yu和Jolly Chen在Postgres中添加了一个SQL（Structured Query Language，结构化查询语言）翻译程序，该版本叫做Postgres95，在开放源代码社区发放。
在1996年，再次对Postgres95做了较大的改动，并将其命名为PostgresSQL 6.0版发布，PostgresSQL 的名字就此定型，从1995年算起，大概有**24年**历史MySQL
在1996年，MySQL 1.0发布，它只面向一小拨人，相当于内部发布。
到了1996年10月，MySQL 3.11.1发布(MySQL没有2.x版本)，最开始只提供Solaris操作系统下的二进制版本，一个月后，Linux版本出现
从1996年算起，大概有**23年**历史

## 小结

上面的对比表还不是很完善，只有一些本人认为比较关键的特性拿出来对比

总的来说，两种数据库都有优缺点，大家在选型的时候需要谨慎选择，MySQL需要多折腾，PGSQL让你少折腾，因为PGSQL本身已经做的比较完善，不太需要依赖一些第三方工具

当然，如果在MySQL上选择Percona 分支，MariaDB分支，或者Oracle官方的MySQL企业版就另当别论

MySQL因为需要支持更换存储引擎，所以某些功能都要受制于存储引擎层，例如：物理复制

而PGSQL不支持更换存储引擎（在PGSQL V12开始也支持可插拨的表存取接口），而且一直由官方统一开发和维护，所以相对比较稳定，功能也比较完善，对得上它的称号：《世界上功能最为强大的开源数据库》

PGSQL V12 支持可插拨的表存取接口之后，有可能由第三方存储引擎来改进PGSQL本身的MVCC实现机制，而不需要等待官方去解决，聚集索引、undo表空间这些都不再是问题

如果业务复杂的话，其实PGSQL是最佳选择，分享一篇文章：[为什么“去O”唯有PG](https://mp.weixin.qq.com/s/hieS4AScOyUpiyMBI0sG7w)