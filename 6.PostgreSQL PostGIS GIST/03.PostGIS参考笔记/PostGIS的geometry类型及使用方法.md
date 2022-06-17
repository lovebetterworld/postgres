- [PostGIS的geometry类型及使用方法_supermapsupport的博客-CSDN博客_geometry类型](https://blog.csdn.net/supermapsupport/article/details/123573338)

PostGIS中矢量数据如何存储和构造，有哪些注意事项？其空间数据模型体系又遵循哪些标准规范？本文进行了详细介绍，并提供实操内容供读者参考。

此外，Yukon构建在PostGIS的基础能力之上（参见文章：[Yukon及其模块简介](https://blog.csdn.net/supermapsupport/article/details/123502585?spm=1001.2014.3001.5501) ），完全兼容PostGIS的矢量数据能力。

# PostGIS的geometry数据类型

geometry是PostGIS的基本空间数据类型，用于表达点线面等空间要素，具体类型涵盖了OGC的简单对象模型，并扩展实现了 SQL/MM ( ISO/IEC 13249-3 SQL Multimedia - Spatial ) Curver相关类型，定义了包含圆弧曲线的几何子对象类型 CircularString、 CompoundCurve、 CurvePolygon、MultiCurve、 MultiSurface。

> OGC在 SFA( Simple Features Access Standard ) 中定义了几何对象的类型，其中包括原子类型的 Point、LineString、LinearRing 和 Polygon，以及集合类型 MultiPoint、MultiLineString、MultiPolygon 和 GeometryCollection。

以下为各几何对象的类型、构成及有效性限定：

| 对象分类         | 对象类型           | 描述         | 构成                                                         | 有效性限定                                                   |
| ---------------- | ------------------ | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SFA简单对象      | POINT              | 点           | 单点                                                         | /                                                            |
|                  | MULTIPOINT         | 多点         | 由n个点组成                                                  | /                                                            |
|                  | LINESTRING         | 线，折线     | 由点串构成                                                   | /                                                            |
|                  | LINEARRING         | 线环         | 由点串构成                                                   | 首尾闭合，非自相交                                           |
|                  | MULTILINESTRING    | 点串         | 由n个线串组成                                                | /                                                            |
|                  | POLYGON            | 面           | 由n个首尾相交的线串组成                                      | 边缘为线环、内部为洞                                         |
|                  | MULTIPOLYGON       | 多面         | 由n个面组成                                                  | 非覆盖、非相邻                                               |
|                  | POLYHEDRALSURFACE  | 多面体表面   | 由n个具有相邻边的Polygon组成                                 | 具有相邻边                                                   |
|                  | TRIANGLE           | 三角形       | 由3个非共线顶点组成                                          | 首尾闭合、非共线                                             |
|                  | TIN                | 不规则三角网 | 由n个三角形组成                                              | 非覆盖                                                       |
|                  |                    |              |                                                              |                                                              |
| SQL/MM参数化对象 | CIRCULARSTRING     | 曲线串       | 用点串描述,三个点确定一段圆弧                                | 前一个圆弧的最后一个点与后一个圆弧的第一个点共用；特别地，如果圆弧的第一个点与第三个点重合，则第二个点表示圆心，以此来表达圆形 |
|                  | COMPOUNDCURVE      | 复合线       | 由 n 个部分组成，每个部分可以是LineString 或 CircularString  | 前一部分的最后一个点与后续部分的第一个点重合，保证复合线对象的连续性 |
|                  | CURVEPOLYGON       | 曲面         | 由 n 个部分组成，每个部分是首尾相连的 CircularString 或 LineString 或 CompoundCurve | 与 Polygon 类似，都表达一个闭合的区域，区别在于是否有CircularString对象参与构造 |
|                  | MULTICURVE         | 多(曲)线     | 由 n 个子对象组成，每个子对象可以是CircularString 或 LineString 或 CompoundCurve | /                                                            |
|                  | MULTISURFACE       | 多面         | 由 n 个子对象组成，每个子对象可以是Polygon 或 CurvePolygon 类型 | 与 MultiPolygon类似，都表达多面对象，区别在于是否有CircularString对象参与构造 |
|                  |                    |              |                                                              |                                                              |
|                  | GEOMETRYCOLLECTION | 复合对象     | 由n个任意子对象类型构成                                      | /                                                            |
|                  |                    |              |                                                              |                                                              |

*（注意：以上每种类型，可以指定是否带Z或M值。例如，点可以指定为：POINT、POINTZ、POINTM、POINTZM。）*

# geometry的对象构成关系

点线面几何对象的关系较为清晰，当加入新的类型 CircularString 后，衍生出带参数化对象的线和面，进而构成PostGIS的全部17种子类型，见下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e72ded600ae64673b51c4e002ed734cd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAc3VwZXJtYXBzdXBwb3J0,size_18,color_FFFFFF,t_70,g_se,x_16#pic_center)

# geometry的元数据

PostGIS中geometry元数据存储在geometry_columns中，通过该表可以查询出当前数据库中有哪些表存储了geometry及geometry的类型、坐标系信息。
用户在创建geometry类型时，PostGIS自动维护该表,其定义遵循OGC SFSQL（Simple Features for SQL）规范。

geometry_columns的表结构及与其它表格的关联关系见下图：
[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-aRhn004X-1647583326711)(./01_metadata.png)]

geometry_columns中字段信息描述如下：

- f_table_catalog，f_table_schema，和f_table_name提供各个几何图形（geometry）的要素表（feature table），即空间数据表 的完全限定名称，分别是数据库名、模式名、空间数据表名。
- f_geometry_column包含对应空间数据表中用于记录几何信息的属性列的列名。
- coord_dimension定义几何图形的维度（2维、3维或4维）
- srid会引用自spatial_ref_sys表的空间参考标识符
- type列定义了几何图形的类型。比如"点（Point）"和"线串（Linestring）"等类型。

示例数据通过SQL语句查看：

```sh
SELECT * FROM geometry_columns;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a9888bc57d234ba19d94c05453398f31.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAc3VwZXJtYXBzdXBwb3J0,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

# geometry构建示例

## 简单几何对象

Point、MultiPoint、LineString、MultiLineString、Polygon、MultiPolygon，构造示例如下：

```sh
--- 指定 srid 为 4326
CREATE TABLE testgeomobj (id serial, geom geometry NOT NULL);

--- Point 对象，如下图id=1
insert into testgeomobj (geom) VALUES ('SRID=4326;POINT(-95.363151 29.763374)');

--- MultiPoint 对象，如下图id=2
insert into testgeomobj (geom) VALUES ('SRID=4326;MULTIPOINT(-95.4 29.8,-96 30)');

--- LineString 对象，如下图id=3
insert into testgeomobj (geom) values ('SRID=4326;LINESTRING(-71.1031880899493 42.3152774590236,-71.1031627617667 42.3152960829043,-71.102923838298 42.3149156848307,-71.1023097974109 42.3151969047397,-71.1019285062273 42.3147384934248)');

--- MultiLineString 对象，如下图id=4
insert into testgeomobj (geom) values ('SRID=4326;MultiLineString (
(-71.1031880899493 42.3152774590236,-71.1031627617667 42.3152960829043,-71.102923838298 42.3149156848307,-71.1023097974109 42.3151969047397,-71.1019285062273 42.3147384934248),
(-71.1766585052917 42.3912909739571, -71.1766820268866 42.391370174323896, -71.1766063012595 42.3913825660754, -71.17658265830809 42.391303365353096)
)');

--- Polygon 对象，如下图id=8
insert into testgeomobj(geom) values ('SRID=4326;POLYGON (
(-71.1776585052917 42.3902909739571, -71.1776820268866 42.3903701743239, -71.1776063012595 42.3903825660754, -71.1775826583081 42.3903033653531,-71.1776585052917 42.3902909739571)
)');

--- MultiPolygon 对象，如下图id=9
insert into testgeomobj(geom) values ('SRID=4326; MultiPolygon (
((-71.1776585052917 42.3902909739571, -71.1776820268866 42.3903701743239, -71.1776063012595 42.3903825660754, -71.1775826583081 42.3903033653531,-71.1776585052917 42.3902909739571)),
((-71.1766585052917 42.3912909739571, -71.1766820268866 42.391370174323896, -71.1766063012595 42.3913825660754, -71.17658265830809 42.391303365353096, -71.1766585052917 42.3912909739571))
)');
```

*（注：可使用数据库管理工具输出SQL语句，本例使用Dbeaver，可在其面板进行几何对象可视化，也可使用SuperMMap iDesktopX或QGIS可视化）*

结果如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/49f6766ffc8444b7b3d15c7231ac5241.gif#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a5e35838a814401aa2dce50321439af.gif#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac1c0251beee4da8ab5c12abcbbb3937.gif#pic_center)

## 参数化几何对象

CircularString CompoundCurve CurvePolygon
以相同的方式插入，但由于大部分可视化工具不支持CURVES类型几何，因此可通过*st_curvetoline* 函数将CURVES转化为LINE进行可视化。

*(需要特别注意的是，在转化为LINE进行面积及周长计算时，会产生一定的误差)*

```sh
--- CircularString：由四段组成，如下图id=14&15
INSERT INTO testgeomobj (geom) VALUES (st_curvetoline('CIRCULARSTRING(0 2, -1 1, 0 0, 0.5 0, 1 0, 2 1, 1 2, 0.5 2, 0 2)'));

--- CompoundCurve：由圆弧和折线段组成，'LINESTRING'关键字可省略，生成的图形，id=15
INSERT INTO testgeomobj(geom) VALUES  (st_curvetoline('COMPOUNDCURVE(CIRCULARSTRING(0 0, 1 1, 1 0),LINESTRING(1 0, 2 0))'));
```

结果如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ca341587c0c24152b217b017684043f0.gif#pic_center)

## 岛洞对象

同时还可以生成复杂岛洞类型，通过创建含有COMPOUNDCURVE（复合曲线）的CURVEPOLYGON来实现。

```sh
---Circles with triangle hole、Triangle with arcish hole,如下图id=17&18
INSERT INTO testgeomobj (geom) VALUES
	(st_curvetoline('CURVEPOLYGON(
			CIRCULARSTRING(2.5 2.5, 4.5 2.5, 4.5 3.5, 2.5 4.5, 2.5 2.5),
			(3.5 3.5, 3.25 2.25, 4.25 3.25, 3.5 3.5))', 4326)),     
	(st_curvetoline('CURVEPOLYGON(
			(-0.5 7, -1 5, 3.5 5.25, -0.5 7),
			CIRCULARSTRING(0.25 5.5, -0.25 6.5, -0.5 5.75, 0 5.75, 0.25 5.5))', 4326));
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/037eae7ee7754952a052bf251dbc4ba4.gif#pic_center)

# 输入与输出格式

OGC SFA SQL规范的WKB和WKT,提供了基于geom转二进制或文本的I/O能力，但其只定义了2D几何图形，并且相关的SRID没有嵌入在I/O的表示中。

因此，PostGIS 扩展了EWKB/EWKT，在OGC的基础上，增加了对3DM坐标、3DZ坐标、4D坐标和嵌入式SRID信息的支持。

数据格式间的转换通过以下接口实现：

- *bytea EWKB = ST_AsEWKB(geometry);*
- *text EWKT = ST_AsEWKT(geometry);*
- *geometry = ST_GeomFromEWKB(bytea EWKB);*
- *geometry = ST_GeomFromEWKT(text EWKT);*

此外，PostGIS还支持了多种其他常用格式的互转，满足多场景需求：

- Geographic Mark-up Language (GML)
- Keyhole Mark-up Language (KML)
- GeoJSON、Scalable Vector Graphics (SVG)
- …