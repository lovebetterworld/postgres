- [Linux中PostgreSQL和PostGIS的安装和使用](https://cloud.tencent.com/developer/article/1722171)



# 一、安装 [PostgreSQL](https://cloud.tencent.com/product/postgresql?from=10680) 和 PostGIS 

PostgreSQL 和 PostGIS 已经是热门的开源工程，已经收录在各大 Linux 发行版的 yum 或 apt 包中。Ubuntu 为例，安装以下包即可：

```bash
$ sudo apt-get install postgresql-client postgresql postgis -y
```

RedHat 系列则请安装：

```bash
$ sudo yum install postgresql-server postgresql postgis
```

初次安装后，默认生成一个名为 postgres 的数据库和一个名为 postgres 的数据库用户。这里需要注意的是，同时还生成了一个名为 postgres 的 Linux 系统用户。我们以后在操作 PostgreSQL 的时候都应该在这个新创建的 postgres 用户中进行。

# 二、PostgreSQL 配置 

不建议从源码安装，我曾经试过从源码安装，实在是太麻烦了，而且各种 make install 容易出错。最后我还是用 rpm 安装了。不过既然花了些时间研究并且我成功安装过，所以还是记录一下吧——不过，可能有错漏，所以读者如果要从源码安装的话，请做好回滚的准备。

如果使用的是通过 source 编译并且 make install 安装，那么这一节是需要额外配置的。 

貌似 CentOS 系列的安装也需要……

默认的 make install 之后，PostgreSQL 安装目录在：/usr/local/pgsql/

首先根据这个链接的参考，需要配置环境变量

```bash
$ set $PGDATA = "/usr/local/pgsql/database"
```

但是执行了 pg_ctl start 之后，会出现错误：

```bash
pg_ctl: directory "/usr/local/pgsql/database" is not a database cluster directory
```

这样的话，就需要参照 PostGreSQL 官方文档的步骤创建真正的 database：<br/> 

PostgreSQL: Documentation: 9.1: Creating a Database Cluster

首先创建一个用户账户，名叫 postgres

```bash
$ usradd postgres
$ sudo chown postgres /usr/local/pgsql/database
```

然后进入这个账户，创建 database

```bash
$ sudo su postgres
$ initdb -D /usr/local/pgsql/database/
```

此时 shell 会输出：

```bash
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "C".
The default database encoding has accordingly been set to "SQL_ASCII".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /usr/local/pgsql/database ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
creating template1 database in /usr/local/pgsql/database/base/1 ... ok
initializing pg_authid ... ok
initializing dependencies ... ok
creating system views ... ok
loading system objects' descriptions ... ok
creating collations ... ok
creating conversions ... ok
creating dictionaries ... ok
setting privileges on built-in objects ... ok
creating information schema ... ok
loading PL/pgSQL server-side language ... ok
vacuuming database template1 ... ok
copying template1 to template0 ... ok
copying template1 to postgres ... ok
syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.
Success. You can now start the database server using:
pg_ctl -D /usr/local/pgsql/database/ -l logfile start
```

恭喜你，接下来就可以启动 PostgreSQL 了：

```bash
pg_ctl -D /usr/local/pgsql/database/ -l /usr/local/pgsql/database/psql.log start
```

# 三、PostgreSQL 安装好后

进入 postgres 账户，并且进入 PostgreSQL 控制台：

```bash
$ sudo su postgres
$ psql
```

这时相当于系统用户 postgres 以同名数据库用户的身份，登录数据库，否则我们每次执行 psql 的时候都要在参数中指定用户，容易忘。

在 psql 中设置一下密码——需要注意的是，这里设置的密码并不是 postgres 系统帐户的密码，而是在数据库中的用户密码：

```bash
postgres=# \password postgres
```

然后按照提示输入密码就好。

从源码安装 PostGIS 

如果选择了从源码安装 PostgreSQL 的话，那么首先需要判断你安装的 PostgreSQL 是什么版本

然后，再到 PostGIS 的网页上去查其对应的是 PostGIS 的哪个版本。

最后，按照 PostGIS 的版本去下载对应的 source

最后的导入很麻烦，笔者就是卡在这一步，所以才最终放弃从源码安装的……

导入 PostGIS 扩展 

根据 postgresql 和 postgis 的版本不同，路径会有些差异，主要是路径中包含版本信息：

```bash
$ sudo su postgres
$ createdb template_postgis
$ createlang plpgsql template_postgis
$ psql -d template_postgis -f /usr/share/postgresql/9.5/contrib/postgis-2.2/postgis.sql
$ psql -d template_postgis -f /usr/share/postgresql/9.5/contrib/postgis-2.2/spatial_ref_sys.sql
```

上面的操作中，创建了一个叫做 “template_postgis” 的空数据库。这个数据库是空的，并且属于 postgres 用户。注意，不要往这个数据库中添加数据，这个数据库之所以称为 “模板”（template），就说明它是用来派生用的。

相应的 PostGIS 路径可能不同，如果失败，就在上面的路径附近多尝试一下，找几个 .sql 文件试试看。

转换 .shp 文件到 PostGIS 数据库中 

转换 .shp 到 .sql 文件 

首先找到需要转换的文件，假设需要转换的 .shp 文件是：/tmp/demo.shp，那么就做以下操作：

```bash
$ sudo su postgres
$ cd /tmp
$ shp2pgsql -W GBK -s 3857 ./demo.shp entry > demo.sql
```

这里需要说明一下最后一句各部分所代表的含义：

- -W GBK：如果你的 .shp 文件包含中文字符，那么请加上这个选项
- -s 3857：指明文件的参考坐标系统。我的 .shp 文件使用的是 EPSG:3857
- ./demo.shp：.shp 文件的路径
- entry：表示要导入的数据库表名——假设这个 .shp 文件表示的是各个入口，所以我命名为 “entry”
- demo.sql

得到了 .sql 文件后，就可以直接导入到 PostgreSQL 数据库了。

# 四、创建一个 PostGIS 数据库 

这里就需要用到前面的 template 了。

```bash
sudo su postgres
psql
CREATE DATABASE newdb WITH TEMPLATE originaldb OWNER dbuser;
```

- newdb: 新的数据库名
- originaldb：也就是前面的 template_postgis
- dbuser：你的账户名，我一般使用 postgres

导入 .sql 文件

```bash
sudo su postgres
psql
\c newdb
\i demo.sql
\d
```

可以看到，.sql 文件已经被导入了。

# 五、设置数据库权限 

OK，现在我们在本机（服务器 IP 假设是 192.168.1.111）用以下命令登录 psql，会发现一段输出：

```bash
$ psql -h 192.168.1.111 -p 5432
psql: could not connect to server: Connection refused
    Is the server running on host "100.94.110.105" and accepting
    TCP/IP connections on port 5432?
```

这是因为 PostgreSQL 默认不对外开放权限，只对监听环回地址。要修改的话，需要找到 postgresql.conf 文件，修改值 listen_addresses：

```bash
listen_addresses 
```