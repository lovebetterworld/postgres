- [POSTGIS路径规划的简单配置(代码编写)_简单GIS的博客-CSDN博客_gis路径规划](https://blog.csdn.net/weixin_39683229/article/details/121771680)

![思维导图](https://img-blog.csdnimg.cn/51732260008b4cecb2ff585bb047b6e8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA566A5Y2VR0lT,size_20,color_FFFFFF,t_70,g_se,x_16)

- [PostGis路径分析_qgbihc的博客-CSDN博客_postgis路径规划](https://blog.csdn.net/qgbihc/article/details/108609968?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-2.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-2.pc_relevant_paycolumn_v3&utm_relevant_index=4)

## 1 生成拓扑

要生成最佳路径，首先要生成合法的拓扑。

生成拓扑前，需要添加两个字段，用来存储线段的首尾编号

```sql
-- Add "source" and "target" column
ALTER TABLE nyc_roads ADD COLUMN "source" integer;
ALTER TABLE nyc_roads ADD COLUMN "target" integer;
```

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

## 2 生成最佳路径

pgrouting支持的最佳路径算法很多。

官方说明：[https://docs.pgrouting.org/3.1/en/search.html?q=+shortest+path&check_keywords=yes&area=default](https://docs.pgrouting.org/3.1/en/search.html?q= shortest path&check_keywords=yes&area=default)
这里以Shortest Path A*和Shortest Path Dijkstra（狄克斯特拉）为例，介绍如何生成最佳路径

如果考虑回程成本的话，需要增加回程成本的字段，并设置为公里数。

```sql
ALTER TABLE nyc_roads ADD COLUMN reverse_cost double precision;
UPDATE nyc_roadsSET reverse_cost = length;
```

### 2.1 Shortest Path Dijkstra算法举例

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