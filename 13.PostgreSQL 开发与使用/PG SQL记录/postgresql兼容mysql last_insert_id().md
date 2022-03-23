- [postgresql兼容mysql last_insert_id()_Ruby-PGer的博客-CSDN博客](https://blog.csdn.net/qq_32935175/article/details/122296941)

PG 中有类似的用法，[INSERT](https://so.csdn.net/so/search?q=INSERT&spm=1001.2101.3001.7020) INTO student1() VALUES () RETURNING id;就像这样。

如果不想改代码，可以直接在PG 数据库封装一个同名函数，使用lastval()实现：

lastval() bigint 返回最近一次用 nextval 获取任何序列的数值。

[自定义函数](https://so.csdn.net/so/search?q=自定义函数&spm=1001.2101.3001.7020)：

```sql
create or replace function last_insert_id() returns int4 as

$$

begin

 return lastval();

end ;

$$

language plpgsql;
```

一个差异就是，如果一次插入多行的话，MySQL是返回第一个值， PG是返回最后一个值。