Postgres空间地理类型POINT POLYGON实现附近的定位和电子围栏功能

原文地址：https://cloud.tencent.com/developer/article/1379951

## 一、需求和背景

在已有大量经纬度坐标点的情况下，给定一组经纬度如何快速定位到附近的POI有哪些？

现在使用经纬度转geohash的算法，将二维的距离运算转换为like前缀匹配。通过比较9位到5位前缀，来获取附近5米到3km之内的坐标，为了寻求更快的定位方法，测试一下postgres的空间类型。

## 二、安装插件postgis

先安装了pg-10, 并且是通过yum安装的。导入过repo.

检查插件

```bash
yum search postgis

postgis-docs.x86_64 : Extra documentation for PostGIS
postgis-jdbc.x86_64 : The JDBC driver for PostGIS
postgis-utils.x86_64 : The utils for PostGIS
postgis23_10-client.x86_64 : Client tools and their libraries of PostGIS
postgis23_10-devel.x86_64 : Development headers and libraries for PostGIS
postgis23_10-docs.x86_64 : Extra documentation for PostGIS
postgis23_10-utils.x86_64 : The utils for PostGIS
postgis24_10-client.x86_64 : Client tools and their libraries of PostGIS
postgis24_10-debuginfo.x86_64 : Debug information for package postgis24_10
postgis24_10-devel.x86_64 : Development headers and libraries for PostGIS
postgis24_10-docs.x86_64 : Extra documentation for PostGIS
postgis24_10-utils.x86_64 : The utils for PostGIS
postgis.x86_64 : Geographic Information Systems Extensions to PostgreSQL
postgis23_10.x86_64 : Geographic Information Systems Extensions to PostgreSQL
postgis24_10.x86_64 : Geographic Information Systems Extensions to PostgreSQL
```

安装

```bash
yum install postgis.x86_64 postgis24_10.x86_64
```

系统安装了插件之后，数据库还要继续启用插件才行。

针对数据库启用插件

```bash
# 添加空间插件
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
```

安装之后，public下会新增一个表spatial_ref_sys。

## 三、点POINT类型和距离

点POINT类型的数据结构为`POINT(0 0)`，正好可以用作存储经纬度。

### 3.1 表添加POINT类型

#### 1. AddGeometryColumn

