- [PostGIS测试 - geometry_columns_万里归来少年心的博客-CSDN博客_geometry_columns](https://blog.csdn.net/liyazhen2011/article/details/89101902)

创建空间数据库后，会默认生成数据表geometry_columns表，它存放了当前数据库中所有表的几何字段信息。用工具pgAdmin查看该表。

![img](https://img-blog.csdnimg.cn/20190408185221798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeWF6aGVuMjAxMQ==,size_16,color_FFFFFF,t_70)

  该表各列的含义如下：

- f_table_catalog表示数据库名。
- f_table_schema表示空间表所在的模式。
- f_table_name表示空间表的表名。
- f_geometry_column表示空间表中几何字段的名称。
- coord_dimension表示几何字段维数。
- srid表示空间表的空间参考。
- type表示几何字段的类型。