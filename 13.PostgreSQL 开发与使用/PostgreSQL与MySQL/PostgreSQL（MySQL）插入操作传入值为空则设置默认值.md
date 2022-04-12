- [PostgreSQL（MySQL）插入操作传入值为空则设置默认值_张志翔 ̮的博客-CSDN博客](https://vegetable-chicken.blog.csdn.net/article/details/103740673)

**1、mysql写法**

IFNULL(p1,p2)，如果p1有值就是p1，如果p1是空，则值为p2

```lua
<insert id="insertForeach" parameterType="java.util.List" >
    insert into user_message
        ( skip_id )
    values
        <foreach collection="list" item="userMessage" index="index" separator=",">
            ( ifnull(#{userMessage.skipId},"0") )
        </foreach>
</insert>
```

**2、postgresql写法**

COALESCE函数是返回参数中的第一个非null的值，它要求参数中至少有一个是非null的，如果参数都是null会报错

```sql
select COALESCE(null,null); //报错
select COALESCE(null,null,now(),''); //结果会得到当前的时间
select COALESCE(null,null,'',now()); //结果会得到''
 
//可以和其他函数配合来实现一些复杂点的功能：查询学生姓名，如果学生名字为null或''则显示“姓名为空”
select case when coalesce(name,'') = '' then '姓名为空' else name end from student;
```

问题到此解决。