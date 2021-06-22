# PostGIS教程二十：索引集群

数据库只能以从磁盘获取信息的速度检索信息。小型数据库将完全浮动于RAM缓存，并摆脱物理磁盘限制。但是对于大型数据库，对物理磁盘的访问将限制信息检索速度。

  数据是偶尔写入磁盘的，因此存储在磁盘上的有序数据与应用程序访问或组织该数据的方式之间不需要存在任何关联。

![img](https://img-blog.csdnimg.cn/20190301091519135.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  加速数据访问的一种方法是确保可能在同一结果集中一起检索的记录位于硬盘上的相似物理位置。这就是所谓的"**集群**（**clustering**）"。

  要使用正确的集群方案可能很棘手，但可以遵循一条一般规则：索引定义了数据的自然排序方案，该方案类似于检索数据的访问模式。

![_images/clustering2.jpg](https://postgis.net/workshops/postgis-intro/_images/clustering2.jpg)

  正因为如此，在某些情况下，以与索引相同的顺序对磁盘上的数据进行排序可以加速数据访问速度。

# 一、R-Tree上的集群

  空间数据倾向于在客户端的窗口中访问：想想Web应用程序或桌面应用程序中的地图窗口。窗口中的所有数据都具有相似的位置信息（否则它们将不在相同的窗口中！）。

  因此，基于空间索引的集群对于将通过空间查询访问的空间数据是有意义的：相似的事物往往具有相似的位置（**地理学第一定律**）。

  让我们根据nyc_census_blocks的空间索引对该表数据进行集群：

```sql
-- Cluster the blocks based on their spatial index
CLUSTER nyc_census_blocks USING nyc_census_blocks_geom_idx;
```

  ![img](https://img-blog.csdnimg.cn/20190301094049651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  该命令按照空间索引nyc_census_blocks_geom_idx所定义的顺序重新写入nyc_census_blocks。你能感觉到访问数据的速度的差异吗？可能不会，因为表很小，很容易装入内存，所以磁盘访问开销不会影响性能。

  R-Tree的一个令人惊讶的地方是，基于空间数据而递增构建的R-Tree可能没有很高的**叶节点**（每个叶节点对应一个区域和一个磁盘页）的空间协调性（spatial coherence）。例如，请参见不列颠哥伦比亚省道路索引的空间索引**叶节点**的可视化。

![_images/clustering3.jpg](https://postgis.net/workshops/postgis-intro/_images/clustering3.jpg)

  我们更喜欢使用空间更均衡紧凑、排列合理的R-tree索引结构进行集群，比如这种平衡的R-Tree（balanced R-Tree）。

![_images/clustering4.jpg](https://postgis.net/workshops/postgis-intro/_images/clustering4.jpg)

  我们在PostGIS中没有**平衡R-Tree的**算法，但我们有一个有用的代替方法，可以将空间数据进行空间自相关顺序排列，即**ST_GeoHash()**函数。

# 二、GeoHash上的集群

  要使用ST_GeoHash()函数进行集群，首先需要在数据上有一个**geohash索引**。幸运的是，它们很容易构建。

  geohash算法仅适用于地理（经度/纬度）坐标中的数据，因此我们需要在对其进行哈希操作之前先转换几何图形（转换为EPSG:4326，即经度/纬度）。

```sql
CREATE INDEX nyc_census_blocks_geohash ON nyc_census_blocks (ST_GeoHash(ST_Transform(geom,4326)));
```

![img](https://img-blog.csdnimg.cn/20190301101243177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  一旦有了geohash索引，就可以使用和R-Tree集群相同的语法进行集群。

```sql
CLUSTER nyc_census_blocks USING nyc_census_blocks_geohash;
```

 ![img](https://img-blog.csdnimg.cn/20190301101502147.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70) 

  现在，数据就很好地以空间相关的顺序排列！

# 三、函数列表

![img](https://img-blog.csdnimg.cn/20190301101644279.png)