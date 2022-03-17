- [PG查询字段注释，表注释](https://www.cnblogs.com/yugege9213/p/15798077.html)

```plsql
select a.attnum AS 序号,
c.relname AS 表名,
cast(obj_description(c.oid) as varchar) AS 表描述,
a.attname AS 列名,
concat_ws('',t.typname,SUBSTRING(format_type(a.atttypid,a.atttypmod) from '\(.*\)')) as 字段类型,
d.description AS 备注
from pg_attribute a
    left join   pg_class c on c.oid=a.attrelid
    LEFT JOIN  pg_type t on t.oid=a.atttypid
    left join  pg_description d on d.objoid=a.attrelid and d.objsubid=a.attnum
where c.relname='表名'  and a.attnum>0  order by c.relname desc ,a.attnum asc;
```

