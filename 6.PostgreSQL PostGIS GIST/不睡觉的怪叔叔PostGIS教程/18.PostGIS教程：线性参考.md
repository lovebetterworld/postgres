# PostGIS教程十八：线性参考

附：关于线性参考的概念，可以参考这篇文章：[ArcGIS中的线性参考/动态分段技术（一）：Linear Referencing背景](https://blog.csdn.net/qq_35732147/article/details/87458168)

  **线性参考**是一种表示要素的方法，这些要素可以通过引用一个基本的线性要素来描述。使用线性参照建模的常见示例包括：

- 高速公路资源，使用沿着公路网中的英里作为参照。
- 公路维护作业，参照的是沿着公路网的一对英里测量。
- 水产库存，其中鱼的存在位置被记录为距离上游的一段位置之间。
- 河流的水文特征，以河流的某一个点到另一个点作为参考。

  线性参考模型的优点是，从属空间观测信息不需要与基准空间观测信息分开记录，并且对基础观测信息进行更新时，从属观测信息将自动更新从而追踪新几何图形。

  注意：ESRI的线性参照约定是有一个线性空间要素的**基表**和一个非空间的**事件表**，其中非空间的**事件表**包括对空间要素的外键引用和沿参照要素的测量值。我们将使用术语"**事件表**(event table)"来表示我们构建的非空间表。

 

# 一、创建线性参考

  如果要参照线性网络的现有点表，请使用**ST_LineLocatePoint**函数，该函数接受线串和点，并返回该点沿线串的线性参考比例。

```sql
-- Simple example of locating a point half-way along a line
SELECT ST_LineLocatePoint('LINESTRING(0 0, 2 2)', 'POINT(1 1)');
-- Answer 0.5
-- What if the point is not on the line? It projects to closest point
-- 即做(0, 2)点到线串(0 0, 2 2)的垂线，使用对应的垂足点来求比例
SELECT ST_LineLocatePoint('LINESTRING(0 0, 2 2)', 'POINT(0 2)');
-- Answer 0.5
```

  我们可以使用ST_LineLocatePoint函数根据nyc_subway_stations创建相对于街道的"**事件表**"。

```sql
-- 下面所有的SQL都是用来创建新的事件表的
CREATE TABLE nyc_subway_station_events AS
-- 我们首先需要找到一组可能最接近的候选者
-- streets, 按id和distance排列...
WITH ordered_nearest AS (
SELECT
  ST_GeometryN(streets.geom,1) AS streets_geom,
  streets.gid AS streets_gid,
  subways.geom AS subways_geom,
  subways.gid AS subways_gid,
  ST_Distance(streets.geom, subways.geom) AS distance
FROM nyc_streets streets
  JOIN nyc_subway_stations subways
  ON ST_DWithin(streets.geom, subways.geom, 200)
ORDER BY subways_gid, distance ASC
)
-- 我们使用'distinct on'PostgreSQL功能为每各唯一的街道gid获取第一条街道（最近的街道）。
-- 然后，我们可以将这条街道信息置入ST_LinLocatePoint函数，使其沿着它的候选地铁站来计算
SELECT
  DISTINCT ON (subways_gid)
  subways_gid,
  streets_gid,
  ST_LineLocatePoint(streets_geom, subways_geom) AS measure,
  distance
FROM ordered_nearest;
-- 主码对于可视化软件很有用
ALTER TABLE nyc_subway_station_events ADD PRIMARY KEY (subways_gid);
```

![img](https://img-blog.csdnimg.cn/2019021615581741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  一旦我们有了一个**事件表**，将其转换回一个空间视图是很有趣的，这样我们就可以将**事件**相对于派生出它们的原始点进行可视化。

  要从线性参考比例值得到对应点，我们使用**ST_LineInterpolatePoint**函数，下面是关于我们前面的简单例子的逆过程：

```sql
-- Simple example of locating a point half-way along a line
SELECT ST_AsText(ST_LineInterpolatePoint('LINESTRING(0 0, 2 2)', 0.5));
-- Answer POINT(1 1)
```

![img](https://img-blog.csdnimg.cn/2019021616312432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  而且，我们可以将nyc_subway_station_events表连接回nyc_streets表，并使用measure属性生成**空间事件点**（这个示例中是地铁站点），而无需引用原始nyc_subway_stations表。

```sql
-- New view that turns events back into spatial objects
CREATE OR REPLACE VIEW nyc_subway_stations_lrs AS
SELECT
  events.subways_gid,
  ST_LineInterpolatePoint(ST_GeometryN(streets.geom, 1), events.measure)AS geom,
  events.streets_gid
FROM nyc_subway_station_events events
JOIN nyc_streets streets
ON (streets.gid = events.streets_gid);
```

![img](https://img-blog.csdnimg.cn/20190216164815575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  查看街道上的**原始地铁站点**（红星）和**事件点**（蓝色圆圈），你可以看到**事件**是如何被直接**捕捉**到最近的街道线的（**事件点**全部位于街道线上）。

![_images/lrs1.jpg](https://postgis.net/workshops/postgis-intro/_images/lrs1.jpg)

  **注意**：**线性参考函数**的一个令人惊讶的用法与线性参考模型无关。如上所示，可以使用这些函数将点**捕捉**到线性要素（即可以使用**线性参考**来实现**捕捉**功能）。对于像GPS轨迹或其他预期参考线性网络的输入这样的用例，**捕捉**是一个方便的功能。

# 二、函数列表

![img](https://img-blog.csdnimg.cn/20190218095643868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)