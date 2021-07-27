# postgresql 中的时间处理小技巧

## 时间格式处理

按照给定格式返回：to_char(timestamp,format)

![img](https://img.jbzj.com/file_images/article/202103/2021032914483987.png)

返回相差的天数：(date(time1) - current_date)

![img](https://img.jbzj.com/file_images/article/202103/2021032914483988.png)

返回时间戳对应的的日期[yyyy-MM-dd]：date(timestamp)

![img](https://img.jbzj.com/file_images/article/202103/2021032914483989.png)

计算结果取两位小数(方便条件筛选)：round((ABS(a-b)::numeric / a), 2) * 100 < 10

![img](https://img.jbzj.com/file_images/article/202103/2021032914483990.png)

## 时间运算

加减运算

'-' :前x天/月/年

'+' :后x天/月/年

current_timestamp - interval 'x day/month/year...' 返回时间戳

![img](https://img.jbzj.com/file_images/article/202103/2021032914483991.png)

date_part('day', current_timestamp - time1) 两个时间相差的天数

![img](https://img.jbzj.com/file_images/article/202103/2021032914483992.png)

返回时间间隔的秒数

两个timestamp 直接相减返回的是 interval类型，而不是毫秒数

extract(epoch from (time1- time2)) * 1000

![img](https://img.jbzj.com/file_images/article/202103/2021032914483993.png)

如果在sql 中使用long类型的 timestamp，需要包裹 to_timestamp() 函数

![img](https://img.jbzj.com/file_images/article/202103/2021032914483994.png)

**参考资料：**

1. https://www.yiibai.com/manual/postgresql/functions-formatting.html

2. http://www.postgres.cn/docs/9.4/functions-datetime.html