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

# 五、查询

1. 如果查询出的数据只有一条，可以通过
	1. 实体类对象接收
	2. List集合接收
	3. Map集合接收
2. 如果查询出的数据有多条，一定不能用实体类对象接收，会抛异常TooManyResultsException，可以通过
	1. 实体类类型的LIst集合接收
	2. Map类型的LIst集合接收
	3. 在mapper接口的方法上添加@MapKey注解

# 六、特殊SQL

最多冲突的是#{value}解析出的值为'value'，解决此冲突就行。

## 6.1 模糊查询

``` SQL
1.select * from t_user where username like '%${mohu}%' 
2.select * from t_user where username like concat('%',#{mohu},'%') 
-- 最常用的
3.select * from t_user where username like "%"#{mohu}"%"
```

## 6.2 批量删除


只能使用${}，如果使用#{}，则解析后的sql语句为delete from t_user where id in ('1,2,3')，这样是将1,2,3看做是一个整体，只有id为1,2,3的数据会被删除。
正确的语句应该是delete from t_user where id in (1,2,3)，或者delete from t_user where id in ('1','2','3')

## 6.3 动态设置表名

``` sql
select * from ${tableName}
```

## 6.4 添加功能获取自增的主键

### 6.4.1 使用场景
* t_clazz(clazz_id,clazz_name)
* t_student(student_id,student_name,clazz_id)

1. 添加班级信息
2. 获取新添加的班级的id
3. 为班级分配学生，即将某学的班级id修改为新添加的班级的id

### 6.4.2 操作方法
* 在mapper.xml中设置两个属性
    * useGeneratedKeys：设置使用自增的主键
    * keyProperty：因为增删改有统一的返回值是受影响的行数，因此只能将获取的自增的主键放在传输的参数user对象的某个属性中

# 七、自定义映射resultMap

## 7.1 resultMap处理字段和属性的映射关系

* 属性：
    * id：表示自定义映射的唯一标识，不能重复
    * type：查询的数据要映射的实体类的类型
* 子标签：
    * id：设置主键的映射关系
    * result：设置普通字段的映射关系
* 子标签属性：
    * property：设置映射关系中实体类中的属性名
    * column：设置映射关系中表中的字段名

若字段名和实体类中的属性名不一致，则可以通过resultMap设置自定义映射，即使字段名和属性名一致的属性也要映射，也就是全部属性都要列出来

``` xml
<resultMap id="empResultMap" type="Emp">
    <id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
</resultMap>
<!--List<Emp> getAllEmp();-->
<select id="getAllEmp" resultMap="empResultMap">
    select * from t_emp
</select>
```

>注：
还有其他两种方式：
一个是起别名，另一个是MyBatis中的核心配置文件中有一个全局配置mapUnderscoreToCamelCase自动将_类型的字段名转换为驼峰。

## 7.2 多对一映射处理

表和表之间的关系延伸到类与类之间的关系

### 7.2.1 级联方式处理映射关系

``` xml
<resultMap id="empAndDeptResultMapOne" type="Emp">
    <id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    <result property="dept.did" column="did"></result>
    <result property="dept.deptName" column="dept_name"></result>
</resultMap>
<!--Emp getEmpAndDept(@Param("eid")Integer eid);-->
<select id="getEmpAndDept" resultMap="empAndDeptResultMapOne">
    select * from t_emp left join t_dept on t_emp.eid = t_dept.did where t_emp.eid = #{eid}
</select>
```

### 7.2.2 使用association处理映射关系

association：处理多对一的映射关系
property：需要处理多对的映射关系的属性名
javaType：该属性的类型

``` xml
<resultMap id="empAndDeptResultMapTwo" type="Emp">
    <id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    <association property="dept" javaType="Dept">
        <id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>
    </association>
</resultMap>
<!--Emp getEmpAndDept(@Param("eid")Integer eid);-->
<select id="getEmpAndDept" resultMap="empAndDeptResultMapTwo">
    select * from t_emp left join t_dept on t_emp.eid = t_dept.did where t_emp.eid = #{eid}
</select>
```

