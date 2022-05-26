# Postgresql+Springboot yml基本使用

# 一、添加依赖

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
</dependency>
```

连接PostgreSQL时需要手动指定schema位置，否则，连接上的database会默认使用`public`这个内置的schema，导致在查询别的schema下的表时，会报类似如下的错误：

```java
nested exception is org.postgresql.util.PSQLException: ERROR: relation "xxxTable" does not exist
```

利用pgAdmin4，在控制界面上输入如下的SQL切换schema：

```java
ALTER ROLE postgres SET SEARCH_PATH ='ROS'; #ROS是schema名，postgres是database用户名
```

也可以通过PostgreSQL提供的命令行界面来做切换：

```bash
root@1dc27bbb5253:/ su - postgres    #首先需要切换用户到[postgres]用户
postgres@1dc27bbb5253:~$ psql        #进入命令行模式
psql (11.2 (Debian 11.2-1.pgdg90+1))
Type "help" for help.

postgres= \c minedb;    #切换数据库
minedb=# ALTER ROLE postgres SET SEARCH_PATH ='ROS';
ALTER ROLE
```

```bash
root@1dc27bbb5253:/# psql -U postgres -d minedb
minedb=# ALTER ROLE postgres SET SEARCH_PATH ='ROS';
ALTER ROLE
```

# 二、修改配置文件

## 2.1 application.yml

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/postgres
    username: postgres
    password: 123456
    driverClassName: org.postgresql.Driver
  jackson:
    time-zone: GMT+8
mybatis:
  type-handlers-package: zsh.demos.postgres.mybatis.typehandler
  mapper-locations: classpath:mapping/*.xml
```

## 2.2 mybatis-generator.xml

我们使用了mybatis-generator-maven-plugin这个插件快速生成通用CRUD配置xml
为了使用这个插件，我们需要新建一个mybatis-generator.xml文件来指导插件工作：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
    PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
    "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--数据库驱动-->
    <classPathEntry location="C:\Users\xxxuser\.m2\repository\org\postgresql\postgresql\42.2.5\postgresql-42.2.5.jar"/>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接地址账号密码-->
        <jdbcConnection driverClass="org.postgresql.Driver" connectionURL="jdbc:postgresql://172.22.122.27:5432/minedb" userId="postgres" password="aq1sw2de"></jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!--生成Model类存放位置-->
        <javaModelGenerator targetPackage="zsh.demos.postgres.dao.pojo" targetProject="./src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!--生成映射文件存放位置-->
        <sqlMapGenerator targetPackage="mapping" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--生成Dao类存放位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="zsh.demos.postgres.dao.mapper" targetProject="./src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!--生成对应表及类名-->
        <!--<table tableName="big_table" domainObjectName="BigTable" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>-->
        <table tableName="vehicle" domainObjectName="Vehicle" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
    </context>
</generatorConfiguration>
```



# 三、测试

## 3.1 使用注解配置

```java
@Configuration
@MapperScan(basePackages = "zsh.demos.postgres.dao.mapper", sqlSessionFactoryRef = "pgSqlSessionFactory")
public class PostgresConfig {

    @Value("${mybatis.mapper-locations}")
    private String MAPPER_LOCATION;
    @Value("${mybatis.type-handlers-package}")
    private String TYPE_HANDLERS_PACKAGE;

    @Bean(name = "pgSqlSessionFactory")
    public SqlSessionFactory postgresSqlSessionFactory(@Autowired DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);

        // case change.
        org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
        configuration.setMapUnderscoreToCamelCase(true);

        sqlSessionFactoryBean.setConfiguration(configuration);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        sqlSessionFactoryBean.setTypeHandlersPackage(TYPE_HANDLERS_PACKAGE);
        return sqlSessionFactoryBean.getObject();
    }
}
```

## 3.2 TypeHandler

TypeHandler是针对`JDBCType=OTHER`的扩展，例如Postgres支持的JSON格式数据，我们需要手动定义如下TypeHandler:

基类

```java
public abstract class JSONTypeHandler<T> implements TypeHandler<T> {

    /**
     * json数据和类名的分隔符号
     */
    protected Class<T> jsonClass = null;

