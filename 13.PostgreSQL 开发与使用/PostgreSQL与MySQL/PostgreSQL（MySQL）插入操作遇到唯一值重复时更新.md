- [PostgreSQL（MySQL）插入操作遇到唯一值重复时更新_张志翔 ̮的博客-CSDN博客](https://vegetable-chicken.blog.csdn.net/article/details/103733927)

**1、mysql写法（on duplicate key update 语法必须配合唯一索引）**

```csharp
<insert id="insertOrUpdateNew">
      insert into test
      (a,b,c)
      values
      (#{a},#{b},#{c})
      on duplicate key update
      c = values(c)
</insert>
 
<insert id="insertOrUpdateNewNew">
      insert into test
      (a,b,c)
      values
      <foreach collection="list" item="l" separator=",">
          (#{l.a},#{l.b},#{l.c})
      </foreach>
      on duplicate key update
      c = values(c)
</insert>
```

**2、postgresql写法（on conflict 语法必须配合主键或唯一索引）**

```csharp
<insert id="insertOrUpdateNew">
      insert into test
      (a,b,c)
      values
      (#{a},#{b},#{c})
      on conflict(a,b)
      do update set
      c = #{c}
</insert>
 
<insert id="insertOrUpdateNewNew">
      insert into test
      (a,b,c)
      values
      <foreach collection="list" item="l" separator=",">
      (#{l.a},#{l.b},#{l.c})
      </foreach>
      on conflict(a,b) do nothing
</insert>
 
推荐下面:
<insert id="insertOrUpdateNew">
      insert into test
      (a,b,c)
      values
      (#{a},#{b},#{c})
      on conflict(a,b) do update set c = excluded.c
</insert>
```

**注：1、PostgreSQL单条插入更新的时候三种写法都可以用，但是！！！批量插入更新的时候只能用第三种写法！！！**

**2、PostgreSQL的conflict语法只在PostgreSQL-9.5以上才可生效，9.5以下版本直接报错。**