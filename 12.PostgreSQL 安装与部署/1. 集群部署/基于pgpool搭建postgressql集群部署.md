- [基于pgpool搭建postgressql集群部署](https://www.cnblogs.com/guapitomjoy/p/15330079.html)

# postgresql集群搭建

基于pgpool中间件实现postgresql一主多从集群部署，这里用两台服务器作一主一从示例

| 虚拟机名   | IP            | 主从划分 |
| ---------- | ------------- | -------- |
| THApps     | 192.168.1.31  | 主节点   |
| YY-Test-01 | 192.168.1.36  | 从节点   |
| vip        | 192.168.1.100 | 虚拟ip   |

## 1.软件版本

| 系统版本   | CentOS Linux release 7.9.2009 (Core) |
| ---------- | ------------------------------------ |
| PostgreSQL | 10                                   |
| pgpool-II  | 4.0.1                                |

## 2.整体架构

[![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cueGlhb21hc3RhY2suY29tL2ltYWdlcy8yMDE5LzA4L1BncG9vbC1JSSVFOSU5QiU4NiVFNyVCRSVBNC5wbmc?x-oss-process=image/format,png)](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cueGlhb21hc3RhY2suY29tL2ltYWdlcy8yMDE5LzA4L1BncG9vbC1JSSVFOSU5QiU4NiVFNyVCRSVBNC5wbmc?x-oss-process=image/format,png)

## 3.安装postgresql

### 3.0.安装postgresql-10

```python
# 1.更新源
网页打开：https://yum.postgresql.org/repopackages.php  
找到版本 CentOS 7 - x86_64 
右键 复制链接
# 在centos系统中运行：
	yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    
# 2.安装postgresql
# 执行命令：
	yum update -y
# 查看postgresql源：
	yum list | grep postgresql
# 我们需要安装的是这两个: postgresql10-contrib postgresql10-server
yum install postgresql10 postgresql10-contrib postgresql10-server postgresql10-devel -y

# 3.初始化数据库 执行：
	/usr/pgsql-10/bin/postgresql-10-setup initdb

# 4.启动数据库并设置开机启动
    sudo systemctl start postgresql-10
    sudo systemctl enable postgresql-10.service

# 5.安装完毕，配置pg环境变量
	vi ~/.bash_profile
# 在PATH后添加
	/usr/pgsql-10/bin
# 比如：
	PATH=$PATH:$HOME/bin:/usr/pgsql-10/bin
# 添加完后执行：使它立即生效
	source ~/.bash_profile
```

### 3.1主节点配置

（1）创建用于主从访问的用户， 修改postgres用户的密码，用于远程登录。(切换到postgres用户操作)

```bash
su - postgres
psql
create role actorcloud login replication encrypted password 'public';
alter role postgres with password 'postgres';
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924135836217-44516831.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924135836217-44516831.png)

（2）修改pg_hba.conf和postgresql.conf配置：
 `vim /var/lib/pgsql/10/data/pg_hba.conf`在最后一行添加下面内容：



```
host    replication    actorcloud    192.168.1.31/32    trust
host    replication    actorcloud    192.168.1.36/32    trust
host    all    all    192.168.1.0/24    md5
host    all    all    0.0.0.0/0    md5
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924135808678-795399152.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924135808678-795399152.png)

修改postgresql.conf配置：
 `vim /var/lib/pgsql/10/data/postgresql.conf `

```
listen_addresses = '*' 
port = 5432 
wal_level = replica
max_wal_senders= 10
wal_keep_segments = 10240
max_connections = 512
```

（3）重启主节点
 建议先停止，再启动，而不是重启。之后再验证一下是否启动成功

```
systemctl stop postgresql-10
systemctl start postgresql-10
```

### 3.2从节点配置

从节点的操作建议全部在postgres用户下进行。

（1）切换postgres用户

```
su - postgres
```

（2）对主节点的数据进行备份到从节点，其中192.168.1.31为主节点IP，actorcloud是上一节主节点创建的用户。

```
rm -rf /var/lib/pgsql/10/data/*
pg_basebackup -h 192.168.1.31 -U actorcloud -D /var/lib/pgsql/10/data -X stream -P
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924135731209-2018505147.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924135731209-2018505147.png)

（3）拷贝recovery.conf，编辑recovery.conf内容，其中192.168.1.31对应主机IP，actorcloud是上一节主机创建的用户。

```
cp /usr/pgsql-10/share/recovery.conf.sample /var/lib/pgsql/10/data/recovery.conf
```

`vim /var/lib/pgsql/10/data/recovery.conf`修改配置：

```
standby_mode = on
primary_conninfo = 'host=192.168.1.31 port=5432 user=actorcloud password=public' 
recovery_target_timeline = 'latest'
trigger_file = '/tmp/trigger_file0'
```

（4）修改从节点的postgresql.conf，用于开启standby模式。

```
hot_standby = on
```

（5）退出postgres用户，重启PostgreSQL

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924133808667-180965971.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924133808667-180965971.png)

## 4.验证主从

### 4.1查看从节点信息

在主节点切换至psql界面，输入命令：

```
select client_addr,sync_state from pg_stat_replication;
```

若从节点显示出来，如下图所示，则说明PostgreSQL集群搭建成功。

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924120019649-566244859.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924120019649-566244859.png)

### 4.2读写测试

（1）在主节点写数据，从节点读数据。

```
create database test;
\l
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115939693-1526705266.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115939693-1526705266.png)

