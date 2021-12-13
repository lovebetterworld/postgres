## PgSQL批量插入测试数据

## 1 pgsql批量插入测试数据测试

- [pgsql批量插入测试数据测试](https://blog.csdn.net/Lei_Da_Gou/article/details/104376092)

### 1.1 测试准备

```sql
-- 1.创建测试表t_user
create table if not exists t_user(
	id serial primary key,
	user_name varchar(255),
	pass_word varchar(255),
	create_time date,
	dr char(1)
)
 
-- 2.注释
comment on column t_user.id is '测试表';
comment on column t_user.user_name is '账号';
comment on column t_user.pass_word is '密码';
comment on column t_user.create_time is '创建日期';
comment on column t_user.dr is 'delete remark';
```

### 1.2 测试4种不同的批量插入测试数据效率

#### 1.2.1 创建存储过程

```sql
-- 创建存储过程插入数据
create or replace function batch_insert_proc(num int) returns void as 
$$
begin
	while num > 0 loop
		insert into t_user(user_name,pass_word,create_time,dr) values('username'||round(random()*num),'password'||round(random()*num),now(),0);
		num = num -1;
	end loop;
exception
	when others then
	raise exception'(%)',SQLERRM;
end;
$$ language plpgsql;
```

#### 1.2.2 直接调用存储过程

```sql
-- 插入100*10000条数据
select batch_insert_proc(100*10000); 
```

#### 1.2.3 尝试关闭自动提交调用存储过程

```sql
-- 也可以使用set autocommit off关闭自动提交
START TRANSACTION;
    select batch_insert_proc(100*10000);
commit;
```

#### 1.2.4 直接使用插入语句

```sql
insert into t_user select generate_series(1,100*10000),'username'||round(random()*1000000),'password'||round(random()*1000000),now(),0;
```

#### 1.2.5 直接使用插入语句并关闭自动提交

```sql
start transaction;
	insert into t_user select         generate_series(1,100*10000),'username'||round(random()*1000000),'password'||round(random()*1000000),now(),0;
commit;
```

### 1.3 测试复制表效率

#### 1.3.1 导出导入数据文件并使用copy复制

```sql
-- 导出数据到文件（100*10000条记录）
copy t_user to 'E:\\sql\\user.sql';
-- 复制表结构
create table t_user_copy as (select * from t_user limit 0);
-- 从数据文件copy数据到新表
copy t_user_copy from 'E:\\sql\\user.sql';
```

#### 1.3.2 直接使用sql语句复制

```sql
-- 测试2：直接复制表结构及数据(100W耗时3.445秒)
create table t_user_copy as (select * from t_user);
```

### 1.4 总结

1、插入测试数据直接使用2.4，复制表数据使用copy即可。

2、postgresql在function不能用start transaction, commit 或rollback,因为函数中的事务其实就是begin...exception.end块。所有你如果要写事务，请直接用begin...exception...end就够了，这也说明了为什么上面开启事务，或者换种说法关闭自动提交根本就没有提升批量插入数据的性能，但是copy确实对性能提升有一定帮助。

3、postgresql在函数（存储过程）中引入了异常的概念，即begin...exception...end块，这里有三种情况，第一种：这个函数体的结构就是一个全局的begin...exception...end块（后文简称bee块），可以把bee块理解为一个事务，当这个函数体，即全局事务出现异常，则全部rollback。第二种情况：如果在某个子bee块中出现异常，则该bee块会回滚，但是不会影响其它bee块。感觉pgsql在函数这块得些细节整得有点晦涩难懂。

## 2 postgres 使用存储过程批量插入数据

- [postgres 使用存储过程批量插入数据](https://www.cnblogs.com/ldxsuanfa/p/10935006.html)

```sql
create or replace function creatData2() returns 
boolean AS
$BODY$
declare ii integer;
  begin
  II:=1;
  FOR ii IN 1..10000000 LOOP
  INSERT INTO ipm_model_history_data (res_model, res_id) VALUES (116, ii);
  end loop;
  return true;
  end;
$BODY$
LANGUAGE plpgsql;
select * from creatData2() as tab;
```

- [postgresql函数存储过程实现数据批量插入](https://www.cnblogs.com/dongyaotou/p/13337042.html)

```sql
create or replace function creatData2() returns 
boolean AS
$BODY$
declare ii integer;
begin
II:=1;
FOR ii IN 1000..2000 LOOP
INSERT INTO "t_member_score_item2"("member_score_item_id", "member_code", "score_type", "score", "sys_no", "remark", "insert_time", "insert_oper", "update_time", "update_oper") VALUES
(ii, '01862204522573', '06', '100.000000', '04', NULL, now(), 'mv', now(), 'mv');
end loop;
return true;
end;
$BODY$
LANGUAGE plpgsql;


select * from creatData2() as tab;
select * from t_member_score_item2
```

## 3 pgsql 批量插入数据（含数组使用）

- [pgsql 批量插入数据（含数组使用）](https://blog.csdn.net/qq_26818839/article/details/110124319)

```sql
do $$
declare 
i integer; -- 定义使用变量
 testArr varchar array; -- 数组定义
begin -- 开始
i :=2200;
testArr := array['张三','李四','王五','六七'];
for index in 1..array_length(testArr, 1) loop -- 循环开始
INSERT INTO public.wares (id,w_name,w_alias,w_unit,w_cost,w_sellprice,w_minnum,w_maxnum,w_description,w_purchase_date,s_id,w_vaild_date,remark,valid_flag,create_date,creator,update_date,updater) VALUES 
(i,testArr[index],testArr[index],'斤',4.000,3.400,20.000,20.000,NULL, now(),NULL,now(),NULL,'1',now(),'管理员',NULL,NULL);
i = i+1;
end loop; -- 循环结束
end $$;
```

## 4 postgreSQL数据库 实现向表中快速插入1000000条数据

```sql
create table tbl_test (id int, info text, c_time timestamp);

--随机字母 
select chr(int4(random()*26)+65);

--随机4位字母
select repeat( chr(int4(random()*26)+65),4);

--随机数字 十位不超过6的两位数
select (random()*(6^2))::integer;

--三位数
select (random()*(10^3))::integer;

insert into t_test SELECT generate_series(1,10000) as key,repeat( chr(int4(random()*26)+65),4), (random()*(6^2))::integer,null,(random()*(10^4))::integer;
```

## 5 批量插入

批量插入是指一次性插入多条数据，主要用于提升数据插入效率，PostgreSQL有多种方法实现批量插入。

### 5.1 方式一：INSERT INTO…SELECT…

通过表数据或函数批量插入，语法如下：

```sql
INSERT INTO table_name SELECT...FROM source_table 
```

比如创建一张表结构和user_ini相同的表并插入user_ini表的全量数据，代码如下所示：

```sql
mydb=> CREATE TABLE tbl_batch1(user_id int8,user_name text);
CREATE TABLE 
mydb=> INSERT INTO tbl_batch1(user_id,user_name) SELECT user_id,user_name FROM user_ini; 
INSERT 0 1000000 
```

以上示例将表user_ini的user_id、user_name字段所有数据插入表tbl_batch1，也可以插入一部分数据，插入时指定where 条件即可。 通过函数进行批量插入，如下所示：

```sql
mydb=> CREATE TABLE tbl_batch2 (id int4,info text); 
CREATE TABLE
mydb=> INSERT INTO tbl_batch2(id,info) 
	   SELECT generate_series(1,5),'batch2'; 
INSERT 0 5 
```

通过SELECT表数据批量插入的方式大多关系型数据库都支持，接下来看看PostgreSQL支持的其他批量插入方式。

### 5.2 方式二：INSERT INTO VALUES（）， （），…（）

PostgreSQL的另一种支持批量插入的方法为在一条 INSERT语句中通过VALUES关键字插入多条记录，通过一个 例子就很容易理解，如下所示：

```sql
mydb=> CREATE TABLE tbl_batch3(id int4,info text); 
CREATE TABLE 
mydb=> INSERT INTO tbl_batch3(id,info) VALUES (1,'a'),(2,'b'),(3,'c'); 
INSERT 0 3 
```

数据如下所示：

```sql
mydb=> SELECT * FROM tbl_batch3; 
id | info 
-------+------ 
	 1 | a 
	 2 | b 
	 3 | c 
(3 rows) 
```

这种批量插入方式非常独特，一条SQL插入多行数据，相 比一条SQL插入一条数据的方式能减少和数据库的交互，减少数据库WAL（Write-Ahead Logging）日志的生成，提升插入效率，通常很少有开发人员了解PostgreSQL的这种批量插入方 式。

### 5.3 方式三：COPY或\COPY元命令

前面介绍了psql导入、导出表数据，使用的是COPY命令或\copy元命令，copy或\copy元命令能够将一定格式的文件数据导入到数据库中，相比INSERT命令插入效率更高，通常大数据量的文件导入一般在数据库服务端主机通过PostgreSQL 超级用户使用COPY命令导入，下面通过一个例子简单看看 COPY命令的效率，测试机为一台物理机上的虚机，配置为4核CPU，8GB内存。 首先创建一张测试表并插入一千万数据，如下所示：

```sql
mydb=> CREATE TABLE tbl_batch4( id int4, info text, create_time timestamp(6) with time zone default clock_timestamp()); 
CREATE TABLE 
mydb=> INSERT INTO tbl_batch4(id,info) SELECT n,n||'_batch4' FROM generate_series(1,10000000) n; 
INSERT 0 10000000 
```

以上示例通过INSERT插入一千万数据，将一千万数据导出到文件，如下所示：

```sql
[postgres@pghost1 ~]$ psql mydb postgres 
psql (10.0) 
Type "help" for help.
mydb=# \timing 
Timing is on. 
mydb=# COPY pguser.tbl_batch4 TO '/home/pg10/tbl_batch4.txt'; 
COPY 10000000 
Time: 6575.787 ms (00:06.576) 
```

一千万数据导出花了6575毫秒，之后清空表tbl_batch4并将文件tbl_batch4.txt的一千万数据导入到表中，如下所示：

```sql
mydb=# TRUNCATE TABLE pguser.tbl_batch4; 
TRUNCATE TABLE 
mydb=# COPY pguser.tbl_batch4 FROM '/home/pg10/tbl_batch4.txt'; 
COPY 10000000 
Time: 15663.834 ms (00:15.664) 
```

一千万数据通过COPY命令导入执行时间为15663毫秒。