# postgresql 中的to_char()常用操作

postgresql中的to_char()用法和Oracle相比，多了一个参数。

![img](https://img.jbzj.com/file_images/article/202102/20210201113818.jpg)

to_char(待转换值，转换格式);

### 常用转换格式有2种：

一个是写若干个0，如果待转换的值位数少于于你定义的转换格式位数，输出值会自动在左边补0，位数补齐到转换格式的长度；如果待转换的值位数多于你定义的转换格式位数，输出值为：##（长度跟你定义的转换格式一样）；

另一个是写若干个9，如果待转换的值位数少于你定义的转换格式位数，正常输出；

如果待转换的值位数多于于你定义的转换格式位数，输出值为：##（长度跟你定义的转换格式一样）；

转换格式如果写其他数字，输出结果为转换格式的值。

**补充：Postgresql中使用to_char进行yyyy-MM-dd HH:mm:ss转换时要注意的问题**

在java和一些常用的数据中(mysql/sqlsever)中进行年月日分秒转换的时候，都是用

```plsql
SELECT to_char(CURRENT_DATE,'yyyy-MM-dd hh:MM:ss')
```

但是在Postgresql中这样用就会出现问题，在pg中执行上面的语句返回的结果为

2015-05-06 12:05:00

看到了，这并不是我们想要的，那怎么处理呢？在pg中要用下面的方法

```plsql
SELECT to_char(CURRENT_DATE,'yyyy-MM-dd hh24:MI:ss')
```

结果如下

2015-05-06 00:00:00