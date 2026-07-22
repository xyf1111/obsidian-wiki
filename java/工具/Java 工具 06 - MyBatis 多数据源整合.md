---
title: "Java 工具 06 - MyBatis 多数据源整合"
date: 2026-07-22
tags: [java, mybatis, datasource, springboot, baomidou]
source: "鱼皮·编程导航 / codefather"
---

# Java 工具 06 - MyBatis 多数据源整合

> 使用 `dynamic-datasource-spring-boot-starter` 快速为 Spring Boot + MyBatis 项目整合多数据源，实现运行时动态切换。

## 背景

业务中常需要查询多个库表的数据，但不想起多个服务。MyBatis 支持整合多数据源并随时切换，通过社区主流的 `dynamic-datasource` 组件即可实现。

## 解决方案

### 引入依赖

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
  <version>${version}</version>
</dependency>
```

### 配置文件

```yaml
spring:
  datasource:
    dynamic:
      primary: master           # 默认数据源
      strict: false
      datasource:
        master:
          url: jdbc:mysql://localhost:3306/db1?useSSL=false&serverTimezone=Asia/Shanghai
          username: root
          password: xxxx
          driver-class-name: com.mysql.cj.jdbc.Driver
        slave_1:
          url: jdbc:mysql://localhost:3306/db2?useSSL=false&serverTimezone=Asia/Shanghai
          username: root
          password: xxxx
          driver-class-name: com.mysql.cj.jdbc.Driver
```

### 使用 @DS 注解切换数据源

在 Service 类或方法上添加 `@DS` 注解，指定数据源名称：

```java
@Service
@DS("master")
public class InterfaceInfoServiceImpl extends ServiceImpl<InterfaceInfoMapper, InterfaceInfo>
        implements InterfaceInfoService {
}

@Service
@DS("slave_1")
public class TEmailServiceImpl extends ServiceImpl<TEmailMapper, TEmail>
        implements TEmailService {
}
```

注解值为配置中的 `datasource` 名称。方法上注解优先级高于类上注解。

## 关键约定

1. **只做数据源切换**，不做其他限制，切换后可执行任何 CRUD
2. 配置文件中以下划线 `_` 分割的数据源首部为组名，同组数据源放在一个组下，切换时采用负载均衡算法
3. 可切换组名或具体数据源名称；默认数据源名称为 `master`，可通过 `spring.datasource.dynamic.primary` 修改
4. **方法注解优先于类注解**；`@DS` 支持继承抽象类上的注解，不支持接口上的注解
5. 底层基于 AOP 实现，同一个方法内不可切换数据源

> 来源：鱼皮·编程导航 / codefather
