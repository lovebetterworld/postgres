- [PostGIS使用2：如何利用PostGIS正确计算距离和面积](https://www.jianshu.com/p/be5049ad8884)
- [PostGIS使用3：PostGIS功能简介（1.概要+空间数据类型）](https://www.jianshu.com/p/9ed3b4dcd32a)



# 一、计算距离和范围的思路

其实计算距离和面积属于典型的**Measurements（测量）**需求，PostGIS也是把相应的函数划分在了Spatial Relationships and Measurements这个板块里。

**ps：**PostGIS的reference：[http://postgis.net/docs/manual-2.3/reference.html](https://link.jianshu.com?t=http%3A%2F%2Fpostgis.net%2Fdocs%2Fmanual-2.3%2Freference.html)

**测量**是一门古老且历史悠久的技术，最早的记录在公元前两千多年大禹治水的时候就有了，他本身跟计算机和互联网没什么关系，思路大体上是：

**我要测量的范围大概有多大？->我要在哪个地方测量？->我要测量多准？**

比较形象的比喻就是我在量一个东西的时候需要根据东西的大小和位置先找把合适的尺子，然后再开始测量。

弄清以上三点选择合适的测量方式（尺子）再进行测量（计算）是我个人推荐的方式。

# 二、在这就需要提一个组织

**EPSG**（[http://www.epsg.org](https://link.jianshu.com?t=http%3A%2F%2Fwww.epsg.org)）
**EPSG**:European Petroleum Survey Group，欧洲石油调查组织(你没看错)，该组织负责专门维护地球上所有的测量坐标系统（找石油），并且给每组坐标系统都赋予了一个编号和一组描述（WKT），比如大家常用的WGS84坐标系编号就是EPSG:4326，再比如互联网地图（谷歌、高德等）常用的伪墨卡托投影编号就是EPSG:3857。



可以理解成EPSG给大家维护了无数把尺子，并且给每把尺子搞了个编号，还标明了这把尺子适合什么条件下用。

**举三个例子帮助大家理解：**
 先推荐一个查EPSG坐标系非常棒的网站：[epsg.io](https://link.jianshu.com?t=http%3A%2F%2Fepsg.io%2F)(需要翻墙)

## 2.1 EPSG:4326

 直接在浏览器里敲：[http://epsg.io/4326](https://link.jianshu.com?t=http%3A%2F%2Fepsg.io%2F4326)(注意要翻墙)，进去你会看到大大的几行字：
 **EPSG:4326
 Geodectic coordinate system
 WGS 84 -- WGS84 - World Geodetic System 1984, used in GPS**  
 翻译过来就是：EPSG:4326，一个**地理坐标系（也叫大地坐标系）**，用于描述WGS84坐标系，其是从1984年开始在GPS中使用的全球地理坐标系统。

 再下面是Attributes、Covered area、Export三个板块：
 **Attributes**中先重点看下**Uint**（单位），这里写的是degree（度），CRS、Ellipsoid等参数看不懂没关系，以后文章会单独讲
 **Corverd area**是重点要看的：

![img](https://upload-images.jianshu.io/upload_images/12078747-a099ac6ab49cd973.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/570)

看到红色的框框没，这是指该坐标系适合计算的范围，几乎覆盖了全球，再看下面的三条描述：
**Center coordinates**，指坐标系的中心点坐标
**WGS84 bounds**，指坐标系的适用范围
**World**，清晰的告诉你这是给全球范围用的坐标系



所以问题来了：**这个坐标系适合计算距离和面积吗？**
 答案是：**否**
 因为该坐标系是大地坐标系（Geodectic coordinate system），单位是度（角度单位），角度用来测量长度和面积是不合适的（尺子不好用啊），但可用于定位，而且它的范围又覆盖了全球，所以很适合全球定位（GPS卫星定位的坐标系用的就是它）

再看一个坐标系

## 2.2 EPSG:3857

 点开[http://epsg.io/3857](https://link.jianshu.com?t=http%3A%2F%2Fepsg.io%2F3857)可以看到：
 **EPSG:3857
 Projected coordinate system
 WGS 84 / Pseudo-Mercator -- Spherical Mercator, Google Maps, OpenStreetMap, Bing, ArcGIS, ESRI**
 意思就是，EPSG:3857是一个**投影坐标系（Projected coordinate system）**，在WGS84坐标系基础上进行了伪墨卡托投影（Pseudo-Mercator）。球形墨卡托地图、谷歌地图、osm地图、bing地图、ArcGIS、ESRI会常用该坐标系。

**投影坐标系（Projected coordinate system）**是在大地坐标系（Geodectic coordinate system）的基础上，经过数学运算，把大地坐标系的曲面坐标映射到平面上产生的一种平面坐标系。

不太严谨的理解就是你有个橘子，没剥开的时候在橘子皮上画的画就是大地坐标系，包开以后把橘子皮拍平了上面画的就是投影坐标系。



再看下**Unit**（单位）是metre（米）
 **Covered area**的**WGS84 bounds**是-180，-85.06到180，85.06，描述是**World between 85.06°S and 85.06°N.**，嗯。。。几乎覆盖整个地球了，感觉用它来计算距离和面积妥妥的。

但很遗憾答案还是：**否，用它算测不了实地距离** >3<

虽然EPSG:3857是平面坐标系，单位是长度（米），但是他用了一个长度和面积都不靠谱的投影坐标系：Pseudo-Mercator（伪墨卡托投影，该投影是正轴等角切圆柱投影，在高纬度地区形变的非常严重）。看到这你貌似突然知道了一个惊天秘密：原来高德地图和谷歌地图上面画的东东都是变形的！！没错，就是变形的，你看到的这些互联网地图用的都是类似的投影，他们在高纬度地区都是拉伸严重的，远比他的实地面积要大。但是正轴墨卡托投影有个优点：投影后角度不变形，所以用来导航和定位非常合适。

这个例子还反应了一件事：能几乎覆盖全球的坐标系一般都算不了正确的距离和面积（没有一个坐标系能完美解决地球上的所有问题），想算正确的距离？找个小点的范围吧。

在这直接扔一个北京市可以用的：

## 2.3 EPSG:4527

 点开[http://epsg.io/4527](https://link.jianshu.com?t=http%3A%2F%2Fepsg.io%2F4527)可以看到：
 **EPSG:4527
 Projected coordinate system
 CGCS2000 / 3-degree Gauss-Kruger zone 39**
 翻译过来就是：EPSG:4527，是投影坐标系，基于国家2000大地坐标系做的高斯-克吕格3度带投影中的第39带坐标系（看不懂没关系，高斯克吕格以后会慢慢讲，知道它形变很小就好了）

 其中**国家2000（CGCS2000）**指的就是咱中国自己的大地坐标系（还有两个比较旧的：北京54、西安80），**Unit**是米，重点看**Covered area：**

![img](https://upload-images.jianshu.io/upload_images/12078747-998a447ae5a37548.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/530)

红色就是该投影最适合使用的范围，**WGS84 bounds:**115.5 22.6到118.5，49.88，正好把北京市（116.46，39.92）包括在内，下面的描述也很清楚：**China - onshore between 115°30'E and 118°30'E.**（适合中国内陆东经115°30'到东经118°30'的范围），跟图上红框区域完全匹配有木有，恭喜你找到了一把测量北京市距离和面积的好尺子。

# 三、PostGIS算北京市范围内的距离和面积

坐标系（尺子）有了，就差运算(测量)了：

## 3.1 算距离

在北京选Mobike总部曼宁国际北门这面墙的两个端点试验：

![img](https://upload-images.jianshu.io/upload_images/12078747-3d017e49e8f07c71.png?imageMogr2/auto-orient/strip|imageView2/2/w/534)

左边点是：经度116.4677961543，纬度39.9486461337
 右边点是：经度116.4680989087，纬度39.9486998528
 他们的坐标系都是WGS84大地坐标系，也就是EPSG:4326。

执行查询:

```plsql
SELECT st_distance(st_transform(st_geometryfromtext('POINT(116.4677961543 39.9486461337)',4326),4527),st_transform(st_geometryfromtext('POINT(116.4680989087 39.9486998528)',4326),4527));
```

结果是：26.55米

```sql
st_distance
------------------
 26.5520226594034
(1 row)
```

其中用到几个函数：
 **st_geometryfromtext（geometry，srid）**：该方法作用是根据描述的几何对象（geometry）的字符串转化成几何对象，POINT说明几何对象是点类型，第二个参数srid是4326，是指这个点类型对象的空间参考（也就是EPSG的编号，也是所在的坐标系）是EPSG:4326,即WGS84大地坐标系。

**st_transform（geometry，srid）**：该方法是把某个几何对象（geometry）的所有坐标从一个坐标系转换到另一个坐标系。在这做的就是把EPSG:4326转换为EPSG:4527（量北京的尺子到手）。

**st_distance（geometry，geometry）**：该方法用于计算两点距离，所用坐标系根据geometry带的srid（EPSG编号）决定。

**上述查询的含义就是：**
 两个4236坐标系下的点对象转换成4527坐标系后计算直线距离，这个距离与地面实际距离很接近。

再用google地图上瞄着位置手动量了下：结果差不多，26.52米

![img](https://upload-images.jianshu.io/upload_images/12078747-1ef9f4962d21f6a8.png?imageMogr2/auto-orient/strip|imageView2/2/w/602)

如果用3857试试：

```plsql
SELECT st_distance(st_transform(st_geometryfromtext('POINT(116.4677961543 39.9486461337)',4326),3857),st_transform(st_geometryfromtext('POINT(116.4680989087 39.9486998528)',4326),3857));
```

结果是：34.59米，距离明显变长了，结果错误

```sql
   st_distance
------------------
 34.5933990100333
(1 row)
```

## 3.2 算面积

同样还是曼宁国际选了它前面停车场的矩形范围的四个端点：
 116.4679312706，39.9482801227
 116.4677961543，39.9486461337
 116.4680989087，39.9486998528
 116.4682182670，39.9483181633

执行查询:

```plsql
SELECT st_area(ST_Transform(st_geometryfromtext('POLYGON((116.4679312706 39.9482801227,116.4677961543 39.9486461337,116.4680989087 39.9486998528,116.4682182670 39.9483181633,116.4679312706 39.9482801227))',4326),4527));
```

结果：1100平方米约等于1.65亩

```sql
     st_area
------------------
 1101.47601530779
(1 row)
```

这里geometry的类型用的是POLYGON（多边形），注意要闭合（首尾的经纬度要一样）

**st_area(geometry,srid)**:该函数用于计算在某个srid（某个坐标系下）一个几何对象的面积

**上述查询的含义就是：**
 一个4236坐标系下的多边形对象转换成4527坐标系后计算其面积，这个面积与地面实际实际面积很接近。

**可能有人会好奇PostGIS是怎么知道这些编号的？**
 请执行下面语句：

```plsql
SELECT * FROM spatial_ref_sys WHERE srid IN (4326,3857,4257);
```



结果：

![img](https:////upload-images.jianshu.io/upload_images/12078747-b8c3f787e7fea53e.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1096)

看到没，PostGIS自带的这张表spatial_ref_sys（空间参考系统）包含了几乎EPSG所有坐标系编号定义和描述，你甚至可以自己定义一套坐标系统放到里面（只要你会写WKT和proj4text）。

# 四、总结

计算地物距离先要预估计算的范围和大小，选择合适的坐标系（空间参考），坐标系分为：大地坐标和投影坐标，而投影坐标是在大地坐标基础之上投影而来，常用的坐标系在EPSG上都有分组和编号，而这一切PostGIS都支持，想怎么算都可以。





