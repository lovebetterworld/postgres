- [ST_Transform (postgis.net)](https://postgis.net/docs/ST_Transform.html)

## Synopsis

`geometry ST_Transform(`geometry g1, integer srid`)`;

`geometry ST_Transform(`geometry geom, text to_proj`)`;

`geometry ST_Transform(`geometry geom, text from_proj, text to_proj`)`;

`geometry ST_Transform(`geometry geom, text from_proj, integer to_srid`)`;

## Description

返回一个新的几何图形，其坐标转换为不同的空间参考系统。指向_srid的目标空间引用可以通过一个有效的SRID整数参数来标识(即它必须存在于spatial_ref_sys表中)。或者，定义为project .4字符串的空间引用可以用于to_proj和/或from_proj，但是这些方法没有优化。如果目标空间引用系统用project .4字符串而不是SRID表示，那么输出几何图形的SRID将被设为零。除了使用from_proj的函数外，输入几何图形必须有一个定义好的SRID。

ST_Transform经常与ST_SetSRID混淆。**ST_Transform实际上是将几何图形的坐标从一个空间引用系统更改为另一个空间引用系统**，而ST_SetSRID()只是更改几何图形的SRID标识符。

- 需要PROJ支持PostGIS编译。使用PostGIS_Full_Version来确认你已经编译了PROJ支持。
- 如果使用不止一个转换，在常用转换上有一个函数索引是很有用的，以便利用索引的使用。