# 一、PgAdmin

  PostgreSQL有许多管理工具，主要的一个是**psql**，一个输入SQL命令查询的命令行工具。

  另一个流行的PostgreSQL工具是免费的开源图形工具**pgAdmin**，在pgAdmin中完成的所有查询都可以使用psql完成。

## **1.1、**找到pgAdmin并启动它

  ![img](https://img-blog.csdnimg.cn/20181223220147830.png)

  ![img](https://img-blog.csdnimg.cn/20181223220217621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 1.2、如果是第一次运行pgAdmin，应该有一个已在pgAdmin中配置的PostGIS服务器条目（localhost:5432）。双击该条目，并在密码框中输入密码，以连接到数据库。

  ![img](https://img-blog.csdnimg.cn/20181223220555359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 二、创建一个数据库

## **2.1、**打开数据库的树结构选项，查看可用的数据库。postgres数据库是默认的postgres用户所属的用户数据库，我们不对该数据库感兴趣。

  ![img](https://img-blog.csdnimg.cn/20181223220841429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## **2.2、**鼠标右击数据库**选项并选择**新建数据库：

![img](https://img-blog.csdnimg.cn/20181223221052462.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## **2.3、**如下图所示，填写“**新建数据库**”表单，然后单击“**确定**”：

![img](https://img-blog.csdnimg.cn/20181223221212519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## 2.4、选择nyc这个新建的数据库，并打开它以显示对象树，将会看到public架构（schema）：

![img](https://img-blog.csdnimg.cn/20181223221437795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## **2.5、**单击下面所示的SQL查询按钮（或转到**工具** > **查询工具**）。

![img](https://img-blog.csdnimg.cn/20181223221602701.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## **2.6、**在查询文本区域中输入以下查询语句以加载**PostGIS空间扩展**：

```sql
CREATE EXTENSION postgis;
```

![img](https://img-blog.csdnimg.cn/20181223221852662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## **2.7、**单击工具栏中的**执行查询**按钮（或按F5）以"执行查询"。

![img](https://img-blog.csdnimg.cn/20181223222052915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

## **2.8、**现在，通过运行PostGIS函数来确认是否安装了PostGIS：

```sql
SELECT postgis_full_version();
```

![img](https://img-blog.csdnimg.cn/20181223222408865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  至此，已经成功地创建了PostGIS空间数据库！

# 三、函数列表

  **PostGIS_Full_Version()**  ——  返回完整的PostGIS版本信息和配置信息。