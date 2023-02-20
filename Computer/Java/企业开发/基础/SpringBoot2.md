# 一、简介

SpringBoot简化了Spring技术栈的整合。

优点：


创建独立Spring应用

内嵌web服务器

自动starter依赖，简化构建配置

自动配置Spring以及第三方功能

提供生产级别的监控、健康检查及外部化配置

无代码生成、无需编写XML

## 1.1 微服务

James Lewis and Martin Fowler (2014)  提出微服务完整概念。
中文文档：https://blog.cuicc.com/blog/2015/07/22/microservices/

微服务是一种架构风格
一个应用拆分为一组小型服务
每个服务运行在自己的进程内，也就是可独立部署和升级
服务之间使用轻量级HTTP交互
服务围绕业务功能拆分
可以由全自动部署机制独立部署
去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

## 1.2 分布式

部署在不同服务器上的微服务

### 分布式的困难


远程调用
服务发现
负载均衡
服务容错
配置管理
服务监控
链路追踪
日志管理
任务调度

### 分布式的解决

SpringBoot+SpringCloud

![](images/2023-02-12-18-45-25.png)

## 1.3 云原生

原应用上云。

### 上云的困难

服务自愈
弹性伸缩
服务隔离
自动化部署
灰度发布
流量治理

### 上云的解决

容器技术

# 二、开发过程

1. 引入依赖
2. 创建主程序(MainApplication)
3. 写入业务
4. 编写配置文件(application.properties/application.yml)
5. 测试运行
6. 简化部署(打包jar)


# 三、自动配置

## 3.1 SpringBoot特点

### 3.1.1 依赖管理

#### 父项目做依赖管理

1. 声明了几乎所有开发中常用的依赖的版本号。
2. 可以查找到父项目中规定依赖版本的key使用\<properties>标签来自定义版本号


#### starter场景启动器

1. spring-boot-starter-* 表示引入*场景所需要的所有依赖
2. SpringBoot所有支持的场景：https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter
3. *-spring-boot-starter： 第三方为我们提供的简化开发的场景启动器。

### 3.1.2 自动配置

自动配置Tomcat
自动配置SpringMVC
自动按规则创建包

>可以使用@SpringBootApplication(scanBasePackages="com.atguigu")
或者@ComponentScan 指定扫描路径

配置项是按需加载的

## 3.2 容器功能

### 3.2.1 组件添加

#### i @Configuration

作用：声明配置类

与Spring5的全注解开发相同

Full模式与Lite模式
配置类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断
配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式

#### ii @Import

给容器中调用无参构造创建对象。

#### iii @Conditional 

条件装配：满足Conditional指定的条件，则进行组件注入

#### iv @ImportResource

导入原始的配置文件

### 3.2.2 配置绑定

如何使用Java读取到properties文件中的内容，并且把它封装到JavaBean中，以供随时使用；

1. @ConfigurationProperties
2. @EnableConfigurationProperties + @ConfigurationProperties
3. @Component + @ConfigurationProperties

## 3.3 自动配置原理入门

@SpringBootApplication是由：@SpringBootConfiguration、@ComponentScan、@EnableAutoConfiguration合成的注解

``` java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}

```

### 3.3.1 @SpringBootConfiguration

@Configuration 代表是一个配置类

### 3.3.2 @ComponentScan

指定扫描哪些，Spring注解；

### 3.3.3 @EnableAutoConfiguration

``` java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

#### i. @AutoConfigurationPackage

``` java
@Import(AutoConfigurationPackages.Registrar.class)  //给容器中导入一个组件
public @interface AutoConfigurationPackage {}

