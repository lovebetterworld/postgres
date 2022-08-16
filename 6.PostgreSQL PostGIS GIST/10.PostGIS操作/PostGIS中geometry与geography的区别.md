- [PostGIS中geometry与geography的区别_supermapsupport的博客-CSDN博客](https://blog.csdn.net/supermapsupport/article/details/123801084?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5-123801084-blog-118405266.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5-123801084-blog-118405266.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=10)

摘要：除了geometry，PostGIS还定义了geography，用于不同的场景需求。以下通过实战对比介绍geometry与geography之间的差异、适用场景及互转方法。

## 1 geography基础介绍

与geometry不同的是，geography原生支持地理坐标（也称为大地坐标或经纬度坐标），基于球面模型进行函数分析和计算。下图为两个坐标系：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3eaa3a8e56754575b861b07832c68b94.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAc3VwZXJtYXBzdXBwb3J0,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 2 geometry与geography对比

以下列举了二者在各个维度间的对比：

| 对比项     | geometry                                                     | geography                                                    |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 名称       | 几何对象                                                     | 地理对象                                                     |
| 坐标系     | 支持平面坐标系和球面坐标系                                   | 仅支持球面坐标系                                             |
| 对象类型   | 支持 POINT、MULTIPOINT、LINESTRING、LINEARRING、MULTILINESTRING、POLYGON、MULTIPOLYGON、POLYHEDRALSURFACE、TRIANGLE、TIN、GEOMETRYCOLLECTION等简单对象，还支持CIRCULARSTRING、COMPOUNDCURVE、CURVEPOLYGON、MULTICURVE、MULTISURFACE | 仅支持POINT、LINESTRING、POLYGON、MULTIPOINT、MULTILINESTRING、MULTIPOLYGON、GEOMETRYCOLLECTION |
| 函数限制   | 类型较丰富                                                   | 支持类型较少，仅支持类型转换、长度面积距离计算、交并差运算函数 |
| 索引局限性 | 几何索引不能正确处理极地地区查询                             | 地理索引可以正确处理覆盖极点或国际日期变更线的查询           |
| 元数据     | geometry_columns：提供了数据库中所有空间数据表的描述信息，分别是数据库名、模式名、空间数据表名，属性列名称，几何图形维度，空间参考标识符，几何图形类型 | 无定义的元数据，需自定义                                     |
|            |                                                              |                                                              |

**根据geometry与geography的特点，在使用时可根据以下情况选择：**

| geometry                                                     | geography                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 如果数据在地理范围上是紧凑的（包含在州、县或市内），推荐使用基于笛卡尔坐标的geometry类型 | 如果需要测量在地理范围上是分散的数据集（覆盖世界大部分地区）距离，推荐使用geography类型。 |
| 当做数据存储时，推荐使用geometry                             | 由于地理坐标较为精确，因此在进行距离、面积等量算时，建议使用geography |
| 如果用户较为了解投影信息知识，推荐使用geometry               | geography不需要了解专业的投影知识只需要知道经纬度就可以进行计算，因此使用门槛较低 |
| 当场景需要运用大量复杂函数时，推荐使用geometry               | geography的支持函数较少，且计算复杂，因此应用时需要占用较多计算资源。 |
|                                                              |                                                              |

## 3 geometry与geography应用实践

首先我们分别使用geometry和geography类型通过*ST_Distance*计算北京到昆明之间的距离，空间参考设置为SRID=4326

```sh
--geometry对象计算距离
SELECT ST_Distance(
  'SRID=4326;POINT(116.415767 39.916042)'::geometry,    -- Beijing
  'SRID=4326;POINT(102.833963 24.916456)'::geometry     -- Kunming
  );
--结果
20.234944528360142 
```

可以看到，采用geometry在平面坐标系上计算的结果并不正确

```sh
 --转成geography对象计算距离
 SELECT ST_Distance(
  'SRID=4326;POINT(116.415767 39.916042)'::geography,     -- Beijing
  'SRID=4326;POINT(102.833963 24.916456)'::geography      -- Kunming
  );
 --结果
2091929.28729987 
```

计算结果是2092千米，采用geography对象在球面坐标系计算的结果正确，两点的距离量算为大圆航线的一部分。但是该函数只适用于点对象。那我们可以尝试接入构造函数解决这一问题：

```sh
-- ST_GeographyFromText(text)
SELECT ST_Distance(
  ST_GeographyFromText('POINT(116.415767 39.916042)'),    -- Beijing
  ST_GeographyFromText('POINT(102.833963 24.916456)')     -- Kunming
);
--结果
2091929.28729987
```

可以看到，使用 *ST_GeographyFromText(text)* 函数计算，也得到了正确的结论。因此在加入其它几何对象的计算需求下，可使用这种方法进行计算，如计算北京到昆明航线离上海的最短距离时，可采用以下做法：

```sh
--北京到昆明航线离上海的最短距离，即垂线段距离：
SELECT ST_Distance(
  ST_GeographyFromText('LINESTRING(116.415767 39.916042, 102.833963 24.916456)'), -- Beijing to Kunming
  ST_GeographyFromText('POINT(121.452027 31.242725)')                        -- Shanghai
);
--结果
988436.03069308
```

得到正确的结果为988千米

## 4 geometry与geography互转

为了方便进行准确运算并且可以灵活运用丰富的函数类型，PostGIS提供了二者之间的互转，满足多场景应用。

- **geometry转为geography**

为了将geometry数据加载到geography表中，首先需要将geometry转换到EPSG:4326（经度/纬度），然后再将其转换为geography。 *ST_Transform(geometry, srid)* 函数能将坐标转换为地理坐标，*Geography(geometry)* 函数能将基于EPSG:4326的geometry数据类型转换为geography数据类型，流程如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d0f5783f6c444469a0c68a68b1e288b5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAc3VwZXJtYXBzdXBwb3J0,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```sh
--使用ST_Transform(geometry, srid)和Geography(geometry)两个嵌套函数将nyc_subway_stations示例数据从geom转换为geog
CREATE TABLE nyc_subway_stations_geog AS
SELECT
  Geography(ST_Transform(geom,4326)) AS geog,
  name,
  routes
FROM nyc_subway_stations;
```

同时，当数据量大时，支持创建空间索引，在geography表上构建空间索引与在geometry表上构建空间索引的方法完全相同：

```sh
--创建GiST索引类型
CREATE INDEX nyc_subway_stations_geog_gix
ON nyc_subway_stations_geog USING GIST (geog);
```

- **geography转为geometry**

虽然geography类型的空间函数可以解决许多问题，但有时仍需要使用geometry类型支持的其他空间函数，需要将对象从geography转换为geometry。以下city表为例：

```sh
--GEOGRAPHY的列名为geog
CREATE TABLE city (
    code VARCHAR(3),
    geog GEOGRAPHY(Point)
  );

INSERT INTO city
  VALUES ('BJ', 'POINT(116.41 39.916042)');
INSERT INTO city
  VALUES ('KM', 'POINT(102.833963 24.956)');
INSERT INTO city
  VALUES ('SH', 'POINT(121.452 31.245)');
```

利用SQL语句查询geography元数据，可以看到geography给city表的geog列指定了默认的srid=4326：

```sh
SELECT * FROM geography_columns;
```

结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3df25e68db274e2596ae7ddaafb9b2e3.png#pic_center)

利用类型转换方法，函数将geog转为geometry类型，并调用 *ST_X* 方法读取转换后的坐标：

```sh
SELECT code, ST_X(geog::geometry) AS longitude FROM city;
```

结果为如下图，可以看到采用类型转换的方法，并不会改变geometry本身的坐标值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4704bf35ba5c43698507bb3fe81da1c2.png#pic_center)