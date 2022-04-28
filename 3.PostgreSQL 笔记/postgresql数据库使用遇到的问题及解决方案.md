- [postgresql数据库使用遇到的问题及解决方案_Master_Shifu_的博客-CSDN博客](https://blog.csdn.net/Master_Shifu_/article/details/108144442)

## 1.column is of type [timestamp](https://so.csdn.net/so/search?q=timestamp&spm=1001.2101.3001.7020) without time zone but expression is of type character varying解决

column is of type timestamp without time zone but expression is of type character varying解决

java插入postgresql问题：
ERROR: column is of type timestamp without time zone but expression is of type character varying
建议：You will need to rewrite or cast the expression.

解决：
jdbc:postgresql://127.0.0.1:5432/testdb?stringtype=unspecified
后面添加stringtype=unspecified，即自动格式化数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200821111644155.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hc3Rlcl9TaGlmdV8=,size_16,color_FFFFFF,t_70#pic_center)

## 2.关于jdbc向mysql和postgresql批量插入大量数据时候的优化！

1.对于mysql数据库，driverurl中加入:allowMultiQueries=true&rewriteBatchedStatements=true; 这样在使用jdbctemplate插入的时候，类似:

```java
private void insertData(JdbcTemplate insertJdbcTemplate, String insertCoreSql, List<Map<String, Object>> dataList, int cols) {
        insertJdbcTemplate.batchUpdate(insertCoreSql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement preparedStatement, int i) throws SQLException {
                Map<String, Object> dataMap = dataList.get(i);
                for (int index = 1; index <= cols; index++) {
                    preparedStatement.setObject(index, dataMap.get(index + ""));
                }
            }
 
            @Override
            public int getBatchSize() {
                return dataList.size();
            }
        });
123456789101112131415
```

使用[mybatis](https://so.csdn.net/so/search?q=mybatis&spm=1001.2101.3001.7020)批量插入的时候，类似foreach转成insert into values(),(),(),()…这总， 性能有巨大提升！

2.对于postgresql数据库，或基于postgresql的greenplum数据仓库，driverurl中加入:reWriteBatchedInserts=true,这个是从postgresql的jdbc高版本驱动(9.4.1209开始加，但是有bug)才加入的特性，详细见：https://jdbc.postgresql.org/documentation/changelog.html#version_42.2.2，建议升级驱动到42.2.2版本，否则即使你将sql写成insert into values(),(),(),()这种形式，一样被转化成单条插入。

## 3.PostgreSQL数据库使用函数生成[uuid](https://so.csdn.net/so/search?q=uuid&spm=1001.2101.3001.7020)

```sql
**uuid_generate_v1**
postgres=# select uuid_generate_v1();

--------------------------------------
86811bd4-22a5-11df-b00e-ebd863f5f8a7
(1 row)

**uuid_generate_v4**
postgres=# select uuid_generate_v4();

--------------------------------------
5edbfcbb-1df8-48fa-853f-7917e4e346db
(1 row)
12345678910111213
```

主要就是uuid_generate_v1和uuid_generate_v4，当然还有uuid_generate_v3和uuid_generate_v5。

其他使用可以参见PostgreSQL官方文档 http://www.postgresql.org/docs`在这里插入代码片`/8.3/static/uuid-ossp.html

## 4.postgresql 如何设置[主键](https://so.csdn.net/so/search?q=主键&spm=1001.2101.3001.7020)defaultValue为uuid

–更新字段索引

```sql
alter table t_mqtt_env_data alter column id set default (MD5(cast(uuid_generate_v4() as VARCHAR))); 
1
```

因为直接使用uuid_generate_v4生成uuid会有一定的几率重复,而且需要自己去下划线,所以在使用md5再转一下,保证长度只有32位并且不会重复

## 5.mybatis中取自增列的值

其中order分
order=“BEFORE”(再插入之前查询),查询的结果放入vo的主键id上,再之后的insert语句中插入的#{id,jdbcType=VARCHAR}取到当前插入id
order=“AFTER”, 看单词意思就是在插入之后获取插入的主键id,一般建议在之前获取

使用注意事项: 该获取主键方法只适用于一条插入语句的情况,如果是一个list批量插入,则需要单独获取主键在java代码中设置值或者直接设置数据库主键默认自增字段,就可以减去自己获取主键在设置的烦恼

```xml
<selectKey keyProperty="pkId" resultType="String" order="BEFORE">
    select MD5(cast(uuid_generate_v4() as VARCHAR))
</selectKey>
123
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200821113940549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hc3Rlcl9TaGlmdV8=,size_16,color_FFFFFF,t_70#pic_center)

## 6.创建表内容之–java数据类型对应postgresql的数据类型

| java8          | postgreSQL                 |
| -------------- | -------------------------- |
| LocalDate      | date                       |
| LocalTime      | time                       |
| LocalDateTime  | timestamp without timezone |
| OffsetDateTime | timestamp with timezone    |
| String         | varchar                    |
| String         | text                       |
| Integer        | int2                       |
| Integer        | int4                       |
| Long           | int8                       |
| Float          | float4                     |
| Double         | float8                     |
| BigDecimal     | numeric                    |
| Boolean        | bool                       |

## 7.创建表内容之–postgresql 如何设置主键自增

法一：

```sql
CREATE TABLE customers  

(  

  customerid SERIAL primary key ,  

  companyname character varying,  

  contactname character varying,  

  phone character varying,  

 country character varying  

)  
123456789101112131415
```

法二

```sql
CREATE SEQUENCE gateway_id_seq  

START WITH 1  

INCREMENT BY 1  NO MINVALUE  NO MAXVALUE  

CACHE 1;

alter table event alter column id set default nextval('gateway_id_seq'); 
123456789
```

**应用:pg数据库主键生成固定字符加自增序列的默认主键**
–更新字段索引

```sql
alter table t_mqtt_env_data alter column id set default ('lcc'||nextval('gateway_id_seq')); 
1
```

## 8.创建表内容之–sql – 使用liquibase标签将最大列值设置为序列起始值

当时用图二语句创建主键默认值时,默认值不会生效,初步猜测liquibase识别到有单引号,所以语句被跳过了,那有没有另外一种方法执行呢,有的,直接写sql就可以了

alter table XXX alter column id set default ('lcc'||nextval('gateway_id_seq'))

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020082112494827.png#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200821124613454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01hc3Rlcl9TaGlmdV8=,size_16,color_FFFFFF,t_70#pic_center)

## 9.PostgreSQL的时间/日期函数使用

PostgreSQL的常用时间函数使用整理如下：

一、获取系统时间函数

1.1 获取当前完整时间
**now**
select now();

2013-04-12 15:39:40.399711+08
(1 row)

------

**current_timestamp** 同 now() 函数等效。

2013-04-12 15:40:22.398709+08
(1 row)

二、时间的计算
2.3 三周前

select now() - interval ‘3 week’;

2013-03-22 16:00:04.203735+08
(1 row)

------

**当前时间+十分钟后**

select now() + ‘10 min’;

2013-04-12 16:12:47.445744+08
(1 row)

说明：

interval 可以不写，其值可以是：

| Abbreviation | Meaning                    |
| ------------ | -------------------------- |
| Y            | Years                      |
| M            | Months (in the date part)  |
| W            | Weeks                      |
| D            | Days                       |
| H            | Hours                      |
| M            | Minutes (in the time part) |
| S            | Seconds                    |

**应用:**

------

当前时间+一秒后
select now() +‘1 second’
在数据库中使用creationDate查询时间上下边界的时候,结束时间一般需要+1s才能查询出完整的下界时间
如:
creation_date <= to_timestamp(‘2020-08-20 16:20:08’,‘yyyy-MM-dd HH24:mi:SS’ )
如果不加 +‘1 second’, 则查询的时间区间是 = 2020-08-20 16:19:42 到 =2020-08-20 16:20:07
我们一般传的下界是需要等于,且是包含在时间区间内的,所以在下界中 +'1 second’是最好的处理方式,
否则creation_date也需要做time_timestamp的处理

```sql
SELECT * 
FROM
	XXX 
WHERE
	active_flag = 'Y'
	
	and creation_date >= to_timestamp('2020-08-20 16:19:42','yyyy-MM-dd HH24:mi:SS' )
	and creation_date <= to_timestamp('2020-08-20 16:20:08','yyyy-MM-dd HH24:mi:SS' )+'1 second'
12345678
```

其他时间方面的使用方法参考如下链接:
[PostgreSQL的时间/日期函数使用](https://www.cnblogs.com/mchina/archive/2013/04/15/3010418.html)

## 10.PostgreSQL标准建表语句

```sql
DROP TABLE IF EXISTS "t_table_a";
CREATE TABLE "t_table_a" (
  "id" varchar(32) PRIMARY KEY NOT NULL ,
  "type" int4,
  "creation_date" timestamp without time zone DEFAULT now(),
  "last_update_time" timestamp without time zone DEFAULT now(),
  "created_by" varchar(100) DEFAULT 'system',
  "last_updated_by" varchar(100) DEFAULT 'system',
  "ope" char(1),
  "active_flag" char(1) DEFAULT 'Y',
  "soo" NUMERIC(6,2),
  "extra_field" varchar(2000)
)with (oids = false);

COMMENT ON COLUMN "t_table_a"."id" IS '主键ID';
COMMENT ON COLUMN "t_table_a"."type" IS '消息类型信息';
COMMENT ON COLUMN "t_table_a"."creation_date" IS '数据创建时间';
COMMENT ON COLUMN "t_table_a"."last_update_time" IS '数据最后更新时间';
COMMENT ON COLUMN "t_table_a"."created_by" IS '数据创建人';
COMMENT ON COLUMN "t_table_a"."last_updated_by" IS '数据最后更新人';
COMMENT ON COLUMN "t_table_a"."ope" IS '';
COMMENT ON COLUMN "t_table_a"."active_flag" IS '数据是否激活标志位,激活状态为为Y,失效状态为N';
COMMENT ON COLUMN "t_table_a"."soo" IS 'SOO 数据;
COMMENT ON COLUMN "t_table_a"."extra_field" IS '数据扩展字段,建议使用JSON格式存储';


-- 主键 （如果建表语句里面没添加主键就执行该语句）
alter table t_table_a alter column id set default ('lcc'||nextval('gateway_id_seq'));


-- 索引或唯一索引
CREATE INDEX "index_creation_Date.env_data" ON "t_table_a" USING btree (
  "creation_date" "pg_catalog"."timestamp_ops" DESC NULLS LAST
);
COMMENT ON INDEX "index_creation_Date.env_data" IS '创建时间索引';

-- 授权
GRANT ALL ON TABLE public.user TO mydata;
GRANT SELECT, UPDATE, INSERT, DELETE ON TABLE public.user TO mydata_dml;
GRANT SELECT ON TABLE public.user TO mydata_qry;


--如果没有创建序列,用以下语句创建序列
CREATE SEQUENCE gateway_id_seq  

START WITH 1  

INCREMENT BY 1  NO MINVALUE  NO MAXVALUE  

CACHE 1;
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950
```

**NUMERIC(6,2)介绍**: 精确浮点型
MySQL中称为精确浮点型，PG中原英文是Arbitrary Precision Numbers，翻译成中文是任意精度数，那么也好理解，decimal可以指定精确度，例如，数字23.5141的精度为6，小数位数为4。可以将整数视为小数位数为零，那么定义的时候即为decimal(6,4)或者NUMERIC（6,4)。实际存储空间类似于varchar(n)，两个字节对于每组四个十进制数字，再加上三到八个字节的开销。
其他关于浮点或者货币类型介绍可参考如下链接:
[第八章、PG数据类型（数字类型、货币类型、字符串类型）【一】](https://www.cnblogs.com/yaochong-chongzi/p/12637681.html)

## 11.mybatis常用jdbcType数据类型

Mybatis中javaType和jdbcType对应和CRUD例子

```xml
<resultMap type="java.util.Map" id="resultjcm">
  <result property="FLD_NUMBER" column="FLD_NUMBER"  javaType="double" jdbcType="NUMERIC"/>
  <result property="FLD_VARCHAR" column="FLD_VARCHAR" javaType="string" jdbcType="VARCHAR"/>
  <result property="FLD_DATE" column="FLD_DATE" javaType="java.sql.Date" jdbcType="DATE"/>
  <result property="FLD_INTEGER" column="FLD_INTEGER"  javaType="int" jdbcType="INTEGER"/>
  <result property="FLD_DOUBLE" column="FLD_DOUBLE"  javaType="double" jdbcType="DOUBLE"/>
  <result property="FLD_LONG" column="FLD_LONG"  javaType="long" jdbcType="INTEGER"/>
  <result property="FLD_CHAR" column="FLD_CHAR"  javaType="string" jdbcType="CHAR"/>
  <result property="FLD_BLOB" column="FLD_BLOB"  javaType="[B" jdbcType="BLOB" />
  <result property="FLD_CLOB" column="FLD_CLOB"  javaType="string" jdbcType="CLOB"/>
  <result property="FLD_FLOAT" column="FLD_FLOAT"  javaType="float" jdbcType="FLOAT"/>
  <result property="FLD_TIMESTAMP" column="FLD_TIMESTAMP"  javaType="java.sql.Timestamp" jdbcType="TIMESTAMP"/>
 </resultMap>
12345678910111213
```

其他关于mybatis常用jdbcType数据类型介绍可参考如下链接:
[mybatis常用jdbcType数据类型](https://www.cnblogs.com/lixuwu/p/5916585.html)