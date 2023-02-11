# 一、简介

原名iBatis来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。

## 1.1 特性

1. 支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架
2. 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
3. 可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old JavaObjects，普通的Java对象）映射成数据库中的记录
4. 是一个 半自动的ORM（Object Relation Mapping）框架

## 1.2 和其它持久化层技术对比

* JDBC
  * SQL 夹杂在Java代码中耦合度高，导致硬编码内伤
  * 维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见
  * 代码冗长，开发效率低
* Hibernate 和 JPA
  * 操作简便，开发效率高
  * 程序中的长难复杂 SQL 需要绕过框架
  * 内部自动生产的 SQL，不容易做特殊优化
  * 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难。
  * 反射操作太多，导致数据库性能下降
* MyBatis
  * 轻量级，性能出色
  * SQL 和 Java 编码分开，功能边界清晰。Java代码专注业务、SQL语句专注数据
  * 开发效率稍逊于HIbernate，但是完全能够接受

# 三、配置文件

核心配置文件中的标签必须按照固定的顺序(有的标签可以不写，但顺序一定不能乱)： properties、settings、typeAliases、typeHandlers、objectFactory、objectWrapperFactory、reflectorFactory、plugins、environments、databaseIdProvider、mappers

* 相关概念：ORM（Object Relationship Mapping）对象关系映射。
    * 对象：Java的实体类对象
    * 关系：关系型数据库
    * 映射：二者之间的对应关系
* 映射文件的命名规则
    * 表所对应的实体类的类名+Mapper.xml
    * 例如：表t_user，映射的实体类为User，所对应的映射文件为UserMapper.xml
    * 因此一个映射文件对应一个实体类，对应一张表的操作
    * MyBatis映射文件用于编写SQL，访问以及操作表中的数据
    * MyBatis映射文件存放的位置是src/main/resources/mappers目录下
* MyBatis中可以面向接口操作数据，要保证两个一致
    * mapper接口的全类名和映射文件的命名空间（namespace）保持一致
    * mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致


``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//MyBatis.org//DTD Config 3.0//EN"
        "http://MyBatis.org/dtd/MyBatis-3-config.dtd">
<configuration>
    <!--引入properties文件，此时就可以${属性名}的方式访问属性值-->
    <properties resource="jdbc.properties"></properties>
    <settings>
        <!--将表中字段的下划线自动转换为驼峰-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!--开启延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
    <typeAliases>
        <!--
        typeAlias：设置某个具体的类型的别名
        属性：
        type：需要设置别名的类型的全类名
        alias：设置此类型的别名，且别名不区分大小写。若不设置此属性，该类型拥有默认的别名，即类名
        -->
        <!--<typeAlias type="com.jiao.mybatis.bean.User"></typeAlias>-->
        <!--<typeAlias type="com.jiao.mybatis.bean.User" alias="user">
        </typeAlias>-->
        <!--以包为单位，设置改包下所有的类型都拥有默认的别名，即类名且不区分大小写-->
        <package name="com.jiao.mybatis.bean"/>
    </typeAliases>
    <!--
    environments：设置多个连接数据库的环境
    属性：
        default：设置默认使用的环境的id
    -->
    <environments default="mysql_test">
        <!--
        environment：设置具体的连接数据库的环境信息
        属性：
            id：设置环境的唯一标识，可通过environments标签中的default设置某一个环境的id，表示默认使用的环境
        -->
        <environment id="mysql_test">
            <!--
            transactionManager：设置事务管理方式
            属性：
                type：设置事务管理方式，type="JDBC|MANAGED"
                type="JDBC"：设置当前环境的事务管理都必须手动处理
                type="MANAGED"：设置事务被管理，例如spring中的AOP
            -->
            <transactionManager type="JDBC"/>
            <!--
            dataSource：设置数据源
            属性：
                type：设置数据源的类型，type="POOLED|UNPOOLED|JNDI"
                type="POOLED"：使用数据库连接池，即会将创建的连接进行缓存，下次使用可以从缓存中直接获取，不需要重新创建
                type="UNPOOLED"：不使用数据库连接池，即每次使用连接都需要重新创建
                type="JNDI"：调用上下文中的数据源
            -->
            <dataSource type="POOLED">
                <!--设置驱动类的全类名-->
                <property name="driver" value="${jdbc.driver}"/>
                <!--设置连接数据库的连接地址-->
                <property name="url" value="${jdbc.url}"/>
                <!--设置连接数据库的用户名-->
                <property name="username" value="${jdbc.username}"/>
                <!--设置连接数据库的密码-->
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--引入映射文件-->
    <mappers>
        <!-- <mapper resource="UserMapper.xml"/> -->
        <!--
        以包为单位，将包下所有的映射文件引入核心配置文件
        注意：
            1. 此方式必须保证mapper接口和mapper映射文件必须在相同的包下
            2. mapper接口要和mapper映射文件的名字一致
        -->
        <package name="com.jiao.mybatis.mapper"/>
    </mappers>
</configuration>
```

# 四、MyBatis获取参数值的两种方式

MyBatis获取参数值的两种方式：${}和#{}

${}的本质就是字符串拼接，#{}的本质就是占位符赋值

${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动加单引号；但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自动添加单引号

## 4.1 实体类类型的参数

### i. 单个字面量类型的参数

可以使用\${}和#{}以任意的名称（最好见名识意）获取参数的值,${}需要手动加单引号

``` xml 
<select id="getUserByUsername" resultType="User">
    //#{}
    select * from t_user where username = #{username}
    //${}
    select * from t_user where username = '${username}' 
</select>
```

### ii. 多个字面量类型的参数

* 参数为多个时，此时MyBatis会自动将这些参数放在一个map集合中:
  * 以arg0,arg1...为键，以参数为值；
  * 以param1,param2...为键，以参数为值；
* 只需要通过${}和#{}访问map集合的键就可以获取相对应的值
* 使用arg或者param都行，arg是从arg0开始的，param是从param1开始的

### iii. map集合类型的参数

可以手动创建map集合，将这些数据放在map中只需要通过${}和#{}访问map集合的键就可以获取相对应的值

``` java
public void checkLoginByMap() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
    Map<String,Object> map = new HashMap<>();
    // select * from t_user where username = #{username} and password = #{password}
    map.put("usermane","admin");
    map.put("password","123456");
    User user = mapper.checkLoginByMap(map);
}
```

### iv. 实体类类型的参数

直接使用\${}和#{}

``` java 
public void insertUser() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
    // insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
    User user = new User(null,"Tom","123456",12,"男","123@321.com");
    mapper.insertUser(user);
}
```

## 4.2 使用@Param标识参数

可以通过@Param注解标识mapper接口中的方法参数，此时，会将这些参数放在map集合中
1. 以@Param注解的value属性值为键，以参数为值；
2. 以param1,param2...为键，以参数为值；

``` java
public void checkLoginByParam() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    ParameterMapper mapper = sqlSession.getMapper(ParameterMapper.class);
    //User CheckLoginByParam(@Param("username") String username, @Param("password") String password);
    //select * from t_user where username = #{username} and password = #{password}
    mapper.CheckLoginByParam("admin","123456");
}
```

# 参考文献

[尚硅谷](https://www.bilibili.com/video/BV1VP4y1c7j7)