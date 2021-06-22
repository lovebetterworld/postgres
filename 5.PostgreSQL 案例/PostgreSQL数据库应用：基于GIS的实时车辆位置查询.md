- [PostgreSQL数据库应用：基于GIS的实时车辆位置查询](https://blog.csdn.net/m0_37657841/article/details/88636605?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.nonecase)
- https://github.com/KiroScarlet/CarLoc

# 一、目的

近年来，随着我国城市化进程的加快，不少城市的规模在扩大、人口在增加、道路在延伸，城市的公交线路也在不断地增加。在城市中，选择合适的公交车前往目的地就成为与广大普通市民出行密切相关的一个问题。地理信息系统(GIS)作为一门融计算机图形和数据库于一体，储存和处理空间信息的边缘综合性学科，能把地理位置和相关属性有机结合起来，根据实际需要准确真实、图文并茂地输出给用户，满足不同部门、不同用户对空间信息的要求，并借助其空间分析能力和可视化表达，用于各种辅助决策。

PostGIS是对象关系型数据库系统PostgreSQL的一个扩展，PostGIS提供如下空间信息服务功能:空间对象、空间索引、空间操作函数和空间操作符。同时，PostGIS遵循OpenGIS的规范。

由此，我们通过使用PostGIS来存储公交车位置信息，并借助socket.io来实现在谷歌地图上实时显示公交车位置，借此来展示PostgreSQL的特性，为用户出行节省时间。

# 二、内容与设计思想

## 1.为什么选择postgreSQL而不用MySQL？

PostGIS的GIS功能相比MySQL强大太多，PostGIS有几百个操作函数, 对GIS支持强大。PostGIS为 PostgreSQL 提供了存储、查询和修改空间关系的能力,如空间对象、空间索引、空间操作函数和空间操作符等。PostGIS最大的特点是符合并且实现了OpenGIS的一些规范，是最著名的开源GIS数据库。

## 2.实现公交车位置实时更新功能
node.js一直以异步io著称，其语言特性尤其擅长于在realtime应用中。本系统主要需要实时更新插入等功能，而nodejs+socket.io在实时应用中具有较好的表现能力。本实验中，PostgreSQL通过触发器监听是否有数据采集终端将新坐标写入或者更新，然后在触发器中notify消息，数据库一旦广播了消息，服务器端监听，并继续以socke.io广播到客户端实时展示。

## 3.用户查询公交车位置

用户可以通过指定公交线路查询，数据库接收到用户请求，只返回用户需求的那些公交车，然后动态更新。

## 4.通过聚类算法帮助用户更好选择公交线路

对数据库中的所有车辆进行聚类，按照每类中车辆的数量区分等级，如果用户查询的公交车等级靠前，说明此辆公交的附近车辆较多，有堵车的可能，推荐用户选择其他公交线路。

# 三、具体设计

## 3.1 数据库端

### 3.1.1 建立数据表，用来存储车辆的坐标点

![img](https://img-blog.csdnimg.cn/20190318124253545.png)

### 3.1.2 对表的增删改建立一个触发器，触发器中发送变化数据出去

![img](https://img-blog.csdnimg.cn/20190318124349102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

## 3.2 服务器端和客户端

创建一个项目，项目中包含两个文件，分别是客户端文件index.html和服务端文件index.js。如图所示：

![img](https://img-blog.csdnimg.cn/20190318124447427.png)

服务端文件负责连接数据库，不断监听数据库的gps的消息，数据库一旦获得通知，将通知消息通过socket.io发送到各个客户端展示。

客户端是一个html文档，客户端接收服务端发送的消息，根据数据库的消息是INSERT，UPDATE还是DELETE确定在地图中新增点、修改已有点还是删除点。通过浏览器访问本地端口localhost：8081，可以在谷歌地图中看到数据库中动态更新的所有点。

## 3.3 实现效果展示

1.当系统的管理员的插入当前车辆的信息，车辆信息可以显示在谷歌地图上面，如下图所示：

![img](https://img-blog.csdnimg.cn/20190318124525797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

2.系统管理员可以获取车辆的位置，并且实时更新当前车辆的位置。

```plsql
update t_gps set geom=st_geomfromtext('Point(121.406317 31.261377)',4326) where id=1;
```

![img](https://img-blog.csdnimg.cn/20190318124602729.png)

更新前：

![img](https://img-blog.csdnimg.cn/20190318124657801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

更新后：

![img](https://img-blog.csdnimg.cn/20190318124716914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

## 3.4 聚类分析

对所有车辆的GPS坐标进行聚类，类簇中坐标点的数目可以代表车辆密集的程度。这样就可以根据用户输入的车辆信息定位到该车辆位置所属的类簇，进而判断该类簇的拥挤程度从而给出合理的出行建议。

我们对给定数据用k-means聚成k类，根据k个类簇中车辆的数目决定每个类簇的拥塞等级，在这里我们将拥塞程度按照大小分为3个等级，分别是“拥堵”，“一般”和“畅通”。每个等级设置一个最小阈值，这样在用户输入车辆id后我们可以返回车辆所属的拥塞等级。

### 3.4.1 数据输入

从数据表中读出所有车辆位置坐标。

![img](https://img-blog.csdnimg.cn/20190318124825999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

### 3.4.2 聚类处理

使用sklearn 的 kmeans方法聚类，根据sklearn.inertia_ 属性大小调节类簇数量，为每个类簇打上[0,k]范围内的标签。

### 3.4.3 数据输出

在数据库中创建表t-cluster,创建两个属性kinds和坐标，每个坐标点对应一个类簇。将聚类处理的结果根据阈值给定所处等级，然后存储到数据库中。

 ![img](https://img-blog.csdnimg.cn/20190318124920894.png)

![img](https://img-blog.csdnimg.cn/20190318124939678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

### 3.4.4 使用聚类结果

当用户查询公交车时，根据用户查询的公交车id在第一张表中查询出车辆坐标。

然后对当时的所有车辆进行聚类，更新数据库中的表t-cluster。

通过表t-cluster查询到车辆对应的等级。

![img](https://img-blog.csdnimg.cn/20190318125017511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjU3ODQx,size_16,color_FFFFFF,t_70)

如果返回值是3，说明此公交车所处位置为拥堵状态，建议用户重新选择出行方案。

# 四、总结

- 数据库表的设计

    在设计聚类分析表的时候，考虑到聚类分析的分类结果，所以不将分类结果设置成主键形式。

- 索引的建立

    当数据库的记录增大的时候，如果没有建立索引的话，操作的效率就显著下降。
    PostGIS建议当记录数超过几千的时候就应该建立索引，而GIS数据库一般都是海量数据，所以对PostGIS而言，索引就非常重要，所以为了应对实时性能，对数据库建立索引。