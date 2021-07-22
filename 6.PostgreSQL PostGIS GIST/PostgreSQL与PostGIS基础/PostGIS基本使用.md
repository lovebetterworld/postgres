- [PostGIS的基础入门](https://cloud.tencent.com/developer/article/1414516?from=article.detail.1722171)

# 一、PostGIS

PostgreSQL数据库安装PostGIS扩展，数据库将可以进行空间数据管理、数量测量与几何拓扑分析。

## 1.1 在testdb数据库下安装PostGIS扩展

安装PostGIS扩展：

```plsql
CREATE EXTENSION postgis;
```

验证PostGIS扩展是否安装成功：

```plsql
SELECT postgis_full_version();
```

还可以安装其它的一些扩展：

```plsql
-- Enable Topology
CREATE EXTENSION postgis_topology;
-- Enable PostGIS Advanced 3D-- and other geoprocessing algorithms
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

可使用`\dx`命令查看已安装的扩展。

## 1.2 创建空间数据表

- 先建立一个常规的表存储

```plsql
CREATE TABLE cities(id smallint,name varchar(50));
```

- 添加一个空间列，用于存储城市的位置。 习惯上这个列叫做 “the_geom”。它记录了数据的类型（点、线、面）、有几维（这里是二维）以及空间坐标系统。这里使用 EPSG:4326 坐标系统： SELECT AddGeometryColumn ('cities', 'the_geom', 4326, 'POINT', 2);

## 1.3 插入数据到空间表

批量插入三条数据：

```plsql
INSERT INTO cities(id, the_geom, name) VALUES (1,ST_GeomFromText('POINT(-0.1257 51.508)',4326),'London, England'), (2,ST_GeomFromText('POINT(-81.233 42.983)',4326),'London, Ontario'), (3,ST_GeomFromText('POINT(27.91162491 -33.01529)',4326),'East London,SA');
```

## 1.4 简单查询

标准的PostgreSQL语句都可以用于PostGIS，这里我们查询cities表数据：

```plsql
SELECT * FROM cities;
```

这里的坐标是无法阅读的 16 进制格式。要以WKT文本显示，使用ST_AsText(the_geom)或ST_AsEwkt(the_geom)函数。也可以使用ST_X(the_geom)和ST_Y(the_geom)显示一个维度的坐标：

```plsql
SELECT id, ST_AsText(the_geom), ST_AsEwkt(the_geom), ST_X(the_geom), ST_Y(the_geom) FROM cities;
```

## 1.5 空间查询

以米为单位并假设地球是完美椭球，上面三个城市相互的距离是多少？

执行以下代码计算距离：

```plsql
SELECT p1.name,p2.name,ST_Distance_Sphere(p1.the_geom,p2.the_geom) FROM cities AS p1, cities AS p2 WHERE p1.id > p2.id;
```

# 二、总结

关于PostgreSQL的一些官方学习资料如下，请参考：

- https://www.postgresql.org/files/documentation/pdf/9.6/postgresql-9.6-A4.pdf
- https://wiki.postgresql.org/wiki/9.1%E7%AC%AC%E4%BA%8C%E7%AB%A0
- https://wiki.postgresql.org/wiki/Main_Page
- 易百教程：https://www.yiibai.com/postgresql/postgresql-datatypes.html
- 中文手册：http://www.postgres.cn/docs/9.6/index.html
- Postgres中文社区：http://www.postgres.cn/v2/home

关于PostGIS的官方学习资料如下，请参考：

- 英文官方资料：http://www.postgis.net/stuff/postgis-2.4.pdf
- 中文社区资料：http://www.postgres.cn/docs/PostGis-2.2.0dev_Manual.pdf