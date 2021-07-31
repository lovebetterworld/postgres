- [postgis中geometry字段在mybatis generator插件中成果转换对照的解决方案](https://blog.csdn.net/a030455/article/details/105968207/)



# 1.文章总述

本文通过使用BaseTypeHandler接口，解决了postgis中geometry结果在pojo中的对照问题。
利用BaseTypeHandler实现了geometry的二进制成果，在pojo中以文本类型wkt格式来直观展现结果，便于前端接收结果后的再处理与图形绘制。

本解决方案github项目地址：
[github-PostgisGeometryTypeHandler](https://github.com/WHU-Linyue/PostgisGeometryTypeHandler)

## 1.1问题描述

现阶段，我们项目中使用的数据库为postgis，对象生成工具为mybatis-generator。
具体问题如下：

1. geometry内容为二进制形式存储；
2. mybatis-generator中默认的字段类型转换不支持geometry，无法以常规的地理信息数据格式直观展现。

## 1.2最终效果

postgis的geometry值最终以wkt格式存储在pojo对象中。

本文主要描述实现的核心环节，至于mybatis-generator的配置等其余技术问题请自行百度。

## 1.3解决步骤

1. maven依赖配置参考

```xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>

        <!--postgis支持包，用于geometry格式转换-->
        <dependency>
            <groupId>net.postgis</groupId>
            <artifactId>postgis-jdbc</artifactId>
            <version>2.3.0</version>
        </dependency>

        <!-- MyBatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.4</version>
        </dependency>
    </dependencies>
```

2. mybatis-generator中 columnOverride 的配置
    说明

```xml
<columnOverride column="postgis中geometry字段名" 
				property="pojo对象中的文本字段" 
				javaType="java.lang.String"
				typeHandler="MyGeometryTypeHandler类型转换器"/>
```

参考样例

```xml
<columnOverride column="smgeometry" property="smgeometry" javaType="java.lang.String"
    typeHandler="cn.fcch.propertysalesonline.mbg.typeHandler.MyGeometryTypeHandler"/>
```

3. MyGeometryTypeHandler具体代码片段

```java
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedTypes;
import org.postgis.PGgeometry;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@MappedTypes({String.class})
public class MyGeometryTypeHandler extends BaseTypeHandler<String> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
        PGgeometry pGgeometry = new PGgeometry(parameter);
        ps.setObject(i, pGgeometry);
    }

    @Override
    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
        PGgeometry pGgeometry = new PGgeometry(rs.getString(columnName));
        if (pGgeometry == null) {
            return null;
        }
        return pGgeometry.toString();
    }

    @Override
    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        PGgeometry pGgeometry = new PGgeometry(rs.getString(columnIndex));
        if (pGgeometry == null) {
            return null;
        }
        return pGgeometry.toString();
    }

    @Override
    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {

        PGgeometry pGgeometry = new PGgeometry(cs.getString(columnIndex));
        if (pGgeometry == null) {
            return null;
        }
        return pGgeometry.toString();
    }
}
```

这样就能实现，在pojo对象中以wkt形式存储几何，在postgis中以geometry存储几何。

## 1.4 展示效果

geometry转换后的wkt文本：

```java
SRID=4490;MULTIPOLYGON(((120.075362920772 30.2956581115747,120.078442096721 30.2955132722879,120.078903436672 30.294268727305,120.075647234928 30.2941828966165,.....)))
```

geometry二进制成果：

```java
01060000208A1100000100000001030000000100000005000000FE0200BFD2045E40AD020040B04B3E40F802003205055E40AE02...
```