### 7.2.3 分步查询

select：设置分布查询的sql的唯一标识（namespace.SQLId或mapper接口的全类名.方法名）
column：设置分步查询的条件

``` xml
<!-- 员工信息 -->
<resultMap id="empAndDeptByStepResultMap" type="Emp">
    <id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    <association property="dept"
                 select="com.jiao.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                 column="did"></association>
</resultMap>
<!--Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);-->
<select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMap">
    select * from t_emp where eid = #{eid}
</select>

<!-- 部门信息 -->
<resultMap id="EmpAndDeptByStepTwoResultMap" type="Dept">
    <id property="did" column="did"></id>
    <result property="deptName" column="dept_name"></result>
</resultMap>
<!--Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);-->
<select id="getEmpAndDeptByStepTwo" resultMap="EmpAndDeptByStepTwoResultMap">
    select * from t_dept where did = #{did}
</select>
```

## 7.3 一对多映射处理

### 7.3.1 collection

collection：用来处理一对多的映射关系
ofType：表示该属性对饮的集合中存储的数据的类型

``` xml
<resultMap id="DeptAndEmpResultMap" type="Dept">
    <id property="did" column="did"></id>
    <result property="deptName" column="dept_name"></result>
    <collection property="emps" ofType="Emp">
        <id property="eid" column="eid"></id>
        <result property="empName" column="emp_name"></result>
        <result property="age" column="age"></result>
        <result property="sex" column="sex"></result>
        <result property="email" column="email"></result>
    </collection>
</resultMap>
<!--Dept getDeptAndEmp(@Param("did") Integer did);-->
<select id="getDeptAndEmp" resultMap="DeptAndEmpResultMap">
    select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did = #{did}
</select>
```

### 7.3.2 分布查询

``` xml
<!-- 部门信息 -->
<resultMap id="DeptAndEmpByStepOneResultMap" type="Dept">
    <id property="did" column="did"></id>
    <result property="deptName" column="dept_name"></result>
    <collection property="emps"
                select="com.jiao.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
                column="did"></collection>
</resultMap>
<!--Dept getDeptAndEmpByStepOne(@Param("did") Integer did);-->
<select id="getDeptAndEmpByStepOne" resultMap="DeptAndEmpByStepOneResultMap">
    select * from t_dept where did = #{did}
</select>

<!-- 员工信息 -->
<!--List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);-->
<select id="getDeptAndEmpByStepTwo" resultType="Emp">
    select * from t_emp where did = #{did}
</select>

```

## 7.4 延迟加载

对于分步查询来说可以根据访问信息执行相对的SQL不执行全部。

需要在全局设置中设置
* lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载
* aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载

全局设置之后，可通过association和collection中的fetchType属性设置当前的分步查询是否使用延迟加载，fetchType="lazy(延迟加载)|eager(立即加载)"


# 八、动态SQL

是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决拼接SQL语句字符串时的痛点问题

## 8.1 if

if标签可通过test属性（即传递过来的数据）的表达式进行判断，若表达式的结果为true，则标签中的内容会执行；反之标签中的内容不会执行

>注意:
在where后面添加一个恒成立条件1=1,这个恒成立条件并不会影响查询的结果,这个1=1可以用来拼接and语句，例如：当empName为null时
1.如果不加上恒成立条件，则SQL语句为select * from t_emp where and age = ? and sex = ? and email = ?，此时where会与and连用，SQL语句会报错
2.如果加上一个恒成立条件，则SQL语句为select * from t_emp where 1= 1 and age = ? and sex = ? and email = ?，此时不报错

