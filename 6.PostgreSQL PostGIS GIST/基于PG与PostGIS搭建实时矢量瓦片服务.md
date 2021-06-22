- [基于PG与PostGIS搭建实时矢量瓦片服务](https://blog.csdn.net/qq_35241223/article/details/106439268)



# 一、矢量瓦片（MVT）

本文中提到的矢量瓦片为Mapbox Vector Tile格式，简称MVT。

## 1.1 MVT标准

MVT标准参考Mapbox官方文档。

[传送门链接](https://github.com/mapbox/vector-tile-spec)

## 1.2 矢量瓦片优势

- 可以支持高分辨率屏幕的显示
- 地图渲染在前端，可以支持一套数据随意更改配图方案，解决传统栅格瓦片动态样式上的问题
- 要素查询可以在前端进行
- 保持了矢量数据的优势，同时采用切片方式又提高了传输上的效率

## 1.3 实时矢量瓦片

顾明思议，矢量瓦片不在使用工具线下进行预先切片，采用即时浏览即时传输矢量瓦片。



### 1.3.1 为什么要有实时的矢量瓦片

采用实时切的矢量瓦片可以做到数据编辑功能。如果采用预先切片方式，那么在数据编辑后，浏览的数据还是旧的数据，做到实时的矢量瓦片后，编辑数据之后浏览到的矢量瓦片就是最新数据。这样就可以解决数据编辑后瓦片不是最新的问题。



# 二、PostGIS中矢量切片相关函数

本文介绍使用PG+PostGIS来做到实时的矢量瓦片，在实战之前先介绍几个用到的矢量瓦片相关的函数。

## 2.1 主要函数：

- ST_AsMvtGeom 将Geom转化为MVT的geom
- ST_AsMVT 将geom转换为MVT数据
- ST_TileEnvelope(3.0以上支持） 根据行列号获取Envelope

## 2.2 辅助函数：

- ST_Transform 坐标转换函数，用它可以做到支持任何坐标系的矢量瓦片
- ST_Simplify 简化，用它来做线或者面的简化
- ST_SimplifyPreserveTopology 与简化类似

# 三、实战

思路：

- 前端地图库浏览时，将请求URL传到后台
- 后端实现URL的服务，根据x、y、z获取对应范围数据，使用sql查询到数据返回给前端

## 3.1 只有点层的SQL语句

![点的矢量切片](https://img-blog.csdnimg.cn/20200531151106248.png)

![效果](https://img-blog.csdnimg.cn/20200531151128340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)


## 3.2 点、线、面层在一起的SQL

![点线面](https://img-blog.csdnimg.cn/20200531151146153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

![点线面效果](https://img-blog.csdnimg.cn/2020053115120053.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)