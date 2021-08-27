  到目前为止，我们只使用了**测量**（ST_Area、ST_Length）、**序列化**（ST_GeomFromText）或者**反序列化**（ST_AsGML）几何图形（geometry）的空间函数。这些函数的共同之处在于它们一次只能处理一个几何图形。

  空间数据库之所以强大，是因为它们不仅能存储几何图形，而且还能够比较几何图形之间的关系。

  诸如"**哪一个是离公园最近的自行车位？**"或者"**地铁线路和街道的交叉路口在哪里?**"的问题，只能通过比较表示自行车位、街道和地铁线路的几何图形来回答。

  **OGC标准**定义了以下一组用于比较几何图形的方法。

# 一、ST_Equals

  **ST_Equals(geometry A, geometry B)**用于测试两个图形的空间相等性。

![_images/st_equals.png](https://postgis.net/workshops/postgis-intro/_images/st_equals.png)

  如果两个相同类型的几何图形具有相同的x、y坐标值，即如果第二个图形与第一个图形的坐标信息相等（相同），则ST_Equals()返回TRUE。

  首先，让我们从nyc_subway_stations表中检索点数据，我们只选"Broad St"的条目。

```sql
SELECT name, geom, ST_AsText(geom)
FROM nyc_subway_stations
WHERE name = 'Broad St';
```

![img](https://img-blog.csdnimg.cn/20190102144536308.png)

![img](https://img-blog.csdnimg.cn/20190102144800134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  然后，将几何图形表示数据插入ST_Equals()进行测试：

```sql
SELECT name
FROM nyc_subway_stations
WHERE ST_Equals(geom, '0101000020266900000EEBD4CF27CF2141BC17D69516315141');
```

![img](https://img-blog.csdnimg.cn/20190102144942970.png)

![img](https://img-blog.csdnimg.cn/20190102145051787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：点在空间数据表中的表示不是很容易理解（0101000020266900000EEBD4CF27CF2141BC17D69516315141），但它是坐标值的精确表示。对于像相等这样的测试，使用精确的坐标信息进行比较是必要的。

# 二、ST_Intersects、ST_Disjoint、ST_Crosses和ST_Overlaps

  **ST_Intersects**、**ST_Crosses**和**ST_Overlaps**测试几何图形是否相交。

![_images/st_intersects.png](https://postgis.net/workshops/postgis-intro/_images/st_intersects.png)

  如果两个图形有相同的部分，即如果它们的边界或内部相交，则ST_Intersects(geometry A, geometry B)返回TRUE。

![_images/st_disjoint.png](https://postgis.net/workshops/postgis-intro/_images/st_disjoint.png)

  ST_Intersects()方法的对立方法是**ST_Disjoint(geometry A, geometry B)**。

  如果两个几何图形没有重合的部分，则它们不相交，反之亦然。

  事实上测试"not intersect"通常比测试"disjoint"更有效，因为intersect测试可以使用**空间索引**。

![_images/st_crosses.png](https://postgis.net/workshops/postgis-intro/_images/st_crosses.png)

  对于multipoint/polygon、multipoint/linestring、linestring/linestring、linestring/polygon和linestring/multipolygon的比较，如果相交生成的几何图形的维度小于两个源几何图形的最大维度，且相交集位于两个源几何图形的内部，则**ST_Crosses(geometry A, geometry B)**将返回TRUE。

![_images/st_overlaps.png](https://postgis.net/workshops/postgis-intro/_images/st_overlaps.png)

  **ST_Overlaps(geometry A, geometry B)**比较两个相同维度的几何图形，如果它们的结果集与两个源几何图形都不同但具有相同维度，则返回TRUE。

  让我们以宽街地铁站（Broad Street subway station）为例，使用ST_Intersects()函数确定其所在社区：

```sql
SELECT name, ST_AsText(geom)
FROM nyc_subway_stations
WHERE name = 'Broad St';
```

![img](https://img-blog.csdnimg.cn/20190102152040983.png)

![img](https://img-blog.csdnimg.cn/20190102152201317.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

```sql
SELECT name, boroname
FROM nyc_neighborhoods
WHERE ST_Intersects(geom, ST_GeomFromText('POINT(583571 4506714)',26918));
```

![img](https://img-blog.csdnimg.cn/20190102152221327.png)

![img](https://img-blog.csdnimg.cn/20190102152436860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 三、ST_Touches

  **ST_Touches()**测试两个几何图形是否在它们的边界上接触，但在它们的内部不相交。

![_images/st_touches.png](https://postgis.net/workshops/postgis-intro/_images/st_touches.png)

  如果两个几何图形的**边界相交**，或者只有一个几何图形的内部与另一个几何图形的边界相交，则**ST_Touches(geometry A, geometry B)**将返回TRUE。

# 四、ST_Within和ST_Contains

  ST_Within()和ST_Contains()测试一个几何图形是否完全位于另一个几何图形内。

![_images/st_within.png](https://postgis.net/workshops/postgis-intro/_images/st_within.png)

  如果第一个几何图形完全位于第二个几何图形内，则ST_Within(geometry A, geometry B)返回TRUE，ST_Within()测试的结果与ST_Contains()完全相反。

  如果第二个几何图形完全包含在第一个几何图形内，则ST_Contains(geometry A, geometry B)返回TRUE。

# 五、ST_Distance和ST_DWithin

  一个常见的GIS问题是"**找到这个物体周围距离X的所有其他物体**"。

  **ST_Distance(geometry A, geometry B)**计算两个几何图形之间的最短距离，并将其作为浮点数返回。这对于实际报告几何图形之间的距离非常有用。

```sql
SELECT ST_Distance(
  ST_GeometryFromText('POINT(0 5)'),
  ST_GeometryFromText('LINESTRING(-2 2, 2 2)'));
```

![img](https://img-blog.csdnimg.cn/20190102155738892.png)

![img](https://img-blog.csdnimg.cn/20190102155906987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  为了测试两个几何图形之间的距离是否在某个范围之内，ST_DWithin()函数提供了一个这样的的功能。

  这对于"**在距离道路500米的缓冲区内有多少棵树?**"这样的问题很有用，你不必计算实际的缓冲区，只需测试距离关系即可。

![_images/st_dwithin.png](https://postgis.net/workshops/postgis-intro/_images/st_dwithin.png)

  再次使用我们的**宽街地铁站**（Broad Street subway station），我们可以找到地铁站附近（10米内）的街道：

```sql
SELECT name
FROM nyc_streets
WHERE ST_DWithin(
        geom,
        ST_GeomFromText('POINT(583571 4506714)',26918),
        10
      );
```

![img](https://img-blog.csdnimg.cn/20190102160524293.png)

![img](https://img-blog.csdnimg.cn/20190102160805539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  我们可以在地图上验证答案，Broad St站实际上是在Wall、Broad和Nassau街道的十字路口。

![_images/broad_st.jpg](https://postgis.net/workshops/postgis-intro/_images/broad_st.jpg)

# 六、空间关系练习

  下面是我们在文章上面部分涉及到的一些函数，它们应该对练习有用！

![img](https://img-blog.csdnimg.cn/20190103084459536.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  还请记住我们现在数据库中已经具有的表：

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

##  6.1  练习

###   ①名为"Atlantic Commonts"的街道的geometry值是什么？

```sql
SELECT ST_AsText(geom)
  FROM nyc_streets
  WHERE name = 'Atlantic Commons';
```

![img](https://img-blog.csdnimg.cn/20190103084742819.png)

![img](https://img-blog.csdnimg.cn/20190103084902182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

###   ②Atlantic Commons（大西洋公地）位于哪个社区（neighborhood）和行政区（borough）？

```sql
SELECT name, boroname
FROM nyc_neighborhoods
WHERE ST_Intersects(
  geom,
  ST_GeomFromText('LINESTRING(586782 4504202,586864 4504216)', 26918)
);
```

![img](https://img-blog.csdnimg.cn/2019010308525196.png)

![img](https://img-blog.csdnimg.cn/20190103090317203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：嘿，为什么要将"MULTILINESTRING"变成"LINESTRING"呢？因为在空间上，它们描述的是相同的形状。

  更重要的是，我们还对坐标进行了四舍五入，以使它们更易于阅读，这实际上改变了结果：我们现在不能使用ST_Touches()方法来找出哪些道路连接Atlantic Commons，因为坐标不再与原来的坐标完全相同。

###   ③Atlantic Commons与哪些街道相连？

```sql
SELECT name
FROM nyc_streets
WHERE ST_DWithin(
  geom,
  ST_GeomFromText('LINESTRING(586782 4504202,586864 4504216)', 26918),
  0.1
);
```

![img](https://img-blog.csdnimg.cn/20190103090140126.png)

![img](https://img-blog.csdnimg.cn/20190103090554135.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/atlantic_commons.jpg](https://postgis.net/workshops/postgis-intro/_images/atlantic_commons.jpg)

###   ④大约有多少人住在Atlantic Commons上（距离Atlantic Commons50米以内）？

```sql
SELECT Sum(popn_total)
  FROM nyc_census_blocks
  WHERE ST_DWithin(
   geom,
   ST_GeomFromText('LINESTRING(586782 4504202,586864 4504216)', 26918),
   50
  );
```

![img](https://img-blog.csdnimg.cn/20190103090937852.png)

![img](https://img-blog.csdnimg.cn/201901030912079.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)