``` xml
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
    select * from t_emp where 1=1
    <if test="empName != null and empName !=''">
        and emp_name = #{empName}
    </if>
    <if test="age != null and age !=''">
        and age = #{age}
    </if>
    <if test="sex != null and sex !=''">
        and sex = #{sex}
    </if>
    <if test="email != null and email !=''">
        and email = #{email}
    </if>
</select>
```

## 8.2 where

where和if一般结合使用：
* 若where标签中的if条件都不满足，则where标签没有任何功能，即不会添加where关键字
* 若where标签中的if条件满足，则where标签会自动添加where关键字，并将条件最前方多余的and/or去掉

``` xml
<select id="getEmpByCondition" resultType="Emp">
    select * from t_emp
    <where>
        <if test="empName != null and empName !=''">
            emp_name = #{empName}
        </if>
        <if test="age != null and age !=''">
            and age = #{age}
        </if>
        <if test="sex != null and sex !=''">
            and sex = #{sex}
        </if>
        <if test="email != null and email !=''">
            and email = #{email}
        </if>
    </where>
</select>
```

## 8.3 trim

trim用于去掉或添加标签中的内容

常用属性
prefix：在trim标签中的内容的前面添加某些内容
suffix：在trim标签中的内容的后面添加某些内容
prefixOverrides：在trim标签中的内容的前面去掉某些内容
suffixOverrides：在trim标签中的内容的后面去掉某些内容

若trim中的标签都不满足条件，则trim标签没有任何效果

``` xml
<select id="getEmpByCondition" resultType="Emp">
    select * from t_emp
    <trim prefix="where" suffixOverrides="and|or">
        <if test="empName != null and empName !=''">
            emp_name = #{empName} and
        </if>
        <if test="age != null and age !=''">
            age = #{age} and
        </if>
        <if test="sex != null and sex !=''">
            sex = #{sex} or
        </if>
        <if test="email != null and email !=''">
            email = #{email}
        </if>
    </trim>
</select>
```

## 8.4 choose、when、otherwise

与if的区别是if都会判断choose只要有一个执行剩下的都不执行

choose、when、otherwise相当于if...else if..else

when至少要有一个，otherwise至多只有一个

``` xml
<select id="getEmpByChoose" resultType="Emp">
    select * from t_emp
    <where>
        <choose>
            <when test="empName != null and empName != ''">
                emp_name = #{empName}
            </when>
            <when test="age != null and age != ''">
                age = #{age}
            </when>
            <when test="sex != null and sex != ''">
                sex = #{sex}
            </when>
            <when test="email != null and email != ''">
                email = #{email}
            </when>
            <otherwise>
                did = 1
            </otherwise>
        </choose>
    </where>
</select>
```

## 8.5 foreach

循环

属性：
collection：设置要循环的数组或集合
item：表示集合或数组中的每一个数据
separator：设置循环体之间的分隔符，分隔符前后默认有一个空格，如,
open：设置foreach标签中的内容的开始符
close：设置foreach标签中的内容的结束符

``` xml
<!-- 批量删除 -->
<!--int deleteMoreByArray(Integer[] eids);-->
<delete id="deleteMoreByArray">
    <!-- 等同于delete from t_emp where eid in (${eids}) -->
    delete from t_emp where eid in
    <foreach collection="eids" item="eid" separator="," open="(" close=")">
        #{eid}
    </foreach>
</delete>
```

## 8.6 SQL片段

可以记录一段公共sql片段，在使用的地方通过include标签进行引入

```xml
<!-- 声明sql片段 -->
<sql id="empColumns">eid,emp_name,age,sex,email</sql>

<!-- 引用sql片段 -->
<!--List<Emp> getEmpByCondition(Emp emp);-->
<select id="getEmpByCondition" resultType="Emp">
    <!-- select eid,emp_name,age,sex,email for t_emp -->
	select <include refid="empColumns"></include> from t_emp
</select>
```

# 九、缓存

一级缓存默认开启

## 9.1 一级缓存

一级缓存是SqlSession级别的

