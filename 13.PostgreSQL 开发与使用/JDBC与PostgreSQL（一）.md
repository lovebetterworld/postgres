# 一、JDBC简介

JDBC（Java DataBase Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。也就是在开发Java程序时，可以使用JDBC来连接数据库，对数据库进行增、删、改、查等等。

JDBC规范了各种接口规范，用于Java应用程序与数据库之间的交互操作，查看JDK文档可以知道大概有以下这些接口：
    
各大数据库实现了这些接口，也就是构建了相应的实现类，这些实现类的集合统称为JDBC驱动程序。
    
各个数据库的JDBC驱动程序是不一样的，因为它们针对JDBC接口的实现方式不一样，PostgreSQL有PostgreSQL的JDBC驱动程序，MySQL有MySQL的JDBC驱动程序，Oracle有Oracle的驱动程序。

# 二、JDBC连接PostgreSQL（方式一）

首先要下载PostgreSQL的JDBC驱动程序，驱动下载地址：https://jdbc.postgresql.org/download.html
    
然后导入将JDBC驱动程序（jar包）导入项目中。
    
现在就可以编写程序连接数据库了：
    

```java
import java.sql.Connection;
import java.sql.Driver;
import java.sql.SQLException;
import java.util.Properties;
 
public class JdbcConnection1 {
	public static void main(String[] args) throws SQLException {
		// 参数：
		// jdbc协议:postgresql子协议://主机地址:数据库端口号/要连接的数据库名
		String url = "jdbc:postgresql://localhost:5432/test";
		// 数据库用户名
		String user = "postgres";
		// 数据库密码
		String password = "123456";
		
		// 1.  创建驱动程序类对象
		Driver driver = new org.postgresql.Driver();
		
		// 2. 设置用户名和密码
		Properties prop = new Properties();
		prop.setProperty("user", user);
		prop.setProperty("password", password);
		
		// 3. 连接数据库，返回连接对象
		Connection conn = driver.connect(url, prop);
	}
}

在这种连接方式中，是通过手动初始化一个Driver对象，然后借助Properties类来设置参数，最后再通过Driver对象的connect()方法来连接数据库的。
```
# 三、JDBC连接PostgreSQL（方式二）

也可以通过将驱动程序对象注册到DriverManager对象中，然后利用DriverManger类的getConnection()方法来连接数据库。
    
这种方式不需要借助Properties类设置参数。
    

```java
import java.sql.Connection;
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.SQLException;
 
public class JdbcConnection2 {
	public static void main(String[] args) throws SQLException {
		// 参数：
		// jdbc协议:postgresql子协议://主机地址:数据库端口号/要连接的数据库名
		String url = "jdbc:postgresql://localhost:5432/test";
		// 数据库用户名
		String user = "postgres";
		// 数据库密码
		String password = "123456";
		
		// 1.  创建驱动程序类对象
		Driver driver = new org.postgresql.Driver();
		
		// 2. 注册驱动程序（可以注册多个驱动程序）
		DriverManager.registerDriver(driver);
		
		// 3. 连接数据库，返回连接对象
		Connection conn = DriverManager.getConnection(url, user, password);	
	}
}

//Driver类对象在创建过程中会自动被注册到DriverManger类中，所以可以省略以上的第二个步骤，直接连接数据库：

package jdbc;
 
import java.sql.Connection;
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.SQLException;
 
public class JdbcConnection2 {
	public static void main(String[] args) throws SQLException {
		// 参数：
		// jdbc协议:postgresql子协议://主机地址:数据库端口号/要连接的数据库名
		String url = "jdbc:postgresql://localhost:5432/test";
		// 数据库用户名
		String user = "postgres";
		// 数据库密码
		String password = "123456";
		
		// 1.  创建驱动程序类对象，驱动程序类对象在创建过程中已经被注册到MangerDriver类中
		Driver driver = new org.postgresql.Driver();
		
		// 2. 连接数据库，返回连接对象
		Connection conn = DriverManager.getConnection(url, user, password);	
 
	}
}
```

# 四、JDBC连接PostgreSQL（方式三，推荐）

上面说Driver类对象在创建时会自动被注册到DriverManger类中，那么具体是在创建Driver类对象的哪个过程中被注册的呢？
    
其实是在Driver类的静态代码块中被注册的，这也就意味着Driver类一加载，Driver类对象就被注册到DriverManager中了。
    

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
 
public class JdbcConnection3 {
	public static void main(String[] args) throws Exception {
		// 参数：
		// jdbc协议:postgresql子协议://主机地址:数据库端口号/要连接的数据库名
		String url = "jdbc:postgresql://localhost:5432/test";
		// 数据库用户名
		String user = "postgres";
		// 数据库密码
		String password = "123456";
		
		// 1. 加载Driver类，Driver类对象将自动被注册到DriverManager类中
		Class.forName("org.postgresql.Driver");
		
		// 2. 连接数据库，返回连接对象
		Connection conn = DriverManager.getConnection(url, user, password);
	}
}

这种方式只需要两行代码就可以让JDBC和数据库进行连接，推荐使用这种方式连接数据库。
```