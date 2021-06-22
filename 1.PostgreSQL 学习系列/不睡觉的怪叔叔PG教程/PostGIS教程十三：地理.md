# 地理

坐标为"地理（geographics）"或者说是"纬度（latitude）/经度（longitude）"的数据非常常见。

  与Mercator（墨卡托）、UTM（通用横轴墨卡托）、Stateplane中的坐标不同，**地理坐标**不是笛卡尔平面坐标（Cartesian coordinates）。**地理坐标**并不表示平面上与原点的线性距离，相反，这些球坐标描述了地球上的角坐标。在球坐标中，点由该点与参考子午线（经度）的旋转角度和该点与赤道的角度（纬度）指定。

  ![_images/cartesian_spherical.jpg](https://postgis.net/workshops/postgis-intro/_images/cartesian_spherical.jpg)

  你可以将**地理坐标**看作近似的笛卡尔平面坐标，并继续进行空间计算，然而，关于距离、长度和面积的测量将会是毫无意义的。由于球坐标测量角度距离，因此单位以"**度**"表示。此外，索引和真/假测试（如**相交**和**包含**）可能会变得非常错误。越与极点或国际日期线接近的区域，点与点之间的距离变得更大。

  例如，下面是**洛杉矶**（Los Angeles）和**巴黎**（Paris）的坐标：

- Los Angeles: `POINT(-118.4079 33.9434)`
- Paris: `POINT(2.3490 48.8533)`

  使用标准的PostGIS笛卡尔平面坐标系空间函数ST_Distance(geometry, geometry)计算**洛杉矶**和**巴黎**之间的距离。请注意，SRID 4326声明了地理空间参考系统。

```sql
SELECT ST_Distance(
  ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
  ST_GeometryFromText('POINT(2.5559 49.0083)', 4326)     -- Paris (CDG)
);
```

![img](https://img-blog.csdnimg.cn/20190115115414899.png)

![img](https://img-blog.csdnimg.cn/20190115115652645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  啊哈！121！但那是什么意思？

  空间参考4326的单位是**度**，所以我们的答案是121度。但是，这表示什么呢？

  在地球球体上，1度对应的地球实际距离的大小是变化的。当远离赤道时，它会变得更小，当越接近两极时，地球上的经线相互之间越来越接近。因此，121度的距离并不意味着什么，这是一个没有意义的数字。

  为了计算出真实的距离，我们不能把**地理坐标**近似的看成笛卡尔平面坐标，而应该把它们看成是球坐标。我们必须把点之间的距离作为球面上的真实路径来测量——大圆的一部分。

  从1.5版开始，PostGIS通过地理（geography）数据类型提供此功能。

  注意：不同的空间数据库有不同的"处理地理"的方法：

- 当SRID是地理坐标系统时，Oracle试图通过透明地进行地理计算来掩盖差异
- SQL Server使用两种空间类型，一种是针对笛卡尔平面坐标数据的"STGeometry"，另一种是针对地理坐标系统数据的"StGeography"
- Informix Spatial是Infomix的纯笛卡尔扩展，而Informix Geodetic是纯地理扩展
- 与SQL Server类似，PostGIS使用两种数据类型："geometry"和"geography"

  关于上面的测量应该使用geography而不是geometry类型，让我们再次尝试测量**洛杉矶**和**巴黎**之间的距离，我们将使用ST_GeographyFromText(text)函数，而不是ST_GeometryFromText(text)。

```sql
SELECT ST_Distance(
  ST_GeographyFromText('POINT(-118.4079 33.9434)'), -- Los Angeles (LAX)
  ST_GeographyFromText('POINT(2.5559 49.0083)')     -- Paris (CDG)
  );
```

![img](https://img-blog.csdnimg.cn/20190115140009852.png)

![img](https://img-blog.csdnimg.cn/20190115140124433.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  得到一个大数字！所有地理计算的返回值都以**米**为单位，所以我们的答案是9124km。

  早期版本的PostGIS支持使用ST_Distance_Spheroid(point, point, measurement)函数对球体进行非常基本的计算。然而，ST_Distance_Spheroid功能是有限的，该函数仅适用于点，不支持跨极点或国际日期变更线的要素的索引。

  当提出这样一个问题时，支持非点的几何图形的需求变得非常明显："**从洛杉矶到巴黎的航班路线距离冰岛有多远?**"

![_images/lax_cdg.jpg](https://postgis.net/workshops/postgis-intro/_images/lax_cdg.jpg)

  在**笛卡尔平面坐标系统**上使用**地理坐标**（紫色线）产生了一个非常错误的答案！使用**大圆路线**（红线）则能得出正确的答案。如果我们将LAX-CDG航班路线转换成一条线串，并利用geography计算其到冰岛某个点的距离，我们可以得到正确的答案（以**米**为单位）。

```sql
SELECT ST_Distance(
  ST_GeographyFromText('LINESTRING(-118.4079 33.9434, 2.5559 49.0083)'), -- LAX-CDG
  ST_GeographyFromText('POINT(-22.6056 63.9850)')                        -- Iceland (KEF)
);
```

![img](https://img-blog.csdnimg.cn/20190115141844791.png)

![img](https://img-blog.csdnimg.cn/20190115141949676.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  因此，在LAX-CDG航班路线距离冰岛的距离（从冰岛的国际机场测量）是一个相对较小的502km这样的一个长度距离。

  笛卡尔平面坐标系统处理地理坐标的方法是完全分解跨越国际日期变更线的要素。从洛杉矶到东京的最短**大圆路线**穿越太平洋，而最短的**笛卡尔平面路线**则穿越大西洋和印度洋。

![_images/lax_nrt.png](https://postgis.net/workshops/postgis-intro/_images/lax_nrt.png)

```sql
SELECT ST_Distance(
  ST_GeometryFromText('Point(-118.4079 33.9434)'),  -- LAX
  ST_GeometryFromText('Point(139.733 35.567)'))     -- NRT (Tokyo/Narita)
    AS geometry_distance,
ST_Distance(
  ST_GeographyFromText('Point(-118.4079 33.9434)'), -- LAX
  ST_GeographyFromText('Point(139.733 35.567)'))    -- NRT (Tokyo/Narita)
    AS geography_distance;
```

![img](https://img-blog.csdnimg.cn/20190115143100345.png)

![img](https://img-blog.csdnimg.cn/20190115144723910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

 

# 一、使用Geography

  为了将geometry数据加载到geography表中，首先需要将geometry投影到EPSG:4326（经度-longitude/纬度-latitude），然后再将其转换为geography。**ST_Transform(geometry, srid)**函数能将坐标转换为地理坐标，**Geography(geometry)**函数能将geometry转换为geography。

![img](https://img-blog.csdnimg.cn/20190421094136548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

```sql
CREATE TABLE nyc_subway_stations_geog AS
SELECT
  Geography(ST_Transform(geom,4326)) AS geog,
  name,
  routes
FROM nyc_subway_stations;
```

![img](https://img-blog.csdnimg.cn/2019011609252365.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  在geography表上构建**空间索**引与在geometry表上构建**空间****索引**完全相同：

```sql
CREATE INDEX nyc_subway_stations_geog_gix
ON nyc_subway_stations_geog USING GIST (geog);
```

![img](https://img-blog.csdnimg.cn/20190116092833694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  不同之处在于：geography空间索引将正确地处理覆盖极点或国际日期变更线的要素的查询，而geometry空间索引则不会。

  对于geography类型，只有相关的少量空间函数：

- **ST_AsText(geography)** returns `text`
- **ST_GeographyFromText(text)** returns `geography`
- **ST_AsBinary(geography)** returns `bytea`
- **ST_GeogFromWKB(bytea)** returns `geography`
- **ST_AsSVG(geography)** returns `text`
- **ST_AsGML(geography)** returns `text`
- **ST_AsKML(geography)** returns `text`
- **ST_AsGeoJson(geography)** returns `text`
- **ST_Distance(geography, geography)** returns `double`
- **ST_DWithin(geography, geography, float8)** returns `boolean`
- **ST_Area(geography)** returns `double`
- **ST_Length(geography)** returns `double`
- **ST_Covers(geography, geography)** returns `boolean`
- **ST_CoveredBy(geography, geography)** returns `boolean`
- **ST_Intersects(geography, geography)** returns `boolean`
- **ST_Buffer(geography, float8)** returns `geography` [[1\]](https://postgis.net/workshops/postgis-intro/geography.html#casting-note)
- **ST_Intersection(geography, geography)** returns `geography` [[1\]](https://postgis.net/workshops/postgis-intro/geography.html#casting-note)

  

# 二、创建一个Geography表

  用于创建含有geography列的新表的SQL与用于创建geography表的SQL非常相似。但是，geography包含在表创建时直接指定表类型的功能。例如：

```sql
CREATE TABLE airports (
  code VARCHAR(3),
  geog GEOGRAPHY(Point)
);
INSERT INTO airports VALUES ('LAX', 'POINT(-118.4079 33.9434)');
INSERT INTO airports VALUES ('CDG', 'POINT(2.5559 49.0083)');
INSERT INTO airports VALUES ('KEF', 'POINT(-22.6056 63.9850)');
```

  在表定义中，GEOGRAPHY(Point)将airport数据类型指定为点。新的geography字段不会在geometry_columns视图中注册，相反，它们是在名为geography_columns视图中注册的。

```sql
SELECT * FROM geography_columns;
```

![img](https://img-blog.csdnimg.cn/2019011609561253.png)

![img](https://img-blog.csdnimg.cn/20190116095707404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  

# 三、转换为Geometry

  虽然geography类型的空间函数已经可以处理许多问题，但有时你可能需要访问仅由geometry类型支持的其他空间函数。幸运的是，你可以将对象从geography转换为geometry。

  PostgreSQL的类型转换语法是将::typename附加到希望转换的值的末尾。因此，2::text将数字2转换为文本字符串"2"；'POINT(0 0)' :: geometry将点的文本表示形式转换为geometry点。

  ST_X(point)函数仅支持geometry类型，那我们怎样才能从我们的geograph类型数据中读取X坐标呢？

```sql
SELECT code, ST_X(geog::geometry) AS longitude FROM airports;
```

![img](https://img-blog.csdnimg.cn/20190116100716814.png)

![img](https://img-blog.csdnimg.cn/20190116100655580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  通过将::geometry附加到geography值后面，可以将对象转换为SRID为4326的geometry。现在，我们就可以使用任何的geometry函数了。但是，请记住-现在我们的对象是geometry，坐标将被解释为**笛卡尔平面坐标**，而不是**球体坐标**。

 

# 四、为什么使用Geography

  地理坐标是大众普遍接受的坐标——每个人都知道经度/纬度的含义，但很少有人理解UTM坐标的含义。为什么不一直用geography呢？

- 首先，如前所述，可直接支持geography类型的函数要少得多。
- 其次，球体上的计算要比笛卡尔计算计算量大得多。例如，计算距离的笛卡尔坐标系的公式（Pythagoras)涉及一次对sqrt()的调用，计算距离的球体坐标的公式包含两次sqrt()调用、一次arctan()调用、四次sin()调用和两次cos()调用，三角函数的计算是非常耗费资源的。

  那么，结论是什么呢？

  **如果你的数据在地理范围上是紧凑的（包含在州、县或市内），请使用基于笛卡尔坐标的geometry类型**，这样使你的数据有意义。有关可能的参考系统的选择，请参见[http://spatialreference.org](http://spatialreference.org/)站点并输入您所在区域的名称。

  **如果你需要测量在地理范围上是分散的数据集（覆盖世界大部分地区）的距离，请使用geography类型。**通过基于geography类型运行而节省的应用程序复杂性将抵消任何性能问题。同时转换为geometry可以抵消大多数功能限制。