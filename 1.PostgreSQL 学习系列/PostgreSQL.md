- [PostgreSQL](https://blog.csdn.net/weixin_41645135/article/details/122851226?utm_source=app&app_version=5.0.1&code=app_1562916241&uLinkId=usr1mkqgl919blen)

## ⛳️1.PostgreSQL 介绍

> PostgreSQL标榜自己是世界上最先进的开源数据库。
>  PostgreSQL的一些粉丝说它能与Oracle相媲美，
>  而且没有那么昂贵的价格和傲慢的客服。
>  它拥有很长的历史，最初是1985年在加利福尼亚大学伯克利分校开发的
>  经过长达15年以上的积极开发和不断改进，
>  PostgreSQL已在可靠性、稳定性、数据一致性等获得了业内相对高的声誉。

![在这里插入图片描述](https://img-blog.csdnimg.cn/297c8be3500144e9bc480ddcb2a81be5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASVTpgqblvrc=,size_20,color_FFFFFF,t_70,g_se,x_16)
 2022年数据库排行，与往年的2月份一样的平静。前十名排位来说较上个月没有变化，PG稳居第四

![在这里插入图片描述](https://img-blog.csdnimg.cn/7f5d735271e8402d83ab1c8e272425a3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASVTpgqblvrc=,size_20,color_FFFFFF,t_70,g_se,x_16)

PostgreSQL数据库有什么优势？

```
PostgreSQL数据库是功能强大的开源数据库，
它支持丰富的数据类型（如JSON和JSONB类型、数组类型）和自定义类型。

PostgreSQL数据库提供了丰富的接口，可以很方便地扩展它的功能，
如可以在GiST框架下实现自己的索引类型，支持使用C语言写自定义函数、触发器，
也支持使用流行的编程语言写自定义函数。

PostgreSQL数据库具有以下优势：

PostgreSQL数据库是目前功能最强大的开源数据库，
它是最接近工业标准SQL92的查询语言，
至少实现了SQL:2011标准中要求的179项主要功能中的160项
（注：目前没有哪个数据库管理系统能完全实现SQL:2011标准中的所有主要功能）。

稳定可靠：PostgreSQL是唯一能做到数据零丢失的开源数据库。
目前有报道称国内外有部分银行使用PostgreSQL数据库。

开源省钱： PostgreSQL数据库是开源的、免费的，
而且使用的是类BSD协议，在使用和二次开发上基本没有限制。

支持广泛：PostgreSQL 数据库支持大量的主流开发语言，
包括C、C++、Perl、Python、Java、Tcl以及PHP等。

PostgreSQL社区活跃：PostgreSQL基本上每3个月推出一个补丁版本，
这意味着已知的Bug很快会被修复，有应用场景的需求也会及时得到响应。
```

## ⛳️2.PostgreSQL相对于MySQL的优势

> 在SQL的标准实现上要比MySQL完善，而且功能实现比较严谨；
>  存储过程的功能支持要比MySQL好，具备本地缓存执行计划的能力；
>  对表连接支持较完整，优化器的功能较完整，支持的索引类型很多，复杂查询能力较强；
>  PG主表采用堆表存放，MySQL采用索引组织表，能够支持比MySQL更大的数据量。
>  PG的主备复制属于物理复制，相对于MySQL基于binlog的逻辑复制，数据的一致性更加可靠，复制性能更高，对主机性能的影响也更小。
>  MySQL的存储引擎插件化机制，存在锁机制复杂影响并发的问题，而PG不存在。

## ⛳️3.PG、Oracle和MySQL对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/d0e5ddf7557048c49e6ddfe31d567b42.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASVTpgqblvrc=,size_20,color_FFFFFF,t_70,g_se,x_16)

## ⛳️4.PG学习指引

```
1. 建议阅读《PostgreSQL学习的九层宝塔》：
https://mp.weixin.qq.com/s/i7b6FvY3PYC2JENCgiVxjQ
2.可自主学习阅读PG应用管理基础文档，
本部分内容主要是中国PG分会培训认证PGCA课程的节选，
相关链接：http://www.postgresqlchina.com/tecdoc
3.PG学习的主力站点
PG国际社区：https://www.postgresql.org/
1) PG概要：https://www.postgresql.org/about/
2) PG在线帮助文档（英文版本，多PG版本）：
https://www.postgresql.org/docs/
3）也可通过PG中文手册查阅学习，访问地址：
http://www.postgres.cn/docs/10/；
http://www.postgres.cn/docs/11/
4)安装介质下载地址：
https://www.postgresql.org/download/
主要有二进制、源码编译安装两种方式，二进制安装介质对应不同的操作系统。
例外还有一种基于PG的产品发布版本的安装，
可以通过产品的公司官网获得安装介质及安装方法，
譬如阿里POLARDB、亚信ANTDB、腾讯TBase、华为GaussDB、瀚高HGDB等。
5)在线学习资源
https://www.postgresql.org/docs/online-resources/
包含丰富的教程、动手练习资源
```

## ⛳️5.PostgreSQL工具

PostgreSQL的工具大体上可以分为以下几类：

备份恢复工具
 监控工具
 逻辑和基于触发器的复制工具
 多主复制工具
 高可用和故障转移工具
 连接池工具
 表分区工具
 迁移工具

### 🐴5.1 备份恢复工具

```
1. Barman
Barman (Backup and Recovery Manager-备份恢复管理器) 
是一个用Python语言实现的PostgreSQL灾难恢复管理工具，
它由第二象限公司(2ndQuadrant)开源并维护。它允许我们在关键业务环境中执行远程备份，
为数据库管理员在恢复阶段提供有效的数据保证。
Barman最优秀的功能包括备份元数据、增量备份、
保留策略、远程回复、WAL文件归档压缩和备份。

2. EDB BART
EDB BART(Backup and Recovery Tool -备份恢复工具)是企业级PostgreSQL数据管理策略的关键组件。
BART为大规模部署的PostgreSQL服务提供保留策略和基于时间点恢复的实现。
BART 2.0版本提供块级别的增量备份。

3. PgBackRest
pgBackRest工具的主要目的是做一款简单可靠的备份恢复工具，
以能够无缝的接入到大规模数据库和工作负载中。pgBackRest放弃了其他传统备份工具依赖tar和rsync的套路，
它的备份功能都是从软件内部实现的，并采用客户端协议与远程服务器交互。移除了对tar和rsync的依赖，
使它能够更好的应对针对特定数据库的备份挑战。客户端远程协议更加灵活，
协议可以按照要求限制连接类型以保证备份过程更安全。
```

### 🐴5.2、监控工具

```
1. PoWA
PoWA(PostgreSQL Workload Analyzer)是PostgreSQL的工作负载分析工具，
它收集性能数据并提供实时的图标和图片展示，以帮助我们监控和调优PostgreSQL服务器。
它和Oracle AWR或者SQL Server MDW很像。

2. PgCluu
pgCluu是一个PostgreSQL的性能监控和审计工具。
它以视图的形式展示您从PostgreSQL数据库集群收集的所有统计信息。
它能展示一份完成的数据库集群信息和系统使用率信息。

3. Pgwatch2
Pgwatch2是监控PostgreSQL数据库工具中最易用的一个。
它基于Grafana并为PostgreSQL数据库提供开箱即用的监控功能。
因为它已经集成到了容器里，所以我们不必担心各种依赖和复杂的安装步骤，
几分钟即可将监控搭建完毕，所有的东西都已经提前配置好。
我们只需要将数据库连接配置到监控中即可运行正常监控操作。
```

### 🐴5.3、逻辑和基于触发器的复制工具

```
1. pgLogical
pglogical是采用PostgreSQL扩展插件的形式实现的逻辑复制工具。
集成完善，不使用任何触发器和外部程序。该插件作为物理复制的替代者，
在有选择性的复制时采用发布/订阅模型，是复制数据的有效方式。

2. Slony-I
Slony-I是PostgreSQL一主多从复制体系的实现，支持级联复制。
开发Slony-I的主要目的是为了实现主从复制，
该复制体系包含大型数据库系统中对合理配置从系统所要求的所有特征和能力。

Slony-I主要为数据中心和备份站点场景设计，
这种场景下通常要求所有节点都是可用的。

3. Bucardo
Bucardo是一个PostgreSQL异步复制系统，允许配置多主多从操作。
它是http://Backcountry.com公司的Jon Jensen和Greg Sabino开发的。
```

### 🐴5.4、多主复制工具

```
BDR
Postgres-BDR(Bi-Directional Replication for PostgreSQL)
是世界上第一个开源PostgreSQL多主复制系统，目的是强化生产环境。
由第二象限(2ndQuadrant)公司开源并维护，BDR为地理分布集群环境特别设计，
使用搞笑的异步逻辑复制方式，支持从2个到48个以上节点在不同地域之间分布。
```

### 🐴5.5、高可用和故障转移工具

```
1. Repmgr
repmgr是一款开源的、用于PostgreSQL服务器集群复制管理和故障转移的工具。
它扩展了PostgreSQL内建的hot-standby能力，
可以设置热备份服务器、监控复制、执行管理任务(故障转移、手工切换等)。
repmgr是第二象限( 2ndQuadrant)公司开发的。

2. PAF
PAF(PostgreSQL Automatic Failover-自动故障转工具)是OCF资源代理贡献给PostgreSQL的，
它的初始目的是在Pacemaker管理和PostgreSQL划清规则，让事情变得简单、文档化和有效。
如果您的PostgreSQL集群启用了内部流复制，
PAF暴露给Pacemaker当前每一个PostgreSQL实例节点的状态：
哪个是主，哪个是从，哪个已停止，哪个正在追复制状态等等。如果主节点失败了，
Pacemaker默认首先恢复失败的主节点。如果失败不可恢复，
PAF会在从节点中选取一个最好的(与已失败主节点数据最为接近)提升为新的主节点。

3. Patroni
Patroni是一个模板，它使用Python为你提供一个自己订制的，高可用的解决方案，为
最大程度的可用性，它的配置信息存储在像ZooKeeper, etcd或者Consul中。
如果DBAs，DevOps工程师或者SRE正在寻找一个在数据中心中快速部署高可用PostgreSQL方案，
或者其他的用途，Patroni 能提供帮助。

4. Stolon
Stolon是一个cloud native的PostgreSQL高可用管理工具。
它之所以是cloud native的是因为它可以在为容器内部的PostgreSQL提供高可用(Kubernetes 集成)，
而且还支持其他种类的基础设施(比如：cloud IaaS，旧风格的基础设施等)
```

### 🐴5.6、Connection Pooling Tools

```
1. PgBouncer
PgBouncer是Skype的研发人员于2007年开发的连接池工具。
在那以后的很多年里，该项目已经由很多开发者参与改进，但是无论怎么变，
其降低PostgreSQL连接代价的角色一直未曾改变。
PgBouncer允许PostgreSQL数据库操作比其自身所能提供的最大连接数更大的客户端访问。
它本质上只追踪每一个客户端连接，然后基于配置信息，
创建一些客户端连接并基于先进先服务的原则服务于客户端访问。

2. PgPool-II
pgpool-II也是连接池，我们通常也习惯称它为pgpool。
它是另一个流行的连接代理，它早于PgBouncer一年左右的时间发布(2006年下半年发布)。
pgpool的使用范围非常关，所能提供的功能包括：基于查询的复制，连接池功能，负载均衡，并行查询等等，
pgpool的一个重要特定就是连接池。如果我们有两台PostgreSQL服务器，我们想使用虚拟IP，
这样客户端就不会感受到主数据库切换的影响。有时候，为了在服务器之间移动IP地址，
首先需要从主数据库服务器上把IP移除，然后在另外一台上重建，这就回中断活动链接，
导致短暂的服务不可用。使用pgpool可以缓存服务器直到另一台服务器提升上来，
pgpool会从内部处理故障转移，在应用和客户端的角度，数据库似乎从来没有下过线。
```

### 🐴5.7、表分区工具

```
1. Pg_Partman
pg_partman是PostgreSQL的一个扩展插件，用于创建和管理基于时间或者基于序列的表分区。
也支持多级子分区。子表和触发器都由扩展插件自身管理。已经有数据的表也能很容易的添加细粒度的分区。
可选的保留策略能够自动删除不再需要的分区。
后台工作进程(BGW)能够自动运行分区维护定时执行任务，而不需要依赖于linux cron等程序从外部进行维护。

2. pg_Pathman
pg_pathman是PostgreSQL Pro公司开源的扩展插件，可以为大型分布式数据库提供优化的分区解决方案。使用pg_pathman可以给大型数据库不停机分区，加速分区表查询，动态管理和增加分区，
为分区增加外部表，操作联合分区等。
```

### 🐴5.8、迁移工具

```
Ora2pg
Ora2Pg是一个用于将Oracle或MySQL数据库迁移到PostgreSQL的免费工具。
它能连接到Oracle数据库，然后自动扫描和导出源端的表结构或者数据，
转化为PostgreSQL数据库SQL脚本。Ora2Pg可以当作Oracle数据库的反向引擎，
用于大型企业级数据库迁移或者Oracle数据复制到PostgreSQL数据库等场景。
它易于使用，不需要任何Oracle数据库背景，你所需要做的仅仅是建立与Oracle数据库的连接而已。
```