在从节点上查看创建之后的数据库。可以看见，数据库同步了。

```
\l
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115902679-694240720.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115902679-694240720.png)

## 5.postgres用户间免密登录

（0）关闭防火墙和SETLINUX

```bash
# 查看防火墙状态
firewall-cmd --state
# 停止firewall
systemctl stop firewalld.service
# 禁止firewall开机启动
systemctl disable firewalld.service 

# 修改/etc/selinux/config 文件
# 将
SELINUX=enforcing
# 改为
SELINUX=disabled

# 重启
reboot
```

以下操作两台机器同时进行：

（1）给postgres用户更改密码：密码修改为postgres

```
passwd postgres
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115834604-1851749215.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115834604-1851749215.png)

（2）生成并同步密钥

以192.168.1.31为例，想要免密登录另外一台机器。先切换到postgres用户执行，ssh-keygen时一路回车往下，提升输入y时输入y，ssh-copy-id时需要输入密码。

```
su postgres
ssh-keygen
ssh-copy-id 192.168.1.36
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115804709-477700049.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115804709-477700049.png)

测试免密登录：

```
ssh 192.168.1.36
ip addr
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115735418-2031386675.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115735418-2031386675.png)

同理在192.168.1.36机器上也是相同操作

## 6.安装pgpool

（1）安装(两台机器都需要安装)

```
# 添加源
yum install http://www.pgpool.net/yum/rpms/4.0/redhat/rhel-7-x86_64/pgpool-II-release-4.0-1.noarch.rpm
 
# 安装
 yum install pgpool-II-pg10-debuginfo pgpool-II-pg10-devel  pgpool-II-pg10-extensions  pgpool-II-pg10
```

安装完之后pgpool的配置文件在/etc/pgpool-II/下

（2）修改配置（两台机器都需要配置）

pool_hba.conf和之前配置的PostgreSQL中的配置时一样的

```
vim /etc/pgpool-II/pool_hba.conf
host    replication    actorcloud    192.168.1.31/32    trust
host    replication    actorcloud    192.168.1.36/32    trust
host    all    all    192.168.1.0/24    md5
host    all    all    0.0.0.0/0    md5
```

（3）对postgres的密码进行加密。本文将postgres的密码设置为和用户名相同，将加密结果复制，并粘贴到pcp.conf中相应的位置，取消掉该行的注释。

```
pg_md5 postgres
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115651021-1888794991.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115651021-1888794991.png)

```
vim /etc/pgpool-II/pcp.conf
postgres:81dc9bdb52d04dc20036dbd8313ed055
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115601217-1364264802.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115601217-1364264802.png)

（4）执行命令(先切换到postgres用户再执行然后输入密码):

```
pg_md5 -m -p -u postgres pool_passwd
```

## 7.修改集群配置

**以192.168.1.31主节点为例**：

```
vim /etc/pgpool-II/pgpool.conf
```

（1）修改监听地址，将localhost改为*，即监听所有地址发来的请求。

```
修改前：listen_addresses = 'localhost'
修改后：listen_addresses = '*'
```

（2）修改backend相关参数，对应的是PostgreSQ两个节点的相关信息。

```
修改前：
	backend_hostname0 = 'localhost'                           
	backend_port0 = 5432
	backend_weight0 = 1
	backend_data_directory0 = '/var/lib/pgsql/data'
	backend_flag0 = 'ALLOW_TO_FAILOVER'
修改后：
	backend_hostname0 = '192.168.1.31'
	backend_port0 = 5432
	backend_weight0 = 1
	backend_data_directory0 = '/var/lib/pgsql/10/data'
	backend_flag0 = 'ALLOW_TO_FAILOVER'
 	
    backend_hostname1 = '192.168.1.36'
    backend_port1 = 5432
    backend_weight1 = 1
    backend_data_directory1 = '/var/lib/pgsql/10/data'
    backend_flag1 = 'ALLOW_TO_FAILOVER'
```

（3）pg_hba.conf生效

```
修改前：enable_pool_hba = off
修改后：enable_pool_hba = on
```

