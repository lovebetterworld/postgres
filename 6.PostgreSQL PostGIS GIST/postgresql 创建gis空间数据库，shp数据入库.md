- [postgresql 创建gis空间数据库，shp数据入库](https://blog.csdn.net/yangniceyangyang/article/details/104047479)
- https://blog.csdn.net/gis_zzu/article/details/91045052
- https://www.jianshu.com/p/2c4f714c62b5
- [PostgreSQL创建空间数据库](https://www.cnblogs.com/jiefu/p/13904912.html)



# 一、postgresql创建空间数据库

## **1.1 创建普通数据库**

```plsql
CREATE DATABASE gisdbname;
```

## 1.2 数据库添加空间扩展

```plsql
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION postgis_tiger_geocoder;
CREATE EXTENSION address_standardizer;
```
# 二、导入shp文件到数据库

## **2.1 shp数据准备**

**注意**：postGIS导入shp数据路径不能含有中文，如果含有中文会报错，而且自己要知道自己的数据的坐标系

## 2.2 打开PostGIS 2.0 Shapefile and DBF Loader Exporter

![img](https://img-blog.csdnimg.cn/20190606154216367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dpc196enU=,size_16,color_FFFFFF,t_70)

弹出如下图：

![img](https://img-blog.csdnimg.cn/20190606154258929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dpc196enU=,size_16,color_FFFFFF,t_70)

## 2.3 连接数据库

![img](https://img-blog.csdnimg.cn/20190606154412337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dpc196enU=,size_16,color_FFFFFF,t_70)

## 2.4 选择要入库的shp文件

![img](https://img-blog.csdnimg.cn/20190606154507931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dpc196enU=,size_16,color_FFFFFF,t_70)

## 2.5 修改SRID的值，双击SRID的值，设置导入数据的坐标系

![img](https://img-blog.csdnimg.cn/2019060615464535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dpc196enU=,size_16,color_FFFFFF,t_70)

# 三、PostgreSQL创建空间数据库练习

## 3.1 创建数据库

![img](https://img2020.cnblogs.com/blog/997646/202010/997646-20201030234347026-1976099327.png)
 ![img](https://img2020.cnblogs.com/blog/997646/202010/997646-20201030234704561-1754943698.png)

## 3.2 添加postgis扩展，使之成为支持空间类型的空间数据库

```plsql
create extension postgis
```

## 3.3 字段设置为geometry类型

![img](https://img2020.cnblogs.com/blog/997646/202010/997646-20201030235541420-888055935.png)

## 3.4 插入空间数据

```plsql
insert into test(id,shape) values(1,point(12.32232442,43.2324535)::geometry);
```

![img](https://img2020.cnblogs.com/blog/997646/202010/997646-20201031000045954-299499116.png)

## 3.5 查询空间数据

```plsql
insert into test(id,shape) values(1,point(12.32232442,43.2324535)::geometry);
```

![img](https://img2020.cnblogs.com/blog/997646/202010/997646-20201031000230929-326474463.png)

# 四、【Postgres】空间数据库创建

## 4.1 扩展PG的空间数据库功能

```plsql
-- Enable PostGIS (includes raster) 
CREATE EXTENSION postgis; 
-- Enable Topology 
CREATE EXTENSION postgis_topology; 
-- Enable PostGIS Advanced 3D 
-- and other geoprocessing algorithms 
-- sfcgal not available with all distributions 
CREATE EXTENSION postgis_sfcgal; 
-- fuzzy matching needed for Tiger 
CREATE EXTENSION fuzzystrmatch; 
-- rule based standardizer 
CREATE EXTENSION address_standardizer; 
-- example rule data set 
CREATE EXTENSION address_standardizer_data_us; 
-- Enable US Tiger Geocoder 
CREATE EXTENSION postgis_tiger_geocoder;
```

# 五、【Postgres】根据字段数据创建空间字段

```plsql
--添加空间字段
SELECT AddGeometryColumn ('GIS', '四至', 4326, 'POLYGON', 2);

--根据其他字段更新空间字段数据
update "GIS" b 
set "四至"=ST_GeomFromText ('POLYGON((' || to_char(a."东经起",'999.9999') || to_char(a."北纬起",'999.9999') || ',' || to_char(a."东经止",'999.9999') || to_char(a."北纬起",'999.9999') || ',' || to_char(a."东经止",'999.9999') || to_char(a."北纬止",'999.9999') ||',' || to_char(a."东经起",'999.9999') || to_char(a."北纬止",'999.9999') || ',' || to_char(a."东经起",'999.9999') || to_char(a."北纬起",'999.9999') || '))',4326)
from "GIS" a
where b."ID"=a."ID"

--创建索引
CREATE INDEX shape_index_sz1
ON "GIS"
USING gist
(四至); 

--查询与指定范围相交的多边形
SELECT * FROM "GIS" where 
ST_Intersects(
ST_GeomFromText('POLYGON((86 44.1667,87.3333 44.1667,87.3333 45.1667,86 45.1667,86 44.1667))'), ST_GeomFromText(ST_AsText("四至")))
```

