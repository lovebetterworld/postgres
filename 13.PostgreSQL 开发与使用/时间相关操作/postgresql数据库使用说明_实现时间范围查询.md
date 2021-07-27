# postgresql数据库使用说明_实现时间范围查询

## 按照日期查询通常有好几种方法：

按照日期范围查询有好几种方法，日期字段类型一般为：

```plsql
Timestamp without timezone
```

### 方法一：

```plsql
select * from user_info where create_date
>= '2015-07-01' and create_date < '2015-08-15';
```

### 方法二：

```plsql
select * from user_info where create_date
between '2015-07-01' and '2015-08-15';
```

### 方法三：

```plsql
select * from user_info where create_date
>= '2015-07-01'::timestamp and create_date < '2015-08-15'::timestamp;
```

### 方法四：

```plsql
select * from user_info where create_date
between to_date('2015-07-01','YYYY-MM-DD') and to_date('2015-08-15','YYYY-MM-DD');
```

pandas.to_sql 遇到主键重复的，怎么能够跳过继续执行呢，其实很简单，就一条一条的插入就可以了，因为to_sql还没有很好的解决办法。

**具体的代码如下所示：**

```plsql
  for exchange in exchange_list.items():
    if exchange[1]==True:
      pass
    else:
      continue
    sql = """ SELECT * FROM %s WHERE "time" BETWEEN '2019-07-05 18:48' AND '2019-07-09' """ % (exchange[0])
    data = pd.read_sql(sql=sql, con=conn)
    print(data.head())
    for i in range(len(data)):
      #sql = "SELECT * FROM `%s` WHERE `key` = '{}'"%(exchange).format(row.Key)
      #found = pd.read_sql(sql, con=conn2)
      #if len(found) == 0:
      try:
        data.iloc[i:i + 1].to_sql(name=exchange[0], index=False,if_exists='append', con=conn2)
      except Exception as e:
        print(e)
        pass
```

pandas.to_sql 无法设置主键，这个是肯定的，能做的办法就是在to_sql之前先使用创建表的方法，创建一张表

**建表的代码如下所示：**

```plsql
/*
Create SEQUENCE for table
*/
DROP SEQUENCE IF EXISTS @exchangeName_id_seq;
CREATE SEQUENCE @exchangeName_id_seq
START WITH 1
INCREMENT BY 1
NO MINVALUE
NO MAXVALUE
CACHE 1;
 
/*
Create Table structure for table
*/
DROP TABLE IF EXISTS "public"."@exchangeName";
CREATE TABLE "public"."@exchangeName" (
 "id" int4 NOT NULL DEFAULT nextval('@exchangeName_id_seq'::regclass),
 "time" timestamp(6) NOT NULL,
 "open" float8,
 "high" float8,
 "low" float8,
 "close" float8,
 "volume" float8,
 "info" varchar COLLATE "pg_catalog"."default" NOT NULL
)
;
 
/*
Create Primary Key structure for table
*/
ALTER TABLE "public"."@exchangeName" DROP CONSTRAINT IF EXISTS "@exchangeName_pkey";
ALTER TABLE "public"."@exchangeName" ADD CONSTRAINT "@exchangeName_pkey" PRIMARY KEY ("time", "info");
```

**补充：postgresql 数据库时间间隔数据查询**

当前时间向前推一天：

```plsql
SELECT current_timestamp - interval '1 day'
```

当前时间向前推一个月：

```plsql
SELECT current_timestamp - interval '1 month'
```

当前时间向前推一年：

```plsql
SELECT current_timestamp - interval '1 year'
```

当前时间向前推一小时：

```plsql
SELECT current_timestamp - interval '1 hour'
```

当前时间向前推一分钟：

```plsql
SELECT current_timestamp - interval '1 min'
```

当前时间向前推60秒：

```plsql
SELECT current_timestamp - interval '60 second'
```