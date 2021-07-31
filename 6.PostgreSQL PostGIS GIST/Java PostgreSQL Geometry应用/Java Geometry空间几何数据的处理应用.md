# 一、Java Geometry空间几何数据的处理应用

- [Java Geometry空间几何数据的处理应用](https://www.jianshu.com/p/5e9c9131d75e?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

[WKT](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.baike.com%2Fwiki%2FWKT)，是一种文本标记语言，用于表示矢量几何对象、空间参照系统及空间参照系统之间的转换。它的二进制表示方式，亦即WKB(well-known binary)则胜于在传输和在数据库中存储相同的信息。该格式由开放地理空间联盟(OGC)制定。

WKT可以表示的几何对象包括：点，线，多边形，TIN（[不规则三角网](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.baike.com%2Fsowiki%2F%E4%B8%8D%E8%A7%84%E5%88%99%E4%B8%89%E8%A7%92%E7%BD%91%3Fprd%3Dcontent_doc_search)）及多面体。可以通过几何集合的方式来表示不同维度的几何对象。
 几何物体的坐标可以是2D(x,y),3D(x,y,z),4D(x,y,z,m),加上一个属于线性参照系统的m值。
 以下为几何WKT字串样例：

- POINT(6 10)

- LINESTRING(3 4,10 50,20 25)
- POLYGON((1 1,5 1,5 5,1 5,1 1),(2 2,2 3,3 3,3 2,2 2))
- MULTIPOINT(3.5 5.6, 4.8 10.5)
- MULTILINESTRING((3 4,10 50,20 25),(-5 -8,-10 -8,-15 -4))
- MULTIPOLYGON(((1 1,5 1,5 5,1 5,1 1),(2 2,2 3,3 3,3 2,2 2)),((6 3,9 2,9 4,6 3)))
- GEOMETRYCOLLECTION(POINT(4 6),LINESTRING(4 6,7 10))
- POINT ZM (1 1 5 60)
- POINT M (1 1 80)
- POINT EMPTY
- MULTIPOLYGON EMPTY

# 二、向空间数据库插入数据

```plsql
--GEOM是类型为Geometry的字段--
--我们向该字段新增了一条3D的多边形数据--
--geometry :: STGeomFromText () 是由SQLSERVER提供的函数，它能将WKT文本转换为数据库geometry类型的数据--
INSERT INTO [dbo].[TEST_GEO_TABLE] ( [GEOM] )
VALUES
    ( geometry :: STGeomFromText ( 
    'POLYGON ((
        113.507259000000005 22.24814946 8, 
        113.507188600000006 22.248088559999999 9, 
        113.507117399999998 22.24802743 10, 
        113.507046099999997 22.24796624 11, 
        113.507017300000001 22.247888209999999 12
        ))',4326 )
    );
```

也就是说，将坐标转化为WKT文本，我们就可以插入空间数据。接下来我们要考虑的是如何产生WKT文本。

# 三、使用Java创建Geometry对象

## 3.1 常见Geometry的JavaAPI

wkt文本仅仅是一个字符串而已，直接将坐标点拼接成符合WKT格式的字符串不就可以了吗？
 道理是这个道理，要做好可就难了。

- 拼接工作量巨大
- 拼接过程容易出错
- 拼接的结果不一定合法可用
   我们需要一套JAVA API对数据进行处理，能够方便的创建Geometry对象，进行地理信息的绘制、创建、验证等等功能

市面上常见的GeometryApi有

- [Esri](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FEsri)/[geometry-api-java](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FEsri%2Fgeometry-api-java)

- [locationtech](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Flocationtech)/**[jts](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Flocationtech%2Fjts)** （推荐）

Esri是Arcgis官方提供的javaSDK，可惜功能不多，甚至不能提供基本的空间计算功能。
 jts功能较为齐全，资料也相对丰富一点



## 3.2 JTS的部分API使用方式

```java
    @Test
    public void geoTest() throws ParseException {
        /**
         * GeometryFactory工厂，参数一：数据精度 参数二空间参考系SRID
         */
        GeometryFactory geometryFactory = new GeometryFactory(new PrecisionModel(PrecisionModel.FLOATING), 4326);

        /**
         * 熟知文本WKT阅读器，可以将WKT文本转换为Geometry对象
         */
        WKTReader wktReader = new WKTReader(geometryFactory);

        /**
         * Geometry对象，包含Point、LineString、Polygon等子类
         */
        Geometry geometry = wktReader.read("POINT (113.53896635 22.36429837)");

        /**
         * 将二进制流的形式读取Geometry对象
         */
        WKBReader wkbReader = new WKBReader(geometryFactory);

        /**
         * 单纯的一个坐标点，单点可以创建Point，多点可以创建LineString、Polygon等
         */
        Coordinate coordinate = new Coordinate(1.00, 2.00);
        Point point = geometryFactory.createPoint(coordinate);

        Polygon polygon = geometryFactory.createPolygon(new Coordinate[]{
                new Coordinate(1, 2),
                new Coordinate(1, 2),
                new Coordinate(1, 2),
                new Coordinate(1, 2),
                new Coordinate(1, 2),
        });
        Geometry geometry1 = point;
        Geometry geometry2 = polygon;

        /**
         * WKT输出器，将Geometry对象写出为WKT文本
         */
        WKTWriter wktWriter = new WKTWriter();
        String write = wktWriter.write(point);
    }
