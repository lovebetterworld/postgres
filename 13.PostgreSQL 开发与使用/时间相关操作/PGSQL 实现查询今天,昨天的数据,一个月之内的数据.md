# PGSQL 实现查询今天,昨天的数据,一个月之内的数据

### PGSQL查询今天的数据

```plsql
select *
 from 表名 as n
 where n.create_date>=current_date;
```

### PG查询昨天的数据

**方法1：**

```plsql
 select *
 from 表名 as n
 where
    age(
    current_date,to_timestamp(substring(to_char(n.create_date, 'yyyy-MM-dd hh24 : MI : ss' ) FROM 1 FOR 10),'yyyy-MM-dd')) ='1 days';
```

**方法2：**

```plsql
select *
 from 表名 as n
 where n.create_date>=current_date-1 and n.create_date <current_date;
```

n.create_date 是一个timestamp的数据;

current_date是pgsql数据一个获取当前日期的字段；

to_char（timestamp，text）把timestamp数据转换成字符串；

substring(text from int for int) 截取想要的文本格式 ‘yyyy-MM-dd'；

to_timestamp（text,'yyyy-MM-dd'）转换成timestamp格式；

age（timestamp,timestamp）获取两个时间之差 返回 days

### PG查询最近一个月内的数据

```plsql
select *
 from 表名 as n
 and n.create_date>=to_timestamp(substring(to_char(now(),'yyyy-MM-dd hh24:MI:ss') FROM 1 FOR 10),'yyyy-MM-dd')- interval '30 day';
```

**补充：postgresql 查询当前时间**

需求：PostgreSQL中有四种获取当前时间的方式。

### 解决方案：

**1.now()**

![img](https://img.jbzj.com/file_images/article/202101/20210128094539.jpg)

返回值：当前年月日、时分秒，且秒保留6位小数。

**2.current_timestamp**

![img](https://img.jbzj.com/file_images/article/202101/20210128094548.jpg)

返回值：当前年月日、时分秒，且秒保留6位小数。（同上）

申明：now和current_timestamp几乎没区别，返回值相同，建议用now。

**3.current_time**

![img](https://img.jbzj.com/file_images/article/202101/20210128094557.jpg)

返回值：时分秒，秒最高精确到6位

**4.current_date**

![img](https://img.jbzj.com/file_images/article/202101/20210128094606.jpg)

返回值：年月日