- [PostgreSQL 和 MySQL 在用途、好处、特性和特点上的异同 - 初见洞洞拐 - 博客园 (cnblogs.com)](https://www.cnblogs.com/chujian007/p/16310090.html)

## PostgreSQL 和 MySQL 在用途、好处、特性和特点上的异同。

PostgreSQL 和 MySQL 是将数据组织成表的[关系数据库](https://www.ibm.com/cloud/learn/relational-databases)。这些表可以根据每个表共有的数据链接或关联。关系数据库使您的企业能够更好地了解可用数据之间的关系，并帮助获得新的见解以做出更好的决策或发现新的机会。

## PostgreSQL 和 MySQL 有何相似之处？

PostgreSQL 和 MySQL 都依赖于 SQL（结构化查询语言），这是与管理系统交互的标准语言。SQL 允许使用具有简单结构的几行源代码连接表，大多数非技术员工可以快速学习。

使用 SQL，分析师不需要知道订单表在磁盘上的位置、如何执行查找以查找特定订单或如何连接订单表和客户表。数据库编译查询并计算出正确的数据点。

MySQL 和 PostgreSQL 都支持 JavaScript Object Notation (JSON) 来存储和传输数据，尽管 PostgreSQL 也支持 JSONB，这是 JSON 的二进制版本，它消除了键的重复和无关的空格。

除了传统的支持机制外，这两个数据库都提供强大的社区支持。

## 什么是 PostgreSQL？

[PostgreSQL](https://www.ibm.com/cloud/learn/postgresql)，也称为 Postgres，是一种开源关系数据库，因其可靠性、灵活性和对开放技术标准的支持而享有盛誉。PostgreSQL 支持非关系和关系数据类型。它被称为当今可用的最兼容、最稳定和最成熟的关系数据库之一，并且可以轻松处理复杂的查询。

PostgreSQL 的特性包括：

- 时间点恢复 (PITR) 将数据库还原到特定时间点。
- 使用 pgBackRest 等工具记录对数据库的所有更改的预写日志 (WAL)。
- 用于创建和保留自定义子例程的[存储过程。](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_74/sqlp/rbafysproeg.htm)

## PostgreSQL是如何使用的？

PostgreSQL 是一个“一刀切”的解决方案，适用于许多寻求经济高效的方法来改进其数据库管理系统 (DBMS) 的企业。它具有足够的可扩展性和多功能性，可以通过强大的扩展生态系统快速支持各种专业用例，涵盖时间序列数据类型和地理空间分析等工作。作为开源数据库解决方案构建的 PostgreSQL 完全不受许可限制、供应商锁定的可能性或过度部署的风险。PostgreSQL 通过对象关系数据库管理系统 (ORDBMS) 进行管理。

PostgreSQL 为负责管理业务活动的[在线事务处理 (OLTP)](https://www.ibm.com/cloud/learn/oltp)协议的企业数据库管理员提供了理想的解决方案，包括电子商务、客户关系管理系统 (CRM) 和财务分类帐。它也是管理接收、创建和生成的数据分析的理想选择。

## PostgreSQL 的好处

这些是 PostgreSQL 的一些主要优点：

- **性能和可扩展性**——包括地理空间支持和无限制的并发性——以及跨多种数据类型的深入、广泛的数据分析。
- 通过使用多版本并发控制 (MVCC)**支持并发，这使得写操作和读操作同时发生。**
- 由于它的兼容性和对多种编程语言的支持，包括[Python](https://www.ibm.com/cloud/blog/python-vs-r)、Java、JavaScript、C/C++ 和 Ruby，因此支持**深度语言。**
- **业务连续性**，通过跨服务器的异步或同步复制方法提供高可用性服务。
- 通过开源数据库管理技术实现**更大的灵活性和成本效益的创新。**

## 什么是 MySQL 数据库？

MySQL — 一种快速、可靠、可扩展且易于使用的开源关系数据库系统 — 旨在处理关键任务、高负载的生产应用程序。它是一种常见且易于启动的数据库，内存、磁盘和 CPU 利用率较低，由关系数据库管理系统 (RDMS) 管理。MySQL Community Edition 是一个由活跃的在线社区支持的免费下载版本。

MySQL 功能包括所有 SQL 标准命令以及事务和 ACID 合规性（代表原子性、一致性、隔离性和持久性）。

两个最常见的关系数据库是 MySQL 和 Oracle。MySQL 不是 SQL Server 的同义词，SQL Server 是 Microsoft 许可产品，与 MAC OS X 缺乏兼容性。

MariaDB 经常与 MySQL 混淆，它是 MySQL 的一个开源分支，速度更快，提供更多存储引擎 (12)，但功能有限。MySQL 和 MariaDB 使用的存储引擎都是 InnoDB。InnoDB 提供标准的 ACID 兼容特性。与 MySQL 不同，MariaDB 不支持数据屏蔽或动态列。

## MySQL是如何使用的？

MySQL 通常用作 Web 数据库来存储各种信息类型，从单个信息数据点到为组织提供的产品或服务的完整列表。它是[LAMP](https://www.ibm.com/cloud/learn/lamp-stack-explained)（Linux 操作系统、Apache HTTP 服务器、MySQL RDBMS 和 PHP 编程语言）的基础组件，这是一种有助于创建[API](https://www.ibm.com/cloud/learn/api)、Web 应用程序和网站的软件堆栈模型。

MySQL Workbench 是一个单一的、集成的可视化 SQL 平台，用于 MySQL 数据库的创建、开发、设计和管理。

## MySQL的好处

MySQL 为市场提供了许多好处，包括：

- **与其他数据库管理平台相比，无与伦比的数据安全性**是由于它使用了安全套接字层 (SSL)。这有助于确保数据完整性，使其成为 Web 应用程序的流行数据库。
- **高性能**，因为 MySQL 的存储引擎框架支持具有高速部分索引、全文索引和独特内存缓存的要求苛刻的应用程序，以实现卓越的数据库性能。
- **可扩展性**和对在小空间内无限存储增长的支持。
- **支持事务处理的灵活开源框架**，但不如 NoSQL 等非关系数据库灵活。

## PostgreSQL vs MySQL：有什么区别？

PostgreSQL 和 MySQL 之间有很多不同之处。特性、功能和优势方面的一些差异如下：

- 数据库类型
  - **MySQL**：关系型
  - **PostgreSQL**：对象关系
- 编程语言
  - **MySQL**： C/C++
  - **PostgreSQL**： C
- 支持级联
  - **MySQL**：没有
  - **PostgreSQL**：是的
- 用户界面
  - **MySQL**：工作台 GUI
  - **PostgreSQL**：PgAdmin
- 支持的过程复杂度
  - **MySQL**：SQL 语法和存储过程
  - **PostgreSQL**：高级过程和存储过程
- 支持的索引类型
  - **MySQL**：二叉搜索树（B-Tree）
  - **PostgreSQL**：很多，包括 GIN 和 Hash
- 客户端和服务器之间的加密
  - **MySQL**：传输层安全 (TLS) 协议
  - **PostgreSQL**： SSL
- XML 数据类型支持
  - **MySQL**：没有
  - **PostgreSQL**：是的
- 支持物化视图和表继承
  - **MySQL**：没有
  - **PostgreSQL**：是的
- 支持高级数据类型
  - **MySQL**：没有
  - **PostgreSQL**：是的——hstore 和用户定义的 tdtaa

总之，PostgreSQL 和 MySQL 都有不同的用途，它们之间的选择取决于企业目标和资源。一般来说，PostgreSQL 是一个更强大、更高级的数据库管理系统，非常适合需要在大型环境中快速执行复杂查询的组织。但是，对于预算和空间更受限制的公司来说，MySQL 是一个理想的解决方案。