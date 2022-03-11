- [PostgreSQL简介（四）—— Client Authentication_迷途的攻城狮-CSDN博客](https://blog.csdn.net/chenleiking/article/details/81195592)

### 1、客户端身份认证

与Unix计算机用户登陆类似，客户端应用程序通过指定的数据库用户名登陆数据库服务器。数据库用户名决定了客户端程序的[访问权限](https://so.csdn.net/so/search?q=访问权限&spm=1001.2101.3001.7020)，因此，明确哪些用户可以连接到数据库显得尤为重要。

[PostgreSQL](https://so.csdn.net/so/search?q=PostgreSQL&spm=1001.2101.3001.7020)提供了好几种客户端身份认证方式，这些验证方式基于client host、database、user来进行特定的客户端身份认证。

PostgreSQL server的数据库用户与PostgreSQL server所在的操作系统的用户是逻辑隔离的，在特定的身份认证方式下，需要建立与数据库用户相同的操作系统用户，这些特定的认证方式通常都限制为本地登陆。如果特定的数据库用户只用来远程登陆，那么就没有必要建立与之对应的操作系统用户。

#### 1.1、pg_hba.conf

PostgreSQL通过data目录下的pg_hba.conf配置文件来控制客户端身份认证（HBA：host-based authentication）。默认的pg_hba.conf文件会在 `initdb` 之后初始化。

pg_hba.conf文件和传统的Linux系统配置文件类似，一行表示一条记录，”#”后面会被当作注释信息，空白行会被忽略，单行记录内，字段之间使用空格或者退格键分隔，当使用双引号时，单个字段可以包含空格。每一行记录内字段依次为：连接方式（connection type）、数据库名称（database name）、用户名称（user name）、客户端地址范围（client IP address range）、认证方式（authentication method）。在认证过程中，其中一行记录认证失败，不会继续使用后续的记录进行认证，如果没有记录匹配，则认证失败。

pg_hba.conf文件内的记录有一下七种格式：

```properties
local      database  user  auth-method  [auth-options]
host       database  user  address  auth-method  [auth-options]
hostssl    database  user  address  auth-method  [auth-options]
hostnossl  database  user  address  auth-method  [auth-options]
host       database  user  IP-address  IP-mask  auth-method  [auth-options]
hostssl    database  user  IP-address  IP-mask  auth-method  [auth-options]
hostnossl  database  user  IP-address  IP-mask  auth-method  [auth-options]1234567
```

1. local：匹配Unix套接字（socket）连接，如果没有此记录，则socket连接不可用；
2. host：匹配TCP/IP连接，并且同时匹配SSL连接和非SSL连接；*默认情况下，TCP/IP远程连接可能不可用，因为PostgreSQL默认的监听地址是localhost，通过设置[listen_addresses](https://www.postgresql.org/docs/10/static/runtime-config-connection.html#GUC-LISTEN-ADDRESSES)来修改监听地址；*
3. hostssl：仅匹配SSL加密的TCP/IP连接；
4. hostnossl：仅匹配非SSL的TCP/IP连接；
5. database：指定当前记录匹配的数据库名称，
   - ***all\***表示匹配所有数据库，
   - ***sameuser\***表示请求建立连接的用户名与数据库名同名，
   - ***samerole\***表示请求建立连接的用户必须是请求建立连接的数据库的同名角色的成员；PostgreSQL中，superuser可以访问所有对象，但是此处，superuser并不被认为是samerole的成员，除非显示的指定superuser为samerole的成员，
   - ***replication\***表示匹配一个物理复制（physical replication）连接，
   - 其他值可以指定特定的数据库名称，多个数据库之间用逗号隔开，
   - *@filePath*指定一个文件路径，文件中包含数据库的名称
6. user：指定匹配的数据库用户名称，
   - ***all\***表示匹配所有的用户，
   - *+groupName*表示匹配的组的成员，包括直接成员或者间接成员，superuser并不被认为是groupName的成员，除非显示的指定superuser为groupName的成员，
   - 其他值可以指定特定的角色名称，多个角色之间用逗号隔开，
   - *@filePath*指定一个文件路径，文件中包含角色名称
7. address：匹配客户端IP列表，可以指定hostname、IP范围，或者下面提到的任何一个关键字
   - IP address range表示为127.1.0.0/16，掩码长度必须完全匹配，檐马之外的右侧部分应该设置为0，
   - IP address表示匹配指定的IP地址
   - ***all\***表示匹配所有的IP地址
   - ***samehost\***表示匹配数据库服务器所具备的IP（*不知道有什么用～*）
   - ***samenet\***表示匹配数据服务器所在的子网内的IP
   - *hostname*表示服务器将通过客户端IP反向查找客户端hostname（比如逆向DNS），hostname匹配不区分大小写。如果hostname匹配，然后根据hostname查找IP，然后匹配查找到的IP和客户端IP，如果hostname和IP都匹配，则通过验证。hostname必须以dot（.）开头，正确的事*.example.com*，而不是*foo.example.com*
8. IP-address、IP-mask：与*IP-address*/*mask-length*格式类似，只是掩码用255来表示，将一个字段拆成两个，例如127.0.0.1/32表示为127.0.0.1 255.255.255.255、127.1.0.0/16表示为127.1.0.0 255.255.0.0；
9. auth-method：以下是可用的身份验证方式
   - trust：无条件的允许连接，这种方式允许任何人使用任何数据库账号连接PostgreSQL服务而不需要密码认证；
   - reject：无条件的拒绝连接，可以用来设置黑名单，阻止部分主机的连接请求；
   - scram-sha-256：使用SCRAM-SHA-256算法验证用户密码；
   - md5：使用SCRAM-SHA-256或者MD5来验证用户密码；
   - password：要求客户端提供明文密码用于验证，因为密码是明文传输，不建议在未知网络环境使用；
   - gss：使用GSSAPI进行用户身份认证，在TCP/IP下可用，也就是连接类型为local时不能用；
   - sspi：使用SSPI进行用户身份认证，只能在Windows环境下使用；
   - ident：客户端自验证方式，获取客户端操作系统用户名，然后访问客户端的ident server进行身份验证，要求TCP/IP连接类型，local方式可以使用下面的peer方式代替；
   - peer：获得客户端操作系统用户名，然后访问客户端的ident server进行身份验证，仅用于local类型；
   - ldap：使用LDAP服务进行身份认证；
   - radius：使用RADIUS服务进行身份认证；
   - cert：使用SSL客户端证书进行身份认证；
   - pam：使用操作系统提供的Pluggable Authentication Modules (PAM)服务进行身份认证；
   - bsd：使用操作系统提供的BSD服务进行身份认证；
10. auth-options：验证方式字段后还可以提供一个name=value形式的键值对表单，主要用于配置验证方式所需的额外参数

> - 使用@符号指定文件时，文件内容可以使用逗号或者空格分隔，并且允许@嵌套，即文件中使用@
> - 除非@后面跟绝对路径，否则以当前引用文件所在目录为相对路径

PostgreSQL按照pg_hba.conf内记录的顺序依次进行检查，所以pg_hba.conf文件内的配置顺序是很重要的。典型的配置方式：前面的记录应该具备更加严格的匹配规则和较弱的验证方式，后面的记录配置宽松的匹配规则和强验证方式。 For example, one might wish to use `trust` authentication for local TCP/IP connections but require a password for remote TCP/IP connections. In this case a record specifying `trust` authentication for connections from 127.0.0.1 would appear before a record specifying password authentication for a wider range of allowed client IP addresses.

pg_hba.conf在PostgreSQL启动或者收到SIGHUP信号量的时候读取。因此，当你修改了pg_hba.conf文件时，你可以发送对应的信号量（使用`pg_ctl reload` 或者 `kill -HUP`）让PostgreSQL重新读取配置文件。

> 在windows环境下，pg_hba.conf文件的任何更改都会在新建立的连接上立即生效，无效额外操作

视图[`pg_hba_file_rules`](https://www.postgresql.org/docs/10/static/view-pg-hba-file-rules.html)可以用来测试和诊断配置文件是否正确并生效。下面是一个pg_hba.conf文件的例子，包含了各种配置实例：

```properties
# Allow any user on the local system to connect to any database with
# any database user name using Unix-domain sockets (the default for local
# connections).
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust

# The same using local loopback TCP/IP connections.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             127.0.0.1/32            trust

# The same as the previous line, but using a separate netmask column
#
# TYPE  DATABASE        USER            IP-ADDRESS      IP-MASK             METHOD
host    all             all             127.0.0.1       255.255.255.255     trust

# The same over IPv6.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             ::1/128                 trust

# The same using a host name (would typically cover both IPv4 and IPv6).
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             localhost               trust

# Allow any user from any host with IP address 192.168.93.x to connect
# to database "postgres" as the same user name that ident reports for
# the connection (typically the operating system user name).
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    postgres        all             192.168.93.0/24         ident

# Allow any user from host 192.168.12.10 to connect to database
# "postgres" if the user's password is correctly supplied.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    postgres        all             192.168.12.10/32        scram-sha-256

# Allow any user from hosts in the example.com domain to connect to
# any database if the user's password is correctly supplied.
#
# Require SCRAM authentication for most users, but make an exception
# for user 'mike', who uses an older client that doesn't support SCRAM
# authentication.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             mike            .example.com            md5
host    all             all             .example.com            scram-sha-256

# In the absence of preceding "host" lines, these two lines will
# reject all connections from 192.168.54.1 (since that entry will be
# matched first), but allow GSSAPI connections from anywhere else
# on the Internet.  The zero mask causes no bits of the host IP
# address to be considered, so it matches any host.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             192.168.54.1/32         reject
host    all             all             0.0.0.0/0               gss

# Allow users from 192.168.x.x hosts to connect to any database, if
# they pass the ident check.  If, for example, ident says the user is
# "bryanh" and he requests to connect as PostgreSQL user "guest1", the
# connection is allowed if there is an entry in pg_ident.conf for map
# "omicron" that says "bryanh" is allowed to connect as "guest1".
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    all             all             192.168.0.0/16          ident map=omicron

# If these are the only three lines for local connections, they will
# allow local users to connect only to their own databases (databases
# with the same name as their database user name) except for administrators
# and members of role "support", who can connect to all databases.  The file
# $PGDATA/admins contains a list of names of administrators.  Passwords
# are required in all cases.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   sameuser        all                                     md5
local   all             @admins                                 md5
local   all             +support                                md5

# The last two lines above can be combined into a single line:
local   all             @admins,+support                        md5

# The database column can also use lists and file names:
local   db1,db2,@demodbs  all                                   md5123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687
```

#### 1.2、用户名映射（User Name Maps）

当使用PostgreSQL外部的身份认证服务时（比如：Ident或者GSSAPI），操作系统的用户名可能与数据库用户名不一致。在这种情况下，可以配置两者之间的对应关系。然后在pg_hba.conf文件中引用这种对象关系配置。

例如Ident对应的映射关系默认配置在data目录下的[pg_ident](https://www.postgresql.org/docs/10/static/runtime-config-file-locations.html#GUC-IDENT-FILE).conf文件中，配置格式：

```properties
map-name system-username-1 database-username-1
map-name system-username-2 database-username-212
```

> system-username和database-username之间时多对多的关系，没有限制

除此之外，还可以配置正则表达式，从操作系统用户名中提取对应的数据库用户：

```properties
mymap   /^(.*)@mydomain\.com$      \1
mymap   /^(.*)@otherdomain\.com$   guest12
```

与pg_hba.conf文件一样，pg_ident.conf文件也是在PostgreSQl启动时读取，可以通过 `pg_ctl reload` 或 `kill -HUP` 来重新加载配置文件。

结合上一小节中的pg_hba.conf文件配置样例，其中 *map=omicron* 引用了下面例子中的映射：

```properties
# MAPNAME       SYSTEM-USERNAME         PG-USERNAME

omicron         bryanh                  bryanh
omicron         ann                     ann
# bob has user name robert on these machines
omicron         robert                  bob
# bryanh can also connect as guest1
omicron         bryanh                  guest112345678
```

在这个例子中，只有192.168网段的bryanh、ann、robert三个操作系统用户可以连接PostgreSQL数据库，且robert只能连接数据库用户bob，ann只能连接数据库用户ann，而bryanh可以选择连接数据库用户bryanh或者guest1。

#### 1.3、认证方式（Authentication Methods）

前面的内容简单介绍过，PostgreSQL目前支持十一种身份认证方式。这里详细介绍几种常用验证方式，所有认证方式的介绍参见[官方文档](https://www.postgresql.org/docs/10/static/auth-methods.html)。

- Trust authentication

```properties
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust12
```

Trust配置允许以任何数据库用户建立connection，包括superuser，且不需要验证密码。当然，database、user和address字段配置依然有效。此验证方式应该在OS层面具备足够的保护措施的前提下使用。

```shell
-bash-4.2$ pwd
/var/lib/pgsql/10/data
--- 查看默认配置 ---
-bash-4.2$ cat pg_hba.conf | grep -Ev "^#|^$"
local   all             all                                     peer
host    all             all             127.0.0.1/32            ident
host    all             all             192.168.0.0/16          password
host    all             all             ::1/128                 ident
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
--- 将第一行的peer修改为trust，表示本机上允许任何人以任何身份访问任何数据库 ---
-bash-4.2$ vi pg_hba.conf 
--- 让PostgreSQL重新加载配置 ---
-bash-4.2$ kill -HUP `head -1 postmaster.pid`
--- 登陆，不需要输入密码，默认的peer也不需要输入密码 ---
-bash-4.2$ psql 
psql (10.4)
Type "help" for help.
--- 验证修改是否生效 ---
postgres=# select * from pg_hba_file_rules;
 line_number | type  |   database    | user_name |   address   |                 netmask                 | auth_method | options | error 
-------------+-------+---------------+-----------+-------------+-----------------------------------------+-------------+---------+-------
          80 | local | {all}         | {all}     |             |                                         | trust       |         | 
          82 | host  | {all}         | {all}     | 127.0.0.1   | 255.255.255.255                         | ident       |         | 
          83 | host  | {all}         | {all}     | 192.168.0.0 | 255.255.0.0                             | password    |         | 
          85 | host  | {all}         | {all}     | ::1         | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | ident       |         | 
          88 | local | {replication} | {all}     |             |                                         | peer        |         | 
          89 | host  | {replication} | {all}     | 127.0.0.1   | 255.255.255.255                         | ident       |         | 
          90 | host  | {replication} | {all}     | ::1         | ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff | ident       |         | 
(7 rows)
--- 创建新role，验证trust ---
postgres=# create role role_trust login;
CREATE ROLE
postgres=# \q
-bash-4.2$ psql -U role_trust -d postgres
psql (10.4)
Type "help" for help.

postgres=> 12345678910111213141516171819202122232425262728293031323334353637383940
```

> - trust方式虽然不需要密码，但是依然需要为role指定login权限
> - 登陆时使用`-d`参数指定登陆数据库，都在默认登陆*role_trust*数据库

如果在一台多用户的操作系统上，你希望在local方式下使用trust方式进行身份认证，但是限制自己以外的其他用户登陆。那么就需要用到操作系统本身的文件系统权限，因为local方式使用socket file进行通信，那么限制socket file的读写权限就可以达到限制目的。如果你打算这么做，你可以设置postgresql.conf 文件中的unix_socket_permissions参数或者unix_socket_group参数。

> 现在socket file的读写权限不会对local类型以外的任何配置起作用，因为其他类型均使用TCP/IP建立连接

- Password authentication

PostgreSQL提供三种基于密码的身份认证方式，下面三种方式非常类似，不同之处在于服务端如何保存用户密码，以及客户端提供的密码格式。

1. scram-sha-256：使用scram-sha-256加密密码，在不受信的环境下避免密码嗅探，在服务端以加密hash值存储密码，这种方式时当前版本所支持的最安全的认证方式，但是不支持较老的客户端。
2. md5：以md5加密防止密码嗅探，并且避免在服务端以明文存储密码。如今，md5已经不被认为时安全的加密算法。为了从md5过度到SCRAM，如果在pg_hba.conf中指定md5方式，但是数据库中存储的密码是使用SCRAM加密的，那么将自动使用SCRAM方式进行身份认证。也就是说，在目前的版本中，即使设置了md5，在进行认证时，也会根据数据库中密码的实际加密方式动态选择认证方式。
3. password：尽可能避免使用此方式，此方式以名为传输密码，在秘密嗅探攻击面前非常脆弱。如果是在SSL环境下，那么password尚且可以安全使用。

PostgreSQL的用户密码独立与操作系统的用户密码，PostgreSQL将密码保存在*pg_authid*中，如果用户密码为空，在进行密码校验时，将始终返回失败

```shell
postgres=# select rolname, rolpassword from pg_authid;
       rolname        |             rolpassword             
----------------------+-------------------------------------
 pg_monitor           | 
 pg_read_all_settings | 
 pg_read_all_stats    | 
 pg_stat_scan_tables  | 
 pg_signal_backend    | 
 postgres             | md53175bce1d3201d16594cebf9d7eb3f9d123456789
```

> trust是不需要进行密码校验的，也就是说，即使role没有设置密码，依然可以登陆

The availability of the different password-based authentication methods depends on how a user’s password on the server is encrypted (or hashed, more accurately). This is controlled by the configuration parameter [password_encryption](https://www.postgresql.org/docs/10/static/runtime-config-connection.html#GUC-PASSWORD-ENCRYPTION) at the time the password is set. If a password was encrypted using the `scram-sha-256` setting, then it can be used for the authentication methods `scram-sha-256` and `password` (but password transmission will be in plain text in the latter case). The authentication method specification `md5` will automatically switch to using the `scram-sha-256` method in this case, as explained above, so it will also work. If a password was encrypted using the `md5` setting, then it can be used only for the `md5` and `password` authentication method specifications (again, with the password transmitted in plain text in the latter case). (Previous PostgreSQL releases supported storing the password on the server in plain text. This is no longer possible.) To check the currently stored password hashes, see the system catalog `pg_authid`.

- Ident authentication

Ident authentication通过获得客户端操作系统用户来登陆数据库，客户端操作系统用户和数据库用户直接可以建立map。Ident authentication仅工作与TCP/IP连接。

Ident authentication支持的额外选项：

1. map：Allows for mapping between system and database user names. See [Section 20.2](https://www.postgresql.org/docs/10/static/auth-username-maps.html) for details.

首先，绝大多数的累Unix操作系统都包含一个ident server，默认的监听端口为113。Ident server的基本功能就是回答问题：“哪一个操作系统用户在你的端口X和我的端口Y之间建立了连接？”。由于PostgreSQl在建立物理连接时，能确切的知道X和Y，所以，它可以询问客户端的ident server，从而得到客户端操作系统用户名，进而使用此用户名登陆数据库。

Ident authentication的缺点是依赖客户端的完整性安全。例如，攻击者可以在客户端机器上运行任何程序监听113端口，并返回他想要登陆的用户名。因此，ident authentication建议在一个隔离的内部网络环境下使用。

部分ident server非标准的加密选项，返回加密之后的操作系统用户名。此时，PostgreSQL将无法正确的得到用户名，因为PostgreSQL不知道如何解密。

- Peer authentication

Peer authentication从客户端操作系统内核获取操作系统用户，此方式仅支持local类型的连接。

Peer authentication支持的额外选项：

1. map：Allows for mapping between system and database user names. See [Section 20.2](https://www.postgresql.org/docs/10/static/auth-username-maps.html) for details.

Peer authentication is only available on operating systems providing the `getpeereid()` function, the `SO_PEERCRED` socket parameter, or similar mechanisms. Currently that includes Linux, most flavors of BSD including macOS, and Solaris.

#### 1.4、认证异常（Authentication Problems）

```shell
FATAL:  no pg_hba.conf entry for host "123.123.123.123", user "andym", database "testdb"1
```

> 出现这种情况的可能原因：pg_hba.conf没有与当前连接请求匹配的条目

```shell
FATAL:  password authentication failed for user "andym"1
```

> 出现此提示，说明你已经与PostgreSQL server建立连接，但是无法通过pg_hba.conf配置的身份认证方法认证。检查你的密码是否正确，或者检查ident service这样的外部验证。

```shell
FATAL:  user "andym" does not exist1
```

> 这说明数据库中找不到此用户，确认数据库pg_roles中是否有此角色

```shell
FATAL:  database "testdb" does not exist1
```

> 如果你指定了请求连接的数据库，那么你指定的数据库不存在。如果你没有明确指定需要连接的数据库，默认连接与用户名同名的数据库。