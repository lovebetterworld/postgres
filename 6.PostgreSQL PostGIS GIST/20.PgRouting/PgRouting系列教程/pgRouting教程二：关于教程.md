- [pgRouting教程二：关于教程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/82227139)

本教程使用了多个[FOSS4G](https://link.zhihu.com/?target=http%3A//www.osgeo.org/)（Free and Open Source Software For Geo）工具，大多数FOSS4G软件都与其他开放源代码项目相关，这里就不一一列出来了。

## 一、pgRouting概览

pgRouting扩展了PostgreSQL/PostGIS地理空间数据库的功能，以提供地理空间路由功能。pgRouting就是PostgreSQL中的一个插件。

数据库的路由功能的优点是：

- 数据和属性可以在许多客户端修改，比如**QGIS**和**uDig**可以通过**JDBC**、**ODBC**进行修改，或者直接使用**PL/pgSQL**进行修改。客户端可以是PC，也可以是移动设备。
- 数据更改可以通过路由引擎即时反映（实时计算），不需要预先计算。
- 可以通过SQL动态计算"**cost**"（成本、代价）参数，其值可以来自多个字段或表。

pgRouting库包含以下核心功能：

- [Dijkstra Algorithm](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_dijkstra.html)
- [Johnson’s Algorithm](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_johnson.html)
- [Floyd-Warshall Algorithm](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_floydWarshall.html)
- [A* Search Algorithm](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_aStar.html)
- [Bi-directional Dijkstra](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_bdDijkstra.html)
- [Bi-directional A*](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_bdAstar.html)
- [Traveling Salesperson Problem](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_TSP.html)
- [Driving Distance](https://link.zhihu.com/?target=https%3A//docs.pgrouting.org/latest/en/pgr_drivingDistance.html)
- Turn Restriction Shortest Path（TRSP）
- 更多！

pgRouting是开放源代码的，基于GPLv2许可，受[Georepublic](https://link.zhihu.com/?target=http%3A//georepublic.info/)、[iMaptools](https://link.zhihu.com/?target=http%3A//imaptools.com/)和广大用户社区的支持和维护。

pgRouting是[OSGeo Foundation](https://link.zhihu.com/?target=http%3A//osgeo.org/)的[OSGeo Community Projects](https://link.zhihu.com/?target=http%3A//wiki.osgeo.org/wiki/OSGeo_Community_Projects)项目，[OSGeoLive](https://link.zhihu.com/?target=http%3A//live.osgeo.org/)里也包含了它。

pgRouting官网：[http://www.pgrouting.org](https://link.zhihu.com/?target=http%3A//www.pgrouting.org/)

OSGeoLive: [https://live.osgeo.org/en/overview/pgrouting_overview.html](https://link.zhihu.com/?target=https%3A//live.osgeo.org/en/overview/pgrouting_overview.html)

## 二、osm2pgrouting概览

![img](https://pic3.zhimg.com/80/v2-0b58ef3ccecc54d2b4d1afd62c2b9e5a_720w.jpg)

**osm2pgrouting（Open Street Map To PgRouting）**是一个命令行工具，用于将OpenStreetMap数据导入pgRouting数据库。它会自动构建路由网络拓扑，并为要素类型和道路类别创建表。osm2pgrouting主要是由Daniel Wendt编写，现在托管在[pgRouting项目站点](https://link.zhihu.com/?target=https%3A//github.com/pgRouting/osm2pgrouting)上。

osm2pgrouting也是基于GPLv2许可。

维基：[https://github.com/pgRouting/osm2pgrouting/wiki](https://link.zhihu.com/?target=https%3A//github.com/pgRouting/osm2pgrouting/wiki)

## 三、OpenStreetMap概览

![img](https://pic1.zhimg.com/80/v2-4b16b26139724fbc7527b27f246e9728_720w.png)

"OpenStreetMap是一个旨在创建并向任何需要的人提供免费地理数据（如街道地图）的项目，启动该项目的目的是因为大多数你认为免费的地图在使用时实际上都有法律或技术的限制，阻碍了人们以创造性、生产性或奇特的方式自由使用它们。"

（来源: [http://wiki.openstreetmap.org/index.php/Press](https://link.zhihu.com/?target=http%3A//wiki.openstreetmap.org/index.php/Press))

对于pgRouting来说，OpenStreetMap是一个丰富的数据源，因为在处理数据方面没有技术限制。各国的数据可获取性仍然不尽相同，但是数据的世界范围覆盖性正在日益改善。

OpenStreetMap使用一种拓扑数据结构：

- 节点（Nodes） —— 具有地理位置的点。
- 道路（Ways） —— 节点的序列，被表示为折线（Poly Line）或者多边形
- 关系（Relations） —— 指可以被赋予为确定属性的一组节点、道路和其他地理要素之间的关系
- 属性（Properties） —— 可以指定给节点、道路或者关系，由name=value对组成

OpenStreetMap网站：[http://www.openstreetmap.org](https://link.zhihu.com/?target=http%3A//www.openstreetmap.org/)