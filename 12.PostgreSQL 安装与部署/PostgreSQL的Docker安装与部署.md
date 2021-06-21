- [docker安装并持久化postgresql数据库](https://www.cnblogs.com/mingfan/p/11863509.html)

# 一、docker安装并持久化postgresql数据库

## 1.1 拉取postgresql镜像

```bash
docker pull postgresql
```

## 1.2 创建本地卷，数据卷可以在容器之间共享和重用， 默认会一直存在，即使容器被删除（`docker volume inspect `pgdata可查看数据卷的本地位置）

```bash
docker volume create pgdata
```

## 1.3 启动容器

```bash
docker run --name postgres2 -e POSTGRES_PASSWORD=password -p 5432:5432 -v pgdata:/var/lib/postgresql/data -d postgres:9.6
```

![img](https://img2018.cnblogs.com/i-beta/1660349/201911/1660349-20191114230953551-384672462.png)

##  1.4 进入postgres容器执行sql

```bash
docker exec -it postgres2 bash

psql -h localhost -p 5432 -U postgres --password
```

![img](https://img2018.cnblogs.com/i-beta/1660349/201911/1660349-20191114231316599-1839621880.png)

 

至此，postgresql安装成功。

# 二、docker 部署带postgis扩展的postgresql

- [hey laosha](https://blog.csdn.net/geol200709)

- [docker 部署带postgis扩展的postgresql](https://blog.csdn.net/geol200709/article/details/89481194)

拉取 postgresql 9.6 版本以及postgis 2.4 版本

```bash
docker pull kartoza/postgis:9.6-2.4

docker run -d --name postgresql9.6 -e ALLOW_IP_RANGE=0.0.0.0/0 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -v /var/minio/postgresql/data:/var/lib/postgresql/data -p 5432:5432 kartoza/postgis:9.6-2.4
```

- -e ALLOW_IP_RANGE=0.0.0.0/0，这个表示允许所有ip访问，如果不加，则非本机 ip 访问不了
- -e POSTGRES_USER=postgres 用户名
- -e POSTGRES_PASS=‘postgres’ 指定密码

