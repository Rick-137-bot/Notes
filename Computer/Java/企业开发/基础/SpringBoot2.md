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
4. 编写配置文件(application.properties)
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

## 3.2 容器功能

### 3.2.1 组件添加

### 3.2.2 原生配置文件导入

### 3.2.3 配置绑定

## 3.3 自动配置原理入门

### 3.3.1 引导加载自动配置类

### 3.3.2 按需开启自动配置项

### 3.3.3 修改默认配置

## 3.4 开发技巧

### 3.4.1 Lombok

### 3.4.2 dev-tools

### 3.4.3 Spring Initailizr























# 参考文献

[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)

[尚硅谷(视频)](https://www.bilibili.com/video/BV19K4y1L7MT)

[尚硅谷(笔记)](https://www.yuque.com/atguigu/springboot)