- [PostGis基本操作-新建空间数据库与shp数据的导入_霸道流氓气质的博客-CSDN博客](https://blog.csdn.net/BADAO_LIUMANG_QIZHI/article/details/114790636)

# 场景

PostGresSQL简介与Windows上的安装教程：

https://blog.csdn.net/BADAO_LIUMANG_QIZHI/article/details/113981563

在上面介绍了PostGreSql的安装，安装完成后可选择安装postgis扩展。

那么怎么使用PostGis新建空间数据库以及shp地图数据的导入。

# 实现

## 空间数据库的创建

首先在电脑中打开pgAdmin,输入密码连接成功之后，点击数据库-新建-数据库

 

![img](https://img-blog.csdnimg.cn/20210314154500198.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

这里取名叫demo，然后再demo数据库上右键-查询工具

 

![img](https://img-blog.csdnimg.cn/20210314154504473.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

然后输入

```
CREATE EXTENSION postgis;
```

并点击上面三角形的运行按钮

 

![img](https://img-blog.csdnimg.cn/20210314154519953.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

然后在扩展里面就有postgis了，并且可以继续在查询工具中执行

```
SELECT postgis_full_version();
```

验证是否安装成功

 

![img](https://img-blog.csdnimg.cn/20210314154531680.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

## shp数据的导入

安装完postgresql以及postgis的扩展后，就可以在电脑中搜索

PostGis Shapefile and DBF Loader Exporter工具来进行shp文件的导入

 

![img](https://img-blog.csdnimg.cn/20210314154537700.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

注意：

首先你要有一个shp文件，这里提供一个中国省级行政区划_shp地图数据文件：

https://download.csdn.net/download/BADAO_LIUMANG_QIZHI/15785012

将其下载之后，注意两点，一是shp文件名不能有中文，而是shp的路径中不能有中文。

打开工具后，点击View connection details

然后输入要连接的用户名和密码以及ip端口和数据库

 

![img](https://img-blog.csdnimg.cn/20210314154546389.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

点击ok后会提示连接成功。

然后为了兼容不同版本的兼容问题，这里的Options下的Import Options中的编码修改为GBK

 

![img](https://img-blog.csdnimg.cn/20210314154550635.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

然后点击import按钮，选择上面下载的或者要导入的shp文件的路径

![img](https://img-blog.csdnimg.cn/20210314154555668.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

这里不能只有shp文件，还需要dbf文件，不然会导入失败

导入成功之后会有成功的提示

 

![img](https://img-blog.csdnimg.cn/20210314154559207.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

查看数据是在架构下public下表下面就可以找到新导入的表，然后右键查看/编辑数据-前100行，就可以看到数据了。

![img](https://img-blog.csdnimg.cn/20210314154606248.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JBREFPX0xJVU1BTkdfUUlaSEk=,size_16,color_FFFFFF,t_70)

 

 