使一级缓存失效的情况：
1. 同一个SqlSession但是查询条件不同
2. 同一个SqlSession两次查询期间执行了任何一次增删改操作
3. 同一个SqlSession两次查询期间手动清空了缓存

## 9.2 二级缓存

二级缓存是SqlSessionFactory级别

二级缓存开启的条件：
1. 在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置
2. 在映射文件中设置标签
3. 二级缓存必须在SqlSession关闭或提交之后有效
4. 查询的数据所转换的实体类类型必须实现序列化的接口

使二级缓存失效的情况：两次查询之间执行了任意的增删改

## 9.3 二级缓存的相关配置

在mapper配置文件中添加的cache标签可以设置一些属性

* eviction属性：缓存回收策略
  * 默认的是 LRU
    * LRU（Least Recently Used） – 最近最少使用的：移除最长时间不被使用的对象。
    * FIFO（First in First out） – 先进先出：按对象进入缓存的顺序来移除它们。
    * SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
    * WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
* flushInterval属性：刷新间隔，单位毫秒
    * 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句（增删改）时刷新
* size属性：引用数目，正整数
    * 代表缓存最多可以存储多少个对象，太大容易导致内存溢出
* readOnly属性：只读，true/false
    * true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。
    * false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是false

## 9.4 查询的顺序

先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用
如果二级缓存没有命中，再查询一级缓存
如果一级缓存也没有命中，则查询数据库
SqlSession关闭之后，一级缓存中的数据会写入二级缓存

## 9.5 整合第三方缓存EHCache

第三方缓存只能代替二级缓存

### 9.5.1 配置文件ehcache.xml

| 属性名 | 是否必须 | 作用 |
| --- | --- | --- |
| maxElementsInMemory | 是 | 在内存中缓存的element的最大数目 |
| maxElementsOnDisk | 是 | 在磁盘上缓存的element的最大数目，若是0表示无穷大 |
| eternal | 是 | 设定缓存的elements是否永远不过期。 如果为true，则缓存的数据始终有效， 如果为false那么还要根据timeToIdleSeconds、timeToLiveSeconds判断 |
| overflowToDisk | 是 | 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上 |
| timeToIdleSeconds | 否 | 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时， 这些数据便会删除，默认值是0,也就是可闲置时间无穷大 |
| timeToLiveSeconds | 否 | 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大 |
| diskSpoolBufferSizeMB | 否 | DiskStore(磁盘缓存)的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区 |
| diskPersistent | 否 | 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false |
| diskExpiryThreadIntervalSeconds | 否 | 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s， 相应的线程会进行一次EhCache中数据的清理工作 |
| memoryStoreEvictionPolicy | 否 | 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。 默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出 |

### 9.5.2 设置二级缓存的类型

在xxxMapper.xml文件中设置二级缓存类型

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

### 9.5.3 加入logback日志

存在SLF4J时，作为简易日志的log4j将失效，此时我们需要借助SLF4J的具体实现logback来打印日志。创建logback的配置文件`logback.xml`，名字固定，不可改变

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <!-- 指定日志输出的位置 -->
    <appender name="STDOUT"
              class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 日志输出的格式 -->
            <!-- 按照顺序分别是：时间、日志级别、线程名称、打印日志的类、日志主体内容、换行 -->
            <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger] [%msg]%n</pattern>
        </encoder>
    </appender>
    <!-- 设置全局日志级别。日志级别按顺序分别是：DEBUG、INFO、WARN、ERROR -->
    <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
    <root level="DEBUG">
        <!-- 指定打印日志的appender，这里通过“STDOUT”引用了前面配置的appender -->
        <appender-ref ref="STDOUT" />
    </root>
    <!-- 根据特殊需求指定局部日志级别 -->
    <logger name="com.jiao.crowd.mapper" level="DEBUG"/>