（4）使负载均衡生效

```
修改前：load_balance_mode = off
修改后：load_balance_mode = on
```

（5）主从流复制生效，并配置用于检查的用户，这个用户就用上方创建的用于主从访问的用户。

```
修改前：
	master_slave_mode = off
	sr_check_period = 0
	sr_check_user = 'nobody'
	sr_check_password = ''
	sr_check_database = 'postgres'
	delay_threshold = 0
修改后：
	master_slave_mode = on
	sr_check_period = 6
	sr_check_user = 'actorcloud'
	sr_check_password = 'public'
	sr_check_database = 'postgres'
	delay_threshold = 10000000
```

（6）健康检查相关配置，并配置用于检查的用户，这个用户就用上方创建的用于主从访问的用户。

```
修改前：
	health_check_period = 0
	health_check_user = 'nobody'
	health_check_password = ''
	health_check_database = ''
修改后：
	health_check_period = 10
	health_check_user = 'actorcloud'
	health_check_password = 'public'
	health_check_database = 'postgres'
```

（7）配置主机故障触发执行的脚本。

```
修改前：failover_command = ''
修改后：failover_command = '/var/lib/pgsql/10/failover_stream.sh %d %H'
```

（8）开启开门狗，IP为本机IP

```
修改前：
	use_watchdog = off
	wd_hostname = ''
修改后：
	use_watchdog = on
	wd_hostname = '192.168.1.31'
```

（9）开启虚拟IP，并修改网卡信息。启动后直接使用虚拟IP进行数据库操作剧哦

```
修改前：
	delegate_IP = ''
	if_up_cmd = 'ip addr add $_IP_$/24 dev eth0 label eth0:0'
	if_down_cmd = 'ip addr del $_IP_$/24 dev eth0'
	arping_cmd = 'arping -U $_IP_$ -w 1 -I eth0'
修改后：
	delegate_IP = '192.168.1.100'
	if_up_cmd = 'ip addr add $_IP_$/24 dev ens192 label ens192:0'
	if_down_cmd = 'ip addr del $_IP_$/24 dev ens192'
	arping_cmd = 'arping -U $_IP_$ -w 1 -I ens192'
```

上方把所有eth0改为了ens192，ens192是该主机的网卡设备名称，可通过命令`ip addr`查看

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115429475-508081309.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115429475-508081309.png)

（10）心跳检查的配置与看门狗配置。IP为从节点的IP

```
修改前：
	heartbeat_destination0 = 'host0_ip1'
	heartbeat_device0 = ''
	#other_pgpool_hostname0 = 'host0'
	#other_pgpool_port0 = 5432
	#other_wd_port0 = 9000
修改后：
	heartbeat_destination0 = '192.168.1.36'
	heartbeat_device0 = 'ens192'
	other_pgpool_hostname0 = '192.168.1.36'
	other_pgpool_port0 = 9999
	other_wd_port0 = 9000
```

以上，是主节点的配置。对于从节点，只有（8）和（10）的IP需要更改，其他的一致。

## 8.编写故障切换脚本

`vim /var/lib/pgsql/10/failover_stream.sh`
 内容为：

```bash
#! /bin/sh
# Failover command for streaming replication.
# Arguments: $1: new master hostname.
failed_node=$1
new_master=$2
trigger_file=$3
# Do nothing if standby goes down.
if [ $failed_node = 1 ]; then
exit 0;
fi
# Create the trigger file.
# use commond 
/usr/bin/ssh -T $new_master /usr/pgsql-10/bin/pg_ctl promote -D /var/lib/pgsql/10/data/
# use file
# /usr/bin/ssh -T $new_master  /bin/touch /tmp/trigger_file0
exit 0
```

## 9.文件权限更改

```bash
chmod u+s /sbin/ifconfig &&chmod u+s /usr/sbin

chown postgres:postgres /var/lib/pgsql/10/failover_stream.sh && chmod 777 /var/lib/pgsql/10/failover_stream.sh

chown -R postgres.postgres /etc/pgpool-II

mkdir /var/log/pgpool

chown -R postgres.postgres /var/log/pgpool

mkdir /var/run/pgpool

chown -R postgres.postgres /var/run/pgpool
```

## 10.运行pgpool

```bash
# 启动
systemctl start pgpool
# 开机自启
systemctl enable pgpool
```

登录虚拟ip查看集群节点。以后所有的操作均可通过连接虚拟IP操作数据库。

```
psql -p 9999 -h 192.168.1.100 -U postgres
show pool_nodes;
```

[![img](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115109046-1954033859.png)](https://img2020.cnblogs.com/blog/1751700/202109/1751700-20210924115109046-1954033859.png)

至此，基于 `Pgpool-II` 中间件的 `PostgreSQL` 集群搭建完成。

