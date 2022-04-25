- [大数据系列5：地理信息系统PostGis](https://blog.csdn.net/kittyzc/article/details/92832369)



# Gis基本概念

数据结构：GIS矢量数据由坐标构成，一个GIS特征可以是点、线、多边形……；栅格数据是影像数据，在地理数据库表示为一组数字矩阵。栅格数据的分辨率表示每个点对应的地理位置距离。
索引算法：四叉树索引、R树索引
概要化：在不同比例尺下加载不同分辨率的缩略图，形成一个影像金字塔
文件结构：Esri的shapefile文件是比较常见的文件类型。

- shp文件：最主要的文件，包含几何元素形状信息。比如从36字节开始包含4个边界值（8字节双精度数值），在python中可以用struct.unpack("<d",f.read(8))来读取。
- shx：存储要素几何图形的索引信息
- dbf：存储地理要素的属性信息（非几何信息）
- prj：存储空间参考信息，即地理坐标系统信息和投影坐标系统信息。使用well-known文本格式进行描述。
- 其他常见的数据格式：xml、geojson、geotiff(后缀可能是.tiff/.tif/.gtif)、jpeg(元数据标记系统称为exif)、bmp、全球文件(以w结尾)、LIDAR(light+radar，三维数据)

# 1. 安装数据库

## 1.1 postgresql安装与配置

在centos 7下安装很简单

```bash
yum -y install postgresql-server
postgresql-setup initdb 
```

此外可以再安装一些额外的插件，比如：

```bash
yum -y install postgis
```

按照这里安装pgrouting。

进行一些配置，让postgresql可被远程连接登录。
进入/var/lib/pgsql/data/postgresql.conf，将第59行取消注释并更改为：listen_addresses = ‘*’，将第395行更改为log_line_prefix = '%t %u %d ’

进入/var/lib/pgsql/data/pg_hba.conf，将最后的peer和ident改为trust

开启防火墙设置：

```bash
systemctl start firewalld
firewall-cmd --add-service=postgresql --permanent 
firewall-cmd --reload 
```

然后启动服务：

```bash
systemctl start postgresql  
systemctl enable postgresql 
```

默认用户名和数据库postgres，我们可以切换进postgres，然后新建用户名和数据库：

```bash
su - postgres
psql -c "alter user postgres with password 'password'"
createuser test_user
createdb test_db -O test_user
exit
```

然后在本机就可以登录

```plsql
 psql -U test_user -d test_db
```

postgresql基础操作总结如下：

- -h：指定连接的地址hostname
- -p指定连接的端口号
- -d指定连接的数据库名称
- -U指定连接的用户名
- -W指定在执行时弹出密码输入提示


## 1.2 创建postGis扩展

首先使用postgres账号登录postgres数据库：

```plsql
psql -U postgres
```

切换数据库：

```plsql
\c test_db
```

创建postgis扩展，并查看postgis的版本号(两种方法)，并退出：

```plsql
CREATE EXTENSION postgis;
SELECT postgis_full_version();
\dx
\q
```

## 1.3 下载数据

openstreetmap可以下载免费的osm格式数据。

geofabrik可以批量下载各个国家的osm和shp格式数据，可以参考链接 https://blog.csdn.net/qq_912917507/article/details/81736041

这个数据并不包含行政边界，坐标WGS84。
我们这里下载geofabrik的数据，并解压：

```bash
wget http://download.geofabrik.de/asia/china-latest-free.shp.zip
unzip china-latest-free.shp.zip
```

## 1.4 导入数据

使用shp2psql命令行工具导入数据：

    shp2pgsql -s 4326 -I "gis_osm_buildings_a_free_1" buildings_a | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_places_a_free_1" places_a | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_pois_free_1" pois | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_traffic_free_1" traffic | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_transport_a_free_1" transport_a | psql -h localhost -p 5432 -d test_db -U postgres
    shp2pgsql -s 4326 -I "gis_osm_railways_free_1 " railways  | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_landuse_a_free_1" landuse_a | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_pofw_a_free_1" pofw_a | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_roads_free_1" roads | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_natural_a_free_1" natural_a | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_water_a_free_1" water_a | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_waterways_free_1" waterways | psql -h localhost -p 5432 -d test_db -U postgres 
    shp2pgsql -s 4326 -I "gis_osm_places_a_free_1" places_a | psql -h localhost -p 5432 -d test_db -U postgres 

shp2pgsql的参数（具体参数使用shp2pgsql --help进行查看）如下：

- -s：指定空间参考系，PostGIS的参考系和EPSG代码是一样的，比如EPSG:4326表示WGS84地理坐标系
- -I：指定在新建的关系表的空间对象的那一列建立空间索引 。双引号引起来的是Shapefile的文件名称（也可以加上扩展名.shp），再加上关系表的全名。
- shp2pgsql的输出是一个标准的SQL，然后Linux的管道操作符’|’将结果传入到psql中进行SQL的执行。
  

# 2. postGis使用

## 2.1 数据库结构

导入数据如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620095827282.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)
注意到，我们的表里面有几个额外的视图，是PostGis为了符合Simple Features for SQL（SFSQL）规范，提供的元数据表。第一张表spatial_ref_sys定义了数据库已知的所有空间参照系统，稍后将对其进行更详细的说明。第二张视图geometry_columns提供了数据库中所有空间数据表的描述信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620163734342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

