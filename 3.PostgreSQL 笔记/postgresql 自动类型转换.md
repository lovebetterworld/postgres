- [postgresql 自动类型转换 - JackGIS - 博客园 (cnblogs.com)](https://www.cnblogs.com/tiandi/p/14611347.html)
- [PostgreSQL 自定义自动类型转换操作(CAST)_PostgreSQL_脚本之家 (jb51.net)](https://www.jb51.net/article/203366.htm)
- [PostgreSQL 自定义自动类型转换(CAST)-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/228271)
- [PostgreSQL 整型int与布尔boolean的自动转换设置(含自定义cast与cast规则介绍)_weixin_33816300的博客-CSDN博客](https://blog.csdn.net/weixin_33816300/article/details/90001704)

## 1 错误案例

### 1.1 赋值错误：

`ERROR druid.sql.Statement:149 - {conn-10005, pstmt-20005} execute error. UPDATE sys_permission SET parent_id=? is_leaf=? internal_or_external=? WHERE id=?`

`org.postgresql.util.PSQLException: 错误: 字段 "is_leaf" 的类型为 integer`, 但表达式的类型为 boolean   建议：你需要重写或转换表达式 位置：126

解决方法：

```sql
update pg_cast set castcontext='a' where castsource ='boolean'::regtype and casttarget='integer'::regtype;  
```

### 1.2 查询错误：

`ERROR druid.sql.Statement:149 - {conn-10005, pstmt-20003} execute error. SELECT * FROM QRTZ_FIRED_TRIGGERS WHERE SCHED_NAME = 'quartzScheduler' AND INSTANCE_NAME = ? AND REQUESTS_RECOVERY = ?`

`org.postgresql.util.PSQLException: 错误: 操作符不存在: character varying = boolean`

建议：没有匹配指定名称和参数类型的操作符. 您也许需要增加明确的类型转换.

解决方法：

```sql
update pg_cast set castcontext='i' where castsource ='boolean'::regtype and casttarget='text'::regtype;
update pg_cast set castcontext='i' where castsource ='boolean'::regtype and casttarget='character'::regtype;
update pg_cast set castcontext='i' where castsource ='boolean'::regtype and casttarget='character varying'::regtype;
```

用该方法之后应该可以执行转换了，但是有可能把true转换成"true"存到数据库中，这个时候，就需要用实例2的方法了

## 2 知识点总结

PostgreSQL 数据类型直接有3种转换，隐式转换，赋值转换，显式转换；对应的转换类型存在系统表 pg_cast中
三种方式分别对应 i（Implicit），a（Assignment），e（Explicit）

查询所有转换：

```sql
select castsource::regtype,casttarget::regtype,castcontext,castfunc from pg_cast where castsource='boolean'::regtype;   
```

三者的转换关系为 i > a > e；意思是可以隐式转换的一定可以赋值转换和显式转换；可以赋值转换的一定可以显式转换

可以显式转换的不能隐式转换和赋值转换。

**i：主要用户表达式和赋值，a主要用于赋值。**

[https://github.com/TheFrancisHe/Postgresql/blob/master/Postgresql%E4%B8%AD%E7%9A%84%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%26%26pg_cast.md](https://github.com/TheFrancisHe/Postgresql/blob/master/Postgresql中的类型转换%26%26pg_cast.md)

[https://github.com/digoal/blog/blob/master/201801/20180131_01.md](https://github.com/TheFrancisHe/Postgresql/blob/master/Postgresql中的类型转换%26%26pg_cast.md)

相关文章：

https://blog.csdn.net/qq_39727113/article/details/105371412

##  3pg_cast中有的可以进行update，如果没有的可以create新的cast

### 3.1 自动将TEXT转换为TIMESTAMP

  如果没有内置的转换函数，我们可能需要自定义转换函数来支持这种转换。(如果不需要转换函数：WITHOUT FUNCTION)

```sql
--创建转换fun
create or replace function cast_text_to_timestamp(text) returns timestamptz as $$  
  select to_timestamp($1, 'yyyy-mm-dd hh24:mi:ss');  
$$ language sql strict ;  

--创建cast规则
create cast (text as timestamptz) with function cast_text_to_timestamp as ASSIGNMENT;

--使用cast进行操作
insert into tbl123 values (1, text '2017-01-01 10:00:00');
```

### 3.2 自动将boolean转成text类型

```sql
//创建fun
create or replace function bool_to_character(boolean) returns text as $$        
  select CAST($1::int as text);
$$ language sql strict;

--这行创建cast会报错：类型 boolean 到 text 的转换已经存在，因此这行执行不了，只能用下面3行代码的方法进行update
create  cast (bool as text) with function bool_to_character(boolean) as implicit; 


--更新cast（3种text类型都要处理character varying，character，text）,先找fun的id，然后再找casttarget
update pg_cast set castcontext='i',castfunc=42127 where castsource ='boolean'::regtype and casttarget='text'::regtype;

--查询casttarget
select castsource::regtype,casttarget::regtype,castcontext,castfunc from pg_cast where castsource='boolean'::regtype;

--根据fun查询id：
SELECT oid,proname FROM pg_proc WHERE proname='bool_to_character'
```

## 4 cast创建语法

```sql
CREATE CAST (source_type AS target_type) 
 WITH FUNCTION function_name [ (argument_type [, ...]) ] 
 [ AS ASSIGNMENT | AS IMPLICIT ] 

CREATE CAST (source_type AS target_type) 
 WITHOUT FUNCTION
 [ AS ASSIGNMENT | AS IMPLICIT ] 

CREATE CAST (source_type AS target_type) 
 WITH INOUT 
 [ AS ASSIGNMENT | AS IMPLICIT ]
```

解释：

1、WITH FUNCTION，表示转换需要用到什么函数。

2、WITHOUT FUNCTION，表示被转换的两个类型，在数据库的存储中一致，即物理存储一致。例如text和varchar的物理存储一致。不需要转换函数。

3、WITH INOUT，表示使用内置的IO函数进行转换。每一种类型，都有INPUT 和OUTPUT函数。使用这种方法，好处是不需要重新写转换函数。

除非有特殊需求，我们建议直接使用IO函数来进行转换。

4、AS ASSIGNMENT，表示在赋值时，自动对类型进行转换。例如字段类型为TEXT，输入的类型为INT，那么可以创建一个 cast(int as text) as ASSIGNMENT。

5、AS IMPLICIT，表示在表达式中，或者在赋值操作中，都对类型进行自动转换。（包含了AS ASSIGNMENT，它只对赋值进行转换）

6、注意，AS IMPLICIT需要谨慎使用，为什么呢？因为操作符会涉及到多个算子，如果有多个转换，目前数据库并不知道应该选择哪个？

因此，建议谨慎使用AS IMPLICIT。建议使用AS IMPLICIT的CAST应该是非失真转换转换，例如从INT转换为TEXT，或者int转换为numeric。

而失真转换，不建议使用as implicit，例如numeric转换为int。