- [PostgreSQL和mysql数据类型对比兼容](https://blog.csdn.net/weixin_39540651/article/details/103958411)

## 1、数值类型

### 1.1 整数

mysql中的整数类型和pg相比，两者有以下区别：
mysql：mysql中支持int 1,2,3,4,8  字节，同时支持有符号，无符号。并且mysql中支持在数值列中指定zerofill，用来将存储的数值通过填充0的方式达到指定数据类型的长度(mysql8开始不建议使用ZEROFILL属性，并且在将来的MySQL版本中将不再支持该属性)。
pg：pg支持 int 2,4,8 字节，且数值都是有符号的。

### 1.2 mysql整数类型

![image-20211225003453224](https://gitee.com/er-huomeng/img/raw/master/img/image-20211225003453224.png)

### 1.3 pg整数类型

![image-20211225003445366](https://gitee.com/er-huomeng/img/raw/master/img/image-20211225003445366.png)
那么对于mysql中的1，3字节整型，或者无符号整型以及zerofill特性，在pg中该如何实现呢？

在pg中我们可以使用domain来实现mysql中的1，3字节整数以及无符号整型。

创建uint8，8字节无符号整型

```sql
bill=# create domain uint8 as numeric(20,0) check (value <= ((2^64::numeric)::numeric(20,0)-1) and value>=0::numeric(20,0));  
CREATE DOMAIN
```

使用domain，插入整型数据，且大于等于0，小于2^64

```sql
bill=# create table t5(c1 uint8);  
CREATE TABLE
bill=# insert into t5 values (-1);
ERROR:  value for domain uint8 violates check constraint "uint8_check"
bill=# insert into t5 values (0);  
INSERT 0 1
bill=# insert into t5 values (18446744073709551615);  
INSERT 0 1
bill=# insert into t5 values (18446744073709551616);   
ERROR:  value for domain uint8 violates check constraint "uint8_check"
bill=# select  from t5;  
          c1          
----------------------
                    0
 18446744073709551615
(2 rows)
```

同样我们也可以来创建domain实现1,3字节有无符号整型，2,4,8字节无符号等等：

```sql
create domain int1 as int2 CHECK (VALUE <= 127 AND VALUE >= (-128));  
create domain uint1 as int2 CHECK (VALUE <= 255 AND VALUE >= 0);  
create domain uint2 as int4 CHECK (VALUE <= 65535 AND VALUE >= 0);  
create domain int3 as int4 CHECK (VALUE <= 8388607 AND VALUE >= (-8388608));  
create domain uint3 as int4 CHECK (VALUE <= 16777215 AND VALUE >= 0);  
create domain uint4 as int8 CHECK (VALUE <= 4294967295 AND VALUE >= 0);  
create domain uint8 as numeric(20,0) check (value <= ((2^64::numeric)::numeric(20,0)-1) and value>=0::numeric(20,0));  
```

而对于mysql中的zerofill，我们可以使用lpad函数来实现，并且这也是mysql官方文档中推荐的一种方式。

mysql中zerofill使用方式：

```sql
mysql> create table t1(id int1 zerofill);  
Query OK, 0 rows affected (0.00 sec)

mysql> insert into t1 values(4);
Query OK, 1 row affected (0.00 sec)

mysql> select  from t1;
+------+
| id   |
+------+
|  004 |
+------+
1 row in set (0.00 sec)
```

pg中使用lpad函数替代：

```sql
bill=# create table t1(id int);  
CREATE TABLE
bill=# insert into t1 values (123),(123456);  
INSERT 0 2
bill=# select lpad(id::text, greatest(4, length(id::text)), '0'), id from t1; 
  lpad  |   id   
--------+--------
 0123   |    123
 123456 | 123456
(2 rows)
```

**numeric类型：**

pg和mysql一样都支持decimal，numeric类型来表示浮点数。两者的区别在于：mysql中的numeric类型整数和小数部分均最大支持65digits。

而pg中numeric类型支持的最大范围是：[左131072,右16383]digits。

例如:

–mysql中

```sql
mysql> create table t1(id numeric(66,1));
ERROR 1426 (42000): Too-big precision 66 specified for 'id'. Maximum is 65.
mysql> create table t1(id numeric(65,1)); 
Query OK, 0 rows affected (0.01 sec)
```

–pg中

```sql
bill=# create table t4(id numeric(66,1)); 
CREATE TABLE
```

**浮点类型：**

mysql和pg中的浮点数类型基本一致。mysql中4 bytes的浮点数类型有real，float4，4  bytes的浮点数类型double。pg中对应的也有real，float，float4，float8以及double  precision，两者基本兼容。

**bit类型：**

mysql中bit类型一般都是使用整数类型表示，所以支持的bit位数最大只能是64位。而在pg中有专门的bit类型bit(范围1～83886080)，以及可变长度的bit类型varbit。

**序列：**

mysql中创建表时可以使用auto_increment来创建自增列，从而生成一个和该列相关的序列，这个和pg中创建serial类型的列类似，但是两者仍然有明显区别：

mysql：使用auto_increment的自增列必须要建索引，不然会报错。序列的默认初始值是1，步长为1.可以通过修改auto_increment_increment和auto_increment_offset来修改初始值和步长。

pg：pg中创建serial类型的列时会创建相应的序列，支持的数据类型有serial2，serial4，serial8。同时pg创建序列时可以直接指定初始值，步长，maxvalue，

cache，circle等参数。其中序列cache预取多个值，可以保证没有性能问题。circle可以指定序列达到最大值后从初始值开始重新计数。

–mysql

```sql
mysql>  create table t4 (id int auto_increment primary key);
Query OK, 0 rows affected (0.06 sec)
```

–PostgreSQL

```sql
bill=# create table t4(id serial);
CREATE TABLE
```

## 2、时间类型

mysql：mysql中时间相关的类型有日期date、时间time以及datetime、timestamp和year类型。

pg：pg中的时间数据类型基本和mysql一致。区别在于pg中支持timez类型，即带时区的时间类型，这个mysql中不支持，但是pg中不支持mysql中的year类型，不过我们仍然可以通过创建domain的方式来在pg中实现year类型。

mysql中的year类型表示年份从 1901年到2155。

pg中实现mysql中year类型的方式：

```sql
bill=# create domain year as int2 check(value >=1901 and value <=2155);  
CREATE DOMAIN
bill=# create table ts4(c1 year);  
CREATE TABLE
bill=# insert into ts4 values (1000);
ERROR:  value for domain year violates check constraint "year_check"
bill=# insert into ts4 values (2019);
INSERT 0 1
bill=# insert into ts4 values (2156);
ERROR:  value for domain year violates check constraint "year_check"
```

## 3、字符串类型

### 3.1 char/varchar类型

mysql和pg中都支持char类型来表示固定长度的字符串，varchar类型表示可变长度的字符串类型，两者的区别在于：

mysql：char类型最大255字符，varchar类型最大不超过64字节。
pg：char类型最大10485760字符，varchar类型最大1G字节。同时pg中还支持两种特殊的字符串类型：name类型，固定64字节长度，char类型(即不指定长度)，固定1字节长度。

### 3.2 binary/varbinary类型

mysql中binary(n)最大255个字符，varbinary(n)最大不超过64k字节。使用字节流来存储字符串。而pg中使用bytea类型来表示二进制类型，最大不超过1G字节。

### 3.3 blob/text类型

mysql中的blob/text类型分别有以下几种：
tinyblob、tinytext < 2^8字节
blob、text < 2^16字节
mediumblob、mediumtext < 2^24字节
longblob、longtext < 2^32字节

pg中对应的使用bytea类型和text类型，两者最大长度均为1G字节。

### 3.5 enum类型

mysql中的枚举类型最大不超过64K个值，而pg中最大为1GB

### 3.6 set类型

mysql中的集合set类型表示没有重复值的集合，最大64个值，在pg中虽然没有set类型，但是可以通过数组类型去代替，最大支持1GB大小。

## 4、其它类型

json类型：

mysql和pg中的json类型基本一致，区别在于默写json函数可能稍有区别。不过pg中json类型有2种json和jsonb，不过一般都使用jsonb类型。

除了上面列举的这些类型之外，pg中还支持很多mysql中不支持的数据类型。例如：pg中支持IP地址类型，这个在mysql中常常都是使用int或者varchar之类的数据类型代替。

除此之外还有很多pg内置的数据类型在mysql中是不支持的：货币、interval、平面几何、全文检索、uuid、xml、数组、复合类型、范围类型、域类型等等。
同时pg还有很多外置的数据类型：树类型、多维类型、化学分子、DNA、postgis等等类型。