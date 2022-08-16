- [postgresql更改geometry类型列的srid_lwdbcy的博客-CSDN博客](https://blog.csdn.net/lwd18175239125/article/details/126283586)

## 1 问题描述

有一张表包含空间信息列`geom` ，它是地理坐标系（`WGS84` ），srid为4326，现在我不小心向其中插入了一批投影坐标系（CGCS2000）的数据，我要如何修正呢？

## 2 主要使用函数

### 2.1 ST_SetSRID(geom,srid)

> 该函数返回一个与输入几何体相同的几何体，只不过使用空间参考系统标识符 (SRID) 的输入值进行了更新。

就是说对几何体**geom** 设置了空间参考

### 2.2 ST_Transform(geom, srid)

> ST_Transform 返回一个新的几何体，坐标在由输入空间参考系统标识符（SRID）定义的空间参考系统中转换。

看名字就知道是转换，将**geom**转换成指定空间参考下的几何体

## 3 解决步骤

解决这个问题关键就是知道上面的两个函数，其他挺简单的

1. 查询`CGCS2000 / 3-degree Gauss-Kruger CM 120E` 的EPSG代码为**4549**
2. 更新数据

```sql
UPDATE table_name SET geom=ST_TRANSFORM(ST_SETSRID(geom, 4549), 4326) 
WHERE xxxxx
```