- [postgresql 创建gis空间数据库，shp数据入库](https://blog.csdn.net/yangniceyangyang/article/details/104047479)



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

参考博文：

https://blog.csdn.net/gis_zzu/article/details/91045052

https://www.jianshu.com/p/2c4f714c62b5