</configuration>
```

# 十、逆向工程

代码生成器

正向工程：先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的

逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成如下资源：
Java实体类
Mapper接口
Mapper映射文件

## 10.1 逆向工程的配置文件generatorConfig.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
    targetRuntime: 执行生成的逆向工程的版本
    MyBatis3Simple: 生成基本的CRUD（清新简洁版）
    MyBatis3: 生成带条件的CRUD（奢华尊享版）
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.jiao.mybatis.pojo" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.jiao.mybatis.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.jiao.mybatis.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
```

## 10.2 QBC(Query By Criteria)

### 10.2.1 查询
- `selectByExample`：按条件查询，需要传入一个example对象或者null；如果传入一个null，则表示没有条件，也就是查询所有数据
- `example.createCriteria().xxx`：创建条件对象，通过andXXX方法为SQL添加查询添加，每个条件之间是and关系
- `example.or().xxx`：将之前添加的条件通过or拼接其他条件
- `example.createCriteria().xxx`：创建条件对象，通过andXXX方法为SQL添加查询添加，每个条件之间是and关系
- `example.or().xxx`：将之前添加的条件通过or拼接其他条件
  
``` java
public void testMBG() throws IOException {

    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);

    EmpExample example = new EmpExample();
    //名字为张三，且年龄大于等于20
    example.createCriteria().andEmpNameEqualTo("张三").andAgeGreaterThanOrEqualTo(20);
    //或者did不为空
    example.or().andDidIsNotNull();
    List<Emp> emps = mapper.selectByExample(example);
    emps.forEach(System.out::println);
}
```

### 10.2.2 增改

- `updateByPrimaryKey`：通过主键进行数据修改，如果某一个值为null，也会将对应的字段改为null
- `updateByPrimaryKeySelective()`：通过主键进行选择性数据修改，如果某个值为null，则不修改这个字段


# 十一、分页插件

## 11.1 开启分页功能

在查询功能之前使用PageHelper.startPage(int pageNum, int pageSize)开启分页功能
pageNum：当前页的页码
pageSize：每页显示的条数

## 11.2 分页相关数据

>常用数据
pageNum：当前页的页码
pageSize：每页显示的条数
size：当前页显示的真实条数
total：总记录数
pages：总页数
prePage：上一页的页码
nextPage：下一页的页码
isFirstPage/isLastPage：是否为第一页/最后一页
hasPreviousPage/hasNextPage：是否存在上一页/下一页
navigatePages：导航分页的页码数
navigatepageNums：导航分页的页码，[1,2,3,4,5]

### 11.2.1 直接输出

``` java
public void testPageHelper() throws IOException {
    InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
    //访问第一页，每页四条数据
    Page<Object> page = PageHelper.startPage(1, 4);
    List<Emp> emps = mapper.selectByExample(null);
    //在查询到List集合后，打印分页数据
    System.out.println(page);
}
```

输出：
```
Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}
[
    Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, 
    Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}
]
```

### 11.2.2 使用PageInfo

在查询获取list集合之后，使用PageInfo<T> pageInfo = new PageInfo<>(List<T> list, intnavigatePages)获取分页相关数据
* list：分页之后的数据
* navigatePages：导航分页的页码数

```java
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	PageHelper.startPage(1, 4);
	List<Emp> emps = mapper.selectByExample(null);
	PageInfo<Emp> page = new PageInfo<>(emps,5);
	System.out.println(page);
}
```

输出：

```
PageInfo{
    pageNum=1, pageSize=4, size=4, startRow=1, endRow=4, total=8, pages=2, 
    list=Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}
    [
        Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, 
        Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}
    ], 
    prePage=0, nextPage=2, isFirstPage=true, isLastPage=false, hasPreviousPage=false, hasNextPage=true, navigatePages=5, navigateFirstPage=1, navigateLastPage=2, navigatepageNums=[1, 2]
}
```

# 参考文献

[尚硅谷](https://www.bilibili.com/video/BV1VP4y1c7j7)