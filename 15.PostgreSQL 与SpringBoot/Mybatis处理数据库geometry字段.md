- [mybatis 处理 数据库geometry字段_PigZHU'的博客-CSDN博客](https://blog.csdn.net/u010667710/article/details/103807117?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5-103807117-blog-104626105.pc_relevant_antiscanv2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-5-103807117-blog-104626105.pc_relevant_antiscanv2&utm_relevant_index=6)

## 1 Maven依赖

```xml
<dependency>
    <groupId>com.vividsolutions</groupId>
    <artifactId>jts</artifactId>
    <version>1.13</version>
</dependency>    
```

## 2 工具类

```java
/**
 * @Description 坐标标转换工具类
 */
public class CoordinateUtil {

    /**
	 * pi
	 */
    public final static double pi = 3.1415926535897932384626;
    /**
	 * a
	 */
    public final static double a = 6378245.0;
    /**
	 * ee
	 */
    public final static double ee = 0.00669342162296594323;

    /**
	 * @Description WGS84 to 火星坐标系 (GCJ-02)
	 * @param lon
	 * @param lat
	 * @return
	 */
    public static double[] wgs84_To_Gcj02(double lon, double lat) {
        if (outOfChina(lat, lon)) {
            return null;
        }
        double dLat = transformLat(lon - 105.0, lat - 35.0);
        double dLon = transformLon(lon - 105.0, lat - 35.0);
        double radLat = lat / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        double mgLat = lat + dLat;
        double mgLon = lon + dLon;
        return new double[] { mgLon, mgLat };
    }

    /**
	 * @Description 火星坐标系 (GCJ-02) to WGS84
	 * @param lon
	 * @param lat
	 * @return
	 */
    public static double[] gcj02_To_Wgs84(double lon, double lat) {
        double[] gps = transform(lat, lon);
        double lontitude = lon * 2 - gps[1];
        double latitude = lat * 2 - gps[0];
        return new double[] { lontitude, latitude };
    }

    /**
	 * @Description 火星坐标系 (GCJ-02) to 百度坐标系 (BD-09)
	 * @param gg_lon
	 * @param gg_lat
	 * @return
	 */
    public static double[] gcj02_To_Bd09(double gg_lon, double gg_lat) {
        double x = gg_lon, y = gg_lat;
        double z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * pi);
        double theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * pi);
        double bd_lon = z * Math.cos(theta) + 0.0065;
        double bd_lat = z * Math.sin(theta) + 0.006;
        return new double[] { bd_lon, bd_lat };
    }

    /**
	 * @Description 百度坐标系 (BD-09) to 火星坐标系 (GCJ-02)
	 * @param bd_lon
	 * @param bd_lat
	 * @return
	 */
    public static double[] bd09_To_Gcj02(double bd_lon, double bd_lat) {
        double x = bd_lon - 0.0065, y = bd_lat - 0.006;
        double z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * pi);
        double theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * pi);
        double gg_lon = z * Math.cos(theta);
        double gg_lat = z * Math.sin(theta);
        return new double[] { gg_lon, gg_lat };
    }

    /**
	 * @Description 百度坐标系 (BD-09) to WGS84
	 * @param bd_lat
	 * @param bd_lon
	 * @return
	 */
    public static double[] bd09_To_Wgs84(double bd_lon, double bd_lat) {

        double[] gcj02 = CoordinateUtil.bd09_To_Gcj02(bd_lon, bd_lat);
        double[] map84 = CoordinateUtil.gcj02_To_Wgs84(gcj02[0], gcj02[1]);
        return map84;

    }

    /**
	 * @Description 判断是否在中国范围内
	 * @param lat
	 * @param lon
	 * @return
	 */
    public static boolean outOfChina(double lat, double lon) {
        if (lon < 72.004 || lon > 137.8347)
            return true;
        if (lat < 0.8293 || lat > 55.8271)
            return true;
        return false;
    }

    /**
	 * @Description transform
	 * @param lat
	 * @param lon
	 * @return
	 */
    private static double[] transform(double lat, double lon) {
        if (outOfChina(lat, lon)) {
            return new double[] { lat, lon };
        }
        double dLat = transformLat(lon - 105.0, lat - 35.0);
        double dLon = transformLon(lon - 105.0, lat - 35.0);
        double radLat = lat / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        double mgLat = lat + dLat;
        double mgLon = lon + dLon;
        return new double[] { mgLat, mgLon };
    }

    /**
	 * @Description transformLat
	 * @param x
	 * @param y
	 * @return
	 */
    private static double transformLat(double x, double y) {
        double ret = -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y + 0.1 * x * y + 0.2 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(y * pi) + 40.0 * Math.sin(y / 3.0 * pi)) * 2.0 / 3.0;
        ret += (160.0 * Math.sin(y / 12.0 * pi) + 320 * Math.sin(y * pi / 30.0)) * 2.0 / 3.0;
        return ret;
    }

    /**
	 * @Description transformLon
	 * @param x
	 * @param y
	 * @return
	 */
    public static double transformLon(double x, double y) {
        double ret = 300.0 + x + 2.0 * y + 0.1 * x * x + 0.1 * x * y + 0.1 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(x * pi) + 40.0 * Math.sin(x / 3.0 * pi)) * 2.0 / 3.0;
        ret += (150.0 * Math.sin(x / 12.0 * pi) + 300.0 * Math.sin(x / 30.0 * pi)) * 2.0 / 3.0;
        return ret;
    }


    /**
	 * @Description WGS84 to 高斯投影(6度分带)
	 * @param longitude 经度
	 * @param latitude 纬度
	 * @return double[] x y
	 */
    public static double[] wgs84_To_Gauss6(double longitude, double latitude) {
        int ProjNo = 0;
        int ZoneWide; // //带宽
        double[] output = new double[2];
        double longitude1, latitude1, longitude0, X0, Y0, xval, yval;
        double a, f, e2, ee, NN, T, C, A, M, iPI;
        iPI = 0.0174532925199433; // //3.1415926535898/180.0;
        ZoneWide = 6; //6度带宽
        a = 6378137.0;
        f = 1.0 / 298.257223563; //WGS84坐标系参数
        //a = 6378245.0;f = 1.0 / 298.3; // 54年北京坐标系参数
        // //a=6378140.0; f=1/298.257; //80年西安坐标系参数
        ProjNo = (int) (longitude / ZoneWide);
        longitude0 = (double)(ProjNo * ZoneWide + ZoneWide / 2);
        longitude0 = longitude0 * iPI;
        longitude1 = longitude * iPI; // 经度转换为弧度
        latitude1 = latitude * iPI; // 纬度转换为弧度
        e2 = 2 * f - f * f;
        ee = e2 / (1.0 - e2);
        NN = a
            / Math.sqrt(1.0 - e2 * Math.sin(latitude1)
                        * Math.sin(latitude1));
        T = Math.tan(latitude1) * Math.tan(latitude1);
        C = ee * Math.cos(latitude1) * Math.cos(latitude1);
        A = (longitude1 - longitude0) * Math.cos(latitude1);
        M = a
            * ((1 - e2 / 4 - 3 * e2 * e2 / 64 - 5 * e2 * e2 * e2 / 256)
               * latitude1
               - (3 * e2 / 8 + 3 * e2 * e2 / 32 + 45 * e2 * e2 * e2
                  / 1024) * Math.sin(2 * latitude1)
               + (15 * e2 * e2 / 256 + 45 * e2 * e2 * e2 / 1024)
               * Math.sin(4 * latitude1) - (35 * e2 * e2 * e2 / 3072)
               * Math.sin(6 * latitude1));
        // 因为是以赤道为Y轴的，与我们南北为Y轴是相反的，所以xy与高斯投影的标准xy正好相反;
        xval = NN
            * (A + (1 - T + C) * A * A * A / 6 + (5 - 18 * T + T * T + 14
                                                  * C - 58 * ee)
               * A * A * A * A * A / 120);
        yval = M
            + NN
            * Math.tan(latitude1)
            * (A * A / 2 + (5 - T + 9 * C + 4 * C * C) * A * A * A * A / 24 + (61
                                                                               - 58 * T + T * T + 270 * C - 330 * ee)
               * A * A * A * A * A * A / 720);
        X0 = 1000000L * (ProjNo + 1) + 500000L;
        Y0 = 0;
        xval = xval + X0;
        yval = yval + Y0;
        output[0] = xval;
        output[1] = yval;
        return output;
    }	
}
```

## 3 自定义类型

```java
@Setter
@Getter
public class GeoPoint implements Serializable {
    /**
	 * 
	 */
    private static final long serialVersionUID = 1L;

    public GeoPoint(double lng, double lat) {
        this.lng = lng;
        this.lat = lat;
    }
    /* 经度 */
    private double lng;
    /* 纬度 */
    private double lat;

    public String toString(){
        return "lng:" + lng + ";lat:" + lat;
    }

    /**
     * 百度经纬度转换
     */
    @JSONField(serialize = false)
    public Coordinate getCoordinate() {
        //百度坐标系 (BD-09) to WGS84
        double[] wgsPntA = CoordinateUtil.bd09_To_Wgs84(lng, lat); 
        //WGS84->高斯6度分带投影
        double[] gaussPntA = CoordinateUtil.wgs84_To_Gauss6(wgsPntA[0], wgsPntA[1]);
        return new Coordinate(gaussPntA[0], gaussPntA[1]);
    }
}
```

## 4 实体对象

```java
private GeoPoint location;
```

## 5 处理类Handler

```java
/*
 * mybatis查询结果集中 mysql的geometry类型映射到GeoPoint对象
 */
@MappedTypes(value = {GeoPoint.class})
public class MysqlGeoPointTypeHandler extends BaseTypeHandler<GeoPoint> {

    private WKBReader _wkbReader;
    private int _srid = 0;
    public MysqlGeoPointTypeHandler() {
        GeometryFactory _geometryFactory = new GeometryFactory(new PrecisionModel(), _srid);
        _wkbReader = new WKBReader(_geometryFactory);
    }
    public MysqlGeoPointTypeHandler(int srid) {
        GeometryFactory _geometryFactory = new GeometryFactory(new PrecisionModel(), srid);
        _wkbReader = new WKBReader(_geometryFactory);
    }

    @Override
    public GeoPoint getNullableResult(ResultSet arg0, String arg1) throws SQLException {
        // TODO Auto-generated method stub
        return fromMysqlWkb(arg0.getBytes(arg1));
    }

    @Override
    public GeoPoint getNullableResult(ResultSet arg0, int arg1) throws SQLException {
        // TODO Auto-generated method stub
        return fromMysqlWkb(arg0.getBytes(arg1));
    }

    @Override
    public GeoPoint getNullableResult(CallableStatement arg0, int arg1) throws SQLException {
        // TODO Auto-generated method stub
        return fromMysqlWkb(arg0.getBytes(arg1));
    }

    @Override
    public void setNonNullParameter(PreparedStatement arg0, int arg1, GeoPoint arg2, JdbcType arg3)
        throws SQLException {
        // TODO Auto-generated method stub

    }

    /*
     * bytes转GeoPoint对象
     */
    private GeoPoint fromMysqlWkb(byte[] bytes) {
        if (bytes == null) {
            return null;
        }
        try {
            byte[] geomBytes = ByteBuffer.allocate(bytes.length - 4).order(ByteOrder.LITTLE_ENDIAN)
                .put(bytes, 4, bytes.length - 4).array();
            Geometry geometry = _wkbReader.read(geomBytes);
            Point point = (Point) geometry;
            return new GeoPoint(new Double(String.valueOf(point.getX())), new Double(String.valueOf(point.getY())));
        } catch (Exception e) {
        }
        return null;
    }
}
```

**注册到MybatisSqlSessionFactoryBean**

```java
sqlSessionFactory.setTypeHandlers(new TypeHandler[]{new MysqlGeoPointTypeHandler()});
```

