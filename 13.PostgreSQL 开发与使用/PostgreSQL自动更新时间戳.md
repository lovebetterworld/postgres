- [PostgreSQL如何自动更新时间戳？ ](https://juejin.cn/post/7033762606175223839)

# 一、需求描述
对于很多业务表，我们大多数需要记录以下几个字段：

- create_at 创建时间
- update_at 更新时间
- create_by 创建人
- update_by 更新人

为了给这些字段赋值，我们需要在repository层为entity赋值，创建时间和更新时间就取当前系统时间LocalDateTime,创建人和更新人需要用系统用户去赋值。对于创建时间和更新时间，这种与当前业务无关的字段，有没有可能不在repository上每次去手动赋值。
当然，肯定是有的，创建时间无非就是数据新插入行的时间，更新时间就是行数据更新的时间，理解了这一层的含义，那就有解决办法了。
对于Mysql来说，其内部提供的函数对于创建时间和更新时间的字段的自动更新是相当容易的，但对于PostgreSQL事情会稍稍复杂一点。

# 二、如何做

要在插入数据的时候自动填充 create_at列的值，我们可以使用DEFAULT值，如下面所示。
```sql
CREATE TABLE users (
  ...
  create_at timestamp(6) default current_timestamp
)
```

为create_at字段设置一个默认值current_timestamp当前时间戳，这样达到了通过在 INSERT 语句中提供值来显式地覆盖该列的值。
但上面的这种方式只是对于insert行数据的时候管用，如果对行更新的时候，我们需要使用到数据库的触发器trigger。
首先我们编写一个触发器update_modified_column如下面的代码所示，含义是更新表的字段update_at为当前时间戳。
```sql
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.update_at = now();
    RETURN NEW;
END;
$$ language 'plpgsql';
```

然后我们应用这个触发器，如何应用呢？当然是为这个触发器设置触发条件。
```sql
CREATE TRIGGER update_table_name_update_at BEFORE UPDATE ON table_name FOR EACH ROW EXECUTE PROCEDURE  update_modified_column();
```

即代表的含义是更新表table_name行数据的时候，执行这个触发器，我们需要为每一个表设置应用这个触发器！至此，达到目的。