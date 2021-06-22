# PostGIS教程十二：投影数据

地球不是平的，也没有简单的方法把它放在一张平面纸地图上（或电脑屏幕上），所以人们想出了各种巧妙的解决方案（**投影**）。

  每种投影方案都有优点和缺点，一些**投影**保留面积特征；一些**投影**保留角度特征，如**墨卡托投影**（Mercator）；一些**投影**试图找到一个很好的中间混合状态，在几个参数上只有很小的失真。所有**投影**的共同之处在于，它们将（地球）转换为**平面笛卡尔坐标系**，选择哪种投影取决于你将如何使用数据（需要哪些数据特征，面积？角度？或者其他）。

  我们在加载纽约数据时"邂逅"了投影。（回想一下令人讨厌的SRID 26918）。但是，有时需要在空间参考系统之间进行变换和重新投影。PostGIS包含更改数据投影（重投影）的功能，即使用ST_Transform(geometry, srid)函数就可以实现重投影。另外，为了查看和设置几何图形的空间参照标识符，PostGIS提供了ST_SRID(geometry）和ST_SetSRID(geometry，SRID）函数。

  我们可以使用ST_SRID(geometry)函数确认数据的SRID：

```sql
SELECT ST_SRID(geom) FROM nyc_streets LIMIT 1;
```

![img](https://img-blog.csdnimg.cn/20190111153100771.png)

![img](https://img-blog.csdnimg.cn/20190111153258564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  “26918"的定义是什么？正如我们在加载纽约数据那一部分中看到的，该定义包含在spatial_ref_sys表中。事实上，有两个定义。"well-known text"（[WKT](https://postgis.net/workshops/postgis-intro/glossary.html#term-wkt)）定义在srtext列中，"proj.4"格式定义在proj4text列。

```sql
SELECT * FROM spatial_ref_sys WHERE srid = 26918;
```

![img](https://img-blog.csdnimg.cn/20190111154015180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  实际上，对于内部PostGIS投影的计算，依据的是proj4text列的内容。以下是26918投影对应的proj4text列的内容：

```sql
SELECT proj4text FROM spatial_ref_sys WHERE srid = 26918;
```

![img](https://img-blog.csdnimg.cn/20190111154344957.png)

![img](https://img-blog.csdnimg.cn/2019011115431721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  实际上，srtext和proj4text列都很重要：srtext列由GeoServer、uDig和FME等外部程序使用；proj4text列由内部程序使用。

# 一、比较数据

  综合起来，坐标和SRID（严谨的说应该是空间参考系统）一起定义了地球上的一个位置。没有SRID，坐标只是一个抽象而没有实际意义的概念。“**笛卡尔**”坐标平面被定义为放置在地球表面的“平面”坐标系。由于PostGIS函数在这样的坐标系统上工作，因此关于两个几何图形的比较的操作都要基于同一SRID。

  如果输入具有不同SRID的几何图形，则会得到错误：

```sql
SELECT ST_Equals(
         ST_GeomFromText('POINT(0 0)', 4326),
         ST_GeomFromText('POINT(0 0)', 26918)
         );
```

 ![img](https://img-blog.csdnimg.cn/20190114092747946.png) 

![img](https://img-blog.csdnimg.cn/20190114092926881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意：****空间索引**是基于存储的几何图形的SRID构建的。如果在不同的SRID中进行比较，则通常不使用**空间索引**。最佳做法是为数据库中的所有表选择一个SRID。仅在向外部程序读取或写入数据时使用转换函数将数据转换为基于指定SRID的数据。

 

# 二、转换数据

  如果查看SRID 26918的Proj4定义，我们可以看到投影是**UTM**（Universal Transverse Mercator） **zone 18**，度量单位为**米**。

![img](https://img-blog.csdnimg.cn/2019011409483460.png)

  让我们将一些数据从**投影坐标**转换为**地理坐标**（也称为"**经度**（longitude）/ **纬度**（latitude）"）。

  若要将数据从一种SRID转换为另一种SRID，必须首先验证几何图形是否具有有效的SRID。由于我们已经确认了当前数据中的SRID，所以接下来仅需要将**投影坐标系统**的SRID转换为**地理坐标系统**的SRID。

  地理坐标最常见的SRID是4326（WGS84地理坐标系统），对应于"**WGS84球体**上的**经度**/**纬度**"，你可以在spatialreference.org站点上看到该定义：

> http://spatialreference.org/ref/epsg/4326/

  你也可以从spatial_ref_sys表中查到该定义：

```sql
SELECT srtext FROM spatial_ref_sys WHERE srid = 4326;
```

![img](https://img-blog.csdnimg.cn/20190114100020765.png)

![img](https://img-blog.csdnimg.cn/20190114100037360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  让我们将"Broad St（宽街）"地铁站的坐标转换为**地理坐标**：

```sql
SELECT ST_AsText(ST_Transform(geom,4326))
FROM nyc_subway_stations
WHERE name = 'Broad St';
```

![img](https://img-blog.csdnimg.cn/20190114100329673.png)

![img](https://img-blog.csdnimg.cn/20190114100539928.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  如果加载数据或创建新几何图形而未指定SRID，则SRID的值将为0。回想一下，在[几何图形](https://blog.csdn.net/qq_35732147/article/details/85258273)中，当我们创建几何表时，我们并没有指定SRID。如果我们查询数据库，则应该知道所有"nyc_表"的SRID值都为26918，而geometries表的SRID默认值为0。

  若要查看表的SRID，请查询数据库的geometry_columns视图表：

```sql
SELECT f_table_name AS name, srid
FROM geometry_columns;
```

![img](https://img-blog.csdnimg.cn/20190114101343229.png)

![img](https://img-blog.csdnimg.cn/20190114101428323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  然而，如果你知道坐标的SRID是什么，则可以使用ST_SetSRID()对几何图形进行SRID设置。然后，你将能把几何图形的现有坐标系统转换为其他坐标系统。

```sql
SELECT ST_AsText(
 ST_Transform(
   ST_SetSRID(geom,26918),
 4326)
)
FROM geometries;
```

![img](https://img-blog.csdnimg.cn/20190114101854468.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

 

# 三、投影练习

  下面是一些我们已经看过的函数，它们应该对练习有用！

![img](https://img-blog.csdnimg.cn/20190114102239942.png)

  请记住你可以使用的在线资源：

- [http://spatialreference.org](http://spatialreference.org/)
- [http://prj2epsg.org](http://prj2epsg.org/)

  还有请记住我们的数据库中现有的数据表：

- ```
  nyc_census_blocks
  ```

  - name, popn_total, boroname, geom

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

## 3.1、练习

###   ①"基于UTM zone 18投影坐标系统的测量，纽约（New York）所有街道的长度是多少？"

```sql
SELECT Sum(ST_Length(geom))
  FROM nyc_streets;
```

![img](https://img-blog.csdnimg.cn/20190114102624847.png)

![img](https://img-blog.csdnimg.cn/20190114102702151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

###   ②"SRID 2831的WKT定义是什么？"

```sql
SELECT srtext FROM spatial_ref_sys
WHERE SRID = 2831;
```

  或者通过查询[prj2epsg](http://prj2epsg.org/epsg/2831)

```sql
PROJCS["NAD83(HARN) / New York Long Island",
  GEOGCS["NAD83(HARN)",
    DATUM["NAD83 (High Accuracy Regional Network)",
      SPHEROID["GRS 1980", 6378137.0, 298.257222101,
        AUTHORITY["EPSG","7019"]],
      TOWGS84[-0.991, 1.9072, 0.5129, 0.0257899075194932, -0.009650098960270402, -0.011659943232342112, 0.0],
      AUTHORITY["EPSG","6152"]],
    PRIMEM["Greenwich", 0.0,
      AUTHORITY["EPSG","8901"]],
    UNIT["degree", 0.017453292519943295],
    AXIS["Geodetic longitude", EAST],
    AXIS["Geodetic latitude", NORTH],
    AUTHORITY["EPSG","4152"]],
  PROJECTION["Lambert Conic Conformal (2SP)",
    AUTHORITY["EPSG","9802"]],
  PARAMETER["central_meridian", -74.0],
  PARAMETER["latitude_of_origin", 40.166666666666664],
  PARAMETER["standard_parallel_1", 41.03333333333333],
  PARAMETER["false_easting", 300000.0],
  PARAMETER["false_northing", 0.0],
  PARAMETER["scale_factor", 1.0],
  PARAMETER["standard_parallel_2", 40.666666666666664],
  UNIT["m", 1.0],
  AXIS["Easting", EAST],
  AXIS["Northing", NORTH],
  AUTHORITY["EPSG","2831"]]
```

###   ③"基于SRID 2831坐标系统，计算纽约市所有街道的长度是多少？"

```sql
SELECT Sum(ST_Length(ST_Transform(geom,2831)))
  FROM nyc_streets;
```

![img](https://img-blog.csdnimg.cn/2019011410320547.png)

![img](https://img-blog.csdnimg.cn/20190114103244668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：UTM 18与SRID 2831（the State Plane Long Island projection - 国家平面长岛投影）测量的差值为（10421993 - 10418904）/ 10418904 = 0.02%。利用地理法在地球球体上计算出的街道总长度为10421999，也就是说基于SRID 2831计算出来的结果和真实结果更接近。这并不奇怪，因为SRID 2831投影坐标系是精确地校准一个很小的区域（**纽约市**），而UTM 18必须为一个大的区域提供合理的结果。

###   ④" 'Broad St' 地铁站点的KML表示是什么？"

```sql
SELECT ST_AsKML(geom)
FROM nyc_subway_stations
WHERE name = 'Broad St';
```

![img](https://img-blog.csdnimg.cn/20190114104417626.png)

![img](https://img-blog.csdnimg.cn/20190114104500671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  嘿！结果坐标是**地理坐标**，而不是投影坐标，然而我们并没有调用ST_Transform()，为什么？因为KML标准规定所有坐标都必须是**地理坐标**（实际上是EPSG: 4326），所以ST_AsKML()函数会自动进行坐标转换。