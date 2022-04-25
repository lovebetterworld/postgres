- [PostgreSQL存储地理信息数据的注意点](https://blog.csdn.net/qq_35241223/article/details/107734044)



矢量数据分为点、线、面三种，实际存储过程中还有多点、多线、多面，并且数据还要包含坐标系信息。PostgreSQL安装PostGIS扩展后可以存储以上信息。实际使用过程中对于geometry类型的字段会区分类型，具体是点，是线，还是面，以及还要区分是WGS84（4326）坐标系的，是CGCS2000（4490）坐标系，还是Web墨卡托（3857）坐标系。但实际情况下，某些时候在存储过程中发现没有限制了，以下介绍一下该情况。

# 一、正常情况

正常情况下使用AddGeometryColumn方法创建字段时，会指定该字段的数据类型（点、线、面、多点……）以及该字段的坐标系，创建完成后插入其它类型数据时会报错，即不允许存储规定外的数据类型。该情况也是我们预期的，示例如下：

    示例表名为test1，类型为点，坐标系为4326(WGS84坐标系)

查看字段类型与坐标系信息：

    使用ST_GeometryType与ST_SRID查看行数据中类型与坐标系，但实际限制未在这

![行信息](https://img-blog.csdnimg.cn/2020080222483896.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

- 查看`geometry_columns`表中记录的信息，实际限制在此

![表信息](https://img-blog.csdnimg.cn/20200802225341940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

更新`test1`表的字段为线，实际情况插入不进去：

- 坐标系为4326，但类型不符，插入失败
   ![类型不对](https://img-blog.csdnimg.cn/202008022257177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

- 坐标系不符，插入失败

![坐标系不符](https://img-blog.csdnimg.cn/20200802225801818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

# 二、非正常情况

将`test1`表使用navicat复制数据复制一份，此时查看数据类型时会发现类型变为`GEOMETRY`，`srid`变为0，此时不再有限制，字段可以存储任何类型与任何坐标系的数据。

![复制](https://img-blog.csdnimg.cn/20200802230059157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)
 查看类型：

![类型](https://img-blog.csdnimg.cn/20200802230559848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

更新为线以及其他坐标系，不再有限制，结果如下：

    更新id为2的数据，将点更新为线，同时坐标系设置为4490

![成功](https://img-blog.csdnimg.cn/20200802230730473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MjQxMjIz,size_16,color_FFFFFF,t_70)

# 三、建议

1. 新增字段时使用AddGeometryColumn，指定类型、坐标系等信息
2. 新增字段时使用Alter Table test add column aa geometry('point', 4326)要指定类型、坐标系等信息
3. 迁移数据后，记得要使用geometry_column查看类型与坐标系，防止信息丢失，不再有限制，导致一张表中存储不同类型的数据