使用函数[AddGeometryColumn](https://postgis.net/docs/AddGeometryColumn.html), 命令行查看函数

```plsql
\df+  AddGeometryColumn
```

#### 2. Synopsis

```plsql
text AddGeometryColumn(varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean use_typmod=true);

text AddGeometryColumn(varchar schema_name, varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean use_typmod=true);

text AddGeometryColumn(varchar catalog_name, varchar schema_name, varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean use_typmod=true);
```

添加两个点字段

```plsql
SELECT AddGeometryColumn ('poi', 'geom_point', 4326, 'POINT', 2);
SELECT AddGeometryColumn ('poi', 'geom_point_26986', 26986, 'POINT', 2);
```

其中两个重要的坐标体系

- 4326  GCS_WGS_1984  World Geodetic System (WGS)
- 26986  美国马萨诸塞州地方坐标系（区域坐标系） 投影坐标, 平面坐标

### 3.2 直接添加

```plsql
ALTER TABLE poi ADD COLUMN geom_p_alter geometry(POINT,4326);
```

### 3.3 添加空间索引

```plsql
CREATE INDEX idx_point
ON poi
USING gist(geom_point);
```

### 3.4 插入点

使用函数将文本转换为几何类型: [ST_GeomFromText](https://postgis.net/docs/ST_GeomFromText.html)

```plsql
sdx=# SELECT ST_GeomFromText('POINT(120.377041 36.066019)', 4326);
                  st_geomfromtext                   
----------------------------------------------------
 0101000020E61000001310937021185E4012F5824F73084240
(1 row)
```

使用坐标转换函数转换坐标体系：[ST_Transform](https://postgis.net/docs/ST_Transform.html)

```plsql
sdx=# SELECT ST_Transform(ST_GeomFromText('POINT(120.377041 36.066019)', 4326),26986)
                    st_transform                    
----------------------------------------------------
 01010000206A690000B6A9B046D9615AC162C3613707DD6441
```

使用函数将几何类型转换为文本描述：[ST_AsText](https://postgis.net/docs/ST_AsText.html)

```plsql
SELECT ST_AsText(ST_GeomFromText('POINT(120.377041 36.066019)', 4326));

st_astext          
-----------------------------
 POINT(120.377041 36.066019)
(1 row)
```

插入三个点

```plsql
update poi set 
geom_point=ST_GeomFromText('POINT(121.248642 31.380415)', 4326), 
geom_point_26986=ST_Transform(ST_GeomFromText('POINT(121.248642 31.380415)', 4326),26986),
geom_p_alter=ST_GeomFromText('POINT(121.248642 31.380415)', 4326) 
WHERE uuid='462745f185a349bbb8454f70d085baae';

SELECT geom_point,geom_point_26986,geom_p_alter from poi WHERE uuid='462745f185a349bbb8454f70d085baae';

 geom_point | geom_point_26986    |  geom_p_alter                    
----------------------------------------------------+----------------------------------------------------+--
 0101000020E610000085766FC1E94F5E400D1AFA27B858D83F | 01010000206A69000087930146005A5CC1ECE89370F91A6541 | 0101000020E610000085766FC1E94F5E400D1AFA27B858D83F
(1 row)


验证：

SELECT ST_AsText(geom_point),ST_AsText(geom_point_26986),geom_p_alter from s_poi_gaode WHERE uuid='462745f185a349bbb8454f70d085baae';

 st_astext |  st_astext  |     geom_p_alter                   
 
-----------------------------+-------------------------------------------+----------------------------------
 POINT(121.248642 31.380415) | POINT(-7432193.09384621 11065291.5180554) | 0101000020E610000085766FC1E94F5E400D1AFA27B858D83F
(1 row)
```

批量更新现有的经纬度字段为POINT

```plsql
update s_poi_gaode set 
geom_point=ST_GeomFromText('POINT('||longitude||'  ' ||latitude||')', 4326), 
geom_point_26986=ST_Transform(ST_GeomFromText('POINT('||longitude||'   ' ||latitude||')', 4326),26986);

验证：
SELECT longitude,latitude,ST_AsText(geom_point),ST_AsText(geom_point_26986) from s_poi_gaode WHERE uuid='e3ebbcf15cc545408ac8b22d4df64ca6';

longitude  | latitude  |          st_astext          |                st_astext                 
------------+-----------+-----------------------------+------------------------------------------
 121.417666 | 31.281433 | POINT(121.417666 31.281433) | POINT(-7448729.03389232 11054385.435284)
(1 row)
```

其中，需要注意的是，使用pg的字符串拼接符号`||`，POINT经纬度之间要留空格。

### 3.5 两个点之间的距离

距离计算函数 [ST_Distance](https://postgis.net/docs/ST_Distance.html)

文本转换地理几何类型函数 [ST_GeogFromText](https://postgis.net/docs/ST_GeogFromText.html) 。

文本转换为地理几何类型函数 [ST_GeographyFromText](http://yehe.isd.com/column/support-plan/article-edit/3267354)

计算距离，单位是m的方法

```plsql
-- 921.37629155
select ST_Distance(ST_GeographyFromText('SRID=4326;POINT(114.017299 22.537126)'), 
                    ST_GeographyFromText('SRID=4326;POINT(114.025919 22.534866)')
                    );
-- 921.37629155
SELECT ST_Distance(ST_GeomFromText('POINT(114.017299 22.537126)',4326):: geography,
            ST_GeomFromText('POINT(114.025919 22.534866)', 4326):: geography
        );

-- 920.28519
SELECT ST_DistanceSphere(ST_GeomFromText('POINT(114.017299 22.537126)',4326),
            ST_GeomFromText('POINT(114.025919 22.534866)', 4326)
        );
        
-- unit=m   26986  马萨诸塞州 投影平面坐标系 单位m  result=972.989337453172
SELECT ST_Distance(
            ST_Transform(ST_GeomFromText('POINT(114.017299 22.537126)',4326 ),26986),
            ST_Transform(ST_GeomFromText('POINT(114.025919 22.534866)', 4326 ),26986)
        );
```

计算距离，单位是度

```plsql
# unit=degrees   result=0.00891134108875483
SELECT ST_Distance(ST_GeomFromText('POINT(114.017299 22.537126)',4326),
            ST_GeomFromText('POINT(114.025919 22.534866)', 4326)
        );
```

关于单位是m的, 前三种的计算结果是正确的。最后一种坐标转换的计算方法， 参考[PostGIS 坐标转换(SRID)的边界问题引发的专业知识 - ST_Transform](https://github.com/digoal/blog/blob/master/201706/20170622_01.md) 建议国内不要使用马萨诸塞州的投影平面，会使得距离计算不够准确。

### 3.6 附近5公里内的点

使用函数[ST_DWithin](https://postgis.net/docs/ST_DWithin.html) 可以计算两个点之间的距离是否在5公里内。

```plsql
# 计算两个点是否在给定距离内

# 单位米m
SELECT ST_DWithin(
ST_GeographyFromText('SRID=4326;POINT(114.017299 22.537126)'), 
ST_GeographyFromText('SRID=4326;POINT(114.025919 22.534866)'), 
1000);


# 单位度degrees
SELECT ST_DWithin(
ST_GeomFromText('POINT(114.017299 22.537126)',4326), 
ST_GeomFromText('POINT(114.025919 22.534866)', 4326), 
0.00811134108875483);


-- 查找给定经纬度5km以内的点
SELECT
    uuid,
    longitude,
    latitude,
    ST_DistanceSphere (
        geom_point,
    ST_GeomFromText ( 'POINT(121.248642 31.380415)', 4326 )) distance 
FROM
    s_poi_gaode 
WHERE
    ST_DWithin ( geom_point :: geography, ST_GeomFromText ( 'POINT(121.248642 31.380415)', 4326 ) :: geography, 5000 ) IS TRUE 
    order by distance desc
    LIMIT 30;
```

通过指定类型`geom_point :: geography`，单位变成米, 否则默认距离单位是度。

### 3.7 最近的10个点

```plsql
SELECT * FROM s_poi_gaode_gps ORDER BY geom_point <-> ST_GeomFromText ( 'POINT(121.248642 31.380415)', 4326 )  LIMIT 10;
```

速度极快。

## 四、面多边形'POLYGON'

### 4.1 添加字段类型

```bash
SELECT AddGeometryColumn ('basic_mall_v1', 'geom_fence', 4326, 'POLYGON', 2);
或者
ALTER TABLE basic_mall_v1 ADD COLUMN geom_fence_alter geometry(POLYGON,4326);
```

添加索引

```plsql
CREATE INDEX idx_area_fence
ON basic_mall_v1
USING gist(geom_fence);
```

### 4.2 插入值

使用函数 ST_GeomFromText

```plsql
SELECT ST_GeomFromText ( 'POLYGON((118.902957 32.085437,118.9041 32.086069,118.904754 32.085219,118.903592 32.084564,118.902957 32.085437))', 4326 );
                     st_geomfromtext 
-----------------------------------------------------------------------
 0103000020E610000001000000050000006F2C280CCAB95D40266F8099EF0A404012143FC6DCB95D4087191A4F040B4040363B527DE7B95D40B9FFC874E80A4040583B8A73D4B95D40A0353FFED20A40406F2C280CCAB95D40266F8099EF0A4040
(1 row)

-- 验证

SELECT
ST_AsText(
    ST_GeomFromText ( 'POLYGON((118.902957 32.085437,118.9041 32.086069,118.904754 32.085219,118.903592 32.084564,118.902957 32.085437))', 4326 )
    )
                                                     st_astext                                                     
-------------------------------------------------------------------------------------------------------------------
 POLYGON((118.902957 32.085437,118.9041 32.086069,118.904754 32.085219,118.903592 32.084564,118.902957 32.085437))
(1 row)
```

更新到数据库字段

```plsql
UPDATE basic_mall_v1 SET geom_fence=ST_GeomFromText ( 'POLYGON((118.902957 32.085437,118.9041 32.086069,118.904754 32.085219,118.903592 32.084564,118.902957 32.085437))', 4326 )  WHERE id=1000001;

-- 验证：
 SELECT ST_AsText(geom_fence) FROM basic_mall_v1 WHERE id=1000001;
                                                     st_astext                                                     
-------------------------------------------------------------------------------------------------------------------
 POLYGON((118.902957 32.085437,118.9041 32.086069,118.904754 32.085219,118.903592 32.084564,118.902957 32.085437))
(1 row)
```

实际上，我们原始围栏数据可能是这样的

```plsql
- longitude,latitude; longitude,latitude; longitude,latitude; 
119.306413,26.131464;119.307575,26.131739;119.30776,26.131224;119.307336,26.131114;119.307438,26.130791;119.306776,26.13059;119.306413,26.131464
```

需要将这个字段转换成空间类型的围栏字段。

```plsql
UPDATE basic_mall_v1 SET geom_fence=
    ST_GeomFromText(
    'POLYGON(('||  
    replace(
    replace(gaode_shape, ',', ' ' ), 
    ';', ',')
    ||'))'
    , 4326)
```

### 4.3 计算gps附近30m内的围栏

使用函数[ST_DWithin](https://postgis.net/docs/ST_DWithin.html) 判断一个几何对象是否在另一个的r距离以内：

```plsql
SELECT
    ST_Distance(ST_GeomFromText('POINT(120.731069 30.758984)',4326):: geography,
            geom_fence :: geography
        ) AS distance, id, name
FROM
    basic_mall_v1 
WHERE
    ST_DWithin (
        geom_fence :: geography,
        ST_GeomFromText ( 'POINT( 120.731069 30.758984)', 4326 ) :: geography,
        30 
    ) ORDER BY distance 
    
    LIMIT 10;
```

使用函数[boolean ST_Within(geometry A, geometry B);](http://yehe.isd.com/column/support-plan/article-edit/3267354) 判断A是否完全在B内部

```plsql
SELECT
      id, name
    FROM
      basic_mall_v1
    WHERE
      ST_Within (
        ST_GeomFromText('POINT('|| #{longitude} ||' '|| #{latitude} ||')',4326),
        geom_fence
      )
```

