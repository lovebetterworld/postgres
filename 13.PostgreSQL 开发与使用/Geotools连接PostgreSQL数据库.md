- [geotools连接PostgreSQL数据库](https://www.jianshu.com/p/395b58f52b87)



# 一、geotools快速入门

`GeoTools`是一个开放源代码（LGPL）Java代码库，它提供了符合标准的方法来处理地理空间数据，例如实现地理信息系统（GIS）。

## 1.1 步骤

参照官网：[https://geotools.org/](https://links.jianshu.com/go?to=https%3A%2F%2Fgeotools.org%2F)

![img](https:////upload-images.jianshu.io/upload_images/23321330-3e6686493a54adb2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

我使用的是idea

![img](https://upload-images.jianshu.io/upload_images/23321330-80b68eb40db2801a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

 其实我创建的时候并不是按照官网上的步骤来的，因为我比较习惯从[https://start.spring.io/](https://links.jianshu.com/go?to=https%3A%2F%2Fstart.spring.io%2F)
 上面构建一个新的maven项目，填写命名下载压缩包，然后在idea中打开。

##  1.2 pom中增加依赖包

```xml
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <geotools.version>24-SNAPSHOT</geotools.version>
    </properties>
```

```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-shapefile</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-swing</artifactId>
            <version>${geotools.version}</version>
        </dependency>
    </dependencies>
```

```xml
    <repositories>
      <repository>
        <id>osgeo</id>
        <name>OSGeo Release Repository</name>
        <url>https://repo.osgeo.org/repository/release/</url>
        <snapshots><enabled>false</enabled></snapshots>
        <releases><enabled>true</enabled></releases>
      </repository>
      <repository>
        <id>osgeo-snapshot</id>
        <name>OSGeo Snapshot Repository</name>
        <url>https://repo.osgeo.org/repository/snapshot/</url>
        <snapshots><enabled>true</enabled></snapshots>
        <releases><enabled>false</enabled></releases>
      </repository>
    </repositories>
```

## 1.3 测试

```java
/*
 *    GeoTools - The Open Source Java GIS Toolkit
 *    http://geotools.org
 *
 *    (C) 2019, Open Source Geospatial Foundation (OSGeo)
 *
 *    This library is free software; you can redistribute it and/or
 *    modify it under the terms of the GNU Lesser General Public
 *    License as published by the Free Software Foundation;
 *    version 2.1 of the License.
 *
 *    This library is distributed in the hope that it will be useful,
 *    but WITHOUT ANY WARRANTY; without even the implied warranty of
 *    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
 *    Lesser General Public License for more details.
 *
 */

package org.geotools.tutorial.quickstart;

import java.io.File;
import org.geotools.data.FileDataStore;
import org.geotools.data.FileDataStoreFinder;
import org.geotools.data.simple.SimpleFeatureSource;
import org.geotools.map.FeatureLayer;
import org.geotools.map.Layer;
import org.geotools.map.MapContent;
import org.geotools.styling.SLD;
import org.geotools.styling.Style;
import org.geotools.swing.JMapFrame;
import org.geotools.swing.data.JFileDataStoreChooser;

/**
 * Prompts the user for a shapefile and displays the contents on the screen in a map frame.
 *
 * <p>This is the GeoTools Quickstart application used in documentationa and tutorials. *
 */
public class Quickstart {

    /**
     * GeoTools Quickstart demo application. Prompts the user for a shapefile and displays its
     * contents on the screen in a map frame
     */
    public static void main(String[] args) throws Exception {
        // display a data store file chooser dialog for shapefiles
        File file = JFileDataStoreChooser.showOpenFile("shp", null);
        if (file == null) {
            return;
        }

        FileDataStore store = FileDataStoreFinder.getDataStore(file);
        SimpleFeatureSource featureSource = store.getFeatureSource();

        // Create a map content and add our shapefile to it
        MapContent map = new MapContent();
        map.setTitle("Quickstart");

        Style style = SLD.createSimpleStyle(featureSource.getSchema());
        Layer layer = new FeatureLayer(featureSource, style);
        map.addLayer(layer);

        // Now display the map
        JMapFrame.showMap(map);
    }
}
```

# 1.4 运行，打开shp文件

![img](https://upload-images.jianshu.io/upload_images/23321330-9c9ea6faa4ef8258.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

![img](https://upload-images.jianshu.io/upload_images/23321330-ddd910b5c5933ef0.png?imageMogr2/auto-orient/strip|imageView2/2/w/781)

# 二、连接PostgreSQL数据库

在网上找到geotools连接数据库的资料，
 参考：[https://www.cnblogs.com/tuboshu/p/10752286.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Ftuboshu%2Fp%2F10752286.html)
 但是发现导入依赖失败

![img](https:////upload-images.jianshu.io/upload_images/23321330-c1bfb6bd48ba495b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

 最后看到这个：https://www.jianshu.com/p/e04f486b7954
 但是还是失败了，经过多次尝试，终于找到正确的依赖包。

## 2.1 pom依赖

```xml
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <geotools.version>24-SNAPSHOT</geotools.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-shapefile</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-swing</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-geojson</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.geotools</groupId>
            <artifactId>gt-epsg-hsql</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <!-- Provides support for PostGIS. Note the different groupId -->
        <dependency>
            <groupId>org.geotools.jdbc</groupId>
            <artifactId>gt-jdbc-postgis</artifactId>
            <version>${geotools.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
<repositories>
        <repository>
            <id>osgeo</id>
            <name>OSGeo Release Repository</name>
            <url>https://repo.osgeo.org/repository/release/</url>
            <snapshots><enabled>false</enabled></snapshots>
            <releases><enabled>true</enabled></releases>
        </repository>
        <repository>
            <id>osgeo-snapshot</id>
            <name>OSGeo Snapshot Repository</name>
            <url>https://repo.osgeo.org/repository/snapshot/</url>
            <snapshots><enabled>true</enabled></snapshots>
            <releases><enabled>false</enabled></releases>
        </repository>
        <repository>
            <id>maven2-repository.dev.java.net</id>
            <name>Java.net repository</name>
            <url>http://download.java.net/maven/2</url>
        </repository>
    </repositories>
```

![img](https://upload-images.jianshu.io/upload_images/23321330-b3c4fb44c1ac520d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

## 2.2 测试

```java
package com.geotools.geotoolsdemo;

import org.geotools.data.DataStore;
import org.geotools.data.DataStoreFinder;
import org.geotools.data.simple.SimpleFeatureCollection;
import org.geotools.data.simple.SimpleFeatureIterator;
import org.geotools.data.simple.SimpleFeatureSource;
import org.opengis.feature.simple.SimpleFeature;

import org.geotools.data.postgis.PostgisNGDataStoreFactory;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * @program: geotoolsdemo
 * @description: 连接数据库
 * @author: zhudan
 * @create: 2020/6/18 18:45
 */
public class PostGis {
    /**
     * @param dbtype:    数据库类型，postgis or mysql
     * @param host:      ip地址
     * @param port:      端口号
     * @param database:  需要连接的数据库
     * @param userName:  用户名
     * @param password:  密码
     * @param tableName: a需要连接的表名
     * @return: 返回为FeatureCollection类型
     */
    private static SimpleFeatureCollection connAndgetCollection(String dbtype, String host, String port,
                                                                String database, String userName, String password, String tableName) {
        Map<String, Object> params = new HashMap<String, Object>();
        DataStore pgDatastore = null;
        params.put(PostgisNGDataStoreFactory.DBTYPE.key, dbtype); //需要连接何种数据库，postgis or mysql
        params.put(PostgisNGDataStoreFactory.HOST.key, host);//ip地址
        params.put(PostgisNGDataStoreFactory.PORT.key, new Integer(port));//端口号
        params.put(PostgisNGDataStoreFactory.DATABASE.key, database);//需要连接的数据库
        params.put(PostgisNGDataStoreFactory.SCHEMA.key, "public");//架构
        params.put(PostgisNGDataStoreFactory.USER.key, userName);//需要连接数据库的名称
        params.put(PostgisNGDataStoreFactory.PASSWD.key, password);//数据库的密码
        SimpleFeatureCollection fcollection = null;
        try {
            //获取存储空间
            pgDatastore = DataStoreFinder.getDataStore(params);
            //根据表名获取source
            SimpleFeatureSource fSource = pgDatastore.getFeatureSource(tableName);
            if (pgDatastore != null) {
                System.out.println("系统连接到位于：" + host + "的空间数据库" + database
                        + "成功！");
                fcollection = fSource.getFeatures();
            } else {
                System.out.println("系统连接到位于：" + host + "的空间数据库" + database
                        + "失败！请检查相关参数");

            }
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("系统连接到位于：" + host + "的空间数据库" + database
                    + "失败！请检查相关参数");
        }
        return fcollection;
    }

    public static void main(String[] args) {
        //调用方法
        SimpleFeatureCollection featureColls = PostGis.connAndgetCollection("postgis", "localhost", "5432", "beijing", "postgres", "123456", "planet_osm_point");
        SimpleFeatureIterator itertor = featureColls.features();
        //循环读取feature，itertor.hasNext()表示游标下一个是否有数据，有返回ture,否则为false
        while (itertor.hasNext()) {
            //获取每一个要素
            SimpleFeature feature = itertor.next();
            System.out.println(feature.getAttribute("planet_osm_point"));
        }

    }
}
```

注：数据库要导入空间数据，参考这篇文章[https://blog.csdn.net/TcCookEgg/article/details/102496421](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2FTcCookEgg%2Farticle%2Fdetails%2F102496421)