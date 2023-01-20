# 一、Spring概念

# 1.1 概述

1. Spring是轻量级的开源的JavaEE框架
2. Spring可以解决企业应用开发的复杂性
3. Spring有两个核心部分：IOC和Aop
   1. IOC：控制反转，把创建对象过程交给Spring进行管理
   2. Aop：面向切面，不修改源代码进行功能增强

# 1.2 特点

1. 方便解耦，简化开发，尽可能做到能不该源码就不改源码，把所有修改操作都在配置文件中修改。
2. Aop编程支持
3. 方便程序测试
4. 方便和其他框架进行整合
5. 方便进行事务操作
6. 降低API开发难度

# 二、IOC

## 2.1 概念和原理

IOC是控制反转，在此特指把对象创建和对象之间的调用过程，交给Spring进行管理。

**使用IOC目的**：为了耦合度降低

**IOC底层原理**：xml解析、工厂模式、反射

## 2.2 接口

IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

### 2.2.1 BeanFactory

IOC容器基本实现，是Spring内部的使用接口，不提供开发人员进行使用

* 加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象

### 2.2.2 ApplicationContext

BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人员进行使用

* 加载配置文件时候就会把在配置文件对象进行创建

#### 2.2.2.1FileSystemXmlApplicationContext & ClassPathXmlApplicationContext

File对应的文件存放在其他地方，Class对应的文件存放在src文件夹下

## 2.3 Bean管理

Spring创建对象
Spirng注入属性

### 2.3.1 xml实现

#### 2.3.1.1 对象

在Spring配置文件中使用bean标签，标签里面添加对应属性，就可以实现对象创建。

常用属性：
* id属性：唯一标识
* class属性：类全路径（包类路径）

``` xml
<bean id="" class=""></bean>
```

#### 2.3.1.2 属性

1.set方法(注：p名称空间注入做出了简化)

``` xml
<bean id="" class="">
   <property name="" value=""></property>
</bean>
```

2.有参构造注入

``` xml
<bean id="" class="">
   <constructor-arg name="" value=""></constructor-arg>
</bean>
```
### 2.3.2 注解实现







# 三、AOP

# 四、JdbcTemplate

# 五、事务管理

# 六、Spring5新特性