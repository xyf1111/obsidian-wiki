---
title: "Java 项目 01 - 启动与依赖服务排错"
date: 2026-07-15
tags: [java, maven, backend, troubleshooting, devops]
source: "鱼皮·编程导航 / codefather"
---

# Java 项目 01 - 启动与依赖服务排错

> Java 后端项目启动失败或依赖服务连接异常的常见原因与解决方案。

## 版本问题

### JDK 版本不匹配

项目要求 JDK 8，但使用 JDK 5 编译会导致 `java: -source 1.5` 中不支持 diamond 运算符 等错误。

使用更高版本也可能出现兼容问题，如 IDEA 默认 JDK 17 而项目为 JDK 8 时，报错 `无效的源发行版：17`。

**解决方案：修改 IDEA 编译版本**

1. 打开项目设置：`File` → `Project Structure`（或 `Ctrl + Alt + Shift + S`）
2. 选择左侧 `Project`，在 `Project SDK` 下拉菜单中选择正确的 JDK 版本
   ![](../../image/img_idea_sdk.png)
3. 在左侧 `Modules` → 选择模块 → `Sources` 标签页
   - `Language level` 下拉菜单中选择正确的 Java 版本
   - `Target bytecode version` 保持与目标 Java 版本一致
   ![](../../image/img_language_level.png)
   ![](../../image/img_bytecode_version.png)

![](../../image/img_java_version_error.png)

### Maven 版本不兼容

Maven install 时报 `无效的标记：--release`，通常是 Maven 版本与 Java 版本不兼容。对于 JDK 8，建议使用 Maven 3.6.x 系列。

修改 `pom.xml` 中的版本配置后，记得刷新 Maven 项目。

**POM 示例：**
```xml
<properties>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
</properties>
```

![](../../image/img_pom_version.png)

**刷新 Maven**：点击 IDEA 右侧 Maven 面板的刷新按钮，或右键 `pom.xml` → `Maven` → `Reload project`。

如果右侧没有 Maven 菜单，右键 `pom.xml` → `Add as Maven Project`。

![](../../image/img_maven_refresh.png)

## 依赖项问题

除了 Java/Maven 配置外，项目依赖的中间件（Redis、MySQL）启动失败也会导致项目报错。

### Redis 连接失败

报错信息：`Unable to connect to Redis server: localhost/127.0.0.1:6379`

**排查方向：**
- Redis 服务是否已启动
- Redis 配置了密码 → 检查项目配置文件中 `spring.redis.password` 是否正确填写
- Redis 端口和主机名是否与项目配置一致

### MySQL 连接失败

报错信息：`CannotGetJdbcConnectionException: Failed to obtain JDBC Connection` / `Access denied for user 'root'@'localhost' (using password: YES)`

**排查方向：**
- 确认 MySQL 服务已启动
- 检查配置文件中的数据库用户名和密码是否正确
- 确认账号在对应主机是否有查询权限（`GRANT` 权限）
- 检查数据库地址和端口配置

> **要点**：项目启动失败时，先看报错堆栈顶部的关键信息，定位是版本问题、依赖服务连接问题还是配置问题，再逐一排查。

> 来源：鱼皮·编程导航 / codefather
