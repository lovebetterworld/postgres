- [PostgreSQL 与 MySQL 相比，优势何在？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/20010554)

作者：luikore
链接：https://www.zhihu.com/question/20010554/answer/62628256
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



**Pg 没有 MySQL 的各种坑**

MySQL 的各种 text 字段有不同的限制, 要手动区分 small text, middle text, large text... Pg 没有这个限制, text 能支持各种大小.

按照 SQL 标准, 做 null 判断不能用 = null, 只能用 is null

> the result of any arithmetic comparison with NULL is also NULL

但 pg 可以设置 [transform_null_equals](https://www.zhihu.com/search?q=transform_null_equals&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A62628256}) 把 = null 翻译成 is null 避免踩坑

不少人应该遇到过 MySQL 里需要 utf8mb4 才能显示 emoji 的坑, Pg 就没这个坑.

MySQL 的事务隔离级别 repeatable read 并不能阻止常见的并发更新, 得加锁才可以, 但[悲观锁](https://www.zhihu.com/search?q=悲观锁&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A62628256})会影响性能, 手动实现乐观锁又复杂. 而 Pg 的列里有隐藏的[乐观锁](https://www.zhihu.com/search?q=乐观锁&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A62628256}) version 字段, 默认的 repeatable read 级别就能保证并发更新的正确性, 并且又有乐观锁的性能. 附带一个各数据库对隔离级别的行为差异比较调查: 

[http://www.cs.umb.edu/~poneil/iso.pdf](https://link.zhihu.com/?target=http%3A//www.cs.umb.edu/~poneil/iso.pdf)



MySQL 不支持多个表从同一个序列中取 id, 而 Pg 可以.

MySQL 不支持 OVER 子句, 而 Pg 支持. OVER 子句能简单的解决 "每组取 top 5" 的这类问题.

几乎任何数据库的子查询 (subquery) 性能都比 MySQL 好.

更多的坑:

[http://blog.ionelmc.ro/2014/12/28/terrible-choices-mysql/](https://link.zhihu.com/?target=http%3A//blog.ionelmc.ro/2014/12/28/terrible-choices-mysql/)

不少人踩完坑了, 以为换个数据库还得踩一次, 所以很抗拒, 事实上不是!!!



**Pg 不仅仅是 SQL 数据库**

它可以存储 array 和 json, 可以在 array 和 json 上建索引, 甚至还能用表达式索引. 为了实现文档数据库的功能, 设计了 jsonb 的存储结构. 有人会说为什么不用 Mongodb 的 BSON 呢? Pg 的开发团队曾经考虑过, 但是他们看到 BSON 把 ["a", "b", "c"] 存成 {0: "a", 1: "b", 2: "c"} 的时候就决定要重新做一个 jsonb 了... 现在 jsonb 的性能已经优于 BSON.

现在往前端偏移的开发环境里, 用 Pg + PostgREST 直接生成后端 API 是非常快速高效的办法:

[begriffs/postgrest · GitHub](https://link.zhihu.com/?target=https%3A//github.com/begriffs/postgrest)

postgREST 的性能非常强悍, 一个原因就是 Pg 可以直接组织返回 json 的结果.

它支持服务器端脚本: TCL, Python, R, Perl, Ruby, MRuby ... 自带 [map-reduce](https://www.zhihu.com/search?q=map-reduce&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A62628256}) 了.

它有地理信息处理扩展 (GIS 扩展不仅限于真实世界, 游戏里的地形什么的也可以), 可以用 Pg 搭寻路服务器和地图服务器:

[PostGIS — Spatial and Geographic Objects for PostgreSQL](https://link.zhihu.com/?target=http%3A//postgis.net/)

它自带全文搜索功能 (不用费劲再装一个 elasticsearch 咯):

[Full text search in milliseconds with PostgreSQL](https://link.zhihu.com/?target=https%3A//blog.lateral.io/2015/05/full-text-search-in-milliseconds-with-postgresql/)

 不过一些语言相关的支持还不太完善, 有个 bamboo 插件用调教过的 mecab 做中文分词, 如果要求比较高, 还是自己分了词再存到 tsvector 比较好.

它支持 trigram 索引.

trigram 索引可以帮助改进全文搜索的结果: 

[PostgreSQL: Documentation: 9.3: pg_trgm](https://link.zhihu.com/?target=http%3A//www.postgresql.org/docs/9.3/static/pgtrgm.html)

trigram 还可以实现高效的正则搜索 (原理参考 

[https://swtch.com/~rsc/regexp/regexp4.html](https://link.zhihu.com/?target=https%3A//swtch.com/~rsc/regexp/regexp4.html)

MySQL 处理树状回复的设计会很复杂, 而且需要写很多代码, 而 Pg 可以高效处理树结构:

[Scaling Threaded Comments on Django at Disqus](https://link.zhihu.com/?target=http%3A//cramer.io/2010/05/30/scaling-threaded-comments-on-django-at-disqus/)[http://www.slideshare.net/quipo/trees-in-the-database-advanced-data-structures](https://link.zhihu.com/?target=http%3A//www.slideshare.net/quipo/trees-in-the-database-advanced-data-structures)

它可以高效处理图结构, 轻松实现 "朋友的朋友的朋友" 这种功能:

[http://www.slideshare.net/quipo/rdbms-in-the-social-networks-age](https://link.zhihu.com/?target=http%3A//www.slideshare.net/quipo/rdbms-in-the-social-networks-age)

它可以把 70 种外部[数据源](https://www.zhihu.com/search?q=数据源&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A62628256}) (包括 Mysql, Oracle, CSV, hadoop ...) 当成自己数据库中的表来查询



作者：方圆
链接：https://www.zhihu.com/question/20010554/answer/15863274
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



一、 PostgreSQL 的稳定性极强， Innodb 等引擎在崩溃、断电之类的灾难场景下抗打击能力有了长足进步，然而很多 MySQL 用户都遇到过Server级的数据库丢失的场景——mysql系统库是MyISAM的，相比之下，PG数据库这方面要好一些。

二、任何系统都有它的性能极限，在高并发读写，负载逼近极限下，PG的性能指标仍可以维持双曲线甚至[对数曲线](https://www.zhihu.com/search?q=对数曲线&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})，到顶峰之后不再下降，而 MySQL 明显出现一个波峰后下滑（5.5版本之后，在企业级版本中有个插件可以改善很多，不过需要付费）。

三、PG 多年来在 GIS 领域处于优势地位，因为它有丰富的几何类型，实际上不止几何类型，PG有大量字典、数组、bitmap 等数据类型，相比之下mysql就差很多，instagram就是因为PG的空间数据库扩展POSTGIS远远强于MYSQL的[my spatial](https://www.zhihu.com/search?q=my+spatial&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})而采用PGSQL的。

四、PG 的“无锁定”特性非常突出，甚至包括 vacuum 这样的整理数据空间的操作，这个和PGSQL的MVCC实现有关系。

五、PG 的可以使用函数和[条件索引](https://www.zhihu.com/search?q=条件索引&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})，这使得PG数据库的调优非常灵活，mysql就没有这个功能，条件索引在web应用中很重要。

六、PG有极其强悍的 SQL 编程能力（9.x 图灵完备，支持递归！），有非常丰富的统计函数和统计语法支持，比如分析函数（ORACLE的叫法，PG里叫[window函数](https://www.zhihu.com/search?q=window函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})），还可以用多种语言来写存储过程，对于R的支持也很好。这一点上MYSQL就差的很远，很多分析功能都不支持，腾讯内部数据存储主要是MYSQL，但是数据分析主要是HADOOP+PGSQL（听[李元佳](https://www.zhihu.com/search?q=李元佳&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})说过，但是没有验证过）。

七、PG 的有多种集群架构可以选择，plproxy 可以支持语句级的镜像或分片，slony 可以进行字段级的同步设置，standby 可以构建WAL文件级或流式的读写分离集群，同步频率和集群策略调整方便，操作非常简单。

八、一般关系型数据库的字符串有限定长度8k左右，无限长 TEXT 类型的功能受限，只能作为外部大数据访问。而 PG 的 TEXT 类型可以直接访问，SQL语法内置正则表达式，可以索引，还可以全文检索，或使用[xml xpath](https://www.zhihu.com/search?q=xml+xpath&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})。用PG的话，文档数据库都可以省了。

九，对于WEB应用来说，复制的特性很重要，mysql到现在也是异步复制，pgsql可以做到同步，异步，半同步复制。还有mysql的同步是基于binlog复制，类似[oracle golden gate](https://www.zhihu.com/search?q=oracle+golden+gate&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274}),是基于stream的复制，做到同步很困难，这种方式更加适合异地复制，pgsql的复制基于wal，可以做到同步复制。同时，pgsql还提供stream复制。

十，pgsql对于numa架构的支持比mysql强一些，比MYSQL对于读的性能更好一些，pgsql提交可以完全异步，而mysql的内存表不够实用（因为表锁的原因）

最后说一下我感觉 PG 不如 MySQL 的地方。

第一，MySQL有一些实用的运维支持，如 slow-query.log ，这个pg肯定可以定制出来，但是如果可以配置使用就更好了。

第二是mysql的innodb引擎，可以充分优化利用系统所有内存，超大内存下PG对内存使用的不那么充分，

第三点，MySQL的复制可以用多级从库，但是在9.2之前，PGSQL不能用从库带从库。

第四点，从测试结果上看，[mysql 5.5](https://www.zhihu.com/search?q=mysql+5.5&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})的性能提升很大，单机性能强于pgsql，5.6应该会强更多.

第五点，对于web应用来说,mysql 5.6 的内置MC API功能很好用，PGSQL差一些。

另外一些：

pgsql和mysql都是背后有商业公司，而且都不是一个公司。大部分开发者，都是拿工资的。

说mysql的执行速度比pgsql快很多是不对的，速度接近，而且很多时候取决于你的配置。

对于存储过程，函数，视图之类的功能，现在两个数据库都可以支持了。

另外[多线程架构](https://www.zhihu.com/search?q=多线程架构&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A15863274})和多进程架构之间没有绝对的好坏，oracle在unix上是多进程架构，在windows上是多线程架构。

很多pg应用也是24/7的应用，比如skype. 最近几个版本VACUUM基本不影响PGSQL 运行，8.0之后的PGSQL不需要cygwin就可以在windows上运行。

至于说对于事务的支持，mysql和pgsql都没有问题。





作者：冯若航
链接：https://www.zhihu.com/question/20010554/answer/94999834
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



PostgreSQL的Slogan是“**世界上最先进的开源关系型数据库**”

它是一款**一专多长的[全栈数据库](https://www.zhihu.com/search?q=全栈数据库&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})：在可观的规模内，都能做到一招鲜吃遍天**。



​        成熟的应用可能会用到许许多多的数据组件（功能）：缓存，OLTP，OLAP/批处理/数据仓库，流处理/[消息队列](https://www.zhihu.com/search?q=消息队列&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})，搜索索引，NoSQL/文档数据库，[地理数据库](https://www.zhihu.com/search?q=地理数据库&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})，空间数据库，时序数据库，图数据库。传统架构选型可能会组合使用多种组件，典型的如：Redis + MySQL + Greenplum/Hadoop + Kafuka/Flink + ElasticSearch。在这里MySQL只能扮演OLTP关系型数据库的角色，但如果是PostgreSQL，就可以身兼多职，**One hold them all**，比如：

**OLTP**：事务处理是PostgreSQL的本行

**OLAP**：citus分布式插件，ANSI SQL兼容，[窗口函数](https://www.zhihu.com/search?q=窗口函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})，CTE，CUBE等高级分析功能，任意语言写UDF

**流处理**：PipelineDB扩展，Notify-Listen，物化视图，规则系统，灵活的存储过程与函数编写

**[时序数据](https://www.zhihu.com/search?q=时序数据&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})**：timescaledb时序数据库插件，分区表，BRIN索引

**空间数据**：PostGIS扩展（杀手锏），内建的几何类型支持，GiST索引。

**搜索索引**：全文搜索索引足以应对简单场景；丰富的索引类型，支持函数索引，条件索引

**NoSQL**：JSON，JSONB，XML，HStore原生支持，至NoSQL数据库的外部数据包装器

**数据仓库**：能平滑迁移至同属Pg生态的GreenPlum，DeepGreen，HAWK等，使用FDW进行ETL

**图数据**：递归查询

**缓存**：物化视图

![img](https://pica.zhimg.com/50/v2-c35e6b6ce2559302f17665ee1bd6bde5_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-c35e6b6ce2559302f17665ee1bd6bde5_720w.jpg?source=1940ef5c)PostgreSQL知名扩展



​     在探探的实践中，整个系统就是围绕PostgreSQL设计的。几百万日活，几百万全局DB-TPS，几百TB数据的量级下，[数据组件](https://www.zhihu.com/search?q=数据组件&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})只用了PostgreSQL。直到接近千万日活，才开始进行架构调整引入独立的数仓，消息队列和缓存。这只是验证过的规模量级，进一步压榨PG是完全可行的。

![img](https://pic1.zhimg.com/50/v2-b7b31d442e75c4f68f481160e891580f_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-b7b31d442e75c4f68f481160e891580f_720w.jpg?source=1940ef5c)围绕PostgreSQL的架构演进

​     因此在一个很可观的规模内，PostgreSQL都可以扮演多面手的角色，一个组件当多种组件使。**虽然在某些领域它可能比不上专用组件**，至少都做的都还不赖。**而单一数据组件选型可以极大地削减项目额外复杂度，这意味着能节省很多成本。它让十个人才能搞定的事，变成一个人就能搞定的事。**

​     为了不需要的规模而设计完全是白费功夫，实际上这属于过早优化的一种形式。只有当没有单个软件能满足你的所有需求时，才会存在分拆和集成的利弊权衡。集成多种异构技术是相当棘手的工作，如果真有那么一样技术可以满足你所有的需求，那么使用该技术就是最佳选择，而不应试图去集成多个组件来重新实现它。

​     当业务规模增长到一定量级时，可能最终还是不得不使用基于微服务/总线的架构，将这些功能分拆为多个组件。但PostgreSQL的存在极大地推后了这个权衡到来的阈值，而且在分拆之后依然能继续发挥重要的作用。



当然除了功能强大之外，Pg的另外一个重要的优势就是**运维友好**。有很多非常实用的特性：

- DDL能放入事务中，删表，TRUNCATE，创建函数，索引，都可以放在事务里原子生效，或者回滚。
  这就能进行很多骚操作，比如在一个事务里通过RENAME，完成两张表的[王车易位](https://www.zhihu.com/search?q=王车易位&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})。
- 能够并发地创建或删除索引（不锁表）；为表添加新的空字段不锁表，瞬间完成。
  这意味着可以随时在线上按需添加移除索引，添加字段，不影响业务。
- 复制方式多样：段复制，流复制，触发器复制，逻辑复制，插件复制，多种复制方法。
  丰富的复制支持使得不停服务迁移数据变得无比容易。
- 提交方式多样：异步提交，同步提交，法定人数同步提交。
- FDW的存在让ETL变得无比简单，一行SQL就能解决。
- 系统视图非常完备，做监控系统相当简单。



​      除了运维之外，Pg还有一个巨大的优势就是**协议友好**，类BSD/MIT的协议。君不见多少国产数据库，BAT云数据库都是Pg的换皮或二次开发产品。

​     相比之下，MySQL社区版是GPL协议，要不是GPL传染，怎么这么多基于MySQL改的数据库都开源了。而且MySQL还捏在[乌龟壳](https://www.zhihu.com/search?q=乌龟壳&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})手里，React改协议的风波算是一个前车之鉴。



当然，要说PG有什么缺点或者遗憾，那还是有几个的，不过也无伤大雅：

- 因为使用了MVCC，数据库需要定期VACUUM，有额外的维护工作。
- 没有`pt-xxx`那么成熟的MySQL工具脚本。
- 没有很好的开源**集群**监控方案，需要自己做。
- 慢查询日志和普通日志是混在一起的，需要自己解析处理。
- 官方Pg没有很好用的列存储，对数据分析而言算一个小遗憾。



还有一个劣势：MySQL确实是**最流行**的开源关系型数据库，所以Pg招人相对困难。很多时候只好自己培养。不过看DB Engines上的流行度趋势，未来还是很光明的。

![img](https://pica.zhimg.com/50/v2-e180033628691e5d54800bb0da027f89_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-e180033628691e5d54800bb0da027f89_720w.jpg?source=1940ef5c)DB-Engine Trend Top4, Top3都在走下坡路 

​       我自己比较选型过MySQL和PostgreSQL，难得地在阿里这种MySQL和Java的世界中有过选择的自由。我认为只要有条件**自由选择**，没有任何理由不选PostgreSQL。扛着阻力把PG（还有Go之于Java）用了起来，推了起来。我用它做过很多项目，解决了很多需求（小到算统计报表，大到创收一个小目标）。大多数需求PG单挑就搞定了，极少数可能会再用点NoSQL，比如Redis，Cassandra/HBase。最后实在是对Pg爱不释手，以至于专职去研究PG了。

------

用好Pg也有一个立竿见影的优势：对于很多需求来说，一个人能顶一个小团队，[全栈工程师](https://www.zhihu.com/search?q=全栈工程师&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})就该用全栈数据库嘛：

- 后端懒得写怎么办，[PostGraphQL](https://link.zhihu.com/?target=https%3A//github.com/graphile/postgraphile)直接从数据库模式定义生成GraphQL API，自动监听DDL变更，生成相应的CRUD方法与存储过程包装，对于后台开发再方便不过，类似的工具还有PostgREST与pgrest。对于中小数据量的应用都还堪用，省了一大半后端开发的活。
- 需要用到Redis的功能，直接上Pg，模拟普通功能不在话下，缓存也省了。Pub/Sub使用Notify/Listen/Trigger实现，用来广播配置变更，做一些控制非常方便。
- 需要做分析，窗口函数，复杂JOIN，CUBE，GROUPING，自定义聚合，自定义语言，爽到飞起。如果觉得规模大了想scale out可以上[citus](https://link.zhihu.com/?target=https%3A//www.citusdata.com/)扩展（或者换greenplum）；比起数仓可能少个列存比较遗憾，但其他该有的都有了。
- 用到地理相关的功能，PostGIS堪称神器，千行代码才能实现的复杂地理需求，[一行SQL轻松高效解决](https://link.zhihu.com/?target=https%3A//github.com/Vonng/pg/blob/master/case/knn.md)。
- 存储时序数据，[timescaledb](https://link.zhihu.com/?target=http%3A//www.timescale.com/)扩展虽然比不上专用时序数据库，但百万记录每秒的入库速率还是有的。用它解决过硬件传感器日志存储，监控系统Metrics存储的需求。
- 一些流计算的相关功能，可以用[PipelineDB](https://link.zhihu.com/?target=https%3A//github.com/Vonng/hbase_fdw)直接定义流式视图实现：UV，PV，用户画像实时呈现。物化视图也很好使。
- PostgreSQL的[FDW](https://link.zhihu.com/?target=https%3A//wiki.postgresql.org/wiki/Foreign_data_wrappers)是一种强大的机制，允许接入各种各样的数据源，以统一的SQL接口访问。它妙用无穷：
- file_fdw这种自带的扩展，可以将任意程序的输出接入数据表。最简单的应用就是[监控系统信息](https://link.zhihu.com/?target=https%3A//github.com/Vonng/pg/blob/master/fdw/file_fdw-intro.md)。
- 管理多个PostgreSQL实例时，可以在一个元数据库中用自带的postgres_fdw导入所有远程数据库的数据字典。统一访问所有数据库实例的元数据，一行SQL拉取所有数据库的实时指标，监控系统做起来不要太爽。
- 之前做过的一件事就是用[hbase_fdw](https://link.zhihu.com/?target=https%3A//github.com/Vonng/hbase_fdw)和MongoFDW，将HBase中的历史批量数据，MongoDB中的当日实时数据包装为PostgreSQL数据表，一个视图就简简单单地实现了融合批处理与流处理的Lambda架构。
- 使用redis_fdw进行缓存更新推送；使用mongo_fdw完成从mongo到pg的数据迁移；使用mysql_fdw读取MySQL数据并存入数仓；实现跨数据库，甚至跨数据组件的JOIN；使用一行SQL就能完成原本多少行代码才能实现的复杂ETL，这是一件多么美妙的事情。
- 各种丰富的类型与方法支持：例如[JSON](https://link.zhihu.com/?target=http%3A//www.postgres.cn/docs/9.6/datatype-json.html)，从数据库直接生成前端所需的JSON响应，轻松而惬意。范围类型，优雅地解决很多原本需要程序处理的边角情况。其他的例如数组，[多维数组](https://www.zhihu.com/search?q=多维数组&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})，自定义类型，枚举，网络地址，UUID，ISBN。很多开箱即用的数据结构让程序员省去了多少造轮子的功夫。
- 丰富的索引类型：通用的Btree索引；大幅优化顺序访问的Brin索引；等值查询的Hash索引；GIN倒排索引；GIST通用搜索树，高效支持地理查询，KNN查询；Bitmap同时利用多个独立索引；Bloom高效过滤索引；能大幅减小索引大小的条件索引；能优雅替代冗余字段的函数索引。而MySQL就只有那么可怜的几种索引。
- 稳定可靠，正确高效。MVCC轻松实现快照隔离，MySQL的RR隔离等级实现[不完善](https://link.zhihu.com/?target=https%3A//github.com/ept/hermitage)，无法避免PMP与G-single异常。而且基于锁与回滚段的实现会有各种坑；PostgreSQL通过SSI能实现高性能的可序列化。
- 复制强大：WAL段复制，流复制（v9出现，同步、半同步、异步），逻辑复制（v10出现：订阅/发布），触发器复制，第三方复制，各种复制一应俱全。
- 运维友好：可以将DDL放在事务中执行（可回滚），创建索引不锁表，添加新列（不带默认值）不锁表，清理/备份不锁表。各种[系统视图](https://www.zhihu.com/search?q=系统视图&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A94999834})，监控功能都很完善。
- 扩展众多、功能丰富、可定制程度极强。在PostgreSQL中可以使用任意的语言编写函数：Python，Go，Javascript，Java，Shell等等。与其说Pg是数据库，不如说它是一个开发平台。我试过很多没什么卵用但很好玩的东西：数据库**里**的爬虫/ [推荐系统](https://link.zhihu.com/?target=https%3A//github.com/Vonng/pg/blob/master/case/pg-recsys.md) / 神经网络 / Web服务器等等。还有着各种各样功能强悍或脑洞清奇的第三方插件：[https://pgxn.org](https://link.zhihu.com/?target=https%3A//pgxn.org/)。





作者：动力节点在线
链接：https://www.zhihu.com/question/20010554/answer/743955463
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



本文是转载文章。

![img](https://pica.zhimg.com/50/v2-1e6439ce4f3fa05ff322aa57edfd60fd_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-1e6439ce4f3fa05ff322aa57edfd60fd_720w.jpg?source=1940ef5c)

![img](https://pic1.zhimg.com/50/v2-f890171649f892f55a833bc28092016e_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-f890171649f892f55a833bc28092016e_720w.jpg?source=1940ef5c)

![img](https://pic1.zhimg.com/50/v2-1b8f75a8cb6a410e727356b693a161db_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-1b8f75a8cb6a410e727356b693a161db_720w.jpg?source=1940ef5c)

 MySQL相对于PostgreSQL的劣势：

![img](https://pica.zhimg.com/50/v2-226fd5e2cf8dbdd7ab0a43f371d17aee_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-226fd5e2cf8dbdd7ab0a43f371d17aee_720w.jpg?source=1940ef5c)

![img](https://pic2.zhimg.com/50/v2-71bfcfc6933b45d0263683dd3f8d217c_720w.jpg?source=1940ef5c)![img](https://pic2.zhimg.com/80/v2-71bfcfc6933b45d0263683dd3f8d217c_720w.jpg?source=1940ef5c)

PostgreSQL主要优势：

　　1. PostgreSQL完全免费，而且是BSD协议，如果你把PostgreSQL改一改，然后再拿去卖钱，也没有人管你，这一点很重要，这表明了PostgreSQL数据库不会被其它公司控制。[oracle数据库](https://www.zhihu.com/search?q=oracle数据库&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463})不用说了，是商业数据库，不开放。而MySQL数据库虽然是开源的，但现在随着SUN被[oracle公司](https://www.zhihu.com/search?q=oracle公司&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463})收购，现在基本上被oracle公司控制，其实在SUN被收购之前，MySQL中最重要的InnoDB引擎也是被oracle公司控制的，而在MySQL中很多重要的数据都是放在InnoDB引擎中的，反正我们公司都是这样的。所以如果MySQL的市场范围与oracle数据库的市场范围冲突时，oracle公司必定会牺牲MySQL，这是毫无疑问的。

　　2. 与PostgreSQl配合的开源软件很多，有很多分布式集群软件，如pgpool、pgcluster、slony、plploxy等等，很容易做读写分离、负载均衡、数据水平拆分等方案，而这在MySQL下则比较困难。

​      \3. PostgreSQL源代码写的很清晰，易读性比MySQL强太多了，怀疑MySQL的源代码被混淆过。所以很多公司都是基本PostgreSQL做二次开发的。

​      \4. PostgreSQL在很多方面都比MySQL强，如复杂SQL的执行、存储过程、触发器、索引。同时PostgreSQL是多进程的，而MySQL是线程的，虽然并发不高时，MySQL处理速度快，但当并发高的时候，对于现在多核的单台机器上，MySQL的总体处理性能不如PostgreSQL，原因是MySQL的线程无法充分利用CPU的能力。

​     目前只想到这些，以后想到再添加，欢迎大家拍砖。

**PostgreSQL与oracle或InnoDB的多版本实现的差别**

PostgreSQL与oracle或InnoDB的多版本实现最大的区别在于最新版本和历史版本是否分离存储，PostgreSQL不分，而oracle和InnoDB分，而innodb也只是分离了数据,索引本身没有分开。
   PostgreSQL的主要优势在于：
   \1. PostgreSQL没有回滚段，而oracle与innodb有回滚段，oracle与Innodb都有回滚段。对于oracle与Innodb来说，回滚段是非常重要的，回滚段损坏，会导致数据丢失，甚至数据库无法启动的严重问题。另由于PostgreSQL没有回滚段，旧数据都是记录在原先的文件中，所以当数据库异常crash后，恢复时，不会象oracle与Innodb数据库那样进行那么复杂的恢复，因为oracle与Innodb恢复时同步需要redo和undo。所以PostgreSQL数据库在出现异常crash后，数据库起不来的几率要比oracle和mysql小一些。
   \2. 由于旧的数据是直接记录在数据文件中，而不是回滚段中，所以不会象oracle那样经常报ora-01555错误。
   \3. 回滚可以很快完成，因为回滚并不删除数据，而oracle与Innodb，回滚时很复杂，在事务回滚时必须清理该事务所进行的修改，插入的记录要删除，更新的记录要更新回来(见[row_undo函数](https://www.zhihu.com/search?q=row_undo函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463}))，同时回滚的过程也会再次产生大量的redo日志。
   \4. WAL日志要比oracle和Innodb简单，对于oracle不仅需要记录数据文件的变化，还要记录回滚段的变化。
**PostgreSQL的多版本的主要劣势在于：**
   1、最新版本和历史版本不分离存储，导致清理老旧版本需要作更多的扫描，代价比较大，但一般的数据库都有高峰期，如果我们合理安排VACUUM，这也不是很大的问题，而且在PostgreSQL9.0中VACUUM进一步被加强了。
　　2、由于索引中完全没有版本信息，不能实现Coverage [index scan](https://www.zhihu.com/search?q=index+scan&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463})，即查询只扫描索引，直接从索引中返回所需的属性，还需要访问表。而oracle与Innodb则可以;

**[进程模式](https://www.zhihu.com/search?q=进程模式&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463})与线程模式的对比**
PostgreSQL和oracle是进程模式，MySQL是[线程模式](https://www.zhihu.com/search?q=线程模式&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463})。
进程模式对多CPU利用率比较高。
进程模式共享数据需要用到共享内存，而线程模式数据本身就是在进程空间内都是共享的，不同线程访问只需要控制好线程之间的同步。
线程模式对资源消耗比较少。
所以MySQL能支持远比oracle多的更多的连接。
对于PostgreSQL的来说，如果不使用连接池软件，也存在这个问题，但PostgreSQL中有优秀的连接池软件软件，如pgbouncer和pgpool，所以通过连接池也可以支持很多的连接。

**堆表与索引组织表的的对比**

Oracle支持堆表，也支持索引组织表
PostgreSQL只支持堆表，不支持索引组织表
Innodb只支持索引组织表
索引组织表的优势：
表内的数据就是按索引的方式组织，数据是有序的，如果数据都是按主键来访问，那么访问数据比较快。而堆表，按主键访问数据时，是需要先按主键索引找到数据的物理位置。
索引组织表的劣势：
索引组织表中上再加其它的索引时，其它的索引记录的数据位置不再是物理位置，而是主键值，所以对于索引组织表来说，主键的值不能太大，否则占用的空间比较大。
对于索引组织表来说，如果每次在中间插入数据，可能会导致索引分裂，索引分裂会大大降低插入的性能。所以对于使用innodb来说，我们一般最好让主键是一个无意义的序列，这样插入每次都发生在最后，以避免这个问题。
由于索引组织表是按一个索引树，一般它访问数据块必须按数据块之间的关系进行访问，而不是按物理块的访问数据的，所以当做全表扫描时要比堆表慢很多，这可能在OLTP中不明显，但在数据仓库的应用中可能是一个问题。

　 PostgreSQL9.0中的特色功能：   
  PostgreSQL中的Hot Standby功能
    也就是standby在应用日志同步时，还可以提供只读服务，这对做读写分离很有用。这个功能是oracle11g才有的功能。

  PostgreSQL[异步提交](https://www.zhihu.com/search?q=异步提交&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463})（Asynchronous Commit）的功能：
　 这个功能oracle中也是到oracle11g R2才有的功能。因为在很多应用场景中，当宕机时是允许丢失少量数据的，这个功能在这样的场景中就特别合适。在PostgreSQL9.0中把synchronous_commit设置为false就打开了这个功能。需要注意的是，虽然设置为了异步提交，当主机宕机时，PostgreSQL只会丢失少量数据，异步提交并不会导致数据损坏而数据库起不来的情况。MySQL中没有听说过有这个功能。

PostgreSQL中索引的特色功能：

​     PostgreSQL中可以有部分索引，也就是只能表中的部分数据做索引，[create index](https://www.zhihu.com/search?q=create+index&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A743955463}) 可以带where 条件。同时PostgreSQL中的索引可以反向扫描，所以在PostgreSQL中可以不必建专门的降序索引了。