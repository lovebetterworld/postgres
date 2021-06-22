- [如何安装PostgreSQL + PostGIS请点击](http://mp.weixin.qq.com/s?__biz=MzU3MTc1NzU0Mg==&mid=2247484008&idx=1&sn=789825cfd234bd5b664304d110b0b4e2&chksm=fcda04e1cbad8df78d0f95ce9322858180064652411d13ef2595b72fba37193fde6ae561d863&scene=21#wechat_redirect)

PostgreSQL与PostGIS版本的依赖关系可点击：http://trac.osgeo.org/postgis/wiki/UsersWikiPostgreSQLPostGIS

# 一、在线安装

通过下载外部repo源的安装方式，我这里暂且称之为在线安装。

我们首先要使用在线安装的方式，成功安装postgresql + postgis，然后再考虑如何获取相关依赖rpm包的问题。请看具体命令：

```bash
# 安装postgresql依赖的rpm包
rpm -ivh https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
# 安装postgis的依赖包
rpm -ivh https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
```

通过执行上述命令，在/etc/yum.repos.d/目录下会有以下几个文件：

- pgdg-96-centos.repo
- epel.repo
- epel-testing.repo

三个文件含有postgresql + postgis的外部下载源。通过yum的方式来安装：

```bash
# 安装postgresql
yum install postgresql96 postgresql96-server postgresql96-libs postgresql96-contrib postgresql96-devel
# 安装postGIS
yum install postgis24_96
```

安装成功。接下来就是要将postgresql + postgis依赖的rpm包收集起来，然后做一个yum本地源，就可以进行离线安装了。

# 而、收集依赖的rpm包

我们可以使用yum命令的--downloaddir参数及--downloadonly参数来将依赖的rpm包下载到本地。具体步骤如下：

1. 首先需要将postgresql + postgis相关的包进行yum卸载，然后我们再install到本地

```bash
yum remove postgresql96 postgresql96-server postgresql96-libs postgresql96-contrib postgresql96-devel postgis24_96
```

1. 创建目录，指定rpm依赖包的存储目录。我们后续会用到httpd，所以我们先安装httpd服务。

```bash
yum install -y httpd
# httpd安装成功后，会自动创建/var/www/html/目录，我们将要下载的rpm依赖包放置到该目录下
mkdir /var/www/html/postgres
```

1. 下载rpm依赖包

```bash
yum install --downloaddir=/var/www/html/postgres --downloadonly postgresql96 postgresql96-server postgresql96-libs postgresql96-contrib postgresql96-devel postgis24_96
```

等下载完毕之后，rpm依赖包如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/BQQRPo0PQq74jcA7OW9MsDFZuXZtDKsib5uJR2ic0ooY9PTo7lAKauYAqld0QZ7zlOcRVXvJibcfOicxNCuWP3G4ibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后我们再搭建yum本地源。

# 三、搭建yum本地源

1. 下载createrepo工具

```bash
yum install -y createrepo
```

1. 生成repodata目录

```bash
cd /var/www/html/postgres
createrepo .
ll repodata
```

1. 删除之前在线安装时的repo文件

```bash
cd /etc/yum.repos.d
# 删除之前在线安装时的repo文件，以测试yum本地源是否搭建成功
rm -rf epel.repo epel-testing.repo pgdg-96-centos.repo
```

1. 启动httpd服务

```bash
service httpd start
```

1. 制作.repo文件

   新建postgres.repo文件，并将其放入到/etc/yum.repos.d目录下。文件内容如下：

```bash
[postgres]
name=postgresql and postgis
baseurl=http://liuyzh2.xdata/postgres/
gpgcheck=0
enabled=1
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/BQQRPo0PQq74jcA7OW9MsDFZuXZtDKsibhdmibwXmfDfS4US21G6Q7YbMjZRQ9UQL3CqKeNmO0v0bS1mtYHgBRcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 四、yum安装

```bash
# 先卸载postgresql相关包
yum remove postgresql*
# 安装postgresql9.6 + postgis2.4
yum install -y postgresql96 postgresql96-server postgresql96-libs postgresql96-contrib postgresql96-devel postgis24_96
```

安装成功，如下图所示;

![图片](https://mmbiz.qpic.cn/mmbiz_png/BQQRPo0PQq74jcA7OW9MsDFZuXZtDKsibk3hs1TJr2efjwLCNBdvfRlHNUWmn8Z9fj7arDcfqicINdtDNBm7Jbaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 五、总结

总结一下：

- 我们首先下载了外部repo源，然后通过yum install的方式将需要的服务成功安装。
- 然后执行`yum install --downloaddir=/var/www/html/postgres --downloadonly postgresql96 postgis24_96 …`命令，这样就将`postgresql96 postgis24_96 …`等所依赖的rpm包下载到了`/var/www/html/postgres`目录下了。
- 有了依赖的rpm包，就简单多啦。直接制作yum本地源，生成repo文件就行了。