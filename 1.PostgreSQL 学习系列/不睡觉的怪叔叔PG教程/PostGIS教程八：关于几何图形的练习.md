# PostGIS教程八：关于几何图形的练习

# 一、函数列表

  以下是我们迄今为止看到的所有函数的提示，它们应该对练习有用！

- **sum(expression)** aggregate to return a sum for a set of records
- **count(expression)** aggregate to return the size of a set of records
- **ST_GeometryType(geometry)** returns the type of the geometry
- **ST_NDims(geometry)** returns the number of dimensions of the geometry
- **ST_SRID(geometry)** returns the spatial reference identifier number of the geometry
- **ST_X(point)** returns the X ordinate
- **ST_Y(point)** returns the Y ordinate
- **ST_Length(linestring)** returns the length of the linestring
- **ST_StartPoint(geometry)** returns the first coordinate as a point
- **ST_EndPoint(geometry)** returns the last coordinate as a point
- **ST_NPoints(geometry)** returns the number of coordinates in the linestring
- **ST_Area(geometry)** returns the area of the polygons
- **ST_NRings(geometry)** returns the number of rings (usually 1, more if there are holes)
- **ST_ExteriorRing(polygon)** returns the outer ring as a linestring
- **ST_InteriorRingN(polygon, integer)** returns a specified interior ring as a linestring
- **ST_Perimeter(geometry)** returns the length of all the rings
- **ST_NumGeometries(multi/geomcollection)** returns the number of parts in the collection
- **ST_GeometryN(geometry, integer)** returns the specified part of the collection
- **ST_GeomFromText(text)** returns `geometry`
- **ST_AsText(geometry)** returns WKT `text`
- **ST_AsEWKT(geometry)** returns EWKT `text`
- **ST_GeomFromWKB(bytea)** returns `geometry`
- **ST_AsBinary(geometry)** returns WKB `bytea`
- **ST_AsEWKB(geometry)** returns EWKB `bytea`
- **ST_GeomFromGML(text)** returns `geometry`
- **ST_AsGML(geometry)** returns GML `text`
- **ST_GeomFromKML(text)** returns `geometry`
- **ST_AsKML(geometry)** returns KML `text`
- **ST_AsGeoJSON(geometry)** returns JSON `text`
- **ST_AsSVG(geometry)** returns SVG `text`

  还有请记住我们现在数据库中已经有的表：

- ```
  nyc_census_blocks
  ```

  - blkid, popn_total, boroname, geom

- ```
  nyc_streets
  ```

  - name, type, geom

- ```
  nyc_subway_stations
  ```

  - name, geom

- ```
  nyc_neighborhoods
  ```

  - name, boroname, geom

# 二、练习

##   ①'West Village'社区（neighborhood）的面积是多少？

```sql
SELECT ST_Area(geom)
  FROM nyc_neighborhoods
  WHERE name = 'West Village';
```

![img](https://img-blog.csdnimg.cn/20181229150057281.png)

![img](https://img-blog.csdnimg.cn/20181229150216327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**:面积以**平方米**为单位。要得到一个以**公顷**为单位的面积，需要再对其除以10000；要得到一个以**英亩**为单位的面积，需要对其除以4047。

##   ②曼哈顿（Manhattan）行政区的面积是多少英亩？

（提示：nyc_census_blocks和nyc_neighborhoods中都有boroname - rorough name - 行政区名）

```sql
SELECT Sum(ST_Area(geom)) / 4047
  FROM nyc_neighborhoods
  WHERE boroname = 'Manhattan';
```

![img](https://img-blog.csdnimg.cn/20181229150923450.png)

![img](https://img-blog.csdnimg.cn/20181229151010920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  或者：

```sql
SELECT Sum(ST_Area(geom)) / 4047
  FROM nyc_census_blocks
  WHERE boroname = 'Manhattan';
```

![img](https://img-blog.csdnimg.cn/2018122915104284.png)

![img](https://img-blog.csdnimg.cn/20181229151133712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

##   ③纽约市（New York City）有多少个人口普查块（census blocks）多边形里有孔洞？

```sql
SELECT Count(*)
  FROM nyc_census_blocks
  WHERE ST_NumInteriorRings(ST_GeometryN(geom,1)) > 0;
```

  注意：ST_NRings()函数可能让人感觉可以胜任，但是它也计算**多-多边形**的外环和内环。为了运行ST_NumInteriorRings()，我们需要将MultiPolygon**几何图形**转换为简单的**多边形**，因此，我们使用ST_GeometryN()从每个集合中提取第一个**多边形**。

![img](https://img-blog.csdnimg.cn/20181229152639455.png)

![img](https://img-blog.csdnimg.cn/20181229152854486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

##   ④纽约市（New York）的街道总长度（公里）是多少？

**（提示：空间数据的测量单位是**米**，每**公里**有1000米）

```sql
SELECT Sum(ST_Length(geom)) / 1000
FROM nyc_streets;
```

![img](https://img-blog.csdnimg.cn/20181229153140395.png)

![img](https://img-blog.csdnimg.cn/20181229153246872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

##   ⑤'Columbus Cir'（哥伦布圆环——纽约曼哈顿区的一个地标）有多长？

```sql
SELECT ST_Length(geom)
FROM nyc_streets
WHERE name = 'Columbus Cir';
```

![img](https://img-blog.csdnimg.cn/20181229153514230.png)

![img](https://img-blog.csdnimg.cn/20181229153617498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

##   ⑥West Village社区边界的JSON表示是怎样的？

```sql
SELECT ST_AsGeoJSON(geom)
FROM nyc_neighborhoods
WHERE name = 'West Village';
```

![img](https://img-blog.csdnimg.cn/20181229154043505.png)

![img](https://img-blog.csdnimg.cn/20181229154228187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  返回的JSON里的几何类型是"MultiPolygon（**多多边形**）"，有趣！

##   ⑦West Village社区多多边形（MultiPolygon）中有多少个多边形？

```sql
SELECT ST_NumGeometries(geom)
FROM nyc_neighborhoods
WHERE name = 'West Village';
```

![img](https://img-blog.csdnimg.cn/20181229154540477.png)

  **注意**：在**空间表**中找到**单元素多多边形**并不少见。使用**多多边形**允许只有一种几何图形类型的表同时存储**单（single-）几何图形**和**多（multi-）几何图形**，而不必使用GeometryCollection类型。

![img](https://img-blog.csdnimg.cn/20181229155209996.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

##   ⑧按类型（type）列出纽约市街道长度是多少？

```sql
SELECT type, Sum(ST_Length(geom)) AS length
FROM nyc_streets
GROUP BY type
ORDER BY length DESC;
```

![img](https://img-blog.csdnimg.cn/2018122915541148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：**ORDER BY length DESC**子句按长度以降序形式排序。