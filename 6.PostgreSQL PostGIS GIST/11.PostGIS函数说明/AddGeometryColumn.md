# AddGeometryColumn

```plsql
text AddGeometryColumn(varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean use_typmod=true);

text AddGeometryColumn(varchar schema_name, varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean use_typmod=true);

text AddGeometryColumn(varchar catalog_name, varchar schema_name, varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean use_typmod=true);
```

# 描述

将几何列添加到现有属性表中。schema_name是表模式的名称。srid必须是对SPATIAL_REF_SYS表项的整数值引用。type必须是一个对应于几何类型的字符串，如'POLYGON'或'MULTILINESTRING'。如果schemaname不存在(或在当前search_path中不可见)或指定的SRID、几何类型或维度无效，则会抛出错误。



**Notes：**2.0.0这个函数不再更新geometry_columns，因为geometry_columns是一个从系统目录读取的视图。默认情况下，它也不创建约束，而是使用PostgreSQL内置的类型修饰符行为。例如，用这个函数构建一个wgs84 POINT列，现在等价于:

```plsql
ALTER TABLE some_table ADD COLUMN geom geometry(Point,4326);
```

更改:2.0.0如果你需要约束的旧行为，使用默认的use_typmod，但将其设置为false。



**Notes：**更改:2.0.0视图不能再手动注册几何列，但视图构建的几何typmod表几何和使用没有包装器函数将正确注册自己，因为他们继承了其父表列的typmod行为。使用输出其他几何图形的几何函数的视图需要转换为typmod几何图形，以便这些视图几何列在geometry_columns中正确注册。

# 示例：

```plsql
-- Create schema to hold data
CREATE SCHEMA my_schema;
-- Create a new simple PostgreSQL table
CREATE TABLE my_schema.my_spatial_table (id serial);

-- Describing the table shows a simple table with a single "id" column.
postgis=# \d my_schema.my_spatial_table
							 Table "my_schema.my_spatial_table"
 Column |  Type   |                                Modifiers
--------+---------+-------------------------------------------------------------------------
 id     | integer | not null default nextval('my_schema.my_spatial_table_id_seq'::regclass)

-- Add a spatial column to the table
SELECT AddGeometryColumn ('my_schema','my_spatial_table','geom',4326,'POINT',2);

-- Add a point using the old constraint based behavior
SELECT AddGeometryColumn ('my_schema','my_spatial_table','geom_c',4326,'POINT',2, false);

-- Add a curvepolygon using old constraint behavior
SELECT AddGeometryColumn ('my_schema','my_spatial_table','geomcp_c',4326,'CURVEPOLYGON',2, false);

-- Describe the table again reveals the addition of a new geometry columns.
\d my_schema.my_spatial_table
                            addgeometrycolumn
-------------------------------------------------------------------------
 my_schema.my_spatial_table.geomcp_c SRID:4326 TYPE:CURVEPOLYGON DIMS:2
(1 row)

                                    Table "my_schema.my_spatial_table"
  Column  |         Type         |                                Modifiers
----------+----------------------+-------------------------------------------------------------------------
 id       | integer              | not null default nextval('my_schema.my_spatial_table_id_seq'::regclass)
 geom     | geometry(Point,4326) |
 geom_c   | geometry             |
 geomcp_c | geometry             |
Check constraints:
    "enforce_dims_geom_c" CHECK (st_ndims(geom_c) = 2)
    "enforce_dims_geomcp_c" CHECK (st_ndims(geomcp_c) = 2)
    "enforce_geotype_geom_c" CHECK (geometrytype(geom_c) = 'POINT'::text OR geom_c IS NULL)
    "enforce_geotype_geomcp_c" CHECK (geometrytype(geomcp_c) = 'CURVEPOLYGON'::text OR geomcp_c IS NULL)
    "enforce_srid_geom_c" CHECK (st_srid(geom_c) = 4326)
    "enforce_srid_geomcp_c" CHECK (st_srid(geomcp_c) = 4326)

-- geometry_columns view also registers the new columns --
SELECT f_geometry_column As col_name, type, srid, coord_dimension As ndims
    FROM geometry_columns
    WHERE f_table_name = 'my_spatial_table' AND f_table_schema = 'my_schema';

 col_name |     type     | srid | ndims
----------+--------------+------+-------
 geom     | Point        | 4326 |     2
 geom_c   | Point        | 4326 |     2
 geomcp_c | CurvePolygon | 4326 |     2
```

