- [PostGIS教程二：PostGIS的安装](https://blog.csdn.net/qq_35732147/article/details/86299060)



# 一、下载安装程序

  在安装PostGIS前首先必须安装**PostgreSQL**，然后在安装好的**Stack Builder**中选择安装PostGIS组件。

  PostgreSQL安装文件下载地址是https://www.enterprisedb.com/downloads/postgres-postgresql-downloads

  这里使用的PostgreSQL版本是9.6。

# 二、安装PostgreSQL

  双击下载的文件，所有设置都使用默认设置即可，只是需要设置超级用户postgres的密码。

# 三、安装PostGIS

  安装PostgreSQL安装完成后，提示运行**Stack Builder**。通过该工具安装PostGIS。

  Stack Builder运行后，选择安装目标软件为PostgreSQL 9.6 on port 5432。然后在安装程序选择对话框中选择PostGIS 2.3。

![img](https://img-blog.csdn.net/20180723163435278?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180723163652734?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  然后Stack Builder会下载PostGIS 2.3的安装程序。下载后就会安装，在设置安装组件时，最好选择"**Create spatial database**"，以便在创建数据库时可以以此作为模板。对于其他步骤的设置都选择默认值即可。