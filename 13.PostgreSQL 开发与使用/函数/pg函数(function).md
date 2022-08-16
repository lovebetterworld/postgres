- [pg函数(function)_进无止尽-如履平地的博客-CSDN博客_pg 函数](https://blog.csdn.net/weixin_40245601/article/details/108196727)

# 函数的格式

```sql
Create or replace function 过程名(参数名 参数类型,…..) returns 返回值类型 as
                   $body$
                            //声明变量
                            Declare
                            变量名变量类型；
                            如：
                            flag Boolean;
                            变量赋值方式（变量名类型 ：=值；）
                            如：
                            str  text :=值; / str  text;  str :=值；
                            Begin
                                     函数体；
                             return 变量名； //存储过程中的返回语句
                            End;
                   $body$
         Language plpgsql;
```

1. 存储过程（FUNCITON）变量可以直接用 || 拼接。
2. 存储过程的对象不可以直接用变量，要用 quote_ident(objVar)
3. $1 $2是 FUNCTION 参数的顺序
4. SQL语句中的大写全部会变成小写，要想大写存大，必须要用双引号。

# 函数

## 例子

```sql
create or replace function intobatch() returns integer as

$body$

declare

    skyid integer;
    lot   float;
    lat   float;
    sex   varchar;
    level integer;
    ctime int     := 1325404914;
    num   integer := 0;
    total integer := 0;

begin
    lot = '73.6666666';
    lat = '3.8666666';
    FOR skyid IN 404499817 ..404953416
        loop
            if (lot > 135.0416666) then
                lot = 73.6666666;
            end if;
            if (lat > 53.5500000) then
                lat = 3.8666666;
            end if;
            if (skyid % 2 <> 0) then
                sex = '1';
                level = 0;
            else
                sex = '2';
                level = 1;
            end if;
            /*INSERT INTO user_last_location(user_id, app_id, lonlat, sex, accurate_level, lonlat_point, create_time)
            VALUES (skyid, 2934, ST_GeomFromText('POINT(' || lot || ' ' || lat || ')', 4326), sex, level,
                    POINT(lot, lat), to_timestamp(ctime));*/
            INSERT INTO department(id, d_code, d_name,d_parentid,d_sex,d_level) VALUES (5,'1010','success',3,sex,level);
            lot = lot + 0.1;
            lat = lat + 0.1;
            skyid = skyid + 1;
        end loop;
    return skyid;
end
$body$
    language plpgsql;
```