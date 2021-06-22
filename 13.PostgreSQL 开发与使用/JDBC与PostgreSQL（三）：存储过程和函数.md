# 一、过程化SQL简介

SQL的一大优点就是高度非过程化，即开发人员只要面向结果编程，而无需关注具体的实现细节。

然而高度非过程化使SQL语言缺少具体的业务逻辑控制功能，因此嵌入式SQL和过程化SQL应运而生。

嵌入式SQL（Embedded SQL, ESQL）将SQL语句嵌入程序设计与语言（比如C、Java等），借助高级语言的控制功能实现过程化。而过程化SQL(Procedural SQL, PL/SQL)是对SQL的扩展，使其增加了过程化语句功能。因此嵌入式SQL和过程化SQL使SQL具有过程化功能。

本文主要介绍过程化SQL和PostgreSQL的过程化SQL以及怎样使用JDBC调用过程化SQL。

过程化SQL程序的基本结构是块，所有的过程化SQL程序都是由块组成的。过程化SQL块主要有两种类型：命名块和匿名块。

匿名块每次执行时都要进行编译，它不能被存储到数据库中，也不能在其他过程化SQL块中调用，也就是匿名块不能被复用。关于匿名块本文不做具体介绍，本文具体介绍命名块的内容。学会了命名块就相当于学会了匿名块，再去稍微看匿名块的内容就能轻松掌握。

而命名块当然就是能够被复用的了，存储过程和函数就是命名块，它们被编译后保存在数据库中，称为持久性存储模块，可以被反复调用，运行速度较快。
那么什么是存储过程和函数呢？它们之间有什么区别呢（几乎没什么区别）？下面就来结合JDBC与PostgreSQL具体介绍:

# 二、存储过程

存储过程是由过程化SQL语句书写的过程，这个过程经编译和优化后存储在数据库服务器中，因此称它为存储过程，使用时只要调用即可。

存储过程具有以下优点：

- 由于存储过程会先经过编译和优化后存储在数据库服务器中，因此存储过程不像解释执行的SQL语句那样在提出操作请求时才进行语法分析和优化工作，因而运行效率高，它提供了在服务器端快速执行SQL语句的有效途径。
- 客户机上的应用程序只要通过网络向服务器发出调用存储过程的名字和参数，就可以让关系数据库管理系统执行其中的多条SQL语句并进行数据处理。只有最终的处理结果才返回客户端。因此，存储过程降低了客户机和服务器之间的通信量。
- 方便实施企业规则。可以把企业规则的运算程序写成存储过程放入数据库服务器中，由关系数据库管理系统管理，既有利于集中控制，又能够方便地进行维护。当企业规则发生变化时只要修改存储过程即可，无须修改其他应用程序。
      

QL中存储过程的语法是：

```plsql
CREATE OR REPLACE PROCEDURE 过程名（[参数1, 参数2, ...])       /* 函数过程首部 */
AS <过程化SQL块>；                                                               /* 存储过程体，描述该存储过程的操作 */
```


由于PostgreSQL不支持存储过程，而只支持自定义函数，所以下面来介绍函数。
# 三、函数

函数也称为自定义函数。函数和存储过程类似，都是持久性存储模块。函数的定义和存储过程也类似，不同之处是函数必须指定返回的类型。

SQL中函数的定义语句格式为：
    

```plsql
CREATE OR REPLACE FUNCTION 函数名 （[参数 1, 参数2, ...]) RETURNS <类型>
AS <过程化SQL块>;
```

函数的执行语句格式：
    
```plsql
CALL/SELECT 函数名([参数1, 参数2,...]);
```

下面使用PostgreSQL来编写几个函数的例子：
    

## ①具有加法功能的函数：

可以发现PostgreSQL的定义函数的语法和SQL的定义函数的语法基本一致，其中

关键字表示定义字符串，以避免字符串内部的转义操作，而LANGUAGE plpgsql用于标明使用的是PostgreSQL的PL/pgSQL过程语言，PostgreSQL还支持其他过程语言，比如PL/Tcl、PL/Perl以及PL/Python，所以需要标明使用的是哪种过程化语言。

以上函数定义中，首先声明了一个integer类型的result变量用于存放加法计算的结果，最后作为函数返回值返回。

现在使用JDBC来让JAVA应用程序调用这个函数：
    

```java
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
 
public class JdbcPL {
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
		
		// 3. 预编译SQL语句，返回CallableStatement对象
		String sql = "SELECT add(?, ?);";
		CallableStatement callState = conn.prepareCall(sql);
		
		// 4. 设置输入参数
		callState.setInt(1, 100);	// 第一个参数设置为100
		callState.setInt(2, 200);	// 第二个参数设置为200
		
		// 5. 发送参数，执行函数，接收结果
		ResultSet rs = callState.executeQuery();
		
		// 6. 取出结果
		while(rs.next()) {
			System.out.println(rs.getInt(1));			// 打印 300
		}
	}
}
```

调用函数需要使用Connection类的prepareCall()方法预编译SQL，同时它将会返回一个CallableSatement对象，然后再像PreparedStatement类那样设置参数，执行SQL语句，最后再取出结果就好。只要注意调用函数使用的是CallableStatement类而不是PreparedStatement类就好。

上面的add方法使用result来充当返回值，定义一个输出参数也能达到类似的结果，来看下面的例子：

## ②具有加法功能的函数（使用输出函数）

现在使用JDBC来调用这个函数：

```java
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;

public class JdbcPL2 {
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
	
	// 3. 预编译SQL语句，返回CallableStatement对象
	String sql = "{CALL add(?, ?, ?)}";
	CallableStatement callState = conn.prepareCall(sql);
	
	// 4. 设置输入参数与输出参数
	callState.setInt(1, 100);		// 第一个输入参数设置为100
	callState.setInt(2, 200);		// 第二个输入参数设置为200
		// 将输出参数注册为JDBC类型INTEGER
	callState.registerOutParameter(3, java.sql.Types.INTEGER);
	
	// 5. 发送参数，执行函数，接收结果
	callState.execute();
	
	// 6. 取出结果
	int output = callState.getInt(3);	// 注意这里参数是3
	
	System.out.println(output);     // 打印300
	}
}
```

 调用具有输出参数的函数，SQL语句需要用{ }包裹，并且需要使用CallableStatement对象注册输出参数为JDBC类型，最后取出结果也不再使用ResultRest类，而是通过CallableStatement类的getXXX()方法直接取出，注意索引参数要正确！

现在来写一个返回多行记录的函数的示例:

## ③查询以下Student表的中年龄（sage）为18的学生的记录：

使用JDBC调用queryStudent()函数：

```java
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JdbcPL3 {
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
	
	// 3. 预编译SQL语句，返回CallableStatement对象
	String sql = "SELECT * FROM queryStudent(?)";
	CallableStatement callState = conn.prepareCall(sql);
	
	// 4. 设置参数
	callState.setInt(1, 18);
	
	// 5. 发送SQL语句，接收查询结果
	ResultSet rs = callState.executeQuery();
	
	// 6. 取出结果
	while(rs.next()) {
		String sno = rs.getString(1);
		String sname = rs.getString(2);
		String ssex = rs.getString(3);
		int sage = rs.getInt(4);
		String sdept = rs.getString(5);
		System.out.println("sno:" + sno + ", sname:" + sname + 
				", ssex:" + ssex + ", sage:" + sage + ", sdept:" + sdept);
	}
}
}
```

查询结果：

可以让函数返回类型是表（TABLE），然后使用 SELECT * FROM 函数 这种语法来调用函数。