```

## 3.3 JTS中Geometry数据类型的子类

![img](https://upload-images.jianshu.io/upload_images/10533664-cec5c278f4d0240d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1173)

# 四、使用JAVA向空间数据库新增数据

根据上面测试类中Api的使用，让我们总结几个要点

- 工厂类对象只需初始化一次，应放在配置类注入到Spring容器中
- 由前端或Excel导入相关坐标数据，生成Geometry对象
- 持久化Geometry对象到SqlServer



本例中推荐两种方式进行Geometry对象的持久化：

1. 获取Geometry对象的WKT文本，再使用SqlServer提供的`geometry :: STGeomFromText ()`函数将WKT文本存储为数据库Geometry类型
2. 将jts包中Geometry对象转换成SqlServer JDBC包中的Geometry对象，将Geometry对象以二进制的形式持久化到数据库

环境：
 本例代码基于JTS、SpringBoot、Mybatis-Plus、mssql-jdbc环境

# 五、使用`TypeHandler`映射自定义对象字段插入Geometry数据

## 5.1 自定义TypeHandler

当我们使用Mybatis框架时，Mybatis提供了自定义类型转换器TypeHandler实现特殊对象与Sql字段的映射关系

```java
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedTypes;
import org.locationtech.jts.geom.Geometry;
import org.locationtech.jts.io.WKTReader;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * @author wangqichang
 * @since 2019/8/28
 */
@Slf4j
@MappedTypes(value = {Geometry.class})
public class GeometryTypeHandler extends BaseTypeHandler<Geometry> {

    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Geometry geometry, JdbcType jdbcType) throws SQLException {
        /**
         * 获取jts包对象的wkt文本，再转换成sqlserver的Geometry对象
         * 调用ps的setBytes（）方法，以二进制持久化该geometry对象
         */
        com.microsoft.sqlserver.jdbc.Geometry geo = com.microsoft.sqlserver.jdbc.Geometry.STGeomFromText(geometry.toText(), geometry.getSRID());
        preparedStatement.setBytes(i, geo.STAsBinary());
    }

    @Override
    public Geometry getNullableResult(ResultSet resultSet, String s) {
        try {
            /**
             * 从ResultSet中读取二进制转换为SqlServer的Geometry对象
             * 使用jts的WKTReader将wkt文本转成jts的Geometryd对象
             */
            com.microsoft.sqlserver.jdbc.Geometry geometry1 = com.microsoft.sqlserver.jdbc.Geometry.STGeomFromWKB(resultSet.getBytes(s));
            String s1 = geometry1.toString();
            WKTReader wktReader = SpringContextUtil.getBean(WKTReader.class);
            Geometry read = wktReader.read(s1);
            return read;
        } catch (Exception e) {
            log.error(e.getMessage());
            throw new ServiceException(e.getMessage());
        }
    }

    @Override
    public Geometry getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return null;
    }

    @Override
    public Geometry getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return null;
    }
}
```

## 5.2 实体对象

实体对象如下：

- `objectid`为Integer类型，非自增（此字段为Arcgis维护，不能修改）`@TableId`是mybatis-plus插件的注解，告知插件该字段为主键字段，字段名为OBJECT，主键策略为用户输入
- `shape`为jts的Geometry对象（该对象JSON序列化结果非常吓人，所以使用`@JsonIgnore`修饰）
- `@KeySequence`也是mybatis-plus的插件，作用是标识该对象需要使用的主键序列名。此处我实现了一个`IKeyGenerator`，作用类似于插入数据前查询Oracle的序列名以填充主键。

```java
@Data
@TableName("LINE_WELL")
@KeySequence(value = "LINE_WELL",clazz = Integer.class)
public class Well extends MyGeometry implements Serializable {

    @TableId(value = "OBJECTID", type = IdType.INPUT)
    private Integer objectid;

    @JsonIgnore
    protected Geometry shape;
}
```

## 5.3 自定义主键生成策略

在arcgis中，空间表中的主键字段为int，并且非自增，不能进行修改。当修改为自增时arcgis会出现一些错误。因此，java后台插入空间数据需要自己完成主键的查询生成。
 `IKeyGenerator`是Mybatis-Plus提供的接口。此实现的作用是，当指定这个主键生成策略时，mp框架将会在新增数据前调用此实现，将结果赋值给对象的ID（类似于Oracle的序列）
 注意，该类需要注入到Spring容器中

```java
import com.baomidou.mybatisplus.core.incrementer.IKeyGenerator;

/**
 * @author wangqichang
 * @since 2019/8/30
 */
