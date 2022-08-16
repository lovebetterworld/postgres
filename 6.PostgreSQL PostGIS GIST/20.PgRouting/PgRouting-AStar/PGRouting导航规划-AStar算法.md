![思维导图](https://img-blog.csdnimg.cn/51732260008b4cecb2ff585bb047b6e8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA566A5Y2VR0lT,size_20,color_FFFFFF,t_70,g_se,x_16)

- 推荐数据库DBeaver，可以直接根据几何要素查看图形，相当方便。

## 1 生成拓扑

要生成最佳路径，首先要生成合法的拓扑。

生成拓扑前，需要添加两个字段，用来存储线段的首尾编号

```sql
-- 开启执行路网topo的插件
create extension postgis;
create extension postgis_topology;
create extension pgrouting;

-- Add "source" and "target" column
ALTER TABLE nyc_roads ADD COLUMN "source" integer;
ALTER TABLE nyc_roads ADD COLUMN "target" integer;

-- 创建索引，不然巨慢
create index if not exists pgr_source_idx on nyc_roads("source");
create index if not exists pgr_target_idx on nyc_roads("target");
```

- source —— 用于保存路径起始顶点的id
- target —— 用于保存路径终止顶点的id

调用pgr_createTopology生成拓扑，注意就是生成线段的首位编号的过程

```sql
pgr_createTopology(
'<table>',   -- 需要生成拓扑的表名
float tolerance,   --  容错值
'<geometry column>',   --  线段列名
'<gid>')  --  gid
```

容错值：例如线段的端不能完全吻合时，允许多少误差，单位一般为角度或公里数

官方说明：https://docs.pgrouting.org/3.1/en/pgr_createTopology.html

例子

```sql
-- Run topology function
SELECT pgr_createTopology('nyc_roads', 0.00001, 'geom', 'gid');
```

或者：

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

## 2 生成最佳路径

pgrouting支持的最佳路径算法很多。

官方说明：[https://docs.pgrouting.org/3.1/en/search.html?q=+shortest+path&check_keywords=yes&area=default](https://docs.pgrouting.org/3.1/en/search.html?q= shortest path&check_keywords=yes&area=default)
这里以Shortest Path A*和Shortest Path Dijkstra（狄克斯特拉）为例，介绍如何生成最佳路径

如果考虑回程成本的话，需要增加回程成本的字段，并设置为公里数。

```sql
ALTER TABLE nyc_roads ADD COLUMN reverse_cost double precision;
UPDATE nyc_roadsSET reverse_cost = length;
```

- cost —— 用于保存路径正向的成本（或者代价）
- reverse_cost —— 用于保存路径反向的成本（或者代价）

```sql
ALTER TABLE nyc_roads
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

### 2.1 Shortest Path Dijkstra算法举例

Dijkstra算法是第一个在pgRouting中实现的算法。它不需要除id、source、target和cost之外的其他属性。而且可以明确指定将图视为**有向的**或**无向的**。

**pgr_dijkstra函数的签名摘要**

![img](https://img-blog.csdnimg.cn/857bd70f1e1e4ab8a611cd5d126d822b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lya6aOe55qE54yqYml1Yml1,size_9,color_FFFFFF,t_70,g_se,x_16)

函数说明：

```sql
pgr_dijkstra(
text sql, -- 用于计算最佳路径的数据来源, 用SQL表示, 例如 
          -- SELECT id (gid), source (线段起点id), target (线段重点ID), cost (起点到终点的成本) [,reverse_cost (终点到起点的成本)] FROM edge_table
integer source,   --  规划路径的起点
integer target,   --  规划路径的终点
boolean directed   --  是否支持双向，如果为true，sql中必须有reverse_cost
);  
```

官方说明：https://docs.pgrouting.org/3.1/en/pgr_dijkstra.html

返回结果：(seq, path_seq ,node, edge, cost, agg_cost)

node:起点id

edge:目标ID, -1表示终点

```sql
SELECT * from  public.pgr_dijkstra(
	'SELECT
	gid AS id,
	source::integer,
	target::integer,
	length::double precision AS cost,
	reverse_cost
	FROM nyc_roads', 
	1, 
	9, 
	false
) 
```

![img](https://img-blog.csdnimg.cn/20200915215514531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)

**注意：**

- 许多pgRouting函数都将sql :: text作为参数之一，虽然这开始看起来很混乱，但它使函数非常灵活。用户可以传递任何SELECT语句作为函数参数，只要该SELECT语句返回的结果包含所需数量的属性且包含正确的属性名。
- 多数pgRouting实现的算法不需要路网几何信息。
- 大多数pgRouting函数不返回几何信息，而只返回包含节点id或路径id的有序列表。

### 2.2 Shortest Path A*算法举例

与Shortest Path Dijkstra算法类似，只是SQL需要用到每条线段的起点和终点的坐标，其他参数和pgr_dijkstra都一样。

```sql
ALTER TABLE nyc_roads ADD COLUMN x1 double precision;
ALTER TABLE nyc_roads ADD COLUMN y1 double precision;
ALTER TABLE nyc_roads ADD COLUMN x2 double precision;
ALTER TABLE nyc_roads ADD COLUMN y2 double precision;
 
UPDATE nyc_roads SET x1 = ST_x(ST_PointN(geom, 1));  -- 线段起点坐标x
UPDATE nyc_roads SET y1 = ST_y(ST_PointN(geom, 1));  -- 线段起点坐标y
 
UPDATE nyc_roads SET x2 = ST_x(ST_PointN(geom, ST_NumPoints(geom)));  -- 线段终点坐标x
UPDATE nyc_roads SET y2 = ST_y(ST_PointN(geom, ST_NumPoints(geom)));  -- 线段终点坐标y
```

可能出现x1,y1,x2,y2没赋值成功，通过函数进行逐个分析，有可能存入的数据格式不对，通过PostGIS函数进行转换下就好：

```sql
UPDATE "Road"  SET x1 = ST_x(st_pointn(st_geometryn(transform_geom, 1), 1));  -- 线段起点坐标x
UPDATE "Road"  SET y1 = ST_y(st_pointn(st_geometryn(transform_geom, 1), 1));  -- 线段起点坐标y

UPDATE "Road"  SET x2 = ST_x(ST_PointN(st_geometryn(transform_geom, 1), ST_NumPoints(st_geometryn(transform_geom, 1))));  -- 线段终点坐标x
UPDATE "Road"  SET y2 = ST_y(ST_PointN(st_geometryn(transform_geom, 1), ST_NumPoints(st_geometryn(transform_geom, 1))));  -- 线段终点坐标y
```

函数说明：

```sql
pgr_astar(
sql text,     -- SELECT id, source, target, cost, x1, y1, x2, y2 [,reverse_cost] FROM edge_table ，包含了起点和重点坐标，计算速度比Shortest Path Dijkstra算法快一点
source integer,   
target integer, 
directed boolean, 
has_rcost boolean  
);
```

官方说明：https://docs.pgrouting.org/3.1/en/pgr_aStar.html

返回结果与pgr_dijkstra一样

例子：

```sql
SELECT * FROM pgr_astar('
	SELECT gid AS id,
			 source::integer,
			 target::integer,
			 length::double precision AS cost,
			 x1, y1, x2, y2
			FROM nyc_roads',
	1, 9, false);
```

结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200915215844964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)

以上的node列表示会经过的顶点，edge列表示会经过的路径，cost表示对应单条路径的成本，agg_cost表示路径的总成本。

**注意：**

- 返回的cost属性表示edges_sql参数中指定的cost属性。在这个例子中，cost是路径的长度。cost可以是时间、距离或任何其他属性以及自定义公式的组合。
- 结果中的node属性和edge属性取决于使用pgr_createTopology函数时对顶点标识符的分配。

## 3 查询的标识符

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

## 4 效果呈现

### 4.1 pgr_dijkstra

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

###  4.2 pgr_astar

```sql
select * from (
SELECT * FROM pgr_astar('
	SELECT id,
			 source::integer,
			 target::integer,
			 length::double precision AS cost,
			 x1, y1, x2, y2
			 FROM "Road"',
	2, 9, false))t1 left join "Road_vertices_pgr" t2 on t1.node = t2.id
```

## 5 参考链接

- [POSTGIS路径规划的简单配置(代码编写)_简单GIS的博客-CSDN博客_gis路径规划](https://blog.csdn.net/weixin_39683229/article/details/121771680)
- [PgRouting求解大数据量最短路径_言成言成啊的博客-CSDN博客_pgrouting](https://blog.csdn.net/qq_30460361/article/details/124463626?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-3-124463626-blog-72866436.pc_relevant_aa_2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-3-124463626-blog-72866436.pc_relevant_aa_2&utm_relevant_index=6)
- [（四）pgRouting最短路径查询_会飞的猪biubiu的博客-CSDN博客_pgr_dijkstra](https://blog.csdn.net/qq_29384639/article/details/122035726?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8-122035726-blog-83351481.pc_relevant_multi_platform_whitelistv2eslanding&spm=1001.2101.3001.4242.5&utm_relevant_index=11)
