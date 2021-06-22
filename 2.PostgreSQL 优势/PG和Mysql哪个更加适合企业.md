- [PG和Mysql哪个更加适合企业](https://developer.aliyun.com/article/772371?spm=a2c6h.14164896.0.0.795d2bde8eB0kk)

# 一、开源方面

PostgreSQL: The world’s most advanced open source database。

**开源协议：**PostgreSQL基于自由的BSD/MIT许可，组织可以使用、复制、修改和重新分发代码，只需要提供一个版权声明即可。

PG的开源协议特别灵活，任何公司的和个人都可以把PG作为一个产品销售，而不需要像MySQL那样必须修改大部分代码才可以作为公司的产品。

**MySQL：**World’s Most Popular Open Source Database。

**开源协议：**核心代码基于GPL或Commercial License。

MySQL的开源协议是基于GPL协议，任何公司都可以免费使用，不允许修改后和衍生的代码做为闭源的商业软件发布和销售，MySQL的版权在甲骨文手中，甲骨文可以推了其商业闭源版本。
![image.png](https://ucc.alicdn.com/pic/developer-ecology/cb211a617c7a42ac8eb9a64e5adf2283.png)

# 二、ACID支持方面

PostgreSQL支持事务的强一致性，事务保证性好，完全支持ACID特性。

MySQL只有innodb引擎支持事务，事务一致性保证上可根据实际需求调整，为了最大限度的保护数据，MySQL可配置双一模式，对ACID的支持上比PG稍弱弱。

# 三、SQL标准的支持方面

PostgreSQL几乎支持所有的SQL标准，支持类型相当丰富。

MySQL只支持部分SQL标准，相比于PG支持类型稍弱。

# 四、复制

MySQL的复制是基于binlog的逻辑异步复制，无法实现同步复制。
**复制模式：**

一主一备。
一主多备。
级联复制。
循环复制。
主主复制。

**数据流转优势：**通过canal增量数据的订阅和消费，可以同步数据到kafka，通过kafka做数据流转。

MySQL所有的高可用方案都是基于binlog做的同步，以及基于MySQL的分布式数据也是基于MySQL的binlog实现，binlog是MySQL生态圈最基本技术实现。

PostgreSQL可以做到同步，异步，半同步复制，以及基于日志逻辑复制，可以实现表级别的订阅和发布。

**复制模式：**
一主一备。
一主多备。
级联复制。
热备库/流复制。
逻辑复制。

**数据流转优势：**通过逻辑复制实现消息的订阅和消费，可以同步数据到kafka，通过kafka实现数据流转。

# 五、并发控制

**PostgreSQL通过其MVCC实现有效地解决了并发问题，从而实现了非常高的并发性。**

PG新老数据一起存放的基于XID的MVCC机制,新老数据一起存放，需要定时触 发VACUUM，会带来多余的IO和数据库对象加锁开销，引起数据库整体的并发能力下降。而且VACUUM清理不及时，还可能会引发数据膨胀。

当然PostgreSQL还有一点影响比较，为了保证事务的强一致性，未决事务会影响所有表VACUUM清理，导致表膨胀。

MySQL仅在InnoDB中支持MVCC。

**innodb的基于回滚段实现的MVCC机制,但是MySQL的间隙锁影响较大，锁定数据较多。**

# 六、性能

**1、PostgreSQL**
1）PostgreSQL广泛用于读写速度高和数据一致性高的大型系统。此外，它还支持各种性能优化，当然这些优化仅在商业解决方案中可用，例如地理空间数据支持，没有读锁定的并发性等等。

2）PostgreSQL性能最适用于需要执行复杂查询的系统。

3）PostgreSQL在OLTP/ OLAP系统中表现良好，读写速度以及大数据分析方面表现良好，基于PG的GP数据库，在数据仓库领域表现良好。

4）PostgreSQL也适用于商业智能应用程序，但更适合需要快速读/写速度的数据仓库和数据分析应用程序。
**2、MySQL**
1）MySQL是广泛选择的基于Web的项目，需要数据库只是为了简单的数据事务。但是，当遇到重负载或尝试完成复杂查询时，MySQL通常会表现不佳。

2）MySQL的读取速度，在OLTP系统中表现良好。

3）MySQL + InnoDB为OLTP场景提供了非常好的读/写速度。总体而言，MySQL在高并发场景下表现良好。

4）MySQL是可靠的，并且与商业智能应用程序配合良好，因为商业智能应用程序通常读取很多。

# 七、高可用技术的实现

## 7.1 PostgreSQL

1）基于流复制的异步、同步主从。

2）基于流复制的–keepalive。

3）基于流复制的 –repmgr。

4）基于流复制的 –patroni+etcd。

5）共享存储HA（corosync+pacemaker）。

6）Postgres-XC。

7）Postgres-XL。

8）中间件实现：pgpool、pgcluster、slony、plploxy。

## 7.2 MySQL

1）主从复制。

2）主主复。

3）MHA。

4）LVS+KEEPALIVE。

5）MGR分布式数据库，多点写入[不建议]，基于paxos协议。

6）PXC分布式数据库，多点写入[不建议]，基于令牌环协议。

7）INNODB CLUSTER[8.0新技术，基于MGR实现，上层封装命令],基于paxos协议。

8）中间件实现：mycat。

# 八、外部数据源

PostgreSQL FDW –[foreign-data wrapper的一个简称，可以叫外部封装。

PostgreSQL不支持多数据引擎。但支持Extension组件扩充，以及通过名为FDW的技术将Oracle、Hadoop、MongoDB、SQLServer、Excel、CSV文件等作为外部表进行读写操作，因此，可以为大数据与关系型数据库提供良好对接。

MySQL：无。

# 九、数据存储和数据类型

PG主表采用堆表存放，存放的数据量较大，数据访问方式类似于Oracle的堆表。

MySQL采用索引组织表，MySQL必须有主键索引，所有的数据访问都是通过主键实现，二级索引访问时，需要扫描两遍索引（主键和二级索引）。

# 十、PostgreSQL与MySQL优劣对比

## 10.1 PostgreSQL相对于MySQL的优势

1）在SQL的标准实现上要比MySQL完善，而且功能实现比较严谨。 

2）存储过程的功能支持要比MySQL好，具备本地缓存执行计划的能力。

3）对表连接支持较完整，优化器的功能较完整，支持的索引类型很多，复杂查询能力较强。

4）PG主表采用堆表存放，MySQL采用索引组织表，能够支持比MySQL更大的数据量。 

5）PG的主备复制属于物理复制，相对于MySQL基于binlog的逻辑复制，数据的一致性更加可靠，复制性能更高，对主机性能的影响也更小。

6）MySQL的存储引擎插件化机制，存在锁机制复杂影响并发的问题，而PG不存在。 

7）PG对可以实现外部数据源查询，数据源的支持类型丰富。 

8）PG原生的逻辑复制可以实现表级别的订阅发布，可以实现数据通过kafka流转，而不需要其他的组件。 

9）PG支持三种表连接方式，嵌套循环，哈希连接，排序合并，而MySQL只支持嵌套循环。 

10）PostgreSQL源代码写的很清晰，易读性比MySQL强太多了。 

11）PostgreSQL通过PostGIS扩展支持地理空间数据。地理空间数据有专用的类型和功能，可直接在数据库级别使用，使开发人员更容易进行分析和编码。 

12）可扩展型系统，有丰富可扩展组件，作为contribute发布。 

13）PostgreSQL支持JSON和其他NoSQL功能，如本机XML支持和使用HSTORE的键值对。它还支持索引JSON数据以加快访问速度，特别是10版本JSONB更是强大。 

14）PostgreSQL完全免费，而且是BSD协议，如果你把PostgreSQL改一改，然后再拿去卖钱，也没有人管你，这一点很重要，这表明了PostgreSQL数据库不会被其它公司控制。相反，MySQL现在主要是被Oracle公司控制。

## 10.2 MySQL相对于PG的优势

1）innodb的基于回滚段实现的MVCC机制，相对PG新老数据一起存放的基于XID的MVCC机制，是占优的。新老数据一起存放，需要定时触 发VACUUM，会带来多余的IO和数据库对象加锁开销，引起数据库整体的并发能力下降。而且VACUUM清理不及时，还可能会引发数据膨胀。

2）MySQL采用索引组织表，这种存储方式非常适合基于主键匹配的查询、删改操作，但是对表结构设计存在约束。

3）MySQL的优化器较简单，系统表、运算符、数据类型的实现都很精简，非常适合简单的查询操作。

4）MySQL相对于PG在国内的流行度更高，PG在国内显得就有些落寞了。

5）MySQL的存储引擎插件化机制，使得它的应用场景更加广泛，比如除了innodb适合事务处理场景外，myisam适合静态数据的查询场景。

# 总结

总体上来说，开源数据库都不是很完善，商业数据库oracle在架构和功能方面都还是完善很多的。从应用场景来说，PG更加适合严格的企业应用场景（比如金融、电信、ERP、CRM），但不仅仅限制于此，PostgreSQL的json，jsonb，hstore等数据格式，特别适用于一些大数据格式的分析；而MySQL更加适合业务逻辑相对简单、数据可靠性要求较低的互联网场景（比如google、facebook、alibaba），当然现在MySQL的在innodb引擎的大力发展，功能表现良好。

MySQL和PostgreSQL复杂的开源关系型数据库，本文只是根据自己经验写的对PG和MySQL的理解，难免有不当之处，不当之处还请大家多多指正，我们的LCRM系统最终还是用了PostgreSQL