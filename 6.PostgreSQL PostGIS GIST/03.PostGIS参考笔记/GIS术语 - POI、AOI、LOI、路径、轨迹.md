# GIS术语 - POI、AOI、LOI、路径、轨迹

**作者**

digoal

**日期**

2017-12-04

**标签**

PostgreSQL , PostGIS , 空间数据库 , POI , AOI , LOI , 轨迹 , 路径 , pgrouting , ltree , geojson , geoarray , 全文检索

# 背景

# 一、poi

Point of interesting. 可以翻译为兴趣点，就是在地图上任何非地理意义的有意义的点：比如商店，酒吧，加油站，医院，车站等。不属于poi的是有地理意义的坐标：城市，河流，山峰

兴趣点（POI，Point of interest），兴趣点是我们就是导航软件商收录的信息点，比如说XX大厦，XX餐厅等特定的为位置，通过POI我们可以直接在导航软件上查找相应的目的地。

在导航软件上如果收录的POI不完整、不准确或者说未收录，那么我们就会发现我们很难开始导航，因为我们无法在导航软件上找到我们想要去的地方。所以说POI是我们在操作导航仪时首先会遇到的数据库，就好比在字典中查找一个我们未知的词语一样，对于能否得到答案起着至关重要的作用。

在实际应用的过程中，我们一般是直接输入一个地名，然后进行查找，在得到了目的地的具体位置的时候，用导航仪对我们的行车方向进行指引，然后完成导航的全部过程。

# 二、aoi

area of interest (AOI)

the geographic bounds of the map sheet

可以理解为AOI个多边形。

# 三、loi

location of interest (LOI)

兴趣位置(LOI)。LOI可以是多边形、点或多点要素、或者是多个多边形。

# 四、路径

这个很好理解，指规划好的有先后顺序的路径。常见于导航。

# 五、轨迹

实际走过的路径。由多个有时序的点组成。

# 六、对应PostgreSQL中的空间特性

1、POI，指中文描述+经纬度。正反都可以查询。使用空间数据库PostGIS存储经纬度，使用全文检索、模糊查询、拼音检索、首字母检索等 PostgreSQL 技术，搜索中文描述。

[《HTAP数据库 PostgreSQL 场景与性能测试之 6 - (OLTP) 空间应用 - KNN查询（搜索附近对象，由近到远排序输出）》](https://github.com/digoal/blog/blob/master/201711/20171107_07.md)

[《HTAP数据库 PostgreSQL 场景与性能测试之 5 - (OLTP) 空间应用 - 空间包含查询》](https://github.com/digoal/blog/blob/master/201711/20171107_06.md)

[《在PostgreSQL中实现按拼音、汉字、拼音首字母搜索的例子》](https://github.com/digoal/blog/blob/master/201611/20161109_01.md)

[《HTAP数据库 PostgreSQL 场景与性能测试之 14 - (OLTP) 字符串搜索 - 全文检索》](https://github.com/digoal/blog/blob/master/201711/20171107_15.md)

[《HTAP数据库 PostgreSQL 场景与性能测试之 7 - (OLTP) 全文检索 - 含索引实时写入》](https://github.com/digoal/blog/blob/master/201711/20171107_08.md)

[《多国语言字符串的加密、全文检索、模糊查询的支持》](https://github.com/digoal/blog/blob/master/201710/20171020_01.md)

[《PostgreSQL 模糊查询最佳实践》](https://github.com/digoal/blog/blob/master/201704/20170426_01.md)

[《PostgreSQL 全表 全字段 模糊查询的毫秒级高效实现 - 搜索引擎颤抖了》](https://github.com/digoal/blog/blob/master/201701/20170106_04.md)

[《从难缠的模糊查询聊开 - PostgreSQL独门绝招之一 GIN , GiST , SP-GiST , RUM 索引原理与技术背景》](https://github.com/digoal/blog/blob/master/201612/20161231_01.md)

2、AOI

PostGIS中的geometry类型，可以存储2d, 3d, 4d点、线、线段、面、集合。

http://postgis.net/docs/manual-2.4/reference.html

3、LOI

PostGIS中的geometry类型，可以存储2d, 3d, 4d点、线、线段、面、集合。

http://postgis.net/docs/manual-2.4/reference.html

4、路径，可以使用pgrouting生成，根据内置的图搜索算法。

http://pgrouting.org/

5、轨迹，指实际运行生成的轨迹，可以存储为多条记录，也可以存储为一个 线段、GEO 数组、GEO JSON、HSTORE（K-V）、文本等。

http://postgis.net/docs/manual-2.4/ST_LinestringFromWKB.html

# 参考

http://gps.zol.com.cn/243/2436644.html

https://en.wikipedia.org/wiki/Point_of_interest

https://pro.arcgis.com/en/pro-app/help/workflow-manager/specifying-an-area-of-interest-aoi.htm

http://webhelp.esri.com/arcgisdesktop/9.3/index.cfm?TopicName=The_Area_of_Interest_(AOI)_tool

http://zhihu.esrichina.com.cn/article/562