    @Override
    public void setParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
        // TODO Auto-generated method stub
        if (parameter == null) {
            ps.setString(i, "");
            return;
        }
        String json = JSONUtils.jsonToJSONStr(parameter);
        PGobject pGobject = new PGobject();
        pGobject.setType("json");
        pGobject.setValue(json);
//      ps.setString(i, json);
        ps.setObject(i, pGobject);
    }

    @Override
    public T getResult(ResultSet rs, String columnName) throws SQLException {
        // TODO Auto-generated method stub
        String json = rs.getString(columnName);
        return jsonToObject(json);
    }

    @Override
    public T getResult(CallableStatement cs, int columnIndex) throws SQLException {
        // TODO Auto-generated method stub
        String json = cs.getString(columnIndex);
        return jsonToObject(json);
    }

    @Override
    public T getResult(ResultSet rs, int columnIndex) throws SQLException {
        // TODO Auto-generated method stub
        String json = rs.getString(columnIndex);
        return jsonToObject(json);
    }
    /**
     * json 转换成对象
     */
    protected T jsonToObject(String json) {
        if (StringUtils.isEmpty(json)) {
            return null;
        }
        T ob = JSONUtils.jsonStrToJSON(json, jsonClass);
        return ob;
    }
}
```

```java
public class JSONUtils {
    //
    public static <T> String jsonToJSONStr(T json) {
        String jsonString = "";
        ObjectMapper mapper = new ObjectMapper();
        try {
            jsonString = mapper.writeValueAsString(json);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        return jsonString;
    }
    //
    public static <T> T jsonStrToJSON(String json, Class<T> clazz) {
        ObjectMapper mapper = new ObjectMapper();
        T object = null;
        try {
            object = mapper.readValue(json, clazz);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return object;
    }
    //
    public static <T> T jsonStrToJSON(String json, TypeReference<T> type) {
        ObjectMapper mapper = new ObjectMapper();
        T object = null;
        try {
            object = mapper.readValue(json, type);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return object;
    }
}
```

具体业务对象类：

```java
@MappedTypes(BusinessBean.class)
public class BusinessBeanHandler extends JSONTypeHandler<BusinessBean> {
    public BusinessBeanHandler () {this.jsonClass = BusinessBean.class;}
}
```

使用时，需要再resultMap中指定需要typeHandler转换的字段：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="zsh.demos.postgres.dao.mapper.VehicleMapper">
  <resultMap id="BaseResultMap" type="zsh.demos.postgres.dao.pojo.BusinessBean">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="business_json" jdbcType="OTHER" property="businessJSON" typeHandler="zsh.demos.postgres.mybatis.typehandler.BusinessBeanHandler"/>
    <result column="business_string" jdbcType="CHAR" property="businessString" />
  </resultMap>
</mapper>
```

### 3.2.1 TypeHandler的坑

自定义的TypeHandler主要是转换Jsonb和array等类型

如果是使用mybatisplus的内置方法,则需要在实体字段加上@TableField注解,并且需要在类名上启动@TableName(autoResultMap = true)

```java
// autoResultMap = true 必须写,否则无法识别
@TableName(autoResultMap = true)
public class BlogUser implements Serializable {

    private static final long serialVersionUID = 1L;

    private Long id;

    private String name;

    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createTime;

    private Integer version;
    // 使用类型转换,否则无法增删改查
    @TableField(typeHandler= JsonTypeHandler.class)
    private Map<String,Object> relation;
}
```

如果是写在xml里面,则必须在对应字段注明转换器class:

```xml
<insert id="addxml" parameterType="com.hou.postgresql.blog.entity.po.BlogUser">
        INSERT INTO blog_user (name, relation, fans, birthday, points, login_time, write_interval, numbers, adult, address, weight)
        VALUES (#{name},
        /*必须显式的指明转换器,否则编译过程就会报错,主要是List,map这种数组,jsonb对应的实体类型*/
        #{relation,typeHandler=com.hou.postgresql.handler.JsonTypeHandler},
        #{fans,typeHandler=com.hou.postgresql.handler.ArrayTypeHandler}, #{birthday}, #{points}, #{loginTime}, #{writeInterval}::interval,
        #{numbers,typeHandler=com.hou.postgresql.handler.ArrayTypeHandler}, #{adult}, #{address}, #{weight})

    </insert>
```

column is of type jsonb but expression is of type character varying问题

即使写了转换器,查询的时候没问题,但是插入的时候依然会报这个错,这时需要在连接的url后面加上参数stringtype=unspecified就可以正常添加了

```yaml
url: jdbc:postgresql://192.168.1.11:5432/postgres?currentSchema=sys&stringtype=unspecified
```



# 四、事务管理器

如果应用只配置了一个数据源，那么在默认情况下，SpringBoot在
`org.springframework.boot.autoconfig ure.jdbc.DataSourceTransactionManagerAutoConfiguration`
自动配置类中已经为我们配好了一个默认的事务管理器。并且在
`org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration`
中帮我们自动启动了事务管理支持`@EnableTransactionManagement`所以我们无需做任何配置。

# 五、踩坑点

## 5.1 schemas问题

pgsql默认的是public,如果用mybatisplus的内置方法的话,是需要指定连接的currentSchema的,否则只会默认查询public,自己写sql可以在前面加上schemas
但是使用内置方法没有,必须在连接url指定schemsa,否则会报ERROR: relation "item" does not exist表不存在。

## 5.2 所有数据类型参数格式

url后面加上stringtype=unspecified就可以使用任意格式插入了,除了json和array之外,其他的特殊类型,比如地址,间隔,时间等都可以使用string
参数如下

```json
{
  "address": "192.168.1.70",   // inet
  "adult": false,  // boolean
  "birthday": "1994-12-16",  // date
  "fans": ["zhangpeng","zhouhang","pengle"],
  "loginTime": "09:12",  // time
  "name": "侯征",
  "numbers": [12,56,42],   // array
  "points": 10.522,  // numeric
  "relation": {       // jsonb
      "key": "value"
  },
  "weight": "[45,50]",    // 区间
  "writeInterval": "800"  // 时间间隔,单位秒
}
```

