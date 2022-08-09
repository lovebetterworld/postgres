- [（四）pgRouting最短路径查询_会飞的猪biubiu的博客-CSDN博客_pgr_dijkstra](https://blog.csdn.net/qq_29384639/article/details/122035726?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8-122035726-blog-83351481.pc_relevant_multi_platform_whitelistv2eslanding&spm=1001.2101.3001.4242.5&utm_relevant_index=11)

## 1 添加PostGIS插件和pgRouting插件

```sql
create extension postgis;
create extension pgrouting;
```

## 2 修改表

首先为aaa_bbb表添加四个字段：

- source —— 用于保存路径起始顶点的id
- target —— 用于保存路径终止顶点的id
- cost —— 用于保存路径正向的成本（或者代价）
- reverse_cost —— 用于保存路径反向的成本（或者代价）

[SQL语句](https://so.csdn.net/so/search?q=SQL语句&spm=1001.2101.3001.7020)：

```sql
ALTER TABLE aaa_bbb
ADD COLUMN source INTEGER,
ADD COLUMN target INTEGER,
ADD COLUMN cost DOUBLE PRECISION,
ADD COLUMN reverse_cost DOUBLE PRECISION;
```

OSM数据包含一个oneway列，它的值可能是以下三个值之一：

- '**F**' —— 表示该路径是单向的，且路径方向是正向的。
- '**T**' —— 表示该路径是单向的，且路径方向是反向的。
- '**B**' —— 表示该路径是双向的。

现在就根据路径的方向来计算各个路径的成本。

将路径的长度作为成本，如果路径是正向的，则cost值为路径的长度，reverse_cost值为-1；如果路径是反向的，则reverse_cost值为路径的长度，cost值为-1；如果路径是双向的，则cost和reverse_cost的值都为路径的长度。

①正向路径的成本：

```sql
UPDATE aaa_bbb
SET cost = ST_Length(geom), reverse_cost = -1
WHERE oneway = 'F';
```

②反向路径的成本：

```sql
UPDATE aaa_bbb
SET reverse_cost = ST_Length(geom), cost = -1
WHERE oneway = 'T';
```

③双向路径的成本：

```sql
UPDATE aaa_bbb
SET cost = ST_Length(geom), reverse_cost = ST_Length(geom)
WHERE oneway = 'B';
```

## **3 创建路网拓扑**

创建路网拓扑需要调用pgr_createTopology数：

```sql
SELECT pgr_createTopology(
	'aaa_bbb', 
	0.001,
	'geom',
	'gid',
	'source',
	'target'
); 
```

上面的六个参数分别表示：

- 路网表
- 路径之间的容差，两条路径的距离大于这个容差值，就表示它们不相交，否则就是相交。
- 路网表中包含空间信息的列
- 路网表的主码列
- 保存路径起始顶点的id的列
- 保存路径终止顶点的id的列

拓扑路径创建完成后，数据库中会自动多出一张表**aaa_bbb_vertices_pgr**

![img](https://img-blog.csdnimg.cn/e2c8b3bc486b47a48c9011288c7dcd85.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_8,color_FFFFFF,t_70,g_se,x_16)

 这张表保存了路径的起始、终止顶点数据

![img](https://img-blog.csdnimg.cn/4eccd205f8254c2f972e4f8c87ce875e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_20,color_FFFFFF,t_70,g_se,x_16)

 我们再查询路网表aaa_bbb，可以发现source列和target列都被填满了，值就是aaa_bbb_vertices_pgr这张表中对应的的id值

![img](https://img-blog.csdnimg.cn/4ddc7ad4d9594183bde8faefd84ad9bb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_20,color_FFFFFF,t_70,g_se,x_16)

##  4 pgr_dijkstra计算最短路径

Dijkstra算法是第一个在pgRouting中实现的算法。它不需要除id、source、target和cost之外的其他属性。而且可以明确指定将图视为**有向的**或**无向的**。

**pgr_dijkstra函数的签名摘要**

![img](https://img-blog.csdnimg.cn/857bd70f1e1e4ab8a611cd5d126d822b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_9,color_FFFFFF,t_70,g_se,x_16)

详情查看[pgRouting官方文档：pgr_dijkstra - 知乎原文地址： pgr_dijkstra - pgRouting Manual (2.6)本文所用的实验数据请参考下面这篇文章： 不睡觉的怪叔叔：pgRouting官方文档：简单数据 pgr_dijkstra —— 使用dijkstra算法返回最短路径。基于Boost.Graph实现…![img](https://static.zhihu.com/heifetz/assets/apple-touch-icon-152.a53ae37b.png)https://zhuanlan.zhihu.com/p/85905703](https://zhuanlan.zhihu.com/p/85905703) 

**注意：**

- 许多pgRouting函数都将sql :: text作为参数之一，虽然这开始看起来很混乱，但它使函数非常灵活。用户可以传递任何SELECT语句作为函数参数，只要该SELECT语句返回的结果包含所需数量的属性且包含正确的属性名。
- 多数pgRouting实现的算法不需要路网几何信息。
- 大多数pgRouting函数不返回几何信息，而只返回包含节点id或路径id的有序列表。

## 5 查询的标识符

source列和target列的顶点标识符（对应shenzhen_roads_vertices_pgr的id值）的分配可能不同，因为**pgr_createTopology函数**随机分配顶点标识符。所以每次运行pgr_createTopology函数建立路网拓扑，source列和target列的顶点标识符都是不同的。（因此仅以文章中的内容作为参考，然后使用你自己的数据进行测试）。

**选取两个点**

```csharp
select * from aaa_bbb_vertices_pgr
```

点1 id为1521

![img](https://img-blog.csdnimg.cn/b3f9e36d95f644008095c455944f82ba.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_20,color_FFFFFF,t_70,g_se,x_16)

 点2 id为1516

![img](https://img-blog.csdnimg.cn/38a270c611b24815a87c8552430d1775.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_20,color_FFFFFF,t_70,g_se,x_16)

 

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost, reverse_cost
	FROM aaa_bbb',
	1521, 1516,
	directed := FALSE
);
```

![img](https://img-blog.csdnimg.cn/0f855e879f1a41bd9f323490f18ac47d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_20,color_FFFFFF,t_70,g_se,x_16)

 以上的node列表示会经过的顶点，edge列表示会经过的路径，cost表示对应单条路径的成本，agg_cost表示路径的总成本。

**注意：**

- 返回的cost属性表示edges_sql参数中指定的cost属性。在这个例子中，cost是路径的长度。cost可以是时间、距离或任何其他属性以及自定义公式的组合。
- 结果中的node属性和edge属性取决于使用pgr_createTopology函数时对顶点标识符的分配。

**结果查看** 

```sql
select * from (
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost, reverse_cost
	FROM aaa_bbb',
	1521, 1516,
	directed := FALSE
))t1 left join aaa_bbb_vertices_pgr t2 on t1.node = t2.id
```

![img](https://img-blog.csdnimg.cn/b223298bf5874562a81c20f5c3bff4c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_20,color_FFFFFF,t_70,g_se,x_16)

 