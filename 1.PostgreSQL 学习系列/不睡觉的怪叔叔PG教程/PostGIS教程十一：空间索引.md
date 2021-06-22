# PostGIS教程十一：空间索引

 回想一下，**空间索引**是空间数据库的三个关键特性之一。**空间索引**使得使用空间数据库存储大型数据集成为可能。在没有**空间索引**的情况下，对要素的任何搜索都需要对数据库中的每条记录进行"顺序扫描"。**索引**通过将数据组织到**搜索树**中来加快搜索速度，**搜索树**可以快速遍历以查找特定记录。

  **空间索引**是PostGIS的最大价值之一。在前面的示例中，构建**空间连接**需要对整个表进行相互比较。这样做的代价很高：连接两个包含10000条记录的表（每个表都没有**索引**）将需要进行100000000次比较；如果使用**空间索引**，则比较次数可能低至20000次。
   加载nyc_census_blocks表时，**pgShapeLoader**会自动创建名为nyc_census_blocks_geom_idx的**空间索引**。

  为了演示**空间索引**对性能有多重要，让我们在没有**空间索引**的情况下搜索nyc_census_blocks表。

  我们的第一步是删除索引：

```sql
DROP INDEX nyc_census_blocks_geom_idx;
```

![img](https://img-blog.csdnimg.cn/20190110094206325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  注意：**DROP INDEX**语句从数据库系统中删除现有索引。有关更多信息，请参见[PostgreSQL文档](http://www.postgresql.org/docs/7.4/interactive/sql-dropindex.html)。

  现在，查看pgAdmin查询窗口右下角的"**计时表**"并运行以下命令。我们的查询将搜索每个单独的**人口普查块**（census block），以查找**宽街**（Broad Street）那个记录。

```sql
SELECT blocks.blkid
FROM nyc_census_blocks blocks
JOIN nyc_subway_stations subways
ON ST_Contains(blocks.geom, subways.geom)
WHERE subways.name = 'Broad St';
```

![img](https://img-blog.csdnimg.cn/20190110094233576.png)

![img](https://img-blog.csdnimg.cn/20190110094359229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  nyc_census_blocks表非常小（只有几千条记录），因此即时没有**索引**，查询也非常快。

  现在，重新添加**空间索引**并再次进行查询：

```sql
CREATE INDEX nyc_census_blocks_geom_idx
ON nyc_census_blocks
USING GIST (geom);
```

![img](https://img-blog.csdnimg.cn/20190110094741535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：**USING GIST**子句告诉PostgreSQL在构建索引时使用generic index structure（**GIST-通用索引结构**）。创建索引时，如果收到类似错误：ERROR:index row requires 11340 bytes，maximum size is 8911，则可能是因为没有添加USING GIST子句。

![img](https://img-blog.csdnimg.cn/20190110095258530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  在我的测试计算机上，时间下降到11毫秒。表越大，索引查询的相对速度提高就越大。

 

# 一、空间索引是怎样工作的？

  标准数据库索引基于某个列的值创建层次结构树。**空间索引**略有不同-它们不能索引几何要素本身，而是索引几何要素的边界框。

![_images/bbox.png](https://postgis.net/workshops/postgis-intro/_images/bbox.png)

  在上图中，与**黄星**相交的线串数是一条，即**红线**。但是与**黄色框**相交的要素的边界框是两个，红框和蓝框。

  空间数据库回答"**哪些直线与黄星相交**"这一问题的方法是，首先使用**空间索引**（速度非常快）判断"**哪些框与黄色框相交**"，然后仅对第一次返回的几何要素进行"**哪些直线与黄星相交**"的精确计算。

  对于一个大的数据表来说，这种先评估近似索引，然后进行精确测试的"两遍"机制可以从根本上减少计算量。

  PostGIS和Oracle Spatial都具有相同的"R-Tree"空间索引结构。R-Tree将数据分解为**矩形**（rectangle）、**子矩形**（sub-rectangle）和**子-子矩形**（sub-sub rectangle）等。它是一种自调优（self-tuning）索引结构，可自动处理可变数据的密度和对象大小。

![_images/index-01.png](https://postgis.net/workshops/postgis-intro/_images/index-01.png)

 

# 二、纯索引查询

  PostGIS中最常用的函数（ST_Contains、ST_Intersects、ST_DWithin等）都包含**自动索引过滤器**。但有些函数（如ST_Relate）不包括**索引过滤器**。

  要使用索引执行边界框搜索（即**纯索引查询**-Index only Query-没有过滤器），需要使用"**&&**"运算符。对于几何图形，&&运算符表示"边界框重叠或接触"（纯索引查询），就像对于数字，"**=**"运算符表示"值相同"。

  让我们将对"West Village"社区人口的**纯空间索引查询**与更精确的查询进行比较。使用&&操作符的**纯索引查询**如下所示：

```sql
SELECT Sum(popn_total)
FROM nyc_neighborhoods neighborhoods
JOIN nyc_census_blocks blocks
ON neighborhoods.geom && blocks.geom
WHERE neighborhoods.name = 'West Village';
```

![img](https://img-blog.csdnimg.cn/20190110102258505.png)

![img](https://img-blog.csdnimg.cn/20190110102451611.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  现在，让我们使用更精确的ST_Intersects函数执行相同的查询：

```sql
SELECT Sum(popn_total)
FROM nyc_neighborhoods neighborhoods
JOIN nyc_census_blocks blocks
ON ST_Intersects(neighborhoods.geom, blocks.geom)
WHERE neighborhoods.name = 'West Village';
```

![img](https://img-blog.csdnimg.cn/20190110102550732.png)

![img](https://img-blog.csdnimg.cn/20190110102705740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  结果数量低得多！第一个查询汇总与**社区**（neighborhood）关于边界框相交的每个**人口统计块**（census block）；第二个查询仅汇总了与该**社区**几何图形本身相交的**人口统计块**。

 

# 三、分析（ANALYZE）

  PostgreSQL查询规划器（query planner）智能地选择何时使用或不使用**空间索引**来计算查询。与直觉相反，执行**空间索引**搜索并不总是更快：如果搜索将返回表中的每条记录，则遍历索引树以获取每条记录实际上比从一开始线性读取整个表要慢。

  为了弄清楚要处理的数据的大概内容（读取表的一小部分信息，而不是读取表的大部分信息），PostgreSQL保存每个索引列中数据分布的**统计信息**。默认情况下，PostgreSQL定期收集统计信息。但是，如果你在短时间内更改了表的构成，则统计数据将不会是最新的。

  为确保统计信息与表内容匹配，明智的做法是在表中加载和删除大容量数据后运行**ANALYZE命令**。这将强制统计系统收集所有索引列的统计信息。

  ANALYZE命令要求PostgreSQL遍历该表并更新用于查询操作而估算的内部统计信息。

```sql
ANALYZE nyc_census_blocks;
```

 

# 四、清理（VACUUM）

  值得强调的是，仅仅创建**空间索引**不足以让PostgreSQL有效地使用它。每当创建新索引或对表大量更新、插入或删除后，都必须执行**清理（VACUUMing）**。**VACUUM**命令要求PostgreSQL回收表页面中因记录的更新或删除而留下的任何未使用的空间。

  **清理**对于数据库的高效运行非常关键，因此，PostgreSQL提供了一个“**自动清理**（autovacuum）"选项。

  默认情况下，**自动清理机制**会根据活动级别确定的合理时间间隔自动清理（恢复空间）和分析（更新统计信息）。虽然这对于高度事务性的数据库是必不可少的功能，但在添加索引或大容量数据之后等待自动清理运行是不明智的，如果执行大批量更新，则应该手动运行VACUUM命令。

  根据需要，可以单独执行清理和分析。发出VACUUM命令不会更新数据库统计信息；同样，执行ANALYZE命令也不会清理未使用的表空间。这两个命令都可以针对整个数据库、单个表或单个列运行。

```sql
VACUUM ANALYZE nyc_census_blocks;
```

 

# 五、相关函数

![img](https://img-blog.csdnimg.cn/20190111143636742.png)

 

附录： [PostGIS官方教程汇总目录](https://blog.csdn.net/qq_35732147/article/details/85256640)