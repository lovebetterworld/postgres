# PostGIS教程十四：几何图形创建函数

目前我们看到的所有函数都可以处理已有的几何图形并返回结果：

- 分析几何图形（ST_Length(geometry), ST_Area(geometry))
- 几何图形的序列化（ST_AsText(geometry), ST_AsGML(geometry))
- 选取几何图形的某个部分（ST_RingN(geometry, n))
- true/false测试（ST_Contains(geometry, geometry), ST_Intersects(geometry, geometry))

  "**几何图形创建函数**"以几何图形作为输入并输出新的图形。

 

# 一、ST_Centroid / ST_PointOnSurface

  组成空间查询时的一个常见需求是将多边形要素替换为要素的点表示。这对于空间连接（spatial join）非常有用，因为在两个多边形图层上使用St_Intersects(geometry, geometry)通常会导致重复计算：位于两个多边形的边界上的多边形将与两侧的多边形都相交，将其替换为点将强制它位于一侧或另一侧，而不是与两侧的多边形都相交。

- ST_Centroid(geometry)  ——  返回大约位于输入参数的质心上的点。这种简单的计算速度非常快，但有时并不可取，因为返回点不一定在要素本身上。如果输入的几何图形具有凸性（假设字母'C'），则返回的质心可能不在图形的内部。
- ST_PointOnSurface(geometry)  ——  返回保证在输入多边形内的点。从计算上讲，它比centroid操作代价要大得多。

![_images/centroid.jpg](https://postgis.net/workshops/postgis-intro/_images/centroid.jpg)

 

# 二、ST_Buffer

   **缓冲区操作**在GIS工作流中很常见，在PostGIS中也可以进行缓冲区操作。**ST_Buffer(geometry, distance)**接受几何图形和缓冲区距离，并输出一个多边形，这个多边形的边界与输入的几何图形之间的距离与输入的缓冲区距离相等。

![_images/st_buffer.png](https://postgis.net/workshops/postgis-intro/_images/st_buffer.png)

  例如，如果**美国公园管理局**（US Park Service）想要在**自由岛**（Liberty Island）周围建立一个海洋交通区，他们可能会在该岛周围建造一个500米的缓冲多边形。**自由岛**是nyc_census_blocks表中的一个单独的人口普查块，因此我们可以轻松地提取和建立对应的缓冲区。

```sql
-- Make a new table with a Liberty Island 500m buffer zone
CREATE TABLE liberty_island_zone AS
SELECT ST_Buffer(geom,500)::geometry(Polygon,26918) AS geom
FROM nyc_census_blocks
WHERE blkid = '360610001001001';
```

![img](https://img-blog.csdnimg.cn/20190122103000750.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/liberty_positive.jpg](https://postgis.net/workshops/postgis-intro/_images/liberty_positive.jpg)

  ST_Buffer函数也接受负的距离值，从而在输入的多边形内构建内接多边形。而对于线串和点，只会返回空值。

 

# 三、ST_Intersection

  另一个经典的GIS操作 - **叠置**（overlay）- 通过计算两个重叠多边形的**交集**来创建新的几何图形。

  **ST_Intersection(geometry A, geometry B)**函数返回两个参数共有的空间区域（或直线，或点）。如果参数不相交，该函数将返回一个空几何图形。

```sql
-- What is the area these two circles have in common?
-- Using ST_Buffer to make the circles!
SELECT ST_AsText(ST_Intersection(
  ST_Buffer('POINT(0 0)', 2),
  ST_Buffer('POINT(3 0)', 2)
));
```

![img](https://img-blog.csdnimg.cn/20190122104529256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/intersection.jpg](https://postgis.net/workshops/postgis-intro/_images/intersection.jpg)

 

# 四、ST_Union

  在前面的示例中，我们将几何图形相交，创建一个新的几何图形，新的几何图形包含来自两个输入图形的线串。

  **ST_Union**执行相反的操作，它接受输入并删除公共线串。

  ST_Union函数有两种形式：

- ST_Union(geometry, geometry)  ——  接受两个几何图形参数并返回合并的并集。例如，将上面示例中的ST_Intersection()函数替换为ST_Union()函数后，结果如下：

```sql
-- What is the total area these two circles cover?
-- Using ST_Buffer to make the circles!
SELECT ST_AsText(ST_Union(
  ST_Buffer('POINT(0 0)', 2),
  ST_Buffer('POINT(3 0)', 2)
));
```

![img](https://img-blog.csdnimg.cn/20190122105619104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/union.jpg](https://postgis.net/workshops/postgis-intro/_images/union.jpg)

- ST_Union([geometry])  ——  接受一组几何图形并返回全部几何图形的并集。ST_Union([geometry])可与GROUP BY语句一起使用，以创建经过细致合并的基本几何图形集。它非常强大。

  我们的nyc_census_blocks就是ST_Union的一个示例，人口普查地理是精心构建的，这样就可以从较小的地理区域建立起较大的地理区域。因为，我们可以通过合并构成每个区域的块来创建人口普查区域地图，或者我们可以通过合并每个县内的块来创建县地图。

  要执行合并，请注意，唯一的键blkid实际上包含了有关较高级别的地理区划的信息。以下是我们之前使用的自由岛的部分键：

```sql
360610001001001 = 36 061 000100 1 001
36     = State of New York
061    = New York County (Manhattan)
000100 = Census Tract
1      = Census Block Group
001    = Census Block
```

  所以我们可以通过分组合并blkid键前5个数字相同的所有几何图形来创建县地图。要有耐心，这个计算代价比较大，可能需要一到两分钟。

```sql
-- Create a nyc_census_counties table by merging census blocks
CREATE TABLE nyc_census_counties AS
SELECT
  ST_Union(geom)::Geometry(MultiPolygon,26918) AS geom,
  SubStr(blkid,1,5) AS countyid
FROM nyc_census_blocks
GROUP BY countyid;
```

![img](https://img-blog.csdnimg.cn/20190122113610361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/union_counties.png](https://postgis.net/workshops/postgis-intro/_images/union_counties.png)

  面积测试可以确认我们的联合操作没有丢失任何几何图形。首先，我们计算每个**人口普查区块**（census block）的面积，并将这些区域按**人口普查县**（census county）id进行分组。

```sql
SELECT SubStr(blkid,1,5) AS countyid, Sum(ST_Area(geom)) AS area
FROM nyc_census_blocks
GROUP BY countyid;
```

![img](https://img-blog.csdnimg.cn/20190122114156254.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190122114210958.png)

  然后我们从nyc_census_counties表中计算出每个新生成的县多边形的面积:

```sql
SELECT countyid, ST_Area(geom) AS area
FROM nyc_census_counties;
```

![img](https://img-blog.csdnimg.cn/20190122115224374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  得到了同样的答案！我们已经成功地根据我们的nyc_census_blocks建立了纽约市县表。

# 五、函数列表

![img](https://img-blog.csdnimg.cn/20190122115452254.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)