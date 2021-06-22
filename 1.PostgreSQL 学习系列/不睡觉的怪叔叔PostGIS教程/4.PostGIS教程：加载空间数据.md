# PostGIS教程四：加载空间数据

在各种库和应用程序的支持下，PostGIS提供了许多用于加载数据的选项。

  本节将重点介绍使用**PostGIS shapefile加载工具**加载shapefile的基础知识。

# 一、PostGIS shapefile工具

## 1.1、首先，返回到选项板，并单击PostGIS部分中的PostGIS shapefile工具，PostGIS shapefile工具将启动

![img](https://img-blog.csdnimg.cn/20181224085759962.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181224085818608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.2、填写PostGIS连接部分的连接详细信息，然后单击“ok”按钮。程序将测试连接并在日志窗口中报告。

如果安装时使用默认的信息，就如下所示：

![img](https://img-blog.csdnimg.cn/20181224090211670.png)

![img](https://img-blog.csdnimg.cn/20181224090244825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2018122409032230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.3、接下来，打开“**Add File**”按钮并导航到数据目录文件（[数据下载地址](http://s3.cleverelephant.ca/postgis-workshop-2018.zip)）：\postgis-workshop-2018\data。选择nyc-census_block.shp文件。

![img](https://img-blog.csdnimg.cn/20181224090828424.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181224090924978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.4、将文件的的SRID（空间参考信息）**值更改为**26918

请注意，架构、表名和列名已经根据shapefile文件里的信息填充，但是你可以有选择地更改它们（不要这样做！在教程后面部分，还有一些步骤需要默认的名称）。

![img](https://img-blog.csdnimg.cn/20181224091621316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.5、保持配置部分的详细信息如下

![img](https://img-blog.csdnimg.cn/20181224091852123.png)

![img](https://img-blog.csdnimg.cn/20181224091906160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.6、单击"**Options**"按钮查看加载选项。加载程序将使用快速“**COPY**（复制）"模式，并在加载数据后默认创建**空间索引**。

![img](https://img-blog.csdnimg.cn/20181224092150764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181224092359582.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.7、最后，单击"**Import**"按钮并观察导入过程。这可能需要几分钟的时间来加载，但这是教程数据中最大的一个文件。

![img](https://img-blog.csdnimg.cn/20181224092549864.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.8、重复数据目录中其余shapefile文件的导入。通过在按"**Import**"按钮之前添加多个文件，可以在一次导入中加载多个文件：

![img](https://img-blog.csdnimg.cn/20181224092913393.png)

## 1.9、加载所有文件后，单击pgAdmin中的"**Refresh**"按钮更新树状视图。应该可以看到6个表显示在树结构：**数据库>nyc>架构>public>数据表**里。

![img](https://img-blog.csdnimg.cn/20181224093618731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181224093701916.png)

# 二、什么是shapefile？

  你可能会问自己 —— "shapefile是什么？"。

  一个"shapefile"通常指带有.shp、.shx、.dbf和其他扩展名且前缀名称一致的文件集合。

  例如上面的nyc_census_blocks由以下几个文件组成：

![img](https://img-blog.csdnimg.cn/20190214091515461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  一个shapfile必需有的文件：

- .shp  ——  存储地理要素的几何信息
- .shx  ——  存储要素几何图形的索引信息
- .dbf  ——  存储地理要素的属性信息（非几何信息）

  可选文件包括：

- .prj  ——  存储空间参考信息，即地理坐标系统信息和投影坐标系统信息。使用well-known文本格式进行描述。

  PostGIS shapefile工具将shapefile数据从二进制转换为一系列的SQL命令，然后在数据库中运行以加载数据，从而使shapefile数据在PostGIS中可用。

# 三、什么是SRID 26918？

  大多数导入过程都是不言自明的，但即使是经验丰富的GIS专业人员也可能被SRID难倒。

  “SRID"表示“Spatial Reference IDentifier（空间参考标识符)"。它定义了我们数据的**地理坐标系统**和**投影**的所有参数。

  SRID很方便，因为它将有关地图投影的所有信息（可能非常复杂）打包（更具体的说应该是映射）到一个数字中。

  你可以在以下链接中查找我们在上面使用的投影的定义：

- http://spatialreference.org/ref/epsg/26918/

  或直接在PostGIS内部使用对spatial_ref_sys表的查询：

```sql
SELECT srtext FROM spatial_ref_sys WHERE srid = 26918;
```

![img](https://img-blog.csdnimg.cn/2018122410014522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：PostGIS的spatial_ref_sys表是一个**OGC标准表**，用于定义数据库已知的所有空间参考系统。PostGIS附带的数据列出了3000多个已知的空间参考系统以及在它们之间进行转换/重新投影所需的详细信息。

  在以上两种查看方式里，你都会看到26918对应的空间参考系统的文本表示（为了便于观看，此处格式化得很整齐）：

```sql
PROJCS[
  "NAD83 / UTM zone 18N",
  GEOGCS[
    "NAD83",
    DATUM[
      "North_American_Datum_1983",
      SPHEROID[
        "GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]
      ],
      AUTHORITY["EPSG","6269"]
    ],
    PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4269"]
  ],
  UNIT["metre",1,AUTHORITY["EPSG","9001"]],
  PROJECTION["Transverse_Mercator"],
  PARAMETER["latitude_of_origin",0],
  PARAMETER["central_meridian",-75],
  PARAMETER["scale_factor",0.9996],
  PARAMETER["false_easting",500000],
  PARAMETER["false_northing",0],
  AUTHORITY["EPSG","26918"],
  AXIS["Easting",EAST],
  AXIS["Northing",NORTH]
]
```

  如果从data目录打开nyc_neighborhoods.prj文件，将看到相同的投影信息。

  人们开始使用PostGIS的一个常见问题是弄清楚要使用哪个SRID号来使用他们的数据。他们只有一个.prj文件，但是应该如何将.prj文件转换成正确的SRID号呢？

  最简单的答案就是使用电脑。将.prj文件的内容插入[http://prj2epsg.org](http://prj2epsg.org/)。这将为我们提供与投影定义最匹配的SRID编号（或编号列表）。

  世界上并不是每个地图投影都有对应的SRID编号，但大多数常见的投影都有对应的且保存在prj2epsg数据库中的SRID编号。

![_images/prj2epsg_01.png](https://img-blog.csdnimg.cn/20181224103509311)

  你从当地机构收到的数据 —— 如纽约市 —— 通常是基于"洲际飞机“或"UTM"标出的地方投影。

# 四、使用QGIS查看数据

  **QGIS**，是一个桌面端GIS查看器/编辑器，用于快速查看数据。

  QGIS可以查看许多数据格式，包括shapefile和PostGIS数据库。

  它的图形界面允许我们轻松探索我们的数据，以及快速的样式设置。

  尝试使用QGIS连接PostGIS数据库，该应用程序可从[http://qgis.org](http://qgis.org/)下载。

![img](https://img-blog.csdnimg.cn/20181224102417683.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181224102524933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181224102610517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)