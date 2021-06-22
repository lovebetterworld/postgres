- [PostGIS导入导出ESRI Shapefile数据](https://cloud.tencent.com/developer/article/1386147?from=article.detail.1722171)

# 一、PostGIS导入导出ESRI Shapefile数据

PostGIS作为[PostgreSQL](https://cloud.tencent.com/product/postgresql?from=10680)数据库的空间扩展，提供了对空间数据管理的支持。对于空间矢量数据，PostGIS提供了Geometry和Geography俩种类型用于空间对象的存储，Geometry使用笛卡尔坐标系，而Geography使用球面坐标系（默认是WGS84坐标系）。对于空间栅格数据，则提供了Raster类型。

这里介绍如何导入我们常用的ESRI Shapefile数据到PostgreSQL数据库中，我们可以使用PostGIS提供的shp2pgsql和pgsql2shp工具进行导入和导出操作，还可以使用GDAL库提供的ogr2ogr工具，ogr2ogr工具支持更加多样的数据格式。

我的实验环境如下：  OS: Ubuntu 16.04 LTS  PostgreSQL:9.5.5 （安装好PostgreSQL以后可以使用`psql --version`进行查看）  PostGIS: 2.2 （安装好PostGIS，并在数据库中启用PostGIS扩展以后，可以在psql命令行中使用`SELECT PostGIS_Version();`或者`SELECT PostGIS_Full_Version();`进行查看。

我们使用的数据是全球大洲的一个矢量数据，坐标类型为WGS84。数据下载链接：[百度云下载](https://pan.baidu.com/s/1b4udv4)

------

在Ubuntu中安装PostgreSQL和PostGIS非常简单：  首先，使用如下命令安装PostgreSQL：

```javascript
sudo apt-get install postgresql
```

然后，使用如下命令添加[UbuntuGIS](https://wiki.ubuntu.com/UbuntuGIS)的PPA用于安装PostGIS扩展。

```javascript
sudo add-apt-repository ppa:ubuntugis/ppa
sudo apt-get update
```

最后，使用如下命令安装PostGIS：

```javascript
sudo apt-get install postgis
```

安装好了以后，使用`sudo -u postgres psql`命令可以进入psql交互环境。  可以使用SQL修改postgres用户的密码`alter user postgres with password 'new password';` （修改了postgres用户密码和没有修改使用当前用户登录，在后面插入数据时命令会稍有不同）

------

进入psql交互环境以后，我们首先创建数据库。

```javascript
CREATE DATABASE postgis_in_action;
```

然后再创建一个schema，以后我们可以将我们创建的table都存储在我们的schema中，而不是默认的public schema中。

```javascript
CREATE SCHEMA staging;
```

然后，切换到我们新建的postgis_in_action数据库中。

```javascript
\c postgis_in_action
```

然后在postgis_in_action数据库中启用PostGIS扩展。

```javascript
CREATE EXTENSION postgis;
```

可以通过`\dx`命令查看安装的扩展：  

![image-20210622093323730](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093323730.png)

创建好了数据库以后，我们可以使用`\q`命令退出psql。

接下来就是使用shp2psql命令行工具导入数据了，命令如下：

```javascript
shp2pgsql -s 4326 -I "continent" staging.world_continent | psql -h localhost -p 5432 -d postgis_in_action -U postgres -W
```

首先说明的是shp2pgsql的参数（具体参数使用`shp2pgsql --help`进行查看）：  `-s`指定空间参考系，PostGIS的参考系和EPSG代码是一样的，比如EPSG:4326表示WGS84地理坐标系  `-I`指定在新建的关系表的空间对象的那一列建立空间索引  然后，双引号引起来的是Shapefile的文件名称（也可以加上扩展名.shp）  最后是关系表的全名，staging是schema名称，world_continent是关系名称  shp2pgsql的输出是一个标准的SQL，然后Linux的管道操作符’|’将结果传入到psql中进行SQL的执行。  `-h`指定连接的地址hostname  `-p`指定连接的端口号  `-d`指定连接的数据库名称  `-U`指定连接的用户名  `-W`指定在执行时弹出密码输入提示

**注意：**  修改了postgres用户密码的情况下，使用上面的命令插入数据。执行过程中，按照提示输入postgres用户的密码即可。  也可以不给postgres用户设置密码，使用如下的命令插入数据，效果是一样的。其实，raster2pgsql命令及其参数不变，就是进入psql命令的时候，稍微有些不同。要不然会提示password authentication failed for user “postgres”错误。

```javascript
raster2pgsql -s 4326 -C ~/Downloads/gis-data/wsiearth.tif staging.wsiearth | sudo -u postgres psql -d postgis_in_action
```

这条命令执行过程中，需要输入当前用户的密码即可。

执行过程如下：  

![image-20210622093338551](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093338551.png)

执行成功以后，我们可以进入psql从数据库中查看数据。命令如下：`\dt staging.`其中，staging是schema的名称，可以看到staging中有两个关系表。  

![image-20210622093348467](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093348467.png)

此外，我们还可以使用`\d staging.world_continent`查看world_continent关系的表结构：  

![image-20210622093355484](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093355484.png)

  可以看到这里有一个geom的列。**在PostGIS中Geography对象类型保存在名为geog的列，而Geometry对象类型保存在geom的列。**所以，我们的数据被以Geometry对象类型保存在数据库。如果要保存成为Geography对象，则需要在shp2psql命令行导入的时候加入`-G`参数。

------

下面说说数据的导出，我们可以使用psql2shp工具导出数据为Shapefile文件。命令如下：

```javascript
pgsql2shp -f ~/Desktop/continent -h localhost -p 5432 -u postgres -P [passworld] postgis_in_action staging.world_continent
```

`-f`后面是导出的文件全路径  `-P`后面接用户postgres的密码  最后面postgis_in_action是数据库名称，staging.world_continent是关系表名称

![image-20210622093428334](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093428334.png)

最后，看看如何使用QGIS直接连接PostgreSQL数据库进行数据显示。  （添加了UbuntuGIS的PPA以后，我们可直接使用`sudo apt-get install qgis python-qgis qgis-plugin-grass`命令安装QGIS）  打开QGIS，在最左侧的图标中点击`Add PostGIS layers`，在弹出的对话框中点击`New`新建一个连接，输入连接参数。  

![image-20210622093440261](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093440261.png)

  点击`Connect`，可以看到我们的staging中有两个关系表。  

![image-20210622093447923](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093447923.png)

  选择world_continent关系表，然后点击`Add`可以进行数据的显示。  

![image-20210622093455156](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622093455156.png)