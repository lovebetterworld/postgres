# Java  操作Gis geometry类型数据

**geolatte-geom** 和**geolatte-geojson**。用于操作geometry和String以及json的互相转化。而json和geojson个人理解就是输出格式不同。多了一些geometry特有的属性。

pom.xml文件如下：

```xml
<!-- https://mvnrepository.com/artifact/org.geolatte/geolatte-geom -->
<dependency>
    <groupId>org.geolatte</groupId>
    <artifactId>geolatte-geom</artifactId>
    <version>1.6.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.geolatte/geolatte-geojson -->
<dependency>
    <groupId>org.geolatte</groupId>
    <artifactId>geolatte-geojson</artifactId>
    <version>1.6.0</version>
</dependency>
```
```java
public static void main(String[] args) {

    // 模拟数据库中直接取出的geometry对象值（他是二进制的）
    // WKT 是字符串形式，类似"POINT(1 2)"的形式
    // 所以WKT转  geometry，相当于是字符串转geometry
    // WKB转  geometry，相当于是字节转geometry
    String s="01020000800200000097E5880801845C404D064F3AF4AE36400000000000000000290A915F01845C40DC90B1A051AE36400000000000000000";
    Geometry geo = Wkb.fromWkb(ByteBuffer.from(s));

    // geometry对象和WKT输出一致

    //        Geometry geometry1 = Wkt.fromWkt(wkt);
    System.out.println("-----Geometry------"+geo.getPositionN(1));
    System.out.println("-----wkt------"+ Wkt.toWkt(geo));
    System.out.println("-----wkb------"+Wkb.toWkb(geo));
}
```