下图是例子：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620164145251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

f_table_catalog，f_table_schema，和f_table_name提供各个几何图形（geometry）的要素表（feature table）的完全限定名称，分别是数据库名、模式名、空间数据表名。
f_geometry_column包含对应空间数据表中用于记录几何信息的属性列的列名。
coord_dimension定义几何图形的维度（2维、3维或4维）
srid定义引用自spatial_ref_sys表的空间参考标识符
type列定义了几何图形的类型。比如"点（Point）"和"线串（Linestring）"等类型。
通过查询该表，GIS客户端和数据库可以确定检索数据时的预期内容，并可以执行任何必要的投影、处理、渲染而无需检查每个几何图形（geometry）—— 这些就是元数据所带来的作用。
如果数据库的表没有指定srid，那该怎么办呢？通过更新表很容易修复：

```plsql
SELECT UpdateGeometrySRID(‘waterways’,’geom’,4326);
```

## 2.2 数据存储格式

在postGis中，几何形状数据类型为geometry。数据可以通过ST_GeometryFromText函数来生成，也可以通过tuple的方式来生成。文本是wkt（well known text）或者wkb（well known binary）类型的数据，在python中可以用shapely库来处理：

```plsql
SELECT ST_AsText(ST_GeometryFromText('LINESTRING(0 0 0,1 0 0,1 1 2)'));
```

```plsql
CREATE TABLE geometries (name varchar, geom geometry);
INSERT INTO geometries VALUES
  ('Point', 'POINT(0 0)'),
  ('Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
  ('Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
  ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
  ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');
SELECT name, ST_AsText(geom) FROM geometries;
```

这里面显示了常见几种几何图形（geometry）的用法：

- 一个点（POINT）
- 一条线（LINESTRING）
- 一个多边形（POLYGON）
- 一个内含空洞的多边形（POLYGON with a hole）
- 一个图形集合（COLLECTION）
- 单元素多几何图形（MultiPolygon、MultiPoint、……）

PostGIS支持以多种格式进行几何图形的输入和输出：

```plsql
①Well-known text（WKT）
ST_GeomFromText(text, srid)    ——    返回geometry
ST_AsText(geometry)    ——    返回text
ST_AsEWKT(geometry)    ——    返回text

②Well-known binary（WKB）
ST_GeomFromWKB(bytea)    ——    返回geometry
ST_AsBinary(geometry)    ——    返回bytea
ST_AsEWKB(geometry)    ——    返回bytea

③Geographic Mark-up Language（GML）
ST_GeomFromGML(text)    ——    返回geometry
ST_ASGML(geometry)    ——    返回text

④Keyhole Mark-up Language（KML）
ST_GeomFromKML(text)    ——    返回geometry
ST_ASKML(geometry)    ——     返回text

⑤GeoJson
ST_AsGeoJSON(geometry)    ——    返回text

⑥Scalable Vector Graphics(SVG）
ST_AsSVG(geometry)    ——    返回text
```

以上函数最常见的用法是将几何图形的文本（text）表示形式转换为内部表示形式。以下SQL查询展示了一个WKB表示形式的示例（将二进制输出转换为ASCII格式以进行打印时，需要调用encode()）：

```plsql
SELECT encode(ST_AsBinary(ST_GeometryFromText('LINESTRING(0 0,1 0)')),'hex');
```

由于WKT和WKB是在SFSQL规范中定义的，因此它们不能处理3维或4维的几何图形。对于这些情况，PostGIS定义了Extended Well Known Text（EWKT）和Extended Well Known Binary（EWKB）格式以用于处理3维或4维的几何图形。

PostgreSQL包含一个简短形式的语法，允许数据从一种类型转换到另一种类型，即类型转换语法：

```plsql
olddata::newtype
```

例如，将double类型转换为文本字符串类型：

