# PostGIS教程七：几何图形（Geometry）

# 一、介绍

  在前面的章节中，我们已经往数据库中加载了数据，现在让我们来先看一些简单的例子。

  在pgAdmin中，再次选择nyc数据库并打开**SQL查询工具**。将下面的SQL代码粘贴到**pgAdmin SQL Editor窗口**中（删除默认情况下可能存在的任何文本），然后执行。

```sql
CREATE TABLE geometries (name varchar, geom geometry);
INSERT INTO geometries VALUES
  ('Point', 'POINT(0 0)'),
  ('Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
  ('Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
  ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
  ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');
SELECT name, ST_AsText(geom) FROM geometries;
```

![img](https://img-blog.csdnimg.cn/20181226164233896.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  上面的示例创建了一个**表**（geometries），然后向该**表**中插入5个**几何图形**数据（geometry）：

- 一个**点**（POINT）
- 一条**线**（LINESTRING）
- 一个**多边形**（POLYGON）
- 一个**内含空洞的多边形**（POLYGON with a hole）
- 一个**图形集合**（COLLECTION）

  最后，查询表中的数据并输出。

# 二、元数据表

  为了符合**Simple Features for SQL**（[SFSQL](https://postgis.net/workshops/postgis-intro/glossary.html#term-sfsql)）规范，PostGIS提供了两张表用于追踪和报告数据库中的**几何图形**（这两张表中的内容相当于元数据）：

- 第一张表spatial_ref_sys  ——  定义了数据库已知的所有**空间参照系统**，稍后将对其进行更详细的说明。
- 第二张表（实际上是**视图**-view）geometry_columns  ——  提供了数据库中所有空间数据表的描述信息。

  ![img](https://img-blog.csdnimg.cn/201812261711579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

​                 geometry_columns视图的结构

  让我们来看一下**数据库**中的geometry_columns表，像原先那样将以下命令粘贴到**查询工具**中：

```sql
SELECT * FROM geometry_columns;
```

![img](https://img-blog.csdnimg.cn/20181226171802954.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

- f_table_catalog，f_table_schema，和f_table_name提供各个**几何图形**（geometry）的**要素表**（feature table）—— 即空间数据表 —— 的完全限定名称，分别是数据库名、模式名、空间数据表名。
- f_geometry_column包含对应空间数据表中用于记录几何信息的属性列的列名**。**
- coord_dimension定义几何图形的维度（2维、3维或4维）
- srid定义引用自spatial_ref_sys表的空间参考标识符
- type列定义了几何图形的类型。比如"**点**（Point）"和"**线串**（Linestring）"等类型。

  通过查询该表，GIS客户端和数据库可以确定检索数据时的预期内容，并可以执行任何必要的投影、处理、渲染而无需检查每个**几何图形**（geometry）—— 这些就是元数据所带来的作用。

  **注意**：如果nyc**数据库**的**表**没有指定26918的srid，那该怎么办呢？通过更新**表**很容易修复：

```sql
SELECT UpdateGeometrySRID(‘nyc_neighborhoods’,’geom’,26918);
```

# 三、表示真实世界的对象

  **Simple Features for SQL**（[SFSQL](https://postgis.net/workshops/postgis-intro/glossary.html#term-sfsql)）规范是PostGIS开发的原始指导标准，它定义了如何表示真实世界的对象。

  通过形成连续的图形并以固定的分辨率对其进行数字化，实现了对真实世界的合理表示。

  SFSQL只规定了对真实世界对象的二维表示，然而，PostGIS已将其扩展到3维和4维的表示。最近，**SQL-Multimedia Part 3**（[SQL/MM](https://postgis.net/workshops/postgis-intro/glossary.html#term-sql-mm)）规范正式定义了它们自己的三维表示。

  示例的**表**包含不同几何图形类型的混合。我们可以使用读取**几何图形**元数据的函数获取每个对象的基本信息：

- **ST_GeometryType(geometry)**  ——  返回几何图形的类型
- **ST_NDims(geometry)**  ——  返回几何图形的维数
- **ST_SRID(geometry)**  ——  返回几何图形的空间参考标识码

```sql
SELECT name, ST_GeometryType(geom), ST_NDims(geom), ST_SRID(geom)
FROM geometries;
```

![img](https://img-blog.csdnimg.cn/20181227085146701.png)

![img](https://img-blog.csdnimg.cn/20181227090027362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 3.1、点（Points）

![_images/points.png](https://img-blog.csdnimg.cn/2018122711073149)

  空间**点**（Point）表示地球上的单个位置。**点**由单个坐标表示（包括2维、3维或4维）。

  当详细的细节（例如形状和大小）在目标空间尺度上不重要时，真实世界中的对象可以直接用**点**表示。

  例如，世界地图上的城市可以描述为**点**，而在一幅州地图中可以将城市表示为**多边形**。

```sql
SELECT ST_AsText(geom)
FROM geometries
WHERE name = 'Point';
```

![img](https://img-blog.csdnimg.cn/20181227090725958.png)

![img](https://img-blog.csdnimg.cn/20181227090907443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  针对**点**的一些特定**空间函数**包括：

- **ST_X(geometry)**  ——  返回X坐标
- **ST_Y(geometry)**  ——  返回Y坐标

  所以，我们这样来读取一个**点**图形的坐标值：

```sql
SELECT ST_X(geom), ST_Y(geom)
FROM geometries
WHERE name = 'Point';
```

![img](https://img-blog.csdnimg.cn/20181227091508731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **纽约市地铁站**（nyc_subway_stations）表是一个以**点**表示的数据集。以下**SQL查询**将返回一个**点**图形数据（在ST_AsText列中）：

```sql
SELECT name, ST_AsText(geom)
FROM nyc_subway_stations
LIMIT 1;
```

![img](https://img-blog.csdnimg.cn/20181227092245305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 3.2、线串（Linestring）

![_images/lines.png](https://img-blog.csdnimg.cn/2018122711073174)

  **线串**（Linestring）是表示两个或多个位置之间的路径，它的形式是由两个或多个**点**组成的有序序列。道路和河流通常表示为**线串**。

  如果**线串**的起始点和结束点是同一个**点**，则称其是**闭合的**（closed）。

  如果**线串**不与自身交叉或接触（如果**线串**是闭合的，则排除结束点），则称其是**简单的**（simple）。

  **线串**既可以是**闭合的**，也可以是**简单的**。

  **纽约的街道网络数据**（nyc_streets）在前面的章节中已经加载到**数据库**中了，这个数据集包含**名称**和**类型**等详细信息。

  一条真实的街道可能由许多**线串**组成，每条**线串**代表一段具有不同属性特征的道路。

  以下**SQL查询**将返回一个**线串**图形的信息（在ST_AsText列中）

```sql
SELECT ST_AsText(geom)
  FROM geometries
  WHERE name = 'Linestring';
```

![img](https://img-blog.csdnimg.cn/20181227095600396.png)

![img](https://img-blog.csdnimg.cn/20181227095650107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  用于处理**线串**的一些特定**空间函数**包括：

- **ST_Length(geometry)**  ——  返回线串的长度
- **ST_StartPoint(geometry)**  ——  将线串的第一个坐标作为点返回
- **ST_EndPoint(geometry）**  ——  将线串的最后一个坐标作为点返回
- **ST_NPoints(geometry)**  ——  返回线串的坐标数量

  所以，我们的**线串**的长度为：

```sql
SELECT ST_Length(geom)
FROM geometries
WHERE name = 'Linestring';
```

![img](https://img-blog.csdnimg.cn/20181227100113844.png)

![img](https://img-blog.csdnimg.cn/20181227100213660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 3.3、多边形（Polygon）

![_images/polygons.png](https://img-blog.csdnimg.cn/2018122711073189)

  **多边形**（Polygon）是区域的表示形式。**多边形**的外部边界由一个**环**（Ring）表示，这个**环**是一个**线串**，如上面定义的，它既是闭合的，又是简单的。**多边形**中的**孔**（hole）也由**环**表示。

  **多边形**用于表示重视**大小**和**形状**这两个特征的地理对象。城市边界、公园、建筑或水体都通常需要表示为**多边形**，当比例尺足够大时，可以观测它们的面积。道路和河流有时也可以表示为**多边形**。

  以下**SQL查询**将返回两个**多边形**图形的信息（在ST_AsText列中）：

```sql
SELECT ST_AsText(geom)
FROM geometries
WHERE name LIKE 'Polygon%';
```

  **注意**：我们不是在WHERE子句中使用"="符号，而是使用LIKE运算符执行字符串匹配操作。你可能习惯使用"*"符号作为模式匹配中的单字符或多字符匹配，但在SQL中，使用"%"符号和LIKE运算符来告诉系统执行全局匹配。

![img](https://img-blog.csdnimg.cn/20181227101824907.png)

![img](https://img-blog.csdnimg.cn/20181227102029837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  第一个**多边形**只有一个**环**，第二个**多边形**有一个内部的"**孔洞**（hole）"，大多数图形系统都包含**多边形**的概念，但GIS系统在允许**多边形**有**孔**方面是比较独特的。

![img](https://img-blog.csdnimg.cn/20181227102431294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  关于**多边形**图形的一些特定**空间函数**包括：

- **ST_Area(geometry)**  ——   返回多边形的面积
- **ST_NRings(geometry)**  ——  返回多边形中环的数量（通常为1个，其他是孔）
- **ST_ExteriorRing(geometry)**  ——  以线串的形式返回多边形最外面的环
- **ST_InteriorRingN(geometry, n)**  ——  以线串形式返回指定的内部环
- **ST_Perimeter(geometry)**  ——  返回所有环的长度

  我们可以使用**空间函数**计算**多边形**的面积：

```sql
SELECT name, ST_Area(geom)
FROM geometries
WHERE name LIKE 'Polygon%';
```

![img](https://img-blog.csdnimg.cn/20181227103037638.png)

![img](https://img-blog.csdnimg.cn/2018122710345236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  请注意，带**孔**的**多边形**的面积是**多边形外环**的面积（10 x 10正方形）减去**孔**的面积（1 x 1正方形）

## 3.4、集合（Collection）

  有四种**集合**（Collection）类型，它们将多个简单几何图形组合为**图形集合**：

- **MultiPoint**   ——  点集合
- **MultiLineString**  ——  线串集合
- **MultiPolygon**  ——  多边形集合
- **GeometryCollection**  ——  由任意几何图形（包括其他GeometryCollection）组成的异构集合

  **集合**更多地出现在GIS软件中，而不是在通用图形软件中。

  **集合**对于将**真实世界的对象**直接建模为**空间对象**非常有用。例如，如何对被**路权**（**路权**指交通参与者的权利）分割的多个**道路部分**进行建模？答案是将其作为**MultiPolygon**，其组成部分位于**路权**的两侧。

![_images/collection2.png](https://postgis.net/workshops/postgis-intro/_images/collection2.png)

  我们示例中的**几何图形集合**包含一个多边形和一个点：

```sql
SELECT name, ST_AsText(geom)
  FROM geometries
  WHERE name = 'Collection';
```

![img](https://img-blog.csdnimg.cn/20181227110134577.png)

![img](https://img-blog.csdnimg.cn/20181227110152910.png)

![img](https://img-blog.csdnimg.cn/2018122711032982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  用于处理**集合**的一些特定**空间函数**：

- **ST_NumGeometries(geometry)**  ——  返回集合中的组成部分的数量
- **ST_GeometryN(geometry, n)**  ——  返回集合中指定的组成部分
- **ST_Area(geometry)**  ——  返回集合中所有多边形组成部分的总面积
- **ST_Length(geometry)**  ——  返回所有线段组成部分的总长度

# 四、几何图形输入和输出

  在数据库中，**几何图形**（Geometry）以仅供PostGIS使用的格式存储在磁盘上。为了让外部程序插入和检索有用的**几何图形**信息，需要将它们转换为其他应用程序可以理解的格式。

  幸运的是，PostGIS支持以多种格式进行**几何图形**的输入和输出。

  ①Well-known text（[WKT](https://postgis.net/workshops/postgis-intro/glossary.html#term-wkt)）

- **ST_GeomFromText(text, srid)**  ——  返回geometry
- **ST_AsText(geometry)**  ——  返回text
- **ST_AsEWKT(geometry)**  ——  返回text

  ②Well-known binary（[WKB](https://postgis.net/workshops/postgis-intro/glossary.html#term-wkb)）

- **ST_GeomFromWKB(bytea)**  ——  返回geometry
- **ST_AsBinary(geometry)**  ——  返回bytea
- **ST_AsEWKB(geometry)**  ——  返回bytea

  ③Geographic Mark-up Language（[GML](https://postgis.net/workshops/postgis-intro/glossary.html#term-gml)）

- **ST_GeomFromGML(text)**  ——  返回geometry
- **ST_ASGML(geometry)**  ——  返回text

  ④Keyhole Mark-up Language（[KML](https://postgis.net/workshops/postgis-intro/glossary.html#term-kml)）

- **ST_GeomFromKML(text)**  ——  返回geometry
- **ST_ASKML(geometry)**  ——   返回text

  ⑤[GeoJson](https://postgis.net/workshops/postgis-intro/glossary.html#term-geojson)

- **ST_AsGeoJSON(geometry)**  ——  返回text

  ⑥Scalable Vector Graphics([SVG](https://postgis.net/workshops/postgis-intro/glossary.html#term-svg)）

- **ST_AsSVG(geometry)**  ——  返回text

  以上函数最常见的用法是将**几何图形**的**文本**（text）表示形式转换为内部表示形式：

  请注意，除了具有**几何图**形表示形式的**文本**参数外，还可以指定一个提供几何图形**SRID**的数字参数。

  以下**SQL查询**展示了一个WKB表示形式的示例（将**二进制**输出转换为ASCII格式以进行打印时，需要调用encode()）：

```sql
SELECT encode(
  ST_AsBinary(ST_GeometryFromText('LINESTRING(0 0,1 0)')),
  'hex');
```

![img](https://img-blog.csdnimg.cn/20181227141049374.png)

![img](https://img-blog.csdnimg.cn/20181227141613303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  在本教程中，将使用WKT，以确保你能够理解我们正在查看的**几何图形**。

  但是，在大多数实际生产环境中（如查看GIS应用程序中的数据、将数据传输到web客户端或远程处理数据），WKB是首选的格式。

  由于WKT和WKB是在[SFSQ](https://postgis.net/workshops/postgis-intro/glossary.html#term-sfsql)L规范中定义的，因此它们不能处理3维或4维的几何图形。对于这些情况，PostGIS定义了**Extended Well Known Text（EWKT）**和**Extended Well Known Binary（EWKB）**格式以用于处理3维或4维的几何图形。

  它们提供了与WKT和WKB相同的格式化功能，并且是在增加了**维度**的情况下。

  以下是WKT中**三维**（3D）**线串**示例：

```sql
SELECT ST_AsText(ST_GeometryFromText('LINESTRING(0 0 0,1 0 0,1 1 2)'));
```

![img](https://img-blog.csdnimg.cn/20181227142545654.png)

![img](https://img-blog.csdnimg.cn/20181227142754377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  注意：文本表示形式发生了变化！这是因为PostGIS的文本输入程序在使用方面是自由的。它可以使用：

- 十六进制编码的EWKB
- 扩展的WKT
- ISO标准的WKT

  在输出端，ST_AsText()只返回**ISO标准**的WKT。

  除了用于各种形式（WKT、WKB、GML、KML、JSON、SVG）的输出函数外，PostGIS还有基于四种形式（WKT、WKB、GML、KML）的输入函数。

  大多数应用程序使用WKT或WKB几何图形创建函数，但是也可以使用其他形式的几何图形创建函数。

  下面是一个使用GML输入和输出JSON的示例：

```sql
SELECT ST_AsGeoJSON(ST_GeomFromGML('<gml:Point><gml:coordinates>1,1</gml:coordinates></gml:Point>'));
```

![img](https://img-blog.csdnimg.cn/20181228091757202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 五、从文本转换

  到目前为止，我们看到的[WKT](https://postgis.net/workshops/postgis-intro/glossary.html#term-wkt)字符串都是'text'类型，我们使用PostGIS的函数ST_GeomFromText()将它们转换为'gometry'类型。

  PostgreSQL包含一个简短形式的语法，允许数据从一种类型转换到另一种类型，即类型转换语法：

```sql
olddata::newtype
```

  例如，将double类型转换为**文本字符串**类型：

```sql
SELECT 0.9::text;
```

  以下SQL语句将一个WKT字符串转换成一个**几何图形**（geometry）：

```sql
SELECT 'POINT(0 0)'::geometry;
```

![img](https://img-blog.csdnimg.cn/20181228092501947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  关于使用类型转换语法创建**几何图形**，需要注意一点：除非指定SRID，否则将得到一个包含未知SRID的**几何图形**。

  可以使用EWKT形式指定SRID，该形式在前面包含一个SRID：

```sql
SELECT 'SRID=4326;POINT(0 0)'::geometry;
```

![img](https://img-blog.csdnimg.cn/20181228092903483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

 