public class SqlServerKeyGenerator implements IKeyGenerator {
    @Override
    public String executeSql(String incrementerName) {
        return "select max(OBJECTID)+1 from " + incrementerName;
    }
}
```

## 5.4 Geometry对象持久化

当我们调用mybatis-plus提供的方法持久化对象

```java
 String str = "POLYGON ((113.52048666400003 22.248443089000034, 113.5206744190001 22.24822462700007, 113.52082998700007 22.248343788000057, 113.52060468200011 22.248547355000028, 113.52048666400003 22.248443089000034))";
        Geometry read = null;
        try {
            /**
             * 这里使用wkt文本生成了一个jts包下的Geometry对象
             */
            read = SpringContextUtil.getBean(WKTReader.class).read(str);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        Well well = new Well();
        well.setShape(read);
        //这里是Mybatis-Plus提供的save接口，调用其内部实现直接储存对象
        wellService.save(well);
        System.out.println("持久化成功");
```

执行日志如下：
 数据插入前执行了`SqlServerKeyGenerator`中的sql获取主键
 插入代码中字段shape为Geometry对象的二进制

```java
2019-08-30 15:54:23.541  INFO 8484 --- [nio-8905-exec-1] jdbc.sqltiming                           : SELECT max(OBJECTID) + 1 FROM LINE_WELL 
 {executed in 4 msec}
2019-08-30 15:54:23.631  INFO 8484 --- [nio-8905-exec-1] jdbc.sqltiming                           : INSERT INTO LINE_WELL (OBJECTID, shape) VALUES (3, '<byte[]>') 
 {executed in 17 msec}
```

# 六、手写xml插入Geometry数据

使用SqlServer提供的函数`geometry :: STGeomFromText( #{wktText},4326)`将Geometry转换成WKT文本再进行插入

```xml
    <insert id="insertCorridorBySql" parameterType="com.zh.xxx.entity.xxx" useGeneratedKeys="true"
            keyProperty="objectid">
        INSERT INTO [LINE_CORRIDOR] (
         shape
        )
        values (
        geometry :: STGeomFromText( #{wktText},4326)
        )
    </insert>
```

注意,wktText是一个非表字段的临时字段，我在此定义了一个父类，所有包含Geometry的空间表实体均继承此类，用于处理wkt文本

```java
import org.locationtech.jts.geom.Geometry;
import org.locationtech.jts.io.WKTWriter;

import java.io.Serializable;

/**
 * 针对Geometry获取Wkt文本字段做处理的Geometry父类，getWktText替代getText,输出三维wkt文本
 * 针对sql_server无法识别POLYGON Z 语法，对wkt文本进行替换
 */
@Data
public class MyGeometry implements Serializable {

    /**
     * 三维wkt输出，默认为2D不带Z
     */
    @TableField(exist = false)
    @JsonIgnore
    private WKTWriter wktWriter = new WKTWriter(3);

    /**
     * sql_server 与 jts wkt不兼容问题
     */
    @TableField(exist = false)
    @JsonIgnore
    private static final String THREE_D_PRIFIX = "POLYGON Z";
    @TableField(exist = false)
    @JsonIgnore
    private static final String TWO_D_PRIFIX = "POLYGON";

    @JsonIgnore
    protected Geometry shape;


    @TableField(exist = false)
    @JsonIgnore
    private String wktText;

    public String getWktText() {
        if (StrUtil.isBlank(wktText)){
            if (getShape() != null) {
                String wkt = wktWriter.write(shape);
                if (wkt.startsWith(THREE_D_PRIFIX)) {
                    wktText = StrUtil.replace(wkt, THREE_D_PRIFIX, TWO_D_PRIFIX);
                } else {
                    wktText = wkt;
                }
            }
        }
        return wktText;
    }
}
```

# 七、采坑记录

## 7.1 jts与sqlserver识别的wkt不兼容

```java
[2019-07-01 16:40:20,637] [ERROR] [http-nio-8905-exec-5] jdbc.audit 111 7. PreparedStatement.execute() INSERT INTO [zhundergroundcableline].[dbo].[LINE_CORRIDOR] ( [Shape] ) values ( geometry :: STGeomFromText( 'POLYGON Z((113.5079365 22.24850034 
0, 113.5078521 22.24845659 0, 113.5077674 22.24841271 0, 113.5076826 22.24836872 0, 113.5075978 22.24832498 0))',4326) ) 

com.microsoft.sqlserver.jdbc.SQLServerException: 在执行用户定义例程或聚合“geometry”期间出现 .NET Framework 错误: 
System.FormatException: 24142: 在位置 8 处应为 "("，但输入中实际为 "Z"。
System.FormatException: 
   在 Microsoft.SqlServer.Types.WellKnownTextReader.RecognizeToken(Char token)
   在 Microsoft.SqlServer.Types.SqlGeometry.GeometryFromText(OpenGisType type, SqlChars text, Int32 srid)
```