//利用Registrar给容器中导入一系列组件
//将指定的一个包下的所有组件导入进来？MainApplication 所在包下。
```

#### ii. @Import(AutoConfigurationImportSelector.class)

1. 利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
2. 调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
3. 利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4. 从META-INF/spring.factories位置来加载一个文件。
	1. 默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
	2. spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories
    

### 3.3.4 按需开启自动配置项

按照条件装配规则（@Conditional），最终会按需配置。

### 3.3.3 修改默认配置

默认会在底层配好所有的组件。但是如果用户自己配置了以用户的优先


直接自己@Bean替换底层的组件
看这个组件是获取的配置文件什么值就去修改。

## 3.4 配置流程

* 引入场景依赖
    * https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter
* 查看自动配置了哪些（选做）
    * 自己分析，引入场景对应的自动配置一般都生效了
    * 配置文件中debug=true开启自动配置报告。Negative（不生效）\Positive（生效）
* 是否需要修改
    * 参照文档修改配置项
      * https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties
      * 自己分析。xxxxProperties绑定了配置文件的哪些。
    * 自定义加入或者替换组件
      * @Bean、@Component。。。
    * 自定义器  XXXXXCustomizer；

## 3.5 配置技巧

### 3.5.1 Lombok


maven引入并且ide安装插件

#### i. 简化JavaBean开发

@Data：编译时生成属性的get&set方法
@ToString：编译时生成ToString方法
(@NoArgsConsructor/@AllArgsConsructor)：无参/全参构造器
@EqualsAndHashCode：重写Equals和HashCode方法

#### ii. 简化日志开发

@Slf4j：类中注入日志

### 3.5.2 dev-tools

项目或者页面修改以后：Ctrl+F9；就会重新编译项目

### 3.5.3 Spring Initailizr

项目初始化向导，整合在idea里

自动依赖引入
自动创建项目结构
自动编写好主配置类

# 四、配置文件

有两种配置语言properties和yaml，因为yaml更加简洁方便所以多用yaml

## 4.1 yaml

### 4.1.1 简介

YAML 是 "YAML Ain't Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。

非常适合用来做以数据为中心的配置文件

### 4.1.2 基本语法

* key: value；kv之间有空格
* 大小写敏感
* 使用缩进表示层级关系
* 缩进不允许使用tab，只允许空格
* 缩进的空格数不重要，只要相同层级的元素左对齐即可
* '#'表示注释
* 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义

#### 4.1.3 数据类型

``` yaml
# 1、字面量:单个的、不可再分的值。date、boolean、string、number、null.
k: v

# 2、对象：键值对的集合。map、hash、set、object 
k: {k1:v1,k2:v2,k3:v3}
#或
k: 
	k1: v1
  k2: v2
  k3: v3

# 3、数组：一组按次序排列的值。array、list、queue
k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

## 4.2 配置提示

``` xml
<!-- maven导入configuration processor包使得自定义文件有配置提示 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

<!-- 设置Spring打包时不打包只在开发时使用的的工具 -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-configuration-processor</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# 五、web开发

## 5.1 简单功能分析

### 5.1.1 静态资源访问

静态资源：前端的固定页面，这里面包含HTML、CSS、JS、图片等等不需要查数据库也不需要程序处理，直接就能够显示的页面。

#### i. 静态资源目录

放在类路径下 ：/static (or /public or /resources or /META-INF/resources
访问 ： 当前项目根路径/ + 静态资源名 

原理：请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面。

改变默认的静态资源路径：

``` yaml
spring:
  mvc:
    static-path-pattern: /res/**

web:
  resources:
    static-locations: classpath:/haha/
```

当前项目 + static-path-pattern + 静态资源名 = 静态资源文件夹下找

> 注：static-path-pattern为虚拟路径，真实路径为static-locations中的路径

#### ii. webjar

https://www.webjars.org/


把一些东西打包成可以静态访问的jar包

自动映射 /webjars/**

### 5.1.2 欢迎页

* 静态资源路径下  index.html
* controller能处理/index


### 5.1.3 自定义Favicon

Facicon：标签页小图标

favicon.ico 放在静态资源目录下即可。

## 5.2 请求参数处理

### 5.2.1 请求映射

同SpringMVC。

>注
SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
SpringBoot自动配置了默认 的 RequestMappingHandlerMapping

### 5.2.2 普通参数与基本注解

#### 注解

@PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@MatrixVariable、@CookieValue、@RequestBody

#### ServletAPI

#### 复杂参数

#### 自定义对象参数

## 5.3 数据响应与内容协商

## 5.4 视图解析与模板引擎

## 5.5 拦截器

## 5.6 文件上传

## 5.7 异常处理

## 5.8 Web原生组件注入

## 5.9 嵌入式Servelt容器

## 5.10 定制化原理


# 六、数据访问

# 七、单元测试

# 八、指标监控

# 九、原理解析























# 参考文献

[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)

[尚硅谷(视频)](https://www.bilibili.com/video/BV19K4y1L7MT)

[尚硅谷(笔记)](https://www.yuque.com/atguigu/springboot)