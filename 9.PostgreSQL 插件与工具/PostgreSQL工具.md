- [值得推荐的60种不同功能的PostgreSQL工具](https://www.sohu.com/na/449220837_185201)



PostgreSQL(简称Postgres)具有许多现成的功能，它在开发人员和数据库工程师中备受欢迎。作为一款开源的产品，Postgres不但拥有强大的开发人员社区，而且带有许多附加组件。为了方便您迅速向Postgres中添加各种新的工具和插件，也为了让您能够扩展现有的Postgres数据库系统功能，本文为您总结了六十款工具、插件和附加组件，以协助您快速、有效地改善现有Postgres的运维方式和服务性能。

# 一、图形化用户界面(GUI)

虽然Postgres并没有自带原生的GUI，但是您可以通过如下特定的、基于Web的GUI和工具，来轻松地管理Postgres数据库。

## 1.1 DataGrip

作为一款可以协助简化管理多个数据库的工具，DataGrip能够与包括PostgreSQL在内的多种数据库系统相兼容。通过由它提供的图形化界面，您不但可以管理数据库，还能够运行查询，以及完成各种例行的维护任务。

## 1.2 DBeaver

DBeaver的最新版本--7.1.4，带有直观化的数据编辑功能。同时，它能够支持PostgreSQL，以及许多其他类型的数据库系统。

## 1.3 Navicat for PostgreSQL

Navicat在数据库领域已深耕多年。它的Postgres产品旨在为用户提供管理复杂数据库所需的各种工具。同时，它也提供了原生的数据可视化工具。

## 1.4 PgAdmin

在简化Postgres维护和管理方面，pgAdmin可谓老牌产品。如今，它不但能够基于Web选项、支持外部配置文件，并且还可以运行在云端。虽然它可以被用作管理数据库集群，但是与完整的GUI相比，pgAdmin还是略显简陋了一些。

## 1.5 Valentina Studio for PostgreSQL

Valentina Studio支持各种表单，可以与CI/CD管道相集成，还能够简化数据库之间的数据传输。它虽然具有不同的版本，但是即便是其免费的版本，也能够管理多个Postgres数据库。

## 1.6 phpPgAdmin

phpMyAdmin之于MySQL，正如phpPgAdmin之于PostgreSQL。两者在功能上既有相似之处，又有不同的地方。

## 1.7 Metabase

作为一款具有高级UI的数据处理工具，Metabase不但可以完成复杂的查询，还能够使用户通过可视化的方式，从PostgreSQL数据库中收集潜在的数据关系。

## 1.8 Slemma

Slemma远不止为Postgres提供GUI那么简单。通过引入自动化，它能够基于参数和数据之间的关系，自动生成可视化的数据报告。

## 1.9 Windward Studios

作为一款特殊的GUI工具，Windward可以与Microsoft Office进行原生地集成。您既可以使用Office应用来设计报告模板，又可以使用存储在Postgres中的数据，去可视化报告。

# 二、实用工具

Postgres的实用工具通常被设计为，用来处理某项特定的需求。可以说，在将良好的实用工具集成到数据库管理工作流中之后，数据库工程师的工作会比以往轻松许多。下面是目前比较流行的实用工具。

## 2.1 pg_catcheck

众所周知，系统目录的损坏可能会让您丢失数据条目，以及某些有价值的信息。而pg_catheck可以监控系统目录是否被损坏，是否会让整个Postgres数据库因故导致宕机。

## 2.2  pgBouncer

顾名思义，pgBouncer能够像保镖一般，阻止任何未经授权的访问。它经常作为负载平衡器，来管理各种连接。同时，您可以使用它来存储密码，加密SCRAM密钥，进而保障Postgres的安全性。

## 2.3 HypoPG

HypoPG可以在不消耗任何云端资源的情况下，建立虚拟索引，并且能够处理假设的分区。

## 2.4 PostGIS

PostGIS能够提供对空间信息的原生支持。Postgres用户可以使用PostGIS，在查询中为应用提供准确的位置信息。

## 2.5 Postgres_fdw

Postgres_fdw能够让外部数据包装器(Foreign-data  wrapper)轻松地访问外部的Postgres数据库。也就是说，您可以使用其他数据库中的对象，而无需内、外部进行真实同步。在该实用工具安装完成后，您可以创建一个外部服务器对象，以及相应的用户映射。

## 2.6 DB Doc for PostgreSQL

DB Doc for PostgreSQL能够为您所开发的项目，创建对应的文档。

# 三、平台即服务(PaaS)

如今，许多开发团队都希望能够以“零管理”的方式，支持其部署在云端架构中的Postgres。对此，如下PaaS提供了功能齐全、却略有不同的数据库托管服务。

## 3.1 Amazon RDS for PostgreSQL

Amazon的RDS通过提供云关系型数据库作为托管服务。它可以让用户完全使用由Postgres所提供的各项功能，而无需考虑存储、部署周期、可用性、以及备份等问题。

## 3.2 Aiven for PostgreSQL

Aiven for PostgreSQL提供了完全托管的SQL数据库。它可以在AWS、GCP、Azure和其他云生态系统上运行，以提高数据库的可用性。您可以先免费试用该平台，然后再切换到最适合自己需求的付费版本上。

## 3.3 Cloud SQL for PostgreSQL

Cloud SQL for PostgreSQL是Google云端关系型数据库的版本。它能够与其他的GCP服务很好地集成在一起。同时，它通过全面的API，来支持那些在多云环境中运行的应用。

## 3.4 Azure Database for PostgreSQL

Microsoft也提供了一个可扩展性的Azure Database for PostgreSQL。得益于支持机器学习，该PaaS提供了各种智能化的功能与性能。

## 3.5 DigitalOcean Managed Databases

DigitalOcean Managed Databases具有一定的价格方面优势，其起售价仅为每月15美元。它具有易于设置、无缝运维、日常备份、以及多冗余等功能，旨在支持各种应用和微服务。

## 3.6 Heroku PostgreSQL

Heroku PostgreSQL在提供全面的Postgres功能的同时，不会让整个平台显得过于臃肿和复杂。它在美国和欧洲都有销售。

# 四、应用领域

目前，许多工具都是旨在简化Postgres数据库的设计、关系的创建、表的管理、以及整个PostgreSQL平台的构建。下面，我们来讨论两个用于端到端数据库设计和管理的Postgres应用。

## 4.1 agileBase

agileBase以其低代码量(甚至是无代码)而闻名。您不必成为数据库专家，便可构建自己的平台，进而支持应用的交付。由于agileBase将其PostgreSQL功能设计为“积木”式，因此您可以按需定制。

## 4.2 Dataedo

您可以通过Dataedo的简单用户界面，来管理最为复杂的Postgres数据库。它不但可以直观地显示数据关系，还可以对其进行编辑。

# 五、高可用性

在实际应用中，我们往往需要在具有高可用性的环境中，实现PostgreSQL数据库，以避免由于数据库故障所导致的整个应用系统的崩溃。同时，我们可以通过如下工具，持续监控PostgreSQL的可用性。

## 5.1 PostgreSQL Dashboard

根据PostgreSQL Dashboard提供的各项关键性指标，我们可以轻松地获悉数据库的可用性，而无需手动浏览日志。同时，凭借着其直观的洞见显示，我们也可以通过完善云端架构，来提高数据库系统的可靠性。

## 5.2 Stolon

Stolon是一种原生的PostgreSQL管理工具。它旨在易于实现高可用性。通过提供诸如对Kubernetes的原生支持，以及自动化服务发现等功能，Stolon允许多个数据库实例同时运行，并为之提供冗余。

## 5.3 PostgreSQL Automatic Failover

PostgreSQL Automatic  Failover(PAF)是基于高可用性的行业标准—Pacemaker而开发的。您只需一次性配置PAF，定义诸如recovery_target_timeline和standby_mode等参数，即可为PostgreSQL数据库提供高可用性。

# 六、备份

云生态系统虽然能够提供较高的可用性，但是我们在日常运营中也少不了对于数据库的例行备份。下面，我们来讨论一些可以轻松实现Postgres自动化备份的工具。

## 6.1 Barman

作为PostgreSQL的完整灾难恢复方案，Barman以无缝的方式提供了对于热备份和冷备份的管理。它不但支持回滚，而且可以根据已配置的参数，自动对数据库的状态产生快照。更重要的是，Barman可以同时管理在多个云端环境中运行的数据库。

## 6.2 pg_probackup

作为Postgres的简单备份工具，pg_probackup简化了数据库集群中的备份过程。它既支持多个任务的并行化，又支持对数据库的文件进行数据去重等功能。

# 七、命令行界面(CLI)

尽管大部分PostgreSQL管理工具都提供了GUI，但是一些开发人员仍然喜欢使用命令行界面，来批量完成某些特定的操作。下面，我们来看看其中最为流行的、两种可以在终端上运行Postgres命令的工具。

## 7.1 Pgcli

顾名思义，Pgcli是Postgres的命令行界面。它能够为用户提供非常详细的信息，以及愉悦的使用体验。例如，当您输入\d参数时，它将为您可视化地显示数据表，并以序号标注每一个代码行。

## 7.2 pgsh

除了提供与Pgcli类似的功能，pgsh也能够管理数据库迁移等任务。您可以选择JavaScript和Python作为的首选语言。当然，前者在生产环境中被使用得更广一些。

# 八、服务器端

其实，数据库系统的性能在很大程度上取决于集群的可靠性。下面，我们来讨论两个用于创建和管理可扩展式PostgreSQL集群的工具。

## 8.1 Postgres-XL

Postgres-XL能够通过原生地使用负载平衡和多个节点，对OLTP的写入密集型工作负载提供支持。无论您的关系型数据库有多么复杂，Postgres-XL都能够创建和优化完美的数据库集群。

## 8.2 AgensGraph

通过与复杂的PostgreSQL数据库进行无缝的交互，AgensGraph使用图形化查询语言，来提高数据库集群的整体性能。

# 九、监控

虽然大部分云服务提供商，都为开发运营人员提供了诸如AWS CloudWatch之类的监控工具，但是它们往往无法真正提供PostgreSQL的详细性能信息。为此，我们可以选用如下监控管理工具。

## 9.1 Datasentinel

既可以被用于本地，又可以基于云端的Datasentinel，能够显示诸如：SQL统计信息、SQL活动的合并视图、以及会话工作负载等关键性指标。同时，它也可以实时采集数据，并处理各种历史数据。

## 9.2 PostgreSQL Dashboard

通过提供简单的仪表板，PostgreSQL Dashboard可以快速分析PostgreSQL的各项指标。其用户界面虽然缺少了自定义选项，但是方便了用户的使用与设置。因此，与深度分析相比，该工具更适用于快速监控的目的。

## 9.3 Pgbadger

作为一款内置了可视化工具的、快速可靠的日志分析器，Pgbadger允许用户设置为仅报告特定的错误和事件，从而有针对性地对数据库进行取证和详细监控。

## 9.4 Pgcluu

作为一种技术性极强的工具，Pgcluu可以通过可视化PostgreSQL集群节点的详细数据，方便用户持续监控数据库、乃至系统的性能。

## 9.5 Postgrestats

Postgrestats集成了统计信息的收集、显示与分析功能。由于它是用PHP和HTML5开发的，因此在部署时不会占用大量的云端资源。与PostgreSQL Dashboard相似，该软件包不但是轻量级的，而且能够让用户快速获悉数据库的性能状态。

## 9.6 PoWA

PostgreSQL Workload Analyzer(PoWA)不但可以分析数据库集群的工作负载与性能，还能够支持那些被用于创建假设索引(hypothetical indexes)的扩展项。

## 9.7 Check_postgres

Check_postgres可以灵活地与Nagios和MRTG相集成，以实现对数据库指定属性的详细监控，以及对配置进行深入检查。

# 十、扩展

作为一个非常流行的数据库系统，PostgreSQL可以根据不同的特定功能，集成许多自定义的扩展项。下面我们来讨论一些比较流行的扩展功能。

## 10.1 OpenFTS

开源全文搜索引擎(Open-Source Full-Text Search Engine，OpenFTS)能够处理在线索引，并启用搜索引擎等功能。它不但能够基于预定指标，对数据库的搜索结果进行排序，而且可以利用过滤器，来优化搜索结果。

## 10.2 AppOS

AppOS不但能够简化Postgres用户的存储管理，还可以被用于创建高效的、可预测的数据库框架。

## 10.3 PostPic

为了让PostgreSQL数据库中的图像处理功能，在应用程序中发挥作用，PostPic能够与PostGIS协作，对空间数据和图像进行深度处理。

## 10.4 Swarm64

作为一种优化类型的扩展，Swarm64可以提高数据的加载速度，优化存储空间的使用率，进而提升Postgres数据库的查询速度。

## 10.5 CyanAudit

顾名思义，由PL/SQL编写的CyanAudit，主要负责在不影响数据库性能的前提下，审核DML请求，并进行深入的日志记录。

## 10.6 Timescale

通过在关系型数据库系统中采集时序数据，Timescale可以在不牺牲PostgreSQL性能的情况下，堆叠(stack)包括关系查询和时序查询在内的各种复杂查询。

## 10.7 Prefix

常被用于电话领域应用的Prefix，可以提供各种自定义的前缀模式。例如：它既可以验证数据库的各个条目，又可以将它们与主键prefix_range进行比较。

## 10.8 PG-Storm

PG-Storm旨在加速数据库的分析和批处理操作。如果您的集群使用到了NVME-SSD和GPU，那么该扩展便可以加快PostgreSQL分析例程的速度。

## 10.9 PG-Themis

PG-Themis是一种使用Themis库进行加、解密的PostgreSQL扩展。您可以在SQL查询中添加加、解密命令，以确保最大的安全性。

# 十一、业务智能

存储在数据库中的数据需要为业务发挥应有的价值。为了以业务智能的方式处理和利用数据，我们通常会使用如下工具和高级算法，将数据分析的见解显示在仪表板上。

## 11.1 Chartio

作为一个仪表板，Chartio可与PostgreSQL数据库紧密协作。由于Chartio十分易用，因此您不必成为数据专家，即可执行诸如：查询和转换SQL条目之类的操作。

## 11.2 SeekTable

SeekTable能够允许您按需访问各种业务智能工具。SeekTable非常适合处理事件敏感型数据，并按需创建报告。通常，您无需导入现有的PostgreSQL数据库，即可处理各种数据条目。

## 11.3 Ubiq

Ubiq是一种将业务智能与PostgreSQL相集成的专业工具。它可以工作在云端或本地环境中，能够提供包括重复查询、以及自定义字段使用情况等信息的高级报告。

# 十二、集群

如前所述，我们可以通过云端架构和增加节点的方式，提高数据库集群的可扩展性，以及高可用性。如下工具恰好能够帮助您更好地控制数据库集群。

## 12.1 YugabyteDB

Yugabyte是一个高性能的开源分布式SQL数据库，它支持全局化的云原生应用。此类应用往往既能够提供与PostgreSQL相兼容的API，又可以被分布式地部署在多个地理位置。该工具非常适合那些希望通过云原生技术，管理数据库架构的企业。据此，企业可以提供SQL数据建模的灵活性，以及各项事务处理功能。

## 12.2 GridSQL

GridSQL专为PostgreSQL而设计。由于Postgres数据库可以分布在多个服务器上，因此GridSQL可以让数据库实现更快的查询、更短的响应时间、更高的性能、以及获取更多的服务器资源。

## 12.3 Hyperscale

Hyperscale也称为Citus，它是针对Azure用户的原生扩展。用户可以通过Hyperscale轻松地实现独立于集群的水平扩展，例如：将Postgres数据库布置到100多个节点上。

# 十三、优化

对于PostgreSQL数据库的优化，往往需要基于持续的监控，而非一蹴而就。如下优化工具可以方便您详细了解PostgreSQL数据库在支持应用的过程中，存在哪些性能上的瓶颈。

## 13.1 PGHero

PGHero集持续监控功能与数据库运行状况检查功能于一身，能够提供诸如：对于CPU(和云资源)使用情况的预测，更好的扩展性，自动清理，以及各种内置的数据库维护工具。

## 13.2 pgDash

作为一个专为PostgreSQL设计的全面监控方案，pgDash能够显示PostgreSQL数据库所需的所有核心报告，可视化各项功能和指标，创建详细的时序图，分析最新的数据，以及运行重要的诊断程序。

## 13.3 PGTune

PGTune能够为您在部署Postgres数据库时，计算出真正的服务器需求，以便您为此支付合理的费用。

## 13.4 PGMustard

PGMustard可以帮助用户发现那些需要长时间处理，以及更多服务器资源的查询，以便您在将PostgreSQL部署到生产环境之前，及时发现性能瓶颈，并优化查询。

## 13.5 PGConfig

虽然与PGTune非常相似，但是PGConfig提供了其他配置项，可协助用户模拟出不同的条件。例如，您可以根据服务器配置、或系统要求，找到work_mem，以及与检查点相关(checkpoint-related)的配置。