```plsql
SELECT 0.9::text;
```

以下SQL语句将一个WKT字符串转换成一个几何图形（geometry）：

```plsql
SELECT 'POINT(0 0)'::geometry;
SELECT 'SRID=4326;POINT(0 0)'::geometry;
```

## 2.3 重要函数

### 2.3.1 通用类

ST_AsText(geometry) —— 返回几何图形的值
ST_GeometryType(geometry) —— 返回几何图形的类型
ST_NDims(geometry) —— 返回几何图形的维数
ST_SRID(geometry) —— 返回几何图形的空间参考标识码
ST_StartPoint(geometry) —— 返回几何图形的第一个点
ST_EndPoint(geometry) —— 返回几何图形的最后一个点
点：
ST_X(geometry) —— 返回点的X坐标
ST_Y(geometry) —— 返回点的Y坐标
线：
ST_Length(geometry) —— 返回线串的长度
ST_StartPoint(geometry) —— 将线串的第一个坐标作为点返回
ST_EndPoint(geometry） —— 将线串的最后一个坐标作为点返回
ST_NPoints(geometry) —— 返回线串的坐标数量
多边形（环+孔）：
ST_Area(geometry) —— 返回多边形的面积
ST_NRings(geometry) —— 返回多边形中环的数量（通常为1个，其他是孔）
ST_ExteriorRing(geometry) —— 以线串的形式返回多边形最外面的环
ST_InteriorRingN(geometry, n) —— 以线串形式返回指定的内部环
ST_Perimeter(geometry) —— 返回所有环的长度
集合：
ST_NumGeometries(geometry) —— 返回集合中的组成部分的数量
ST_GeometryN(geometry, n) —— 返回集合中指定的组成部分
ST_Area(geometry) —— 返回集合中所有多边形组成部分的总面积
ST_Length(geometry) —— 返回所有线段组成部分的总长度

注意在SQL中，使用"%"符号和LIKE运算符来告诉系统执行全局匹配。
对于经纬度，面积计算以平方米为单位。

### 2.3.2 几何操作

ST_Equals(geometry A, geometry B)用于测试两个图形的空间相等性。
ST_Intersects、ST_Intersects、ST_Crosses和ST_Overlaps测试几何图形是否相交。
ST_Touches判断几何图形是否边界相交。
ST_Within()和ST_Contains()测试一个几何图形是否完全位于另一个几何图形内。
ST_Distance(geometry A, geometry B)计算两个几何图形之间的最短距离，并将其作为浮点数返回。
ST_DWithin测试两个几何图形之间的距离是否在某个范围之内。
ST_Centroid(geometry)返回大约位于输入参数的质心上的点。这种简单的计算速度非常快，但有时并不可取，因为返回点不一定在要素本身上。如果输入的几何图形具有凸性（假设字母’C’），则返回的质心可能不在图形的内部。
ST_PointOnSurface(geometry)返回保证在输入多边形内的点。从计算上讲，它比centroid操作代价要大得多。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624094758765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

ST_Buffer(geometry, distance)接受几何图形和缓冲区距离，并输出一个多边形，这个多边形的边界与输入的几何图形之间的距离与输入的缓冲区距离相等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624094843909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

ST_Intersection(geometry A, geometry B)函数返回两个参数共有的空间区域（或直线，或点）。如果参数不相交，该函数将返回一个空几何图形。
 ST_Union(geometry, geometry)接受两个几何图形参数并返回合并的并集。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095148443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

我们可以使用空间连接来执行sql语句。任何在两个表之间提供true/false关系的函数都可以用来驱动空间连接，但最常用的函数是：ST_Intersects、ST_Contains和ST_DWithin。

## 2.4 空间索引

索引通过将数据组织到搜索树中来加快搜索速度，搜索树可以快速遍历以查找特定记录。空间索引是PostGIS的最大价值之一。在前面的示例中，构建空间连接需要对整个表进行相互比较。这样做的代价很高：连接两个包含10000条记录的表（每个表都没有索引）将需要进行100000000次比较；如果使用空间索引，则比较次数可能低至20000次。用下面的语句查看索引信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190620195625632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

USING GIST子句告诉PostgreSQL在构建索引时使用generic index structure（GIST-通用索引结构）。创建索引时，如果收到类似错误：ERROR:index row requires 11340 bytes，maximum size is 8911，则可能是因为没有添加USING GIST子句。 PostGIS和Oracle Spatial都具有相同的"R-Tree"空间索引结构。R-Tree将数据分解为矩形（rectangle）、子矩形（sub-rectangle）和子-子矩形（sub-sub rectangle）等。它是一种自调优（self-tuning）索引结构，可自动处理可变数据的密度和对象大小。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624081456203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)要使用索引执行边界框搜索（即纯索引查询-Index only Query-没有过滤器），需要使用"&&“运算符。对于几何图形，&&运算符表示"边界框重叠或接触”（纯索引查询），就像对于数字，"=“运算符表示"值相同”。
数据库在添加索引或大容量数据之后可以进行清理工作。VACUUM命令要求PostgreSQL回收表页面中因记录的更新或删除而留下的任何未使用的空间。

## 2.5 空间投影

地球不是平的，也没有简单的方法把它放在一张平面纸地图上（或电脑屏幕上），所以人们想出了各种巧妙的解决方案（投影）。每种投影方案都有优点和缺点，一些投影保留面积特征；一些投影保留角度特征，如墨卡托投影（Mercator）；一些投影试图找到一个很好的中间混合状态，在几个参数上只有很小的失真。所有投影的共同之处在于，它们将（地球）转换为平面笛卡尔坐标系，选择哪种投影取决于你将如何使用数据（需要哪些数据特征，面积？角度？或者其他）。
PostGIS包含更改数据投影（重投影）的功能，即使用ST_Transform(geometry, srid)函数就可以实现重投影。另外，为了查看和设置几何图形的空间参照标识符，PostGIS提供了ST_SRID(geometry）和ST_SetSRID(geometry，SRID）函数。我们可以使用ST_SRID(geometry)函数确认数据的SRID：

```plsql
SELECT ST_SRID(geom) FROM nyc_streets LIMIT 1;
```

返回26919，其定义包含在spatial_ref_sys表中。事实上，有两个定义。“well-known text”（WKT）定义在srtext列中，"proj.4"格式定义在proj4text列。

## 2.6 地理坐标

与Mercator（墨卡托）、UTM（通用横轴墨卡托）、Stateplane中的坐标不同，地理坐标不是笛卡尔平面坐标（Cartesian coordinates）。地理坐标并不表示平面上与原点的线性距离，相反，这些球坐标描述了地球上的角坐标。在球坐标中，点由该点与参考子午线（经度）的旋转角度和该点与赤道的角度（纬度）指定。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624092153974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpdHR5emM=,size_16,color_FFFFFF,t_70)

地理坐标不能直接使用距离计算、相交包含函数等。为了解决这个问题，与SQL Server类似，PostGIS使用两种数据类型：“geometry"和"geography”，我们的数据应该保存为geography而不是geometry类型，如果我们尝试测量洛杉矶和巴黎之间的距离，我们应该使用ST_GeographyFromText(text)函数，而不是ST_GeometryFromText(text)。所有地理计算的返回值都以米为单位。
对于geography类型，只有相关的少量空间函数：

ST_AsText(geography) returns text
ST_GeographyFromText(text) returns geography
ST_AsBinary(geography) returns bytea
ST_GeogFromWKB(bytea) returns geography
ST_AsSVG(geography) returns text
ST_AsGML(geography) returns text
ST_AsKML(geography) returns text
ST_AsGeoJson(geography) returns text
ST_Distance(geography, geography) returns double
ST_DWithin(geography, geography, float8) returns boolean
ST_Area(geography) returns double
ST_Length(geography) returns double
ST_Covers(geography, geography) returns boolean
ST_CoveredBy(geography, geography) returns boolean
ST_Intersects(geography, geography) returns boolean
ST_Buffer(geography, float8) returns geography [1]
ST_Intersection(geography, geography) returns geography [1]
用于创建含有geography列的新表的SQL与用于创建geography表的SQL非常相似。但是，geography包含在表创建时直接指定表类型的功能。例如：

```plsql
CREATE TABLE airports (
  code VARCHAR(3),
  geog GEOGRAPHY(Point)
);

INSERT INTO airports VALUES ('LAX', 'POINT(-118.4079 33.9434)');
INSERT INTO airports VALUES ('CDG', 'POINT(2.5559 49.0083)');
INSERT INTO airports VALUES ('KEF', 'POINT(-22.6056 63.9850)');
```

如果你的数据在地理范围上是紧凑的（包含在州、县或市内），请使用基于笛卡尔坐标的geometry类型，这样使你的数据有意义。有关可能的参考系统的选择，请参见http://spatialreference.org站点并输入您所在区域的名称。
\如果你需要测量在地理范围上是分散的数据集（覆盖世界大部分地区）的距离，请使用geography类型。通过基于geography类型运行而节省的应用程序复杂性将抵消任何性能问题。同时转换为geometry可以抵消大多数功能限制。