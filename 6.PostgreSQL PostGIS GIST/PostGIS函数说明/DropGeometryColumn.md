# DropGeometryColumn

```plsql
boolean DropGeometryTable(varchar table_name);

boolean DropGeometryTable(varchar schema_name, varchar table_name);

boolean DropGeometryTable(varchar catalog_name, varchar schema_name, varchar table_name);
```

## 描述

删除一个表及其在geometry_columns中的所有引用。

**注意：**如果没有提供模式，则在支持模式的pgsql安装上使用current_schema()。



**更改：**2.0.0提供此函数是为了向后兼容。既然geometry_columns现在是针对系统编目的视图，那么您可以像使用drop table一样删除具有几何列的表。

## 示例

```plsql
SELECT DropGeometryTable ('my_schema','my_spatial_table');
----RESULT output ---
my_schema.my_spatial_table dropped.

-- The above is now equivalent to --
DROP TABLE my_schema.my_spatial_table;
```

# DropGeometryColumn

```plsql
text DropGeometryColumn(varchar table_name, varchar column_name);

text DropGeometryColumn(varchar schema_name, varchar table_name, varchar column_name);

text DropGeometryColumn(varchar catalog_name, varchar schema_name, varchar table_name, varchar column_name);
```

## 描述

从空间表中删除几何列。注意，schema_name将需要匹配几何列表中表行的f_table_schema字段。

**更改：**2.0.0提供此函数是为了向后兼容。既然geometry_columns现在是系统编目的一个视图，那么您可以使用ALTER TABLE删除一个几何列，就像删除任何其他表列一样

## 示例

```plsql
SELECT DropGeometryColumn ('my_schema','my_spatial_table','geom');
			----RESULT output ---
			                  dropgeometrycolumn
------------------------------------------------------
 my_schema.my_spatial_table.geom effectively removed.

-- In PostGIS 2.0+ the above is also equivalent to the standard
-- the standard alter table.  Both will deregister from geometry_columns
ALTER TABLE my_schema.my_spatial_table DROP